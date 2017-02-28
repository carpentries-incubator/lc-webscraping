---
title: "Web scraping using Python and Scrapy"
teaching: 90
exercises: 30
questions:
- "How can scraping a web site be automated?"
- "How can I setup a scraping project using the Scrapy framework for Python?"
- "How do I tell Scrapy what elements to scrape from a webpage?"
- "How do I tell Scrapy to follow URLs and scrape their contents?"
- "What to do with the data extracted with Scrapy?"
objectives:
- "Setting up a Scrapy project."
- "Understanding the various elements of a Scrapy projects."
- "Creating a spider to scrape a website and extract specific elements."
- "Creating a two-step spider to first extract URLs, visit them, and scrape their contents."
- "Storing the extracted data."
keypoints:
- "Scrapy is a Python framework that can be use to scrape content from the web."
- "A Scrapy project is a set of configuration files and pieces of code that tell Scrapy what to do."
- "In Scrapy, a \"Spider\" is the code that tells it what to do on a specific website."
- "A Scrapy project can have more than one spider but needs at least one."
- "With Scrapy, we can use XPath, CSS selectors and Regular Expressions to define what elements to scrape from a page."
- "Extracted data can be stored in \"Item\" objects. Such objects must be defined before they can be used."
- "Scrapy will automatically stored extracted data in CSS, JSON or XML format based on the file extension given in the -o option."
---

## Recap
Here is what we have learned so far:

* We can use XPath queries to select what elements on a page to scrape.
* We can look at the HTML source code of a page to find how target elements are structured and
  how to select them.
* We can use the browser console and the `$x(...)` function to try out XPath queries on a live site.
* We can use the Scraper browser extension to scrape data from a single web page. Its interface even
  tries to guess the XPath query to target the elements we are interested in.
* We can use web tools such as import.io to scrape data from more than one web page, by querying a list
  of URLs or even by using one scraper to extract URLs and a second one to harvest data on those pages.

This is quite a toolset already, and it's probably sufficient for a number of use cases, but there are
limitations in using the tools we have seen so far. Scraper requires manual intervention and only scrapes
one page at a time. Even though it is possible to save a query for later, it still requires us to operate
the extension. import.io has a nifty schedule function to allow us to let the scraper run on its own at
certain intervals and update information automatically. However, scraping more than 500 URLs per month is
expensive and the interface can be difficult to operate for more complex queries.

## Introducing Scrapy

Enter [Scrapy](https://scrapy.org/)! Scrapy is a _framework_ for the [Python](https://swcarpentry.github.io/python-novice-inflammation/)
programming language.

>
> A framework is a reusable, "semi-complete" application that can be specialized to produce custom applications.
> (Source: [Johnson & Foote, 1988](http://www1.cse.wustl.edu/~schmidt/CACM-frameworks.html))
>

In other words, the Scrapy framework provides a set of Python scripts that contain most of the code required
to use Python for web scraping. We need only to add the last bit of code required to tell Python what
pages to visit, what information to extract from those pages, and what to do with it. Scrapy also
comes with a set of scripts to setup a new project and to control the scrapers that we will create.

It also means that Scrapy doesn't work on its own. It requires a working Python installation
(Python 2.7 and higher or 3.4 and higher - it should work in both Python 2 and 3), and a series of
libraries to work. If you haven't installed Python or Scrapy on your machine, you can refer to the
[setup instructions](/library-webscraping/setup). If you install Scrapy as suggested there, it should take care to install all
required libraries as well.

You can verify that you have the latest version of Scrapy installed by typing

~~~
scrapy version
~~~
{: .source}

in a shell. If all is good, you should get the following back (as of February 2017):

~~~
Scrapy 1.3.2
~~~
{: .output}

If you have a newer version, you should be fine as well.

To introduce the use of Scrapy, we will reuse the same example we used in the previous
section. We will start by scraping a list of URLs from [the list of members of the Ontario Legislative
Assembly](http://www.ontla.on.ca/web/members/members_current.do?locale=en) and then visit those URLs to
scrape [detailed information](http://www.ontla.on.ca/web/members/members_detail.do?locale=en&ID=7085)
about those ministers.

## Setup a new Scrapy project

The first thing to do is to create a new Scrapy project.

Let's navigate first to a folder on our drive where we want to create our project (refer to Software
Carpentry's lesson about the [UNIX shell](http://swcarpentry.github.io/shell-novice/) if you are
unsure about how to do that). Then, type the following

~~~
scrapy startproject ontariompps
~~~
{: .source}

where `ontariompps` is the name of our project.

Scrapy should respond will something similar to (the paths will reflect your own file structure)

~~~
New Scrapy project 'ontariompps', using template directory '/Users/thomas/anaconda/lib/python3.5/site-packages/scrapy/templates/project', created in:
    /Users/thomas/Documents/Computing/python-projects/scrapy/ontariompps

You can start your first spider with:
    cd ontariompps
    scrapy genspider example example.com
~~~
{: .output}

If we list the files in the directory we ran the previous command

~~~
ls -F
~~~
{: .source}

we should see that a new directory was created:

~~~
ontariompps/
~~~
{: .output}

(alongside any other files and directories you had lying around previously). Moving into that new
directory

~~~
cd ontariompps
~~~
{: .source}

we can see that it contains two items:

~~~
ls -F
~~~
{: .source}

~~~
ontariompps/	scrapy.cfg
~~~
{: .output}

Yes, confusingly, Scrapy creates a subdirectory called `ontariompps` within the `ontariompps` project
directory. Inside that _second_ directory, we see a bunch of additional files:

~~~
ls -F ontariompps
~~~
{: .source}

~~~
__init__.py	items.py	settings.py
__pycache__	pipelines.py	spiders/
~~~
{: .output}

To recap, here is the structure that `scrapy startproject` created:

~~~
ontariompps/			# the root project directory
	scrapy.cfg		# deploy configuration file

	ontariompps/		# project's Python module, you'll import your code from here
		__init__.py

		items.py		# project items file

		pipelines.py	# project pipelines file

		settings.py	# project settings file

		spiders/		# a directory where you'll later put your spiders
			__init__.py
			...
~~~
{: .output}

We will introduce what those files are for in the next paragraphs. The most important item is the
`spiders` directory: this is where we will write the scripts that will scrape the pages we
are interested in. Scrapy calls such scripts _spiders_.

## Creating a spider

Spiders are the business end of the scraper. It's the bit of code that combs through a website and harvests data.
Their general structure is as follows:
* One or more _start URLs_, where the spider will start crawling
* A list of _allowed domains_ to constrain the pages we allow our spider to crawl (this is a good way to
  avoid mistakenly writing an out-of-hand spider that mistakenly starts crawling the entire Internet...)
* A method called `parse` in which we will write what data the spider should be looking for on the pages
  it visits, what links to follow and how to parse found data.
  
To create a spider, Scrapy provides a handy command-line tool:

~~~
scrapy genspider <SCRAPER NAME> <START URL>
~~~
{: .source}
  
We just need to replace `<SCRAPER NAME>` with the name we want to give our spider and `<START URL>` with
the URL we want to spider to crawl. In our case, we can type:

~~~
scrapy genspider mppaddresses www.ontla.on.ca/web/members/members_current.do?locale=en
~~~
{: .source}

This will create a file called `mppaddresses.py` inside the `spiders` directory of our project.
Using our favourite text editor, let's open that file. It should look something like this:

~~~
import scrapy

class MppaddressesSpider(scrapy.Spider):
    name = "mppaddresses"  # The name of this spider
	
	# The allowed domain and the URLs where the spider should start crawling:
    allowed_domains = ["www.ontla.on.ca/web/members/members_current.do?locale=en"]
    start_urls = ['http://www.ontla.on.ca/web/members/members_current.do?locale=en/']

	# And a 'parse' function, which is the main method of the spider. The content of the scraped
	# URL is passed on as the 'response' object:
    def parse(self, response):
        pass
~~~
{: .source}

Note that here some comments have been added for extra clarity, they will not be there upon
first creating a spider.

> ## Don't include http:// when running `scrapy genspider`
>
> The current version of Scrapy (1.3.2 - February 2017) apparently only expects URLs without
> `http://` when running `scrapy genspider`. If you do include the `http` prefix, you might
> see that the value in `start_url` in the generated spider will have that prefix twice, because
> Scrapy appends it by default. This will cause your spider to fail. Either run `scrapy genspider`
> without `http://` or check the resulting spider so that it looks like the code above.
>
{: .callout}

> ## Object-oriented programming and Python classes
>
> You might be unfamiliar with the `class MppaddressesSpider(scrapy.Spider)` syntax used above.
> This is an example of [Object-oriented programming](https://en.wikipedia.org/wiki/Object-oriented_programming).
>
> All elements of a piece of Python code are __objects__: functions, variables, strings, integers, etc.
> Objects of a certain type have certain things in common. For example, it is possible to apply special
> functions to all strings in Python by using syntax such as `mystring.upper()` (this will make the contents
> of `mystring` all uppercase).
>
> We call these types of objects __classes__. A class defines the components of an object (called __attributes__),
> as well as specific functions, called __methods__, we might want to run on those objects.
> For example, we could define a class called `Pet` that would contain the attributes `name`, `colour`, `age` etc.
> as well as the methods `run()` or `cuddle()`. Those are common to all pets.
>
> We can use the Object-oriented paradigm to describe a specific type of pet: `Dog` would __inherit__ the
> attributes and methods of `Pet` (dogs have names and can run and cuddle) but would __extend__ the `Pet` class
> by adding dog-specific things like a `pedigree` attribute and a `bark()` method.
>
> The code in the example above is defining a __class__ called `MppaddressesSpider` that __inherits__ the `Spider` class
> defined by Scrapy (hence the `scrapy.Spider` syntax). We are __extending__ the default `Spider` class by defining
> the `name`, `allowed_domains` and `start_urls` attributes, as well as the `parse()` method.
>
{: .discussion}

> ## The `Spider` class
>
> A `Spider` class will define how a certain site (or a group of sites, defined in `start_urls`) will be scraped,
> including how to perform the crawl (i.e. follow links) and how to extract structured data from their pages
> (i.e. scraping items) in the `parse()` method.
> 
> In other words, Spiders are the place where we define the custom behaviour for crawling and parsing
> pages for a particular site (or, in some cases, a group of sites).
>
{: .callout}

Once we have the spider open in a text editor, we can start by cleaning up a little the code that Scrapy
has automatically generated. We see that by default the entire URL has ended up in the `allowed_domains`
attribute. This might limit our spider once we start visiting other pages, so let's replace that value with
just the base domain `www.ontla.on.ca`:

~~~
import scrapy

class MppaddressesSpider(scrapy.Spider):
    name = "mppaddresses"  
	
    allowed_domains = ["www.ontla.on.ca"]
    start_urls = ['http://www.ontla.on.ca/web/members/members_current.do?locale=en/']

    def parse(self, response):
        pass
~~~
{: .source}

Don't forget to save the file once changes have been applied.

## Running the spider

Now that we have a first spider setup, we can try running it. Going back to the Terminal, we first make sure
we are located in the project's top level directory (where the `scrapy.cfg` file is) by using `ls`, `pwd` and
`cd` as required, then we can run:

~~~
scrapy crawl mppaddresses
~~~
{: .source}

Note that we can now use the name we have chosen for our spider (`mppaddresses`, as specified in the `name` attribute)
to call it. This should produce the following result

~~~
2016-11-07 22:28:51 [scrapy] INFO: Scrapy 1.3.2 started (bot: mppaddresses)

(followed by a bunch of debugging output ending with:)

2017-02-26 22:08:51 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.ontla.on.ca/web/members/members_current.do?locale=en/> (referer: None)
2017-02-26 22:08:52 [scrapy.core.engine] INFO: Closing spider (finished)
2017-02-26 22:08:52 [scrapy.statscollectors] INFO: Dumping Scrapy stats:
{'downloader/request_bytes': 477,
 'downloader/request_count': 2,
 'downloader/request_method_count/GET': 2,
 'downloader/response_bytes': 34187,
 'downloader/response_count': 2,
 'downloader/response_status_count/200': 2,
 'finish_reason': 'finished',
 'finish_time': datetime.datetime(2017, 2, 27, 3, 8, 52, 16404),
 'log_count/DEBUG': 3,
 'log_count/INFO': 7,
 'response_received_count': 2,
 'scheduler/dequeued': 1,
 'scheduler/dequeued/memory': 1,
 'scheduler/enqueued': 1,
 'scheduler/enqueued/memory': 1,
 'start_time': datetime.datetime(2017, 2, 27, 3, 8, 51, 594573)}
2017-02-26 22:08:52 [scrapy.core.engine] INFO: Spider closed (finished)

~~~
{: .output}

The line that starts with `DEBUG: Crawled (200)` is good news, as it tells us that the spider was
able to crawl the website we were after. The number in parentheses is the _HTTP status code_ that
Scrapy received in response of its request to access that page. 200 means that the request was successful
and that data (the actual HTML content of that page) was sent back in response.

However, we didn't do anything with it, because the `parse` method in our spider is currently empty.
Let's change that by editing the spider as follows (note the contents of the `parse` method):

(editing `ontariompps/ontariompps/spiders/mppaddresses.py`)

~~~
import scrapy


class MppaddressesSpider(scrapy.Spider):
    name = "mppaddresses"
    allowed_domains = ["www.ontla.on.ca/web/members/members_current.do?locale=en"]
    start_urls = ['http://www.ontla.on.ca/web/members/members_current.do?locale=en/']

    def parse(self, response):
        with open("test.html", 'wb') as file:
            file.write(response.body)
~~~
{: .source}

Now, if we go back to the command line and run our spider again

~~~
scrapy crawl mppaddresses
~~~
{: .source}

we should get similar debugging output as before, but there should also now be a file called
`test.html` in our project's root directory:

~~~
ls -F
~~~
{: .source}

~~~
ontariompps/	scrapy.cfg	test.html
~~~
{: .output}

We can check that it contains the HTML from our target URL:

~~~
less test.html
~~~
{: .source}

~~~
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" lang="en" xml:lang="en" >
<head>
(...)
<title>
Legislative Assembly of Ontario |
Members (MPPs) |
Current MPPs</title>
(...)_
~~~
{: .output}

## Defining which elements to scrape using XPath

Now that we know how to access the content of the [web page with the list of all Ontario MPPs](http://www.ontla.on.ca/web/members/members_current.do?locale=en),
the next step is to extract the information we are interested in, in that case the URLs pointing
to the detail pages for each politician.

Using the techniques we have [learned earlier](/02-xpath), we can start by looking at
the source code for our [target page](http://www.ontla.on.ca/web/members/members_current.do?locale=en)
by using either the "View Source" or "Inspect" functions of our browser.
Here is an excerpt of that page:

~~~
(...)
<div class="tablebody">
	<table>
		<tr class="oddrow" id="MemberID7085">
			<td class="mppcell" >
				<a href="members_detail.do?locale=en&amp;ID=7085">
					Albanese, Hon Laura&nbsp;
				</a>
			</td>
			<td class="ridingcell" >
				York South&#8212;Weston&nbsp;
			</td>
		</tr>
		<tr class="evenrow" id="MemberID7275">
			<td class="mppcell" >
				<a href="members_detail.do?locale=en&amp;ID=7275">
					Anderson, Granville&nbsp;
				</a>
			</td>
			<td class="ridingcell" >
				Durham&nbsp;
			</td>
		</tr>
		(...)
	</table>
</div>
(...)
~~~
{: .output}

There are different strategies to target the data we are interested in. One of them is to identify
that the URLs are inside `td` elements of the class `mppcell`.

We recall that the XPath syntax to access all such elements is `//td[@class='mppcell']`, which we can
try out in the browser console:

~~~
> $x("//td[@class='mppcell']")
~~~
{: .source}

> ## Selecting elements assigned to multiple classes
>
> The above XPath works in this case because the target `td` elements are only assigned to the
> `mppcell` class. It wouldn't work if those elements had more than one class, for example
> `<td class="mppcell sampleclass">`. The more general syntax to select elements that belong to
> the `mppcell` class and potentially other classes as well is
>
> ~~~
> `//*[contains(concat(" ", normalize-space(@class), " "), " mppcell ")]`
> ~~~
> {: .source}
>
> This [comment on StackOverflow](http://stackoverflow.com/a/9133579) has more details on
> this issue.
>
{: .discussion}

Once we were able to confirm that we are targeting the right cells, we can expand our XPath query
to only select the `href` attribute of the URL:

~~~
> $x("//td[@class='mppcell']/a/@href")
~~~
{: .source}

This returns an array of objects:

~~~
<- Array [ href="members_detail.do?locale=en&amp;ID=7085", href="members_detail.do?locale=en&amp;ID=7275", 103 more… ]
~~~
{: .output}

### Debugging using the Scrapy shell

As we learned in the previous section, using the browser console and the `$x()` syntax can be useful to make sure
we are selecting the desired elements using our XPath queries. But it is not the only way. Scrapy provides a similar
way to test out XPath queries, with the added benefit that we can then also debug how to further work on those
queries from within Scrapy.

This is achieved by calling the _Scrapy shell_ from the command line:

~~~
scrapy shell http://www.ontla.on.ca/web/members/members_current.do?locale=en/
~~~
{: .source}

which launches a Python console that allows us to type live Python and Scrapy code to
interact with the page which Scrapy just downloaded from the provided URL. We can see that we are inside an
interactive python console because the prompt will have changed to `>>>`:

~~~
(similar Scrapy debug text as before)

2017-02-26 22:31:04 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.ontla.on.ca/web/members/members_current.do?locale=en/> (referer: None)
[s] Available Scrapy objects:
[s]   scrapy     scrapy module (contains scrapy.Request, scrapy.Selector, etc)
[s]   crawler    <scrapy.crawler.Crawler object at 0x1114356d8>
[s]   item       {}
[s]   request    <GET http://www.ontla.on.ca/web/members/members_current.do?locale=en/>
[s]   response   <200 http://www.ontla.on.ca/web/members/members_current.do?locale=en/>
[s]   settings   <scrapy.settings.Settings object at 0x111f40908>
[s]   spider     <DefaultSpider 'default' at 0x11320acc0>
[s] Useful shortcuts:
[s]   fetch(url[, redirect=True]) Fetch URL and update local objects (by default, redirects are followed)
[s]   fetch(req)                  Fetch a scrapy.Request and update local objects 
[s]   shelp()           Shell help (print this help)
[s]   view(response)    View response in a browser
>>>
~~~
{: .output}

We can now try running the XPath query we just devised against the `response` object, which in Scrapy
contains the downloaded web page:

~~~
>>> response.xpath("//td[@class='mppcell']/a/@href")
~~~
{: .source}

This will return a bunch of `Selector` objects (one for each URL found):

~~~
[<Selector xpath="//td[@class='mppcell']/a/@href" data='members_detail.do?locale=en&ID=7085'>, 
 <Selector xpath="//td[@class='mppcell']/a/@href" data='members_detail.do?locale=en&ID=7275'>,
 ...]
>>>
~~~
{: .output}

Those objects are pointers to the different element in the scraped page (`href` attributes) as
defined by our XPath query. To get to the actual content of those elements (the text of the URLs),
we can use the `extract()` method. A variant of that method is `extract_first()` which does the
same thing as `extract()` but only returns the first element if there are more than one:

~~~
>>> response.xpath("//td[@class='mppcell']/a/@href").extract_first()
~~~
{: .source}

returns

~~~
'members_detail.do?locale=en&ID=7085'
>>> 
~~~
{: .output}

> ## Dealing with relative URLs
>
> Looking at this result and at the source code of the page, we realize that the URLs are all
> _relative_ to that page. They are all missing part of the URL to become _absolute_ URLs, which
> we will need if we want to ask our spider to visit those URLs to scrape more data. We could
> prefix all those URLs with `http://www.ontla.on.ca/web/members/` to make them absolute, but
> since this is a common occurence when scraping web pages, Scrapy provides a built-in function
> to deal with this issue.
>
> To try it out, still in the Scrapy shell, let's first store the first returned URL into a
> variable:
>
> ~~~
> >>> testurl = response.xpath("//td[@class='mppcell']/a/@href").extract_first()
> ~~~
> {: .source}
>
> Then, we can try passing it on to the `urljoin()` method:
>
> ~~~
> >>> response.urljoin(testurl)
> ~~~
> {: .source}
>
> which returns
>
> ~~~
> 'http://www.ontla.on.ca/web/members/members_detail.do?locale=en&ID=7085'
> ~~~
> {: .output}
>
> We see that Scrapy was able to reconstruct the absolute URL by combining the URL of the current page context
> (the page in the `response` object) and the relative link we had stored in `testurl`.
>
{: .callout}


## Extracting URLs using the spider

Armed with the correct query, we can now update our spider accordingly. The `parse`
methods returns the contents of the scraped page inside the `response` object. That response
objects supports a variety of methods to act on its contents:

|Method|Description|
|-----------------|:-------------|
|`xpath()`| Returns a list of objects, each of which contains the nodes selected by the XPath query given as argument|
|`css()`| Works similarly to the `xpath()` method, but uses CSS expressions to select elements.|
|`extract()`| Returns the entire text content of the response object, as a unicode string.|
|`re()`| Returns a list of unicode strings extracted by applying the regular expression given as argument.|

Since we have an XPath query we know will extract the URLs we are looking for, we can now use
the `xpath()` method and update the spider accordingly:

(editing `ontariompps/ontariompps/spiders/mppaddresses.py`)

~~~
import scrapy

class MppaddressesSpider(scrapy.Spider):
    name = "mppaddresses"
    allowed_domains = ["www.ontla.on.ca"]
	start\_urls = ['http://www.ontla.on.ca/web/members/members_current.do?locale=en/']

    def parse(self, response):
        for url in response.xpath("//*[@class='mppcell']/a/@href").extract():
            print(response.urljoin(url))
~~~
{: .source}

> ## Looping through results
>
> Why are we using `extract()` instead of `extract_first()` in the code above?
> Why is the `print` statement inside a `for` clause?
> > ## Solution
> > 
> > We are not only interested in the first extracted URL but in all of them.
> > `extract_first()` only returns the content of the first in a series of
> > selected elements, while `extract()` will return all of them in the form of an
> > array.
> > 
> > The `for` syntax allows us to loop through each of the returned elements one by one.
> >
> {: .solution}
{: .challenge}

We can now run our new spider:

~~~
scrapy crawl mppaddresses
~~~
{: .source}

which produces a result similar to:

~~~
2017-02-26 23:06:10 [scrapy.utils.log] INFO: Scrapy 1.3.2 started (bot: ontariompps)
(...)
http://www.ontla.on.ca/web/members/members_detail.do?locale=en&ID=2111
http://www.ontla.on.ca/web/members/members_detail.do?locale=en&ID=2139
http://www.ontla.on.ca/web/members/members_detail.do?locale=en&ID=7174
http://www.ontla.on.ca/web/members/members_detail.do?locale=en&ID=2148
(...)
2017-02-26 23:06:11 [scrapy.core.engine] INFO: Spider closed (finished)
~~~
{: .output}

We can now pat ourselves on the back, as we have successfully completed the first stage
of our project by successfully extracing all URLs leading to the minister profiles!

> ## Limit the number of URL to scrape through while debugging
>
> We've seen by testing the code above that we are able to successfully gather all URLs from
> the list of MPPs. But while we're working through to the final code that will allow us
> the extract the data we want from those pages, it's probably a good idea to only run it
> on a handful of pages at a time.
>
> This will not only run faster and allow us to iterate more quickly between different
> revisions of our code, it will also not burden the server too much while we're debugging.
> This is probably not such an issue for a couple of hundred of pages, but it's good
> practice, as it can make a difference for larger scraping projects. If you are planning
> to scrape a massive website with thousands of pages, it's better to start small. Other
> visitors to that site will thank you for respecting their legitimate desire to access
> it while you're debugging your scraper...
>
> An easy way to limit the number of URLs we want to send our spider to is to
> take advantage of the fact that the `extract()` method returns a list of matching elements.
> In Python, lists can be _sliced_ using the `list[start:end]` syntax and we can leave out
> either the `start` or `end` delimiters:
> 
> ~~~
> list[start:end] # items from start through end-1
> list[start:]    # items from start through the rest of the array
> list[:end]      # items from the beginning through end-1
> list[:]         # all items
> ~~~
> 
> We can therefore edit our spider thusly to only scrape the first five URLs:
>
> ~~~
> import scrapy
> 
>	class MppaddressesSpider(scrapy.Spider):
>		name = "mppaddresses"
>		allowed_domains = ["www.ontla.on.ca"]
>	    start_urls = ['http://www.ontla.on.ca/web/members/members_current.do?locale=en/']
>
>	    def parse(self, response):
>	        for url in response.xpath("//*[@class='mppcell']/a/@href").extract()[:5]:
>	            print(response.urljoin(url))
> ~~~
> {: .source}
>
> Note that this only works if there are at least five URLs that are being returned, which
> is the case here.
> 
{: .callout}

## Recursive scraping

Now that we were successful in harvesting the URLs to the detail pages, let's begin by editing
our spider to instruct it to visit those pages one by one.

For this, let's begin by defining a new method `get_details` that we want to run on the detail pages:


(editing `ontariompps/ontariompps/spiders/mppaddresses.py`)

~~~
import scrapy

class MppaddressesSpider(scrapy.Spider):
    name = "mppaddresses" # The name of this spider
    
    # The allowed domain and the URLs where the spider should start crawling:
    allowed_domains = ["www.ontla.on.ca"]
    start_urls = ['http://www.ontla.on.ca/web/members/members_current.do?locale=en/']

    def parse(self, response):
		# The main method of the spider. It scrapes the URL(s) specified in the
		# 'start_url' argument above. The content of the scraped URL is passed on
		# as the 'response' object.
        
		for url in response.xpath("//*[@class='mppcell']/a/@href").extract()[:5]:
			# This loops through all the URLs found inside an element of class 'mppcell'
			
			# Constructs an absolute URL by combining the response’s URL with a possible relative URL:
			full_url = response.urljoin(url)
			print("Found URL: "+full_url)
            
			# The following tells Scrapy to scrape the URL in the 'full_url' variable
			# and calls the 'get_details() method below with the content of this
			# URL:
			yield scrapy.Request(full_url, callback=self.get_details)
    
	def get_details(self, response):
		# This method is called on by the 'parse' method above. It scrapes the URLs
		# that have been extracted in the previous step.
		print("Visited URL: "+response.url)
~~~
{: .source}

We've also added some comments to the code to make it easier to read and understand.

If we now run our spider again:

~~~
scrapy crawl mppaddresses
~~~
{: .source}

We should see the result of our `print` statements intersped with the regular Scrapy
debugging output, something like:

~~~
2017-02-27 20:39:42 [scrapy.utils.log] INFO: Scrapy 1.3.2 started (bot: ontariompps)
(...)
2017-02-27 20:39:43 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.ontla.on.ca/web/members/members_current.do?locale=en/> (referer: None)
Found URL: http://www.ontla.on.ca/web/members/members_detail.do?locale=en&ID=7085
Found URL: http://www.ontla.on.ca/web/members/members_detail.do?locale=en&ID=7275
(...)
2017-02-27 20:39:44 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.ontla.on.ca/web/members/members_detail.do?locale=en&ID=7085> (referer: http://www.ontla.on.ca/web/members/members_current.do?locale=en/)
(...)
Visited URL: http://www.ontla.on.ca/web/members/members_detail.do?locale=en&ID=7085
(...)
2017-02-27 20:39:44 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.ontla.on.ca/web/members/members_detail.do?locale=en&ID=7086> (referer: http://www.ontla.on.ca/web/members/members_current.do?locale=en/)
(...)
Visited URL: http://www.ontla.on.ca/web/members/members_detail.do?locale=en&ID=7225
(...)
2017-02-27 20:39:44 [scrapy.core.engine] INFO: Closing spider (finished)
~~~
{: .output}

We've truncated the results above to make it easier to read, but on your console
you should see that all 5 URLs (remember, we are limiting the number of URLs to scrape
for now) have been first "found" (by the `parse()` method) and then "visited"
(by the `get_details()` method).

> ## Asynchronous requests
>
> If you look closely at the output of the code we've just run, you might be surprised
> to see that the "Found URL" and "Visited URL" statements didn't necessarily get
> printed out one after the other, as we might expect.
>
> The reason this is so is that Scrapy requests are [scheduled and processed asynchronously](http://stackoverflow.com/questions/748175/asynchronous-vs-synchronous-execution-what-does-it-really-mean).
> This means that Scrapy doesn’t need to wait for a request to be finished and processed
> before it runs another or do other things in the meantime. This is more efficient
> than running each request one after the other, and it also allows for Scrapy to keep
> working away even if some requests fails for whatever reason.
>
> This is especially advantageous when scraping large websites. Depending on the resources
> of the computer on which Scrapy runs, it can scrape hundreds or thousands of pages
> simultaneously.
> 
{: .callout}

## Scrape the detail pages

Now that we are able to visit each one of the detail pages, we should work on getting the
data that we want out of them. In our example, we are primarily looking
to extract the following details:

* Phone number(s)
* Email address(es)

Unfortunately, it looks like the content of those pages is not consistent. Sometimes, only
one email address is displayed, sometimes more than one. Some MPPs have one Constituency
address, others have more than one, etc.

To simplify, we are going to stop at the first phone number and the first
email address we find on those pages, although in a real life scenario we might be interested
in writing more precise queries to make sure we are collecting the right information.

> ## Scrape phone number and email address
> Write XPath queries to scrape the first phone number and the first email address
> displayed on each of the detail pages that are linked from
> the [Ontario MPPs list](http://www.ontla.on.ca/web/members/members_current.do?locale=en).
>
> Try out your queries on a handful of detail pages to make sure you are getting
> consistent results.
>
> Tips:
>
> * Look at the source code and try out XPath your queries until you find what
>   you are looking for.
> * You can either use the browser console or the Scrapy shell mode (see above)
>   to try out your queries.
> * The syntax for selecting an element like `<div class="mytarget">` is `div[@class = 'mytarget']`.
> * The syntax to select the value of an attribute of the type `<element attribute="value">`
>   is `element/@attribute`.
>
> > ## Solution
> >
> > This returns an array of phone (and fax) numbers (using the Scrapy shell):
> >
> > ~~~
> > scrapy shell "http://www.ontla.on.ca/web/members/members_detail.do?locale=en&ID=7085"
> > >>> response.xpath("//div[@class='phone']/text()").extract()
> > ~~~
> > {: .source}
> >
> > ~~~
> > [['\n416-325-6200\n', '\n416-325-6195\n', '\n416-243-7984\n', '\n416-243-0327\n']
> > ~~~
> > {: .output}> >
> >
> > And this returns an array of email addresses:
> >
> > ~~~
> > >>> response.xpath("//div[@class='email']/a/text()").extract()
> > ~~~
> > {: .source}> >
> >
> > ~~~
> > ['\nlalbanese.mpp@liberal.ola.org\n', '\nlalbanese.mpp.co@liberal.ola.org\n']
> > ~~~
> > {: .output}> >
> >
> {: .solution}
{: .challenge}

Once we have found XPath queries to run on the detail pages and are happy with the result,
we can add them to the `get_details()` method of our spider:


(editing `ontariompps/ontariompps/spiders/firstspider.py`)

~~~
import scrapy

class MppaddressesSpider(scrapy.Spider):
    name = "mppaddresses" # The name of this spider
    
    # The allowed domain and the URLs where the spider should start crawling:
    allowed_domains = ["www.ontla.on.ca"]
    start_urls = ['http://www.ontla.on.ca/web/members/members_current.do?locale=en/']

    def parse(self, response):
		# The main method of the spider. It scrapes the URL(s) specified in the
		# 'start_url' argument above. The content of the scraped URL is passed on
		# as the 'response' object.
        
		for url in response.xpath("//*[@class='mppcell']/a/@href").extract()[:5]:
			# This loops through all the URLs found inside an element of class 'mppcell'
			
			# Constructs an absolute URL by combining the response’s URL with a possible relative URL:
			full_url = response.urljoin(url)
			print("Found URL: "+full_url)
            
			# The following tells Scrapy to scrape the URL in the 'full_url' variable
			# and calls the 'get_details() method below with the content of this
			# URL:
			yield scrapy.Request(full_url, callback=self.get_details)
    
	def get_details(self, response):
		# This method is called on by the 'parse' method above. It scrapes the URLs
		# that have been extracted in the previous step.
		name_detail = response.xpath("normalize-space(//div[@class='mppdetails']/h1/text())").extract_first()
		phone_detail = response.xpath("normalize-space(//div[@class='phone']/text())").extract_first()
		email_detail = response.xpath("normalize-space(//div[@class='email']/a/text())").extract_first()
		print("Found details: " + name_detail + ', ' + phone_detail + ', ' + email_detail)
~~~
{: .source}

Running our scraper again

~~~
scrapy crawl mppaddresses
~~~
{: .source}

produces something like

~~~
2017-02-27 20:39:42 [scrapy.utils.log] INFO: Scrapy 1.3.2 started (bot: ontariompps)
(...)
2017-02-27 20:39:43 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.ontla.on.ca/web/members/members_current.do?locale=en/> (referer: None)
Found URL: http://www.ontla.on.ca/web/members/members_detail.do?locale=en&ID=7085
Found URL: http://www.ontla.on.ca/web/members/members_detail.do?locale=en&ID=7275
(...)
Found details: Ted Arnott, MPP (Wellington—Halton Hills), 416-325-3880, ted.arnott@pc.ola.org
Found details: Teresa J. Armstrong, MPP (London—Fanshawe), 416-325-1872, tarmstrong-qp@ndp.on.ca
(...)
2017-02-27 20:39:44 [scrapy.core.engine] INFO: Closing spider (finished)
~~~
{: .output}

We appear to be getting somewhere! The last step is doing something useful with the
scraped data instead of printing it out on the terminal. Enter the Scrapy Items.

## Using Items to store scraped data

FIXME: add more details here

This is where the data extracted will be stored. Items work like Python dictionaries.

Edit `ontariompps/ontariompps/items.py`

(You will see Scrapy has pre-populated this file)

~~~
import scrapy


class OntariomppsItem(scrapy.Item):
    # define the fields for your item here like:
    name = scrapy.Field()
    email = scrapy.Field()
    phone = scrapy.Field()
~~~
{: .source}

Then edit the spider accordingly:

(editing `ontariompps/ontariompps/spiders/firstspider.py`)

~~~
import scrapy
from ontariompps.items import OntariomppsItem # We need this so that Python knows about the item object

class MPPSpider(scrapy.Spider):
	name = "getemails"	# The name of this spider

	# The allowed domain and the URLs where the spider should start crawling:
	allowed_domains = ["www.ontla.on.ca"]
	start_urls = ["http://www.ontla.on.ca/web/members/members_current.do?locale=en"]

	def parse(self, response):
		# The main method of the spider. The content of the scraped URL is passed on
		# as the response object:
		for url in response.xpath("//*[@class='mppcell']/a/@href").extract():
		    url = 'http://www.ontla.on.ca/web/members/' + url
		    yield scrapy.Request(url, callback=self.get_details) # callback to get_details

	def get_details(self, response):
	    item = OntariomppsItem() # Defining a new Item object
	    item['name'] = response.xpath("normalize-space(//div[@class='mppdetails']/h1/text())").extract()[0]
	    item['phone'] = response.xpath("normalize-space(//div[@class='phone']/text())").extract()[0]
	    item['email'] = response.xpath("normalize-space(//div[@class='email']/a/text())").extract()[0]
	    yield item # Return the item

~~~
{: .source}

Running it like this:

~~~
scrapy crawl getemails
~~~
{: .source}

will by default just output the extracted information in JSON snippets mixed with the
usual Scrapy logging information:

~~~
(...)
2016-11-08 22:27:13 [scrapy] DEBUG: Scraped from <200 http://www.ontla.on.ca/web/members/members_detail.do?locale=en&ID=7>
{'email': 'gbisson@ndp.on.ca',
 'name': 'Gilles Bisson, MPP (Timmins—James Bay)',
 'phone': '416-325-7122'}
2016-11-08 22:27:13 [scrapy] DEBUG: Scraped from <200 http://www.ontla.on.ca/web/members/members_detail.do?locale=en&ID=7275>
{'email': 'ganderson.mpp.co@liberal.ola.org',
 'name': 'Granville Anderson, MPP (Durham)',
 'phone': '416-325-5494'}
(...)
~~~
{: .output}

But by adding an `-o file.ext` argument, we can save the resulting data directly to a file
in the format we prefer:

~~~
scrapy crawl getemails -o mpps.csv
~~~
{: .source}

will produce a file called `mpps.csv` containing all the extracted information in CSV format:

~~~
less mpps.csv
~~~
{: .source}

~~~
email,name,phone
lalbanese.mpp@liberal.ola.org,"Hon Laura Albanese, MPP (York South—Weston)",416-325-6200
vdhillon.mpp.co@liberal.ola.org,"Vic Dhillon, MPP (Brampton West)",416-325-0241
jdickson.mpp@liberal.ola.org,"Joe Dickson, MPP (Ajax—Pickering)",416-327-0653
sdelduca.mpp.co@liberal.ola.org,"Hon Steven Del Duca, MPP (Vaughan)",416-327-9200
(...)
~~~
{: .output}

By changing the file extension, we can also export it as JSON or XML:

~~~
scrapy crawl getemails -o mpps.xml
~~~
{: .source}

Refer to the [Scrapy documentation](http://doc.scrapy.org/en/latest/topics/feed-exports.html#topics-feed-exports)
for a full list of supported formats.


> ## Add other data elements to the spider
>
> Try modifying the spider code to add more data extracted from the MPP detail page.
> Remember to edit the Item definition to allow for all extracted fields to be taken
> care of.
>
{: .challenge}

FIXME: more challenges, conclusion
FIXME: add more explanations about structure of Scrapy project, how it works, different elements.
FIXME: add brief description of classes and objects

![Scrapy architecture overview](https://doc.scrapy.org/en/latest/_images/scrapy_architecture_02.png)
