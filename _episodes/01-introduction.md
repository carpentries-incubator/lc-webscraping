---
title: "Introduction: What is web scraping?"
teaching: 10
exercises: 0
questions:
- "Key question"
objectives:
- "First objective."
keypoints:
- "First key point."
---

From Wikipedia:
	"Web scraping (web harvesting or web data extraction) is a computer software technique of extracting information from websites."

Closely related to web indexing (used by search engines like Google). They typically use tools called "bots" or "crawlers" to go through websites.
Difference:
	Indexing: going through ALL WEBSITES (unless blocked), store all content into database, follow ALL LINKS, index all stored data, repeat
	Scraping: go through SPECIFIC WEBSITES, follow SPECIFIED LINKS, extract unstructured information and put it in STRUCTURED FORM

Example:
http://www.parl.gc.ca/Parliamentarians/en/members

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



	Difference between UNSTRUCTURED and STRUCTURED data
	
	For humans, unstructured. We recognize names, provinces, political party, etc.
	But a machine doesn't.
	
	Computers need LABELS.
	This is a good website, because data is actually structured. 
		View Source. DIVs
		Export as XML/CSV. -> STRUCTURED DATA

Compare with
https://www.parliament.uk/mps-lords-and-offices/mps/

	As humans, we recognize names, parties, etc.
	
	But this data is not as nicely structured.
		View Source. Table





