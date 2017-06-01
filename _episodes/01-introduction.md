---
title: "Introduction: What is web scraping?"
teaching: 10
exercises: 0
questions:
- "What is web scraping and why is it useful?"
- "What are typical use cases for web scraping?"
objectives:
- "Introduce the concept of structured data"
- "Discuss how data can be extracted from web pages"
- "Introduce the examples that will be used in this lesson"
keypoints:
- "Humans are good at categorizing information, computers not so much."
- "Often, data on a web site is not properly structured, making its extraction difficult."
- "Web scraping is the process of automating the extraction of data from web sites."
---

## What is web scraping?

Web scraping is a technique for extracting information from websites. This can be done manually
but it is usually faster, more efficient and less error-prone if it can be automated. 

Web scraping allows you to convert non-tabular or poorly structured data into a usable, structured format, 
such as a .csv file or spreadsheet.

Scraping can help you acquire data made inaccessible by the way it has been presented online.
But scraping is about more than just acquiring data: it can help you track changes to data online, and
help you archive data. In short, it's a skill worth learning.

It is closely related to the practice of
web _indexing_, which is what search engines like Google do when mass analysing the Web to build
their index. But contrary to _web indexing_, which typically parses the entire content of a web
page to make it searchable, _web scraping_ targets specific information on the pages visited.

For example, online stores will often scour the publicly available pages of their competitors,
scrape item prices, and then use this information to adjust their own prices. Another common
practice is "contact scraping" in which contact information (such as email
addresses or phone numbers) is collected for marketing purposes.

Web scraping is also increasingly being used by scholars to create data sets for
text mining projects, say, collections journal articles or digitised texts. The practice of
[data journalism](https://en.wikipedia.org/wiki/Data_journalism), in particular, relies on the
ability for investigative journalists to harvest data that is not always published in a form
that allows analysis.

## Before you get started

As useful as scraping is, there might be better options. Choose the right (i.e. the easiest) tool for the job.

- Check whether or not you can copy and paste data from a site into Excel or Google Sheets.
- Look for existing structured data (it may exist).
- use Freedom of information requests (be specific about the format you want data in).

## Example: scraping government websites for contact addresses

In this lesson, the examples that we will use all involve extracting contact information
from government websites listing the members of various constituencies. A practical example
of why this information would be useful could be an advocacy group wishing to making it easier
for citizens to contact their representatives about a particular issue. 

Let's start by looking at the current list of members of the Canadian parliament, which is available
on the [Parliament of Canada website](http://www.parl.gc.ca/Parliamentarians/en/members).

This is how this page appears in November 2016:

![Screenshot of the Parliament of Canada website]({{ page.root }}/fig/canparl.png)

There are several features (circled in the image above) that make the data on this page easier to work with.
The search, reorder, refine features and display modes hint that the data is actually stored in a (structured)
database before being displayed on this page. The data can be readily downloaded as a Comma-Separated Values (CSV)
file or XML, which allows anyone to load this data in their own database, spreadsheet or computer program to
reuse it.

Even though the information displayed in the view above isn't labeled, a person visiting this site with some
knowledge of Canadian geography and politics is quickly able to figure out what information pertains to the 
name of the politicians, the geographical area or the political party they represent. This is because human
beings are usually good at using context and prior knowledge to quickly categorize information.

Computers, on the other hand, can't do this unless we provide them with more information.
Fortunately, if we look at the source HTML code of this page, we see that the information displayed is actually
organized inside labeled elements:

~~~
(...)
<div>
    <a href="/Parliamentarians/en/members/Ziad-Aboultaif(89156)"> 
        <img alt="Photo - Ziad Aboultaif - Click to open the Member of Parliament profile" title="Photo - Ziad Aboultaif - Click to open the Member of Parliament profile" src="http://www.parl.gc.ca/Parliamentarians/Images/OfficialMPPhotos/42/AboultaifZiad_CPC.jpg" class="picture" />
        <div class="full-name">
		    <span class="honorific"><abbr></abbr></span>
            <span class="first-name">Ziad</span>
            <span class="last-name">Aboultaif</span>
        </div>
    </a>
    <div class="caucus-banner" style="background-color:#002395"></div>
    <div class="caucus">Conservative</div>
    <div class="constituency">Edmonton Manning</div>
    <div class="province">Alberta</div>        
</div>
(...)
~~~
{: .output}

Thanks to these labels, we could relatively easily instruct a computer to look for all parliamentarians from
Alberta and list their names and caucus information.

> ## Structured vs unstructured data
>
> When presented with information, human beings are good at quickly categorizing it and extracting the data
> that they are interested in. For example, when we look at a magazine rack, provided the titles are written
> in a script that we are able to read, we can rapidly figure out the titles of the magazines, the stories they
> contain, the language they are written in, etc. and we can probably also easily organize them by topic, 
> recognize those that are aimed at children, or even whether they lean toward a particular end of the
> political spectrum. Computers have a much harder time making sense of such _unstructured_ data unless
> we specifically tell them what elements data is made of, for example by adding labels such as
> _this is the title of this magazine_ or _this is a magazine about food_. Data in which individual elements
> are separated and labelled is said to be _structured_.
>
{: .callout}

Let's look now at the current list of members for the [UK House of Commons](https://www.parliament.uk/mps-lords-and-offices/mps/). 

![Screenshot of the UK House of Commons website]({{ page.root }}/fig/ukparl.png)

This page also displays a list of names, political and geographical affiliation. There is a search box and
a filter option, but no obvious way to download this information and reuse it.

Here is the code for this page:

~~~
(...)
<table>
    <tbody>
        (...)
        <tr id="ctl00_ctl00_(...)_trItemRow" class="first">
            <td>Aberavon</td>
            <td id="ctl00_ctl00_(...)_tdNameCellRight">
                <a id="ctl00_ctl00_(...)_hypName" href="http://www.parliament.uk/biographies/commons/stephen-kinnock/4359">Kinnock, Stephen</a>(Labour)
            </td>
        </tr>
        (...)
    </tbody>
</table>
(...)
~~~
{: .output}

We see that this data has been structured for displaying purposes (it is arranged in rows inside
a table) but the different elements of information are not clearly labeled.

What if we wanted to download this dataset and, for example, compare it with the Canadian list of MPs
to analyze gender representation, or the representation of political forces in the two groups?
We could try copy-pasting the entire table into a spreadsheet or even manually
copy-pasting the names and parties in another document, but this can quickly become impractical when
faced with a large set of data. What if we wanted to collect this information for every country that
has a parliamentary system?

Fortunately, there are tools to automate at least part of the process. This technique is called
_web scraping_. 

>
> "Web scraping (web harvesting or web data extraction) is a computer software technique of 
> extracting information from websites."
> (Source: [Wikipedia](https://en.wikipedia.org/wiki/Web_scraping))
>

This technique is closely related to _web indexing_ which is what search engines like Google do
to build the database that is queried when users are searching. The difference is that web indexing
(using tools typically called "bots" or "crawlers") aims to visit _all_ web sites recursively,
following all links (unless blocked), index all data and store the result in a database and repeat
every so often to keep their index current.

Web scraping, on the other hand, is a more focused technique, typically targeting one web site at a
time to extract unstructured information and put it in a structured form for reuse.

In this lesson, we will continue exploring the examples above and try different techniques to extract
the information they contain. But before we launch into web scraping proper, we need to look
a bit closer at how information is organized in an HTML document and how to build queries to access
a specific subset of that information.

# References

* [Web Scraping (Wikipedia)](https://en.wikipedia.org/wiki/Web_scraping)
* [The Data Journalism Handbook: Getting Data from the Web](http://datajournalismhandbook.org/1.0/en/getting_data_3.html)

