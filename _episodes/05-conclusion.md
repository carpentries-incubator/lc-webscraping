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

DOES and DONTS - LEGAL ASPECTS

Can be illegal, check terms of use of target websites.
Generally
	Web scraping = tool.
		"using a computer to read a website"
	If data is publicly available, it's OK to scrape it, provided it does not brake the website.
	Sharing data is the problematic bit.
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

Harvesting personal information (e.g. email addresses, cf Australia's Spam Act of 2013) might be illegal. Be careful, especially if sharing extracted data.


BUT even if legal, badly programmed scrapers can also bring down a website (Denial of Service attack).
Test your code on small dataset first.


Good practice:
- if you extract PUBLIC data, share it
	- code on GitHub
	- data on GitHub / datahub.io etc.
	- or use morph.io
	
IF YOU PUBLISH DATA, PROVIDE IT IN STRUCTURED FORM TO BEGIN WITH.
	Web scraping is a "tool of last resort" when data curators fail to provide structured data.


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
