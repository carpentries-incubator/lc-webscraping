---
title: "Web scraping using Python and Scrapy"
teaching: 60
exercises: 30
questions:
- "How can scraping a web site be automated?"
- "How can I setup a scraping project using the Scrapy framework for Python?"
- "How do I tell Scrapy what elements to scrape from a webpage?"
- "What to do with the data extracted with Scrapy?"
objectives:
- "Setting up a Scrapy project."
- "Understanding the various elements of a Scrapy projects."
- "Creating a spider to scrape a website and extract specific elements."
- "Storing the extracted data."
keypoints:
- "Scrapy is a Python framework that can be use to scrape content from the web."
- "A Scrapy project is a set of configuration files and pieces of code that tell Scrapy what to do."
- "In Scrapy, a \"Spider\" is the code that tells it what to do on a specific website."
- "A Scrapy project can have more than one spider but needs at least one."
- "With Scrapy, we can use XPath, CSS selectors and Regular Expressions to define what elements to scrape from a page."
- "Extracted data can be stored in `Item` objects. Such objects must be defined before they can be used."
- "Scrapy will automatically stored extracted data in CSV, JSON or XML format based on the file extension given in the -o option."
---

## Recap
Here is what we have learned so far:

* We can use XPath queries to select what elements on a page to scrape.
* We can look at the HTML source code of a page to find how target elements are structured and
  how to select them.
* We can use the browser console and the `$x(...)` function to try out XPath queries on a live site.
* We can use the Scraper browser extension to scrape data from a single web page. Its interface even
  tries to guess the XPath query to target the elements we are interested in.

This is quite a toolset already, and it's probably sufficient for a number of use cases, but there are
limitations in using the tools we have seen so far. Scraper requires manual intervention and only scrapes
one page at a time. Even though it is possible to save a query for later, it still requires us to operate
the extension.

## Introducing Scrapy

Enter [Scrapy](https://scrapy.org/)! Scrapy is a _framework_ for the [Python](https://swcarpentry.github.io/python-novice-inflammation/)
programming language.

>
> A framework is a reusable, "semi-complete" application that can be specialized to produce custom applications.
> (Source: [Johnson & Foote, 1988](http://www1.cse.wustl.edu/~schmidt/CACM-frameworks.html)) [ACM link](https://dl.acm.org/citation.cfm?id=262798)
>

In other words, the Scrapy framework provides a set of Python scripts that contain most of the code required
to use Python for web scraping. We need only to add the last bit of code required to tell Python what
pages to visit, what information to extract from those pages, and what to do with it. Scrapy also
comes with a set of scripts to setup a new project and to control the scrapers that we will create.

It also means that Scrapy doesn't work on its own. It requires a working Python installation
(Python 2.7 and higher or 3.4 and higher - it should work in both Python 2 and 3), and a series of
libraries to work. If you haven't installed Python or Scrapy on your machine, you can refer to the
[setup instructions](/lc-webscraping/setup). If you install Scrapy as suggested there, it should take care to install all
required libraries as well.

You can verify that you have the latest version of Scrapy installed by typing

~~~
$ scrapy version
~~~
{: .language-bash}

in a shell. If all is good, you should get the following back (as of February 2017):

~~~
Scrapy 1.5
~~~
{: .output}

If you have a newer version, you should be fine as well.

To introduce the use of Scrapy, we will reuse the same example we used in the previous section. 


We will be working the australian MP data and paths you were just working on. However, instead of scraping the *first twelve* MPs, we will be scraping all of them, and going into their profiles as well. 

* First dozen [Members of the Australian Parliment](https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0). Backup link: [https://perma-archives.org/warc/8ATF-RT3Q/https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=](https://perma-archives.org/warc/8ATF-RT3Q/https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=).


## Setup a new Scrapy project

The first thing to do is to create a new Scrapy project.

Let's navigate first to a folder on our drive where we want to create our project (refer to Software
Carpentry's lesson about the [UNIX shell](http://swcarpentry.github.io/shell-novice/) if you are
unsure about how to do that). Then, type the following

~~~
$ scrapy startproject austmps
~~~
{: .language-bash}

where `austmps` is the name of our project.

Scrapy should respond will something similar to (the paths will reflect your own file structure)

~~~
New Scrapy project 'austmps', using template directory '/home/brian/.local/lib/python3.6/site-packages/scrapy/templates/project', created in:
    /home/brian/resbaz/austmps

You can start your first spider with:
    cd austmps
    scrapy genspider example example.com

~~~
{: .output}

If we list the files in the directory we ran the previous command

~~~ 
$ ls -F
~~~
{: .language-bash}

we should see that a new directory was created:

~~~
austmps/
~~~
{: .output}

(alongside any other files and directories you had lying around previously). Moving into that new
directory

~~~
$ cd austmps
~~~
{: .language-bash}

we can see that it contains two items:

~~~
$ ls -F
~~~
{: .language-bash}

~~~
austmps/    scrapy.cfg
~~~
{: .output}

Yes, confusingly, Scrapy creates a subdirectory called `austmps` within the `austmps` project
directory. Inside that _second_ directory, we see a bunch of additional files:

~~~
$ ls -F austmps
~~~
{: .language-bash}

~~~
__init__.py  items.py  middlewares.py  pipelines.py  __pycache__/  settings.py  spiders/
~~~
{: .output}

To recap, here is the structure that `scrapy startproject` created:

~~~
austmps/            # the root project directory
    scrapy.cfg      # deploy configuration file

    austmps/        # project's Python module, you'll import your code from here
        __init__.py

        items.py        # Holds the structure of the data we want to collect

        middlewares.py  # We aren't going to use this today. 
                        # A file to manipulate how spiders process input, in case pretending to be a normal HTTP browser doesn't work.

        pipelines.py    # We aren't going to use this today, either.
                        # Once our spider writes things to the "item," we can use pipelines to do additional processing before we export it.

        settings.py # project settings file

        spiders/        # a directory where you'll later put your spiders
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
{: .language-bash}
  
We just need to replace `<SCRAPER NAME>` with the name we want to give our spider and `<START URL>` with
the URL we want to spider to crawl. In our case, we can type:

~~~
$ scrapy genspider austmpdata "www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0"
~~~
{: .language-bash}

> ## Important note about URLs and the command line.
> 
> Note how in the URL above we put it in quotation marks? This tells bash to treat everything inside the quotation marks as one argument instead of trying to parse it as a bash command. The character `&` is especially problematic. 
{: .callout}


This will create a file called `austmpdata.py` inside the `spiders` directory of our project.
Using our favourite text editor, let's open that file. It should look something like this:

~~~
import scrapy


class AustmpdataSpider(scrapy.Spider):
    name = 'austmpdata' # The name of this spider

    # The allowed domain and the URLs where the spider should start crawling:

    allowed_domains = ['www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0']
    start_urls = ['http://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0/']

    # And a 'parse' function, which is the main method of the spider. The content of the scraped
    # URL is passed on as the 'response' object:

    def parse(self, response):
        pass

~~~
{: .language-python}

Note that here some comments have been added for extra clarity, they will not be there upon
first creating a spider.

> ## Don't include `http://` when running `scrapy genspider`
>
> The current version of Scrapy (1.5.0 - June 2018) apparently only expects URLs without
> `http://` when running `scrapy genspider`. If you do include the `http` prefix, you might
> see that the value in `start_url` in the generated spider will have that prefix twice, because
> Scrapy appends it by default. This will cause your spider to fail. Either run `scrapy genspider`
> without `http://` or check the resulting spider so that it looks like the code above.
>
{: .callout}

> ## Object-oriented programming and Python classes
>
> You might be unfamiliar with the `class AustmpdataSpider(scrapy.Spider)` syntax used above.
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
> The code in the example above is defining a __class__ called `austmpdataSpider` that __inherits__ the `Spider` class
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
has automatically generated. 
 
> ## Paying attention to `allowed_domains`
>
> Looking at the code that was generated by `genspider`, we see that by default the entire start URL 
> has ended up in the `allowed_domains` attribute. 
> 
> Is this desired? What do you think would happen
> if later in our code we wanted to scrape a page living at the address `https://www.aph.gov.au/Senators_and_Members/Parliamentarian?MPID=EZ5`? 
> > ## Solution
> > 
> > `allowed_domains` is a safeguard for our spider, it will restrict its ability to scrape pages
> > outside of a certain realm. A URL is structured as a path to a resource, with the root directory
> > at the beginning and a set of "subdirectories" after that. In `www.mydomain.ca/house/dog.html`,
> > `http://www.mydomain.ca/` is the root, `house/` is a first level directory and `dog.html` is a file
> > sitting inside the `house/` directory.
> >
> > If we restrict a Scrapy spider with `allowed_domains = ["www.mydomain.ca/house"]`, it means
> > that the spider will be able to scrape everything that's inside the `www.mydomain.ca/house/` directory (including
> > subdirectories), but not, say, pages that would be in `www.mydomain.ca/garage/`. However,
> > if we set `allowed_domains = ["www.mydomain.ca/"]`, the spider can scrape both the contents of
> > the `house/` and `garage/` directories.
> > 
> > To answer the question, leaving `allowed_domains = ["www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0"]`
> > would restrict the spider to pages with URLs of the same pattern, and 
> > `https://www.aph.gov.au/Senators_and_Members/Parliamentarian?MPID=EZ5`
> > is of a different pattern, so Scrapy would prevent the spider from scraping it.
> >
> {: .solution}
>
> How should `allowed_domains` be set to prevent this from happening?
>
> > ## Solution
> > 
> > We should let the spider scrape all pages inside the `https://www.aph.gov.au` domain by editing
> > it so that it reads:
> >
> > ~~~
> > allowed_domains = ["www.aph.gov.au"]
> > ~~~
> > {: .source .language-python}
> >
> {: .solution}
>
{: .challenge} 
 
Here is what the spider looks like after cleaning the code a little:

(editing `austmps/austmps/spiders/austmpdata.py`)

~~~
import scrapy


class AustmpdataSpider(scrapy.Spider):
    name = 'austmpdata'
    allowed_domains = ['www.aph.gov.au']
    start_urls = ['http://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0/']

    def parse(self, response):
        pass


~~~
{: .language-python}

Don't forget to save the file once changes have been applied.

## Running the spider

Now that we have a first spider setup, we can try running it. Going back to the Terminal, we first make sure
we are located in the project's top level directory (where the `scrapy.cfg` file is) by using `ls`, `pwd` and
`cd` as required, then we can run:

~~~
$ scrapy crawl austmpdata  -s DEPTH_LIMIT=1
~~~
{: .language-bash}

> ## A note on DEPTH_LIMIT
>
> We're running `-s DEPTH_LIMIT=1` to limit how many pages the spider crawls. It's always a good idea not to run with "unlimited depth" while debugging a spider designed to go over multiple pages.
{: .callout}


Note that we can now use the name we have chosen for our spider (`austmpdata`, as specified in the `name` attribute)
to call it. This should produce the following result

~~~
2018-06-26 12:14:30 [scrapy.utils.log] INFO: Scrapy 1.5.0 started (bot: austmps)

(followed by a bunch of debugging output ending with:)

2018-06-26 12:14:31 [scrapy.statscollectors] INFO: Dumping Scrapy stats:
{'downloader/request_bytes': 997,
 'downloader/request_count': 3,
 'downloader/request_method_count/GET': 3,
 'downloader/response_bytes': 96519,
 'downloader/response_count': 3,
 'downloader/response_status_count/200': 2,
 'downloader/response_status_count/302': 1,
 'finish_reason': 'finished',
 'finish_time': datetime.datetime(2018, 6, 26, 2, 14, 31, 302189),
 'log_count/DEBUG': 4,
 'log_count/INFO': 7,
 'memusage/max': 53104640,
 'memusage/startup': 53104640,
 'response_received_count': 2,
 'scheduler/dequeued': 2,
 'scheduler/dequeued/memory': 2,
 'scheduler/enqueued': 2,
 'scheduler/enqueued/memory': 2,
 'start_time': datetime.datetime(2018, 6, 26, 2, 14, 30, 973886)}
2018-06-26 12:14:31 [scrapy.core.engine] INFO: Spider closed (finished)



~~~
{: .output}

The line that starts with `DEBUG: Crawled (200)` is good news, as it tells us that the spider was
able to crawl the website we were after. The number in parentheses is the _HTTP status code_ that
Scrapy received in response of its request to access that page. 200 means that the request was successful
and that data (the actual HTML content of that page) was sent back in response.

However, we didn't do anything with it, because the `parse` method in our spider is currently empty.
Let's change that by editing the spider as follows (note the contents of the `parse` method):

(editing `austmps/austmps/spiders/austmpdata.py`)

~~~
import scrapy

class AustmpdataSpider(scrapy.Spider):
    name = 'austmpdata'
    allowed_domains = ['www.aph.gov.au']
    start_urls = ['http://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0/']

    def parse(self, response):
        with open("test.html", 'wb') as file:
            file.write(response.body)



~~~
{: .language-python}

Now, if we go back to the command line and run our spider again. Make sure to change to your project's root directory first before running this, so we don't leave random files around.

~~~
$ scrapy crawl austmpdata  -s DEPTH_LIMIT=1
~~~
{: .language-bash}

we should get similar debugging output as before, but there should also now be a file called
`test.html` in our project's root directory:

~~~
$ ls -F
~~~
{: .language-bash}

~~~
austmps/    scrapy.cfg  test.html
~~~
{: .output}

We can check that it contains the HTML from our target URL:

~~~
$ head -n 12 test.html
~~~
{: .language-bash}

~~~
<!doctype html>
<!--[if lt IE 7 ]> <html lang="en" class="no-js ie6"> <![endif]-->
<!--[if IE 7 ]>    <html lang="en" class="no-js ie7"> <![endif]-->
<!--[if IE 8 ]>    <html lang="en" class="no-js ie8"> <![endif]-->
<!--[if IE 9 ]>    <html lang="en" class="no-js ie9"> <![endif]-->
<!--[if (gt IE 9)|!(IE)]><!--> <html class="no-js" lang="en" xml:lang="en" xmlns="http://www.w3.org/1999/xhtml"> <!--<![endif]-->

<head id="Head1"><meta charset="utf-8" /><meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" /><title>
    Senators & Members Search Results

~~~
{: .language-html .output}

# *Status quo ante*: outputting to a tab delimited file

Before we go exploring the extra capabilities that a scraper offers us, let us make sure we can output the same sort of data that our manual scraper can.

We are interested in their titles, their locations, and their member website links.

While we can use the relative path we discovered in the prior exercise, it is fragile and hard to read. Instead, let us focus on the `h4` that we know is unique to each member's profile. 

`//h4[@class='title']/a/text()`

To review, let us break down this XPath into each component.

We have `//h4[@class='title']` which says "Find any h4 element" with its css class being "title".

Then we have `/a/text()` which says "Then go into the a element inside that h4, and get me the text of whatever is inside that element."

Now let's get data out using the scrapy shell.

## Using the Scrapy shell

Scrapy provides a way to test out XPath queries, with the added benefit that we can then also debug how to further work on those queries from within Scrapy.

This is achieved by once again calling the _Scrapy shell_ from the command line:

~~~
$ scrapy shell "https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0"
~~~
{: .source}

which launches a Python console that allows us to type live Python and Scrapy code to
interact with the page which Scrapy just downloaded from the provided URL. We can see that we are inside an
interactive python console because the prompt will have changed to `>>>`:

~~~
(similar Scrapy debug text as before)

2018-06-26 12:27:02 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0> (referer: None)
[s] Available Scrapy objects:
[s]   scrapy     scrapy module (contains scrapy.Request, scrapy.Selector, etc)
[s]   crawler    <scrapy.crawler.Crawler object at 0x7f4d6b9ea0f0>
[s]   item       {}
[s]   request    <GET https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0>
[s]   response   <200 https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0>
[s]   settings   <scrapy.settings.Settings object at 0x7f4d6a2f3668>
[s]   spider     <AustmpdataSpider 'austmpdata' at 0x7f4d69e86898>
[s] Useful shortcuts:
[s]   fetch(url[, redirect=True]) Fetch URL and update local objects (by default, redirects are followed)
[s]   fetch(req)                  Fetch a scrapy.Request and update local objects 
[s]   shelp()           Shell help (print this help)
[s]   view(response)    View response in a browser
>>>
~~~
{: .output}

We can now try running the XPath query we used last lesson in scrapy.

~~~
>>> response.xpath("//h4[@class='title']/a/text()")
~~~
{: .language-python}


We see:
~~~
[<Selector xpath="//h4[@class='title']/a/text()" data='Hon Tony Abbott MP'>, <Selector xpath="//h4[@class='title']/a/text()" data='Hon Anthony Albanese MP'>, <Selector xpath="//h4[@class='title']/a/text()" data='Mr John Alexander OAM, MP'>, <Selector xpath="//h4[@class='title']/a/text()" data='Dr Anne Aly MP'>, <Selector xpath="//h4[@class='title']/a/text()" data='Hon Karen Andrews MP'>, <Selector xpath="//h4[@class='title']/a/text()" data='Hon Kevin Andrews MP'>, <Selector xpath="//h4[@class='title']/a/text()" data='Mr Adam Bandt MP'>, <Selector xpath="//h4[@class='title']/a/text()" data='Ms Julia Banks MP'>, <Selector xpath="//h4[@class='title']/a/text()" data='Hon Sharon Bird MP'>, <Selector xpath="//h4[@class='title']/a/text()" data='Hon Julie Bishop MP'>, <Selector xpath="//h4[@class='title']/a/text()" data='Hon Chris Bowen MP'>, <Selector xpath="//h4[@class='title']/a/text()" data='Mr Andrew Broad MP'>]
~~~
{: .language-html .output}

We should now exit the shell. We can do so with <kbd>Ctrl</kbd>+><kbd>c</kbd>


This tells us that we have accurately targeted scrapy. We can then use this to find more variables as soon as we're finished with the spider.

## Extracting data with our spider

Armed with the correct query, we can now update our spider accordingly. The `parse` methods returns the contents of the scraped page inside the `response` object. The `response`
object supports a variety of methods to act on its contents:

|Method|Description|
|-----------------|:-------------|
|`xpath()`| Returns a list of selectors, each of which points to the nodes selected by the XPath query given as argument|
|`css()`| Works similarly to the `xpath()` method, but uses CSS expressions to select elements.|

Those methods will return objects of a different type, called `selectors`. As their name implies,
these objects are "pointers" to the elements we are looking for inside the scraped page. In order
to get the "content" that the `selectors` are pointing to, the following methods should be used:

|Method|Description|
|-----------------|:-------------|
|`extract()`| Returns the entire contents of the element(s) selected by the `selector` object, as a list of strings.|
|`extract_first()`| Returns the content of the first element selected by the `selector` object.|
|`re()`| Returns a list of unicode strings within the element(s) selected by the `selector` object by applying the regular expression given as argument.|
|`re_first()`| Returns the first match of the regular expression|

Those objects are pointers to the different elements in the scraped page (`h4` text) as defined by our XPath query. To get to the actual content of those elements (the text of the links), we can use the `extract()` method. A variant of that method is `extract_first()` which does the same thing as `extract()` but only returns the first element if there are more than one:

For example we can see: 

~~~
>>> response.xpath("//h4[@class='title']/a/text()").extract_first()
~~~
{: .language-python}

~~~
>>> response.xpath("//h4[@class='title']/a/text()").extract_first()
'Hon Tony Abbott MP'
>>> response.xpath("//h4[@class='title']/a/text()").extract()
['Hon Tony Abbott MP', 'Hon Anthony Albanese MP', 'Mr John Alexander OAM, MP', 'Dr Anne Aly MP', 'Hon Karen Andrews MP', 'Hon Kevin Andrews MP', 'Mr Adam Bandt MP', 'Ms Julia Banks MP', 'Hon Sharon Bird MP', 'Hon Julie Bishop MP', 'Hon Chris Bowen MP', 'Mr Andrew Broad MP']
~~~
{: .language-python .output}


Since we have an XPath query we know will extract the names we are looking for, we can now use the `xpath()` method and update the spider accordingly:

(editing `austmps/austmps/spiders/austmpdata.py`)

~~~
import scrapy

class AustmpdataSpider(scrapy.Spider):
    name = 'austmpdata'
    allowed_domains = ['www.aph.gov.au']
    start_urls = ['http://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0/']

    def parse(self, response):
        print(response.xpath("//h4[@class='title']/a/text()").extract())
        

~~~
{: .language-python}

We now need to re-run the spider.

~~~
$ scrapy crawl austmpdata -s DEPTH_LIMIT=1
~~~
{: .source}

We see, in the middle:

~~~
2018-06-26 17:09:13 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0/> (referer: None)
['Hon Tony Abbott MP', 'Hon Anthony Albanese MP', 'Mr John Alexander OAM, MP', 'Dr Anne Aly MP', 'Hon Karen Andrews MP', 'Hon Kevin Andrews MP', 'Mr Adam Bandt MP', 'Ms Julia Banks MP', 'Hon Sharon Bird MP', 'Hon Julie Bishop MP', 'Hon Chris Bowen MP', 'Mr Andrew Broad MP']
~~~
{: .output}

But, we want to have multiple items per result and we need to output this to a file!

Let's first see if we can iterate over these results instead of having them in a big line. 

Instead of calling extract on the response, let us call each item returned a "resource" and run extract on that resource.

(editing `austmps/austmps/spiders/austmpdata.py`)


~~~
import scrapy

class AustmpdataSpider(scrapy.Spider):
    name = 'austmpdata'
    allowed_domains = ['www.aph.gov.au']
    start_urls = ['http://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0/']

    def parse(self, response):
        for resource in response.xpath("//h4[@class='title']/a/text()"):
            print(resource.extract())
~~~
{: .language-python}

We see:

~~~
2018-06-26 17:11:39 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0/> (referer: None)
Hon Tony Abbott MP
Hon Anthony Albanese MP
Mr John Alexander OAM, MP
Dr Anne Aly MP
Hon Karen Andrews MP
Hon Kevin Andrews MP
Mr Adam Bandt MP
Ms Julia Banks MP
Hon Sharon Bird MP
Hon Julie Bishop MP
Hon Chris Bowen MP
Mr Andrew Broad MP
~~~
{: .output}

Which is great. 


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


## Getting more data

But! We hear someone cry, there's only one column! Now, let's fix that. First, we need to find the element holding the data we want. 

Let's launch scrapy shell again.

~~~
$ scrapy shell "https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0"
~~~
{: .source}

We know that `response.xpath("//h4[@class='title']/a/text()")` is the link inside the `h4`. What if we go the other direction? Just like changing directories `..` moves to the parent, we can use the same thing here.

Let's try `response.xpath("//h4[@class='title']/..")`

~~~
[<Selector xpath="//h4[@class='title']/.." data='<div class="medium-8 columns">\r\n        '>
~~~
{: .language-html .output}

To the page inspector! 

![The page inspector showing the div with class "medium-8"]({{page.root}}/fig/medium-8.png)

And now all we need is to get our data.

(editing `austmps/austmps/spiders/austmpdata.py`)

~~~
import scrapy

class AustmpdataSpider(scrapy.Spider):
    name = 'austmpdata'
    allowed_domains = ['www.aph.gov.au']
    start_urls = ['http://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0/']

    def parse(self, response):
        for resource in response.xpath("//h4[@class='title']/.."):
            name = resource.xpath("h4/a/text()").extract_first()
            print(name)
~~~
{: .language-python}

Running it produces:

~~~
2018-06-26 18:06:40 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0/> (referer: None)
Hon Tony Abbott MP
Hon Anthony Albanese MP
Mr John Alexander OAM, MP
Dr Anne Aly MP
Hon Karen Andrews MP
Hon Kevin Andrews MP
Mr Adam Bandt MP
Ms Julia Banks MP
Hon Sharon Bird MP
Hon Julie Bishop MP
Hon Chris Bowen MP
Mr Andrew Broad MP
~~~
{: .output}


> ## Exercise: Get the two other columns we want.
>
> Now that we are extracting one column. Reference the XPaths from your prior code to print out the other columns.
> 
> > ## Solution
> > 
> > District was: `/dl/dd`
> > 
> > Link was: `h4/a/@href`
> > 
> > Therefore: 
> > 
> > ~~~
> > import scrapy
> > 
> > class AustmpdataSpider(scrapy.Spider):
> >     name = 'austmpdata'
> >     allowed_domains = ['www.aph.gov.au']
    > > start_urls = ['http://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0/']
> > 
> >     def parse(self, response):
> >         for resource in response.xpath("//h4[@class='title']/.."):
> >             name = resource.xpath("h4/a/text()").extract_first()
> >             link = resource.xpath("h4/a/@href").extract_first()
> >             district = resource.xpath("dl/dd/text()").extract_first()
> >             print(name, district, link)
> > ~~~
> > {: .source .language-python}
> {: .solution}
{: .challenge}



> ## Discussion
> 
> What are the similarities and differences between these three approaches? When would we want the infrastructure of a scrapy solution to a more casual browser based scrape?
{: .discussion}

## Writing to a csv

Now that we have 

~~~
2018-06-26 18:10:49 [scrapy.core.engine] DEBUG: Crawled (200) <GET https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0/> (referer: None)
Hon Tony Abbott MP Warringah, New South Wales /Senators_and_Members/Parliamentarian?MPID=EZ5
Hon Anthony Albanese MP Grayndler, New South Wales /Senators_and_Members/Parliamentarian?MPID=R36
Mr John Alexander OAM, MP Bennelong, New South Wales /Senators_and_Members/Parliamentarian?MPID=M3M
Dr Anne Aly MP Cowan, Western Australia /Senators_and_Members/Parliamentarian?MPID=13050
Hon Karen Andrews MP McPherson, Queensland /Senators_and_Members/Parliamentarian?MPID=230886
Hon Kevin Andrews MP Menzies, Victoria /Senators_and_Members/Parliamentarian?MPID=HK5
Mr Adam Bandt MP Melbourne, Victoria /Senators_and_Members/Parliamentarian?MPID=M3C
Ms Julia Banks MP Chisholm, Victoria /Senators_and_Members/Parliamentarian?MPID=18661
Hon Sharon Bird MP Cunningham, New South Wales /Senators_and_Members/Parliamentarian?MPID=DZP
Hon Julie Bishop MP Curtin, Western Australia /Senators_and_Members/Parliamentarian?MPID=83P
Hon Chris Bowen MP McMahon, New South Wales /Senators_and_Members/Parliamentarian?MPID=DZS
Mr Andrew Broad MP Mallee, Victoria /Senators_and_Members/Parliamentarian?MPID=30379
~~~
{: .output}

## Using `Item`s to store scraped data

Scrapy conveniently includes a mechanism to collect scraped data and output it
in several different useful ways. It uses objects called `Item`s. Those are akin
to Python dictionaries in that each `Item` can contain one or more fields to
store individual data element. Another way to put it is, if you visualize the
data as a spreadsheet, each `Item` represents a row of data, and the fields within
each item are columns.

Before we can begin using `Item`s, we need to define their structure. Using our editor,
let's navigate and edit the following file that Scrapy has created for us when we
first created our project: `austmps/austmps/items.py`

Scrapy has pre-populated this file with an empty "austmpsItem" class:


~~~
import scrapy

class AustmpsItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    pass

~~~
{: .language-python}


Let's add a few fields to store the data we aim to extract from the detail pages
for each politician:

(editing `austmps/austmps/items.py`)

~~~
import scrapy

class AustmpsItem(scrapy.Item):
    # define the fields for your item here like:
    name = scrapy.Field()
    district = scrapy.Field()
    link = scrapy.Field()
~~~
{: .language-python}

Then save this file. We can then edit our spider one more time:

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


        for resource in response.xpath("//h4[@class='title']/.."):
            # Loop over each item on the page. 
            item = AustmpsItem() # Creating a new Item object

            item['name'] = resource.xpath("h4/a/text()").extract_first()
            item['link'] = resource.xpath("h4/a/@href").extract_first()
            item['district'] = resource.xpath("dl/dd/text()").extract_first()

            yield item
~~~
{: .language-python}

We made two significant changes to the file above:
* We've included the line `from austmps.items import AustmpsItem` at the top. This is required
  so that our spider knows about the `austmpsItem` object we've just defined.
* We've also replaced the `print` statement with the creation of an `austmpsItem`
  object, in which fields we are now storing the scraped data. The item is then passed back to the
  main spider method using the `yield` statement.

If we now run our spider again:

~~~
$ scrapy crawl austmpdata -s DEPTH_LIMIT=1
~~~
{: .language-bash}

We see:

~~~
(...)
2018-06-26 18:56:36 [scrapy.core.scraper] DEBUG: Scraped from <200 https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0/>
{'district': 'Warringah, New South Wales',
 'link': '/Senators_and_Members/Parliamentarian?MPID=EZ5',
 'name': 'Hon Tony Abbott MP'}
(...) 
~~~
{: .output}

We see that Scrapy is dumping the contents of the items within the debugging output using
a syntax that looks a lot like JSON.

But let's now try running the spider with an extra `-o` ('o' for 'output') argument that
specifies the name of an output file with a `.csv` file extension:

~~~
$ scrapy crawl austmpdata -o output.csv
~~~
{: .language-bash}

> ## No depth limit?
> 
> Since we're now happy with our scraper and want its output, we'll remove the ` -s DEPTH_LIMIT=1` debug flag to let it run completely. 
> 
{: .callout}

This produces similar debugging output as the previous run, but now let's look inside the
directory in which we just ran Scrapy and we'll see that it has created a file called
`output.csv`, and when we try looking inside that file, we see that it contains the
scraped data, conveniently arranged using the Comma-Separated Values (CSV) format, ready
to be imported into our favourite spreadsheet!

~~~
$ head -3 output.csv
~~~
{: .language-bash}

We see:
~~~
district,link,name
"Warringah, New South Wales",/Senators_and_Members/Parliamentarian?MPID=EZ5,Hon Tony Abbott MP
"Grayndler, New South Wales",/Senators_and_Members/Parliamentarian?MPID=R36,Hon Anthony Albanese MP
~~~
{: .output}


> ## Exercise: Add the rest of the columns from the previous manual browser scraping to the csv.
> 
> On [browser scraping](/03-XPath-browser-scraping), exercise 2, we had other columns: their party affiliation, and their twitter handle. Let's add them to this exporter.
> 
> The XPaths that we found last time were:
> 
> |---|---|
> | `*/dl/dt[text()='Party']/following-sibling::dd` | Party |
> | `*/dl/dd/a[contains(@class, 'twitter')]/@href` | Twitter |
> 
> Adapt these XPaths and edit the spider to add these columns to the csv.
> 
> Tip: make sure to remove the output file before re-running the spider. will just append to the end of the file if you don't.
> 
> > ## Solution
> > 
> > Editing `items.py`
> > ~~~
> > import scrapy
> >
> >
> >class AustmpsItem(scrapy.Item):
> >    # define the fields for your item here like:
> >    name = scrapy.Field()
> >    district = scrapy.Field()
> >    link = scrapy.Field()
> >    twitter = scrapy.Field()
> >    party = scrapy.Field()
> > ~~~
> > {: .language-python} 
> >Editing `austmpdata.py`:
> >~~~
> >import scrapy
> >from austmps.items import AustmpsItem # We need this so that Python knows about the item object
> >
> >class AustmpdataSpider(scrapy.Spider):
> >    name = 'austmpdata'  # The name of this spider
> >
> >    # The allowed domain and the URLs where the spider should start crawling:
> >    allowed_domains = ['www.aph.gov.au']
> >    start_urls = ['http://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0/']
> >    
> >    
> >    def parse(self, response):
> >        # The main method of the spider. It scrapes the URL(s) specified in the
> >        # 'start_url' argument above. The content of the scraped URL is passed on
> >        # as the 'response' object.
> >
> >        for resource in response.xpath("//h4[@class='title']/.."):
> >            # Loop over each item on the page. 
> >            item = AustmpsItem() # Creating a new Item object
> >
> >            item['name'] = resource.xpath("h4/a/text()").extract_first()
> >            item['link'] = resource.xpath("h4/a/@href").extract_first()
> >            item['district'] = resource.xpath("dl/dd/text()").extract_first()
> >            item['twitter'] = resource.xpath("dl/dd/a[contains(@class, 'twitter')]/@href").extract_first()
> >            item['party'] = resource.xpath("dl/dt[text()='Party']/following-sibling::dd/text()").extract_first()
> >
> >            yield item
> >~~~
> >{: .language-python}            
> {: .solution}
> 
> And once we run the spider and head on the csv, we see:
> 
>~~~
>district,link,name,party,twitter
>"Warringah, New South Wales",/Senators_and_Members/Parliamentarian?MPID=EZ5,Hon Tony Abbott MP,Liberal Party of Australia,http://twitter.com/TonyAbbottMHR
>"Grayndler, New South Wales",/Senators_and_Members/Parliamentarian?MPID=R36,Hon Anthony Albanese MP,Australian Labor Party,http://twitter.com/AlboMP
>
>~~~
>{: .output}
{: .challenge}


# Reference

* [Scrapy documentation](https://doc.scrapy.org/en/latest/index.html)


