---
title: "Scraping multiple pages"
teaching: 30
exercises: 30
questions:
- "How do I tell Scrapy to follow URLs and scrape their contents?"
objectives:
- "Creating a two-step spider to first extract the next-page URLs, visit them, and scrape their contents."
keypoints:
- "We can have the spider follow links to collect more data in an automated fashion."
- "We can use a callback to get that data scraped for our output file"
- "We can use .meta to exchange data between callbacks"
---

# Walking over the site we want to scrape

The primary advantage of a spider over a manual tool scraping a website is that it can follow links. Let's use the scraper extension to identify the XPath of the "next page" link. 

* Right click on "Next" and choose Inspect

This is important because whenever we're scraping a site we always want to start from the code. 

![The next link]({{page.root}}/fig/next.png)

Here, we see some useful things. That there is a `class="next` and that there's a characteristic `li/a` with a title "Next Page". These are all attributes we can target.

What happens if we take some cues from the source and run the Scrapy Shell:

~~~
$ scrapy shell "https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0"
~~~
{: .source .language-bash}

with
~~~
>>> response.xpath("//a[@title='Next page']/@href")
~~~
{: .source .language-python}

We see:

~~~
[<Selector xpath="//a[@title='Next page']/@href" data='?page=2&q=&mem=1&par=-1&gen=0&ps=12&st=1'>, <Selector xpath="//a[@title='Next page']/@href" data='?page=2&q=&mem=1&par=-1&gen=0&ps=12&st=1'>]
~~~
{: .language-html .output}

We can use `extract_first()`` here because the links are identical.

~~~
>>> response.xpath("//a[@title='Next page']/@href").extract_first()
~~~
{: .language-python}

returns

~~~
'?page=2&q=&mem=1&par=-1&gen=0&ps=12&st=1'
>>> 
~~~
{: .output}



> ## Dealing with relative URLs
>
> Looking at this result and at the source code of the page, we realize that the URLs are all
> _relative_ to that page. They are all missing part of the URL to become _absolute_ URLs, which
> we will need if we want to ask our spider to visit those URLs to scrape more data. We could
> prefix all those URLs with `https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results` to make them absolute, but
> since this is a common occurrence when scraping web pages, Scrapy provides a built-in function
> to deal with this issue.
>
> To try it out, still in the Scrapy shell, let's first store the first returned URL into a
> variable:
>
> ~~~
> >>> testurl = response.xpath("//a[@title='Next page']/@href").extract_first()
> ~~~
> {: .language-python}
>
> Then, we can try passing it on to the `urljoin()` method:
>
> ~~~
> >>> response.urljoin(testurl)
> ~~~
> {: .language-python}
>
> which returns
>
> ~~~
> 'https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?page=2&q=&mem=1&par=-1&gen=0&ps=12&st=1'
> ~~~
> {: .output}
>
> We see that Scrapy was able to reconstruct the absolute URL by combining the URL of the current page context
> (the page in the `response` object) and the relative link we had stored in `testurl`.
>
{: .callout}

## Moving our page-scraping to its own function

We're going to be doing some work with the spider and moving it around pages. Let's move our page scraper to its own function and make sure we can move the data we care about between functions.

(editing `austmps/austmps/spiders/austmpdata.py`)
~~~
import scrapy
from austmps.items import AustmpsItem # We need this so that Python knows about the item object

class AustmpdataSpider(scrapy.Spider):
    name = 'austmpdata'  # The name of this spider
    
    # The allowed domain and the URLs where the spider should start crawling:
    allowed_domains = ['www.aph.gov.au']
    start_urls = ['http://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0/']
    
    def parse(self, response):
        # The main method of the spider. It scrapes the URL(s) specified in the
        # 'start_url' argument above. The content of the scraped URL is passed on
        # as the 'response' object.

        # the function is an iterator, so we need to iterate over it. There is likely a cleaner way to do this.
        for item in self.scrape(response):
            yield item

    def scrape(self, response):
        for resource in response.xpath("//h4[@class='title']/.."):
            # Loop over each item on the page. 
            item = AustmpsItem() # Creating a new Item object

            item['name'] = resource.xpath("h4/a/text()").extract_first()
            item['link'] = resource.xpath("h4/a/@href").extract_first()
            item['district'] = resource.xpath("dl/dd/text()").extract_first()
            item['twitter'] = resource.xpath("dl/dd/a[contains(@class, 'twitter')]/@href").extract_first()
            item['party'] = resource.xpath("dl/dt[text()='Party']/following-sibling::dd/text()").extract_first()

            yield item
~~~
{: .language-python}            


## Extracting URLs using the spider

Since we have an XPath query we know will extract the URLs we are looking for, we can now use the `XPath()` method and update the spider accordingly.

(editing `austmps/austmps/spiders/austmpdata.py`)

~~~
(...)   
    def parse(self, response):
        # The main method of the spider. It scrapes the URL(s) specified in the
        # 'start_url' argument above. The content of the scraped URL is passed on
        # as the 'response' object.

        nextpageurl = response.xpath("//a[@title='Next page']/@href").extract_first()
        nextpage = response.urljoin(nextpageurl)
        print(nextpage)
       
        for item in self.scrape(response):
            yield item
(...)
~~~
{: .language-python}

~~~
$ scrapy crawl austmpdata -s DEPTH_LIMIT=1
~~~
{: .language-bash}

And this prints out the next page url. 
~~~
(...)
2018-06-26 19:23:50 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0/> (referer: None)
https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?page=2&q=&mem=1&par=-1&gen=0&ps=12&st=1
(...)
~~~
{: .output}

Now we need our spider to follow that link and we need to make sure that the spider *stops* when the link isn't present. We can do this through a technique called "recursion" which means calling a thing from itself.

To our function parse, we add a call to itself: the function parse.

~~~
    def parse(self, response):
        # The main method of the spider. It scrapes the URL(s) specified in the
        # 'start_url' argument above. The content of the scraped URL is passed on
        # as the 'response' object.

        # for item in self.scrape(response):
        #     yield item

        nextpageurl = response.xpath("//a[@title='Next page']/@href")

        if nextpageurl:
            # If we've found a pattern which matches
            path = nextpageurl.extract_first()
            nextpage = response.urljoin(path)
            print("Found url: {}".format(nextpage)) # Write a debug statement
            yield scrapy.Request(nextpage, callback=self.parse) # Return a call to the function "parse"      
~~~
{: .language-python}

And now, with a single invocation of the scraper (note our lack of depth limit, since we're testing recursion):

~~~
$ scrapy crawl austmpdata 
~~~
{: .language-bash}

we get:

~~~
(...)
2018-06-26 19:39:17 [scrapy.downloadermiddlewares.redirect] DEBUG: Redirecting (302) to <GET https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0/> from <GET http://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0/>
2018-06-26 19:39:17 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0/> (referer: None)
Found url: https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?page=2&q=&mem=1&par=-1&gen=0&ps=12&st=1
(...)
2018-06-26 19:39:20 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?page=13&q=&mem=1&par=-1&gen=0&ps=12&st=1> (
(...)
~~~
{: .output}


If we turn our response parsing back on by uncommenting the item for loop, we suddenly get all 145 members of parliament.

Here is the full code of `austmpdata.py`:

~~~
import scrapy
from austmps.items import AustmpsItem # We need this so that Python knows about the item object

class AustmpdataSpider(scrapy.Spider):
    name = 'austmpdata'  # The name of this spider
    
    # The allowed domain and the URLs where the spider should start crawling:
    allowed_domains = ['www.aph.gov.au']
    start_urls = ['http://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0/']
    
    def parse(self, response):
        # The main method of the spider. It scrapes the URL(s) specified in the
        # 'start_url' argument above. The content of the scraped URL is passed on
        # as the 'response' object.

        nextpageurl = response.xpath("//a[@title='Next page']/@href")

        for item in self.scrape(response):
            yield item

        if nextpageurl:
            path = nextpageurl.extract_first()
            nextpage = response.urljoin(path)
            print("Found url: {}".format(nextpage))
            yield scrapy.Request(nextpage, callback=self.parse)

    def scrape(self, response):
        for resource in response.xpath("//h4[@class='title']/.."):
            # Loop over each item on the page. 
            item = AustmpsItem() # Creating a new Item object

            item['name'] = resource.xpath("h4/a/text()").extract_first()
            item['link'] = resource.xpath("h4/a/@href").extract_first()
            item['district'] = resource.xpath("dl/dd/text()").extract_first()
            item['twitter'] = resource.xpath("dl/dd/a[contains(@class, 'twitter')]/@href").extract_first()
            item['party'] = resource.xpath("dl/dt[text()='Party']/following-sibling::dd/text()").extract_first()

            yield item


~~~
{: .language-python}

If we run:

~~~
$ rm output.csv
$ scrapy crawl austmpdata -o output.csv
$ wc -l output.csv 
~~~
{: .language-bash}

We get all 145 members of parliment + 1 line for the header:
~~~
146 output.csv
~~~
{: .output}

## Visiting child pages

Now that we're scraping and following links, what happens if we want to add a member's Electorate Office phone number to this data sheet?

We will need to tell the scraper to load their profile page (which we have the URL for) and to write a second scraper function to find the data we want from this specific page.

First, use the tools we've explored today to find the correct XPath for the Electorate Office phone number.

Tip: `"Electorate Office "` has a space inside the `h3`. And we're going to need to use `following-sibling::`.

Using scrapy shell 

~~~
$ scrapy shell "https://www.aph.gov.au/Senators_and_Members/Parliamentarian?MPID=R36"
~~~
{: .source}


~~~
>>> response.xpath("//h3[text()='Electorate Office ']/following-sibling::dl/dd[1]/text()").extract()
~~~
{: .language-python}

We will use the `dd[1]` here because otherwise we're going to enter into far more complex selectors.

Now that we have the XPath solution, we need to make sure the `items.py` object has a phone number that it can accept.

~~~
import scrapy


class AustmpsItem(scrapy.Item):
    # define the fields for your item here like:
    name = scrapy.Field()
    district = scrapy.Field()
    link = scrapy.Field()
    twitter = scrapy.Field()
    party = scrapy.Field()
    phonenumber = scrapy.Field()
~~~
{: .language-python}
Next, we need to make another function called `get_phonenumber(self, response)`. 

Looking at the [scrapy documentation](https://doc.scrapy.org/en/latest/topics/request-response.html#passing-additional-data-to-callback-functions), we can "pass" the item class around using `.meta` 

Editing `austmpdata.py`

~~~
import scrapy
from austmps.items import AustmpsItem # We need this so that Python knows about the item object

class AustmpdataSpider(scrapy.Spider):
    name = 'austmpdata'  # The name of this spider
    
    # The allowed domain and the URLs where the spider should start crawling:
    allowed_domains = ['www.aph.gov.au']
    start_urls = ['http://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0/']
    
    def parse(self, response):
        # The main method of the spider. It scrapes the URL(s) specified in the
        # 'start_url' argument above. The content of the scraped URL is passed on
        # as the 'response' object.

        nextpageurl = response.xpath("//a[@title='Next page']/@href")

        for item in self.scrape(response):
            yield item

        if nextpageurl:
            path = nextpageurl.extract_first()
            nextpage = response.urljoin(path)
            print("Found url: {}".format(nextpage))
            yield scrapy.Request(nextpage, callback=self.parse)

        
    def scrape(self, response):
        for resource in response.xpath("//h4[@class='title']/.."):
            # Loop over each item on the page. 
            item = AustmpsItem() # Creating a new Item object

            item['name'] = resource.xpath("h4/a/text()").extract_first()


            # Instead of just writing the relative path of the profile page, lets make the full profile page so we can use it later.
            profilepage = response.urljoin(resource.xpath("h4/a/@href").extract_first())
            item['link'] = profilepage

            item['district'] = resource.xpath("dl/dd/text()").extract_first()
            item['twitter'] = resource.xpath("dl/dd/a[contains(@class, 'twitter')]/@href").extract_first()
            item['party'] = resource.xpath("dl/dt[text()='Party']/following-sibling::dd/text()").extract_first()

            # We need to make a new variable that the scraper will return that will get passed through another callback. We're calling that variable "request"
            request= scrapy.Request(profilepage, callback=self.get_phonenumber)
            request.meta['item'] = item #By calling .meta, we can pass our item object into the callback.
            yield request #Return the item + phonenumber back to the parser.

    def get_phonenumber(self, response):
        # A scraper designed to operate on one of the profile pages
        item = response.meta['item'] #Get the item we passed from scrape()        
        item['phonenumber'] = response.xpath("//h3[text()='Electorate Office ']/following-sibling::dl/dd[1]/text()").extract_first()
        yield item #Return the new phonenumber'd item back to scrape
~~~
{: .language-python}

and running:

~~~
$ rm -f output.csv
$ scrapy crawl austmpdata -o output.csv
$ head -3 output.csv
~~~
{: .language-bash}

gets us

~~~
district,link,name,party,phonenumber,twitter
"Warringah, New South Wales",https://www.aph.gov.au/Senators_and_Members/Parliamentarian?MPID=EZ5,Hon Tony Abbott MP,Liberal Party of Australia,(02) 9977 6411,http://twitter.com/TonyAbbottMHR
"Menzies, Victoria",https://www.aph.gov.au/Senators_and_Members/Parliamentarian?MPID=HK5,Hon Kevin Andrews MP,Liberal Party of Australia,(03) 9848 9900,http://twitter.com/kevinandrewsmp
~~~
{: .output}

Triumph! 

> ## Summative exercise: Write a new web scraper
> 
> Now that we've built this web scraper. Use the list of [Members of the house of commons](https://www.parliament.uk/mps-lords-and-offices/mps/) and extract their name, constituency, party, twitter handle, and phone number.