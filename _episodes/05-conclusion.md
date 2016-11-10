---
title: "COMING SOON: Conclusion"
teaching: 15
exercises: 15
questions:
- "What are the legal implications of web scraping?"
objectives:
- "Wrap things up"
keypoints:
- "FIXME"
---

Now that we have seen several different ways to scrape data from websites and are
ready to start working on potentially larger projects, we may ask ourselves whether
there are any legal implications of writing a piece of computer code that downloads
information from the Internet.

## Legal aspects

> ## This section does not consitute legal advice
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

FIXME: add references to use cases and legal cases to illustrated the aspects below

The important thing to stress is that web scraping _can_ be illegal. If the terms
and conditions of the web site we are scraping specifically prohibit downloading
and copying its content, then we could be in trouble for scraping it.

In practice, however, web scraping is a tolerated practice, provided reasonable
care is taken not to disrupt the "regular" use of a web site.

In a sense, web scraping is no different than using a web browser to visit a web page,
in that it amounts to using computer software (a browser vs a scraper) to acccess
data that is publicly available on the web.

In general, if data is publicly available (the content that is being scraped is not
behind a password-protected authentication system), then it is OK to scrape it,
provided we don't break the web site doing so (see below). What is potentially
problematic is if the scraped data will be shared further.


Usually OK if public data (e.g. government data)

Usually OK for personal consumption. Good practice to contact data owners/curators if aim to use it for institutional purposes
- You might receive data already structured
- Clarifies right to share extracted data

Unfortunatly, not all data curators will be sensible to this.

Don't download copies of documents that are clearly not public (c.f. SciHub...)
Content theft is illegal
Generally safe if not for commercial intent.

Case law evolving in the US, cases where scraping was declared illegal involved commercial aspects
e.g. competitor to eBay automatically placing bets
e.g. extracting pricing data to set own prices
"trespass to chattel"

## Denial of Service attacks

BUT even if legal, badly programmed scrapers can also bring down a website (Denial of Service attack).
Test your code on small dataset first.

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
   even from publicly avaialable web sites, can be illegal (e.g. Australia).
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
   requests.
7. __Publish your own data in a reusable way.__ Don't force others to write their own
   scrapers to get at your data. Use open and software-agnostic formats (e.g. JSON, XML),
   provide metadata (data about your data: where it came from, what it represents, how
   to use it, etc.) and make sure it can be indexed by search engines so that people can
   find it.



GOING FURTHER
HTTP-authentication is supported by many frameworks, allows for scraping of protected data
	Warning about legal aspects!
There are frameworks that reproduce the workings of a web browser (e.g. PhantomJS)





	



References:
- https://en.wikipedia.org/wiki/Web_scraping
- http://blog.rubyroidlabs.com/2016/04/web-scraping-1/
- http://schoolofdata.org/handbook/courses/scraping/
- http://schoolofdata.org/handbook/recipes/scraper-extension-for-chrome/
- http://doc.scrapy.org/en/1.1/intro/tutorial.html
