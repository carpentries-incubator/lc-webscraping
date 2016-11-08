---
title: "IN DEVELOPMENT: Web scraping using Python and Scrapy"
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
[setup instructions](setup/). If you install Scrapy as suggested there, it should take care to install all
required libraries as well.

You can verify that you have the latest version of Scrapy installed by typing

~~~
scrapy version
~~~
{: .source}

in a shell. If all is good, you should get the following back:

~~~
Scrapy 1.2.0
~~~
{: .output}

Scrapy version 1.2.0 is the most current at the time of writing this lesson. If you have a newer version,
you should be fine as well.

To introduce the use of Scrapy, we will reuse the same example we used with import.io in the previous
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
cancolleges/			# the root project directory
	scrapy.cfg		# deploy configuration file

	cancolleges/		# project's Python module, you'll import your code from here
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

## Writing our first spider

Spiders are the business end of the scraper. It's the bit of code that combs through a website and harvests data.
Their general structure is as follows:
* One or more _start URLs_, where the spider will start crawling
* A list of _allowed domains_ to constrain the pages we allow our spider to crawl (this is a good way to
  avoid mistakenly writing an out-of-hand spider that mistakenly starts crawling the entire Internet...)
* A method called `parse` in which we will write what data the spider should be looking for on the pages
  it visits, what links to follow and how to parse found data.

Let's start with an example. Using our favourite text editor, let's create a file called `firstspider.py` 
inside the `ontariompps/ontariompps/spiders` directory with the following contents:

~~~
import scrapy

class MPPSpider(scrapy.Spider):
	name = "firstspider"	# The name of this spider
	
	# The allowed domain and the URLs where the spider should start crawling:
	allowed_domains = ["www.ontla.on.ca"]
	start_urls = ["http://www.ontla.on.ca/web/members/members_current.do?locale=en"]
	
	def parse(self, response):
		# The main method of the spider. The content of the scraped URL is passed on
		# as the response object:
		print(response.text)
~~~
{: .source}

Once this file has been saved, we can try running it. First, let's move back to the project's
top level directory (where the `scrapy.cfg` file is) and then type

~~~
scrapy crawl firstspider
~~~
{: .source}

Note that we are able to use the name we specified in the `name` attribute of our spider`.
This should produce the following result

~~~
2016-11-07 22:28:51 [scrapy] INFO: Scrapy 1.2.0 started (bot: ontariompps)

(followed by some additional data and a bunch of HTML content)

2016-11-07 22:28:53 [scrapy] INFO: Closing spider (finished)
2016-11-07 22:28:53 [scrapy] INFO: Dumping Scrapy stats:
{'downloader/request_bytes': 476,
 'downloader/request_count': 2,
 'downloader/request_method_count/GET': 2,
 'downloader/response_bytes': 34319,
 'downloader/response_count': 2,
 'downloader/response_status_count/200': 2,
 'finish_reason': 'finished',
 'finish_time': datetime.datetime(2016, 11, 8, 3, 28, 53, 447656),
 'log_count/DEBUG': 3,
 'log_count/INFO': 7,
 'response_received_count': 2,
 'scheduler/dequeued': 1,
 'scheduler/dequeued/memory': 1,
 'scheduler/enqueued': 1,
 'scheduler/enqueued/memory': 1,
 'start_time': datetime.datetime(2016, 11, 8, 3, 28, 52, 980198)}
2016-11-07 22:28:53 [scrapy] INFO: Spider closed (finished)
~~~
{: .output}

The HTML that was printed before the "Closing spider" statement was in fact the output of our script's
`print(response.text)` function. This means that our first run was successful, the spider opened
the URL we specified, passed it to the `parse` method as the `response` object, which was then printed
to the console using the `print()` function.

Dumping data on the console's standard output is not very practical, however, so we can modify our
script to have the resulted data be stored in a file:

~~~
import scrapy

class MPPSpider(scrapy.Spider):
	name = "firstspider"	# The name of this spider
	
	# The allowed domain and the URLs where the spider should start crawling:
	allowed_domains = ["www.ontla.on.ca"]
	start_urls = ["http://www.ontla.on.ca/web/members/members_current.do?locale=en"]
	
	def parse(self, response):
		with open("test.html", 'wb') as file:
			file.write(response.body)
~~~
{: .source}

Now, if we run our spider again

~~~
scrapy crawl firstspider
~~~
{: .source}

the output on the console should be much shorter. Instead, there is now a file called
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
(...)
~~~
{: .output}

## Define which elements to scrape using XPath


FIXME starts here
	

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

~~~
> $x("//*[@class='mppcell']")
~~~
{: .source}

	This actually only works if there is only one class. More general answer:
	https://stackoverflow.com/questions/8808921/selecting-a-css-class-with-xpath


Drilling down:

~~~
> $x("//*[@class='mppcell']/a/@href")
~~~
{: .source}

Once we found what we're looking for, let's edit the Spider accordingly:

Scrapy Selectors support different methods:
- xpath(): returns a list of selectors, each of which represents the nodes selected by the xpath expression given as argument.
		This is what we'll use
- css(): returns a list of selectors, each of which represents the nodes selected by the CSS expression given as argument.
- extract(): returns a unicode string with the selected data.
- re(): returns a list of unicode strings extracted by applying the regular expression given as argument.


~~~
import scrapy

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
		    print(url)
~~~
{: .source}



~~~
scrapy crawl getemails
~~~
{: .source}

~~~
2016-11-07 22:55:19 [scrapy] INFO: Scrapy 1.2.0 started (bot: ontariompps)
(...)
http://www.ontla.on.ca/web/members/members_detail.do?locale=en&ID=2111
http://www.ontla.on.ca/web/members/members_detail.do?locale=en&ID=2139
http://www.ontla.on.ca/web/members/members_detail.do?locale=en&ID=7174
http://www.ontla.on.ca/web/members/members_detail.do?locale=en&ID=2148
(...)
2016-11-07 22:55:20 [scrapy] INFO: Spider closed (finished)
~~~
{: .output}


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
