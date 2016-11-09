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

FIXME: this section needs more challenges

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
[setup instructions](/setup). If you install Scrapy as suggested there, it should take care to install all
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

Looking at this result and at the source code of the page, we realize that the URLs are all
_relative_ to that page. They are all missing part of the URL to become _absolute_ URLs, which
we will need if we want to ask our spider to visit those URLs to scrape more data. We will therefore
need to append `http://www.ontla.on.ca/web/members/` to all of those ULRs before we can use them.


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

Note also the following additional changes:
* Since `response.xpath(...)` will return more than one object (as many as there are URLs on the page),
  we are using a `for` construct to loop through all of them.
* We use the `extract()` method on the returned objects to extract their text content
* As we saw earlier, the URLs extracted from the page are all relative. We must therefore append
  `http://www.ontla.on.ca/web/members/` to all of them to make them absolute.

We can now run our new spider:

~~~
scrapy crawl getemails
~~~
{: .source}

which produces a result similar to:

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

## Recursive scraping

Now that we were successful in harvesting the URLs to the detail pages, let's look at those
detail pages to find the information that needs to be extracted. We are primarily looking
to extract the following details:

* Phone number(s)
* Email address(es)

Unfortunately, it looks like the content of those pages is not consistent. Sometimes, only
one email address is displayed, sometimes more than one. Some MPPs have one Constituency
address, others have more than one, etc.

To simplify, we are going to stop at the first phone number and the first
email address we find on those pages, although in a real life scenario we might be interested
in writing more precise queries to make sure we are collecting the right information.

> ## Scrape the phone number and email address
> Write XPath queries to scrape the first phone number and the first email address
> displayed on each of the detail pages that are linked from
> the [Ontario MPPs list](http://www.ontla.on.ca/web/members/members_current.do?locale=en).
>
> Try out your queries on a handful of detail pages to make sure you are getting
> consistent results.
>
> Tips:
> 
> * Look at the source code and try out XPath queries in the console until you find what
>   you are looking for.
> * The syntax for selecting an element like `<div class="mytarget">` is `div[@class = 'mytarget']`.
> * The syntax to select the value of an attribute of the type `<element attribute="value">`
>   is `element/@attribute`.
>
> > ## Solution
> > 
> > This returns an array of phone (and fax) numbers:
> > 
> > ~~~
> > > $x("//div[@class='phone']/text()")
> > ~~~
> > {: .source}
> >
> > And this returns an array of email addresses:
> > 
> > ~~~
> > > $x("//div[@class='email']/a/text()")
> > ~~~
> > {: .source}> > 
> >
> {: .solution}
{: .challenge}

> ## Use the Scrapy shell to try out XPath queries
>
> Another way to try out XPath queries is to use Scrapy in interactive mode.
> By running
>
> ~~~
> scrapy shell "http://www.ontla.on.ca/web/members/members_detail.do?locale=en&ID=7085"
> ~~~
> {: .source}
>
> we get an iPython shell in which the `response` object containing the content of the page
> passed on to Scrapy is available, as well as the methods to apply selectors to it.
> We can use that to refine our queries until we get exactly what we are looking for:
>
> ~~~
> In [1]: response.xpath("//div[@class='email']/a/text()").extract()
> Out[1]: ['\nlalbanese.mpp@liberal.ola.org\n', '\nlalbanese.mpp.co@liberal.ola.org\n']
> 
> In [2]: response.xpath("//div[@class='email']/a/text()").extract()[0]
> Out[2]: '\nlalbanese.mpp@liberal.ola.org\n'
> 
> In [3]: response.xpath("normalize-space(//div[@class='email']/a/text())").extract()[0]
> Out[3]: 'lalbanese.mpp@liberal.ola.org'
> ~~~
> {: .source}
>
> Use `Ctrl-D` (`Ctrl-Z` on Windows) to get out of the interactive shell.
{: .callout}


FIXME: describe this step in more detail

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
		    yield scrapy.Request(url, callback=self.get_details)
		    
	def get_details(self, response):
	    name_detail = response.xpath("normalize-space(//div[@class='mppdetails']/h1/text())").extract()[0]
	    phone_detail = response.xpath("normalize-space(//div[@class='phone']/text())").extract()[0]
	    email_detail = response.xpath("normalize-space(//div[@class='email']/a/text())").extract()[0]
	    print(name_detail + ', ' + phone_detail + ', ' + email_detail)
~~~
{: .source}

yield is like return, but for potentially large datasets. Basically when you don't know from the beginning
how big the dataset will be.
Also: return implies that the function is returning control of execution to the point where the function was called. yield, however, implies that the transfer of control is temporary and voluntary, and our function expects to regain it in the future. From http://jeffknupp.com/blog/2013/04/07/improve-your-python-yield-and-generators-explained/

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

Then edit the spider accordingly

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

![Scrapy architecture overview](https://doc.scrapy.org/en/latest/_images/scrapy_architecture_02.png)



