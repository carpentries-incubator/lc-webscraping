---
title: "COMING SOON: Web scraping using Python and Scrapy"
teaching: 60
exercises: 30
questions:
- "How can scraping a web site be automated?"
objectives:
- "Writing a spider to scrape a website using Python and the Scrapy framework."
- "Use popular web-based scraping services."
keypoints:
- "FIXME"
---

Verify that Scrapy 1.1.0 is installed
	Type scrapy version in Terminal

	Requires Python 2.7+ or 3.4+
	Should work in either environment


EXPLAIN EXAMPLE
http://www.collegesinstitutes.ca/our-members/member-directory/


We will work in two steps, first gather all URLs
Then extract data for each college

	
Initialize Scrapy project
	Navigate to where you want to create the project
	Type scrapy startproject cancolleges

	This creates the following structure:
	
	cancolleges/
	    scrapy.cfg            # deploy configuration file

	    cancolleges/          # project's Python module, you'll import your code from here
	        __init__.py

	        items.py          # project items file

	        pipelines.py      # project pipelines file

	        settings.py       # project settings file

	        spiders/          # a directory where you'll later put your spiders
	            __init__.py
	            ...


DEFINING ITEMS
	This is where the data extracted will be stored.
	Items work like Python dictionaries.
	
	Remember that we will start by gathering all the URLs for the college pages.
	Let's start by defining an Item to store the College URLs.
	
	Edit cancolleges/cancolleges/items.py
	
	(You will see Scrapy has pre-populated this file)
	
	import scrapy

	class CancollegesItem(scrapy.Item):
	    # define the fields for your item here like:
	    # name = scrapy.Field()
	    name = scrapy.Field()
	    link = scrapy.Field()


DEFINING A SPIDER
	Spiders are the business end of the scraper. It's the bit of code that combs through a website and harvest data.
	They define an initial list of URLs to download, how to follow links, and how to parse the contents of pages to extract items.
	
	Create a new file called colleges_spider.py under cancolleges/cancolleges/spiders
	with
	
	import scrapy

	class CollegeSpider(scrapy.Spider):
	    name = "colleges"  # Identifies the scraper, has to be UNIQUE but does not need to be same name as spider filename
	
		# allowed_domains = ["collegesinstitutes.ca"] ### REAL URLS
	    # start_urls = ["http://www.collegesinstitutes.ca/our-members/member-directory/"]
	
	    allowed_domains = ["labs.timtom.ch"]
	    start_urls = ["http://labs.timtom.ch/swc-teaching-notes/webscraping/data/www.collegesinstitutes.ca/our-members/member-directory/index.html"]
		
	    def parse(self, response):
			# a method of the spider, which will be called with the downloaded 
			# Response object of each start URL. The response is passed to the method as the first and only argument.
	        print response.text

START CRAWLING
	
	Try it out by moving to project's top level directory and type
		scrapy crawl colleges
						^-- name of the spider we just created
	
		Should return a bunch of text, including the HTML content of the page we just scraped.

	This is not very convenient, so try instead
	
	import scrapy

	class CollegeSpider(scrapy.Spider):
	    name = "colleges"
	   	allowed_domains = ["labs.timtom.ch"]
	    start_urls = ["http://labs.timtom.ch/swc-teaching-notes/webscraping/data/www.collegesinstitutes.ca/our-members/member-directory/index.html"]
	
	    def parse(self, response):
	        with open("test.html", 'wb') as file:
	            file.write(response.body)

	Should create file "test.html". Look inside this file.
	

OK, so how do we now access the URLs we're interested in?
Let's start by going back to the website and use the Inspect tool to find the elements we're looking for.

We need to use "Selectors" to get to the elements we need. Scrapy uses XPath (or CSS) selectors for this.

Refresher:
Here are some examples of XPath expressions and their meanings:
- /html/head/title: selects the <title> element, inside the <head> element of an HTML document
- /html/head/title/text(): selects the text inside the aforementioned <title> element.
- //td: selects all the <td> elements
- //div[@class="mine"]: selects all div elements which contain an attribute class="mine"


In Chrome, we can use the console to try out XPath queries:
> $x('//*[@class="facetwp-results"]')

	This actually only works if there is only one class. More general answer:
	https://stackoverflow.com/questions/8808921/selecting-a-css-class-with-xpath


Drilling down:
> $x('//*[@class="facetwp-results"]/li/a/@href')

Once we found what we're looking for, let's edit the Spider accordingly:

Scrapy Selectors support different methods:
- xpath(): returns a list of selectors, each of which represents the nodes selected by the xpath expression given as argument.
		This is what we'll use
- css(): returns a list of selectors, each of which represents the nodes selected by the CSS expression given as argument.
- extract(): returns a unicode string with the selected data.
- re(): returns a list of unicode strings extracted by applying the regular expression given as argument.



	import scrapy

	class CollegeSpider(scrapy.Spider):
	    name = "colleges"
	    allowed_domains = ["labs.timtom.ch"]
	    start_urls = ["http://labs.timtom.ch/swc-teaching-notes/webscraping/data/www.collegesinstitutes.ca/our-members/member-directory/index.html"]
		
	    def parse(self, response):
	        for url in response.xpath('//*[@class="facetwp-results"]/li/a/@href').extract():
	            print url

Looking good.

OK, so we now have our URLs, how do we get further?

Let's have a look at the first college page.
What information are we interested in extracting?

- Name
- Website
- Address
- Telephone

================================================================================================
CHALLENGE
What are the XPath queries to extract the above details?
Hint: keep using the Inspect tool + console trick

$x('//*[@class="page-title"]')
$x('//*[@class="mem-contact"]/p[2]//a/@href')
$x('//*[@class="mem-contact"]/p[1]')
$x('//*[@class="mem-contact"]/p[2]/node()[2]') OR $x('//*[@class="mem-contact"]/p[2]/text()[1]')
================================================================================================


Other trick, we can also use Scrapy in shell mode to try out XPath selectors:
(actually an iPython shell)

$ scrapy shell "http://www.collegesinstitutes.ca/members/alberta-college-of-art-and-design/"
In [1]: reponse.body
In [2]: response.xpath('//*[@class="page-title"]')
	Whoops, is that what we want?
In [3]: response.xpath('//*[@class="page-title"]/text()').extract()

Etc.

To exit: CTRL-D (or CTRL-Z in Windows?)


How do we bring this back together?

Remember the CanCollegesItem we defined?

	import scrapy
	from cancolleges.items import CancollegesItem # We need this so that Python knows about the Item

	class CollegeSpider(scrapy.Spider):
	    name = "colleges"
	    allowed_domains = ["collegesinstitutes.ca"]
	    start_urls = ["http://www.collegesinstitutes.ca/our-members/member-directory/"]

	    def parse(self, response):
	        for url in response.xpath('//*[@class="facetwp-results"]/li/a/@href').extract():
	            yield scrapy.Request(url, callback=self.get_details) # Callback to the get_details() method

	    def get_details(self, response):
	        item = CancollegesItem()
	        item['name'] = response.xpath('//*[@class="page-title"]/text()').extract()
		    item['link'] = response.xpath('//*[@class="mem-contact"]/p[2]//a/@href').extract()
	        yield item # Return the item

yield is like return, but for potentially large datasets. Basically when you don't know from the beginning
how big the dataset will be.
	Also:
	return implies that the function is returning control of execution to the point where the function was called. 
	yield, however, implies that the transfer of control is temporary and voluntary, and our function expects to regain it in the future.
	From http://jeffknupp.com/blog/2013/04/07/improve-your-python-yield-and-generators-explained/


================================================================================================
CHALLENGE
Add the other elements to the Spider.
Remember to edit the Item definition to allow for all fields to be taken care of.

Advanced:
Check http://www.collegesinstitutes.ca/members/aurora-college/
Look at column with enrollment numbers. Several colleges have these. Can you incorporate them in your spider?
Don't forget to update the Item accordingly
================================================================================================


Running the spider above returns CancollegesItem objects.
What if we want this data in another, more usable format?

Feed exports > very powerful function of Scrapy

Just type
$ scrapy crawl colleges -o items.csv

or
$ scrapy crawl colleges -o items.json

Also works with XML, Pickle, etc.
http://doc.scrapy.org/en/latest/topics/feed-exports.html#topics-feed-exports

MAGIC!!
