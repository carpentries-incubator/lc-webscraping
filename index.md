---
layout: lesson
root: .
---

Web scraping is the process of extracting data from websites. Some data that is available on the web is
presented in a format that makes it easier to collect and use it, for example in the form of downloadable
comma-separated values (CSV) datasets that can then be imported in a spreadsheet or loaded into a data analysis
script. Often however, even though it is publicly available, data is not readily available for reuse. 
For example it can be contained in a PDF, or a table on a website, or spread across multiple web pages.

There are a variety of ways to _scrape_ a website to extract information for reuse.
In its simplest form, this can be achieved by
copying and pasting snippets from a web page, but this can be unpractical if there is a large amount of data to
be extracted, or if it spread over a large number of pages. Instead, specialized tools and techniques can be used
to automate this process, by defining what sites to visit, what information to look for, and whether data extraction
should stop once the end of a page has been reached, or whether to follow hyperlinks and repeat the process recursively.
Automating web scraping also allows to define whether the process should be run at regular intervals and capture changes
in the data.


> ## Prerequisites
>
> As webscraping is a technique to extract data from web pages, it requires some understanding of
> the technologies that are used to display information on the web. 
> This lesson therefore assumes that learners will have some familiarity with [HTML](https://en.wikipedia.org/wiki/HTML)
> and the [Document Object Model](https://en.wikipedia.org/wiki/Document_Object_Model) (DOM).
>  
> The first part of this lesson will use browser extensions to introduce the concepts of web scraping
> as well as introduce the XPath syntax for selecting elements on a web page
> and requires no further specific knowledge.
> The second part will introduce the use of specialized libraries to scrape websites by writing
> custom computer programs and will require some familiarity with the 
> [Python programming language](https://swcarpentry.github.io/python-novice-inflammation/)
> and [object-oriented programming](https://en.wikipedia.org/wiki/Object-oriented_programming).
>
{: .prereq}

## Software requirements

Refer to the [Setup](setup/) section to install the required software to follow along this lesson.

> ## Under development
>
> Please note that the contents of this lesson are still being actively developed. Any feedback is
> appreciated, please do not hesitate to contact [the author](mailto:tom@timtom.ch) or contribute
> to the lesson by [forking it on GitHub](https://github.com/timtomch/library-webscraping/).
>
{: .callout}