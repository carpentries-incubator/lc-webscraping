---
title: "Conclusion"
teaching: 15
exercises: 15
questions:
- "When is web scraping OK and when is it not?"
- "Is web scraping legal? Can I get into trouble?"
- "How can I make sure I'm doing the right thing?"
- "What can I do with the data that I've scraped?"
objectives:
- "Wrap things up"
- "Discuss the legal implications of web scraping"
- "Establish a code of conduct"
keypoints:
- "Web scraping is, in general, legal and won't get you into trouble."
- "There are a few things to be careful about, notably don't overwhelm a web server and don't steal content."
- "Be nice. In doubt, ask."
---

Now that we have seen several different ways to scrape data from websites and are
ready to start working on potentially larger projects, we may ask ourselves whether
there are any legal implications of writing a piece of computer code that downloads
information from the Internet.

In this section, we will be discussing some of the issues to be aware of when
scraping websites, and we will establish a [code of conduct (below)](#web-scraping-code-of-conduct)
to guide our web scraping projects.

> ## This section does not constitute legal advice
> 
> Please note that the information provided on this page is for information
> purposes only and does not constitute professional legal advice on the
> practice of web scraping.
>
> If you are concerned about the legal implications of using web scraping
> on a project you are working on, it is probably a good idea to seek
> advice from a professional, preferably someone who has knowledge of the
> intellectual property (copyright) legislation in effect in your country.
>
{: .callout}

## Don't break the web: Denial of Service attacks

The first and most important thing to be careful about when writing a web scraper is that
it typically involves querying a website repeatedly and accessing a potentially large
number of pages. For each of these pages, a request will be sent to the web server that
is hosting the site, and the server will have to process the request and send a response
back to the computer that is running our code. Each of these requests will consume resources
on the server, during which it will not be doing something else, like for example responding
to someone else trying to access the same site. 

If we send too many such requests over a short span of time, we can prevent other "normal" users
from accessing the site during that time, or even cause the server to run out of resources and crash.

In fact, this is such an efficient way to disrupt a web site that hackers are often doing it on purpose.
This is called a [Denial of Service (DoS) attack](https://en.wikipedia.org/wiki/Denial-of-service_attack).

Since DoS attacks are unfortunately a common occurence on the Internet, modern web servers include
measures to ward off such illegitimate use of their resources. They are watchful for large amounts
of requests appearing to come from a single computer or IP address, and their first line of defense often involves
refusing any further requests coming from this IP address.

A web scraper, even one with legitimate purposes and no intent to bring a website down, can exhibit
similar behaviour and, if we are not careful, result in our computer being banned from accessing
a website.

The good news is that a good web scraper, such as Scrapy, recognizes that this is a risk and includes
measures to prevent our code from appearing to launch a DoS attack on a website. This is mostly
done by inserting a random delay between individual requests, which gives the target server enough
time to handle requests from other users between ours. 

This is Scrapy's default behaviour, and it should prevent most scraping projects from ever causing problems.
To be on the safe side, however, it is good practice to limit the number of pages we are scraping
while we are still writing and debugging our code. This is why in the previous section, we imposed
a limit of five pages to be scraped, which we only removed when we were reasonably certain the scraper
was working as it should.

Limiting requests to a particular domain, by using Scrapy's `allowed_domains` property is another
way to make sure our code is not going to start scraping the entire Internet by mistake.

Thanks to the defenses web servers use to protect themselves against DoS attacks and Scrapy's
measure to avoid inadvertently launching such an attack, the risks of causing trouble is limited.

## Don't steal: Copyright and fair use

It is important to recognize that in certain circumstances web scraping _can_ be illegal. If the terms
and conditions of the web site we are scraping specifically prohibit downloading
and copying its content, then we could be in trouble for scraping it.

In practice, however, web scraping is a tolerated practice, provided reasonable
care is taken not to disrupt the "regular" use of a web site, as we have seen above.

In a sense, web scraping is no different than using a web browser to visit a web page,
in that it amounts to using computer software (a browser vs a scraper) to acccess
data that is publicly available on the web.

In general, if data is publicly available (the content that is being scraped is not
behind a password-protected authentication system), then it is OK to scrape it,
provided we don't break the web site doing so. What is potentially
problematic is if the scraped data will be shared further. For example, downloading
content off one website and posting it on another website (as our own), unless
explicitely permitted, would constitute copyright violation and be illegal.

However, most copyright legislations recognize cases in which reusing some, possibly
copyrighted, information in an aggregate or derivative format is considered
"fair use". In general, unless the intent is to pass off data as our own, copy
it word for word or trying to make money out of it, reusing publicly available
content scraped off the internet is OK.

### Better be safe than sorry
Be aware that copyright and data privacy legislation typically differs from country
to country. Be sure to check the laws that apply in your context. For example, in Australia,
it can be illegal to scrape and store personal information such as names, phone
numbers and email addresses, even if they are publicly available.

If you are looking to scrape data for your own personal use, then the above
guidelines should probably be all that you need to worry about. However,
if you plan to start harvesting a large amount of data for research
or commercial purposes, you should probably seek legal advice first.

If you work in a university, chances are it has a copyright office that
will help you sort out the legal aspects of your project. The
university library is often the best place to start looking for help on
copyright.

## Be nice: ask and share

Depending on the scope of your project, it might be worthwhile to consider asking
the owners or curators of the data you are planning to scrape if they have it
already available in a structured format that could suit your project. If your
aim is do use their data for research, or to use it in a way that could potentially
interest them, not only it could save you the trouble of writing a
web scraper, but it could also help clarify straight away what you can and cannot do
with the data.

On the other hand, when you are publishing your own data, as part of a research project,
documentation or a public website, you might want to think about whether someone might
be interested in getting your data for their own project. If you can, try to provide
others with a way to download your raw data in a structured format, and thus save
them the trouble to try and scrape your own pages!

## Web scraping code of conduct

This all being said, if you adhere to the following simple rules, you will probably
be fine.

1. __Ask nicely.__ If your project requires data from a particular organisation, for example,
   you can try asking them directly if they could provide you what you are looking for.
   With some luck, they will have the primary data that they used on their website in a
   structured format, saving you the trouble.
2. __Don't download copies of documents that are clearly not public.__ For example, academic
   journal publishers often have very strict rules about what you can and what you cannot
   do with their databases. Mass downloading article PDFs is probably prohibited and can
   put you (or at the very least your friendly university librarian) in trouble. If your
   project requires local copies of documents (e.g. for text mining projects), special
   agreements can be reached with the publisher. The library is a good place to start
   investigating something like that.
3. __Check your local legislation.__ For example, certain countries have laws protecting
   personal information such as email addresses and phone numbers. Scraping such information,
   even from publicly avaialable web sites, can be illegal (e.g. in Australia).
4. __Don't share downloaded content illegally.__ Scraping for personal purposes is usually
   OK, even if it is copyrighted information, as it could fall under the fair use provision
   of the intellectual property legislation. However, sharing data for which you don't
   hold the right to share is illegal.
5. __Share what you can.__ If the data you scraped is in the public domain or you got
   permission to share it, then put it out there for other people to reuse it (e.g. on 
   [datahub.io](https://datahub.io)). If you
   wrote a web scraper to access it, share its code (e.g. on GitHub) so that others can
   benefit from it.
6. __Don't break the Internet.__ Not all web sites are designed to withstand thousands of
   requests per second. If you are writing a recursive scraper (i.e. that follows
   hyperlinks), test it on a smaller dataset first to make sure it does what it is
   supposed to do. Adjust the settings of your scraper to allow for a delay between
   requests. By default, Scrapy uses conservative settings that should minimize this risk.
7. __Publish your own data in a reusable way.__ Don't force others to write their own
   scrapers to get at your data. Use open and software-agnostic formats (e.g. JSON, XML),
   provide metadata (data about your data: where it came from, what it represents, how
   to use it, etc.) and make sure it can be indexed by search engines so that people can
   find it.

## Going further

This lesson only provides an introduction to the practice of web scraping and highlights
some of the tools available. Scrapy has many more features than those mentioned in the
previous section, be sure to refer to its [full documentation](https://doc.scrapy.org/en/latest/)
for details.

Happy scraping!


# References

* The [Web scraping Wikipedia page](https://en.wikipedia.org/wiki/Web_scraping) has a concise
  definition of many concepts discussed here.
* The [School of Data Handbook](http://schoolofdata.org/handbook/courses/scraping/) has a
  short introduction to web scraping, with links to resources e.g. for data journalists.
* [This blog](https://blog.rubyroidlabs.com/2016/04/web-scraping-1/) has a discussion on
  the legal aspects of web scraping.
* [This case study](http://naelshiab.com/members-parliament-web-scraping/) is a great example
  of what can be done using web scraping and how to achieve it.
* [Scrapy documentation](https://doc.scrapy.org/en/latest/)
* [morph.io](https://morph.io/) is a cloud-based web scraping platform that supports multiple
  frameworks, interacts with GitHub and provides a built-in way to save and share extracted
  data.
* [import.io](https://www.import.io/) is a commercial web-based scraping service that requires
  little coding.
* [Software Carpentry](https://software-carpentry.org/) is a non-profit organisation that
  runs learn-to-code workshops worldwide. All lessons are publicly available and can be
  followed indepentently. This lesson is heavily inspired by Software Carpentry.
* [Data Carpentry](http://www.datacarpentry.org/) is a sister organisation of Software Carpentry
  focused on the fundamental data management skills required to conduct research.
* [Library Carpentry](https://librarycarpentry.github.io/) is another Software Carpentry spinoff
  focused on software skills for librarians.
