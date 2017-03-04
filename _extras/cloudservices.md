---
title: "COMING SOON: Using cloud services for web scraping"
teaching: 0
exercises: 0
questions:
- "How to deploy web scrapers on cloud services."
- "What other tools are available to automate web scraping?"
objectives:
- "FIXME"
keypoints:
- "FIXME"
---


Going further:
- Pipelines
	Can be used to check validity of returned data
	http://doc.scrapy.org/en/latest/topics/item-pipeline.html



CLOUD-BASED SCRAPING
- ScrapingHub, Scrapy Cloud
- Morph.io


Example deploy on ScrapingHub

- Login/Register on https://app.scrapinghub.com/
	Use Google account


- If not done already, run
	$ pip install shub

- $ shub login
	Provide your API as found on https://app.scrapinghub.com/account/apikey
	This writes the API key into a local file: less ~/.scrapinghub.yml
	Not specific to that repository 
	shub logout to remove it

- Create a new project, note its ID
- cd to the root of a scrapy project directory
	
- $ shub deploy
	Provide the ID of the recently created project
	
- From the web app, run spider
	Arguments can be provided
	
- Items are found under the Items tab
	Free project: one concurrent spider, max 2.5GB of data storage, data retention 7 days
	Items can be downloaded individually or in batch (big green Items button on top)
	Can also be accessed through API calls
		http://doc.scrapinghub.com/api/overview.html
		e.g. curl -u 1e2490bfc15d4e6089e4b842364b5cd1: "https://storage.scrapinghub.com/items/85589/1/1?format=xml"
		

- Jobs can be scheduled, etc.

They have a GUI (work in progress) to define elements and build spiders: Portia


CHECK IF ENOUGH TIME FOR LEGAL ASPECTS



ALTERNATIVE: MORPH.IO
- Port of ScraperWiki Classic
- Open source
- Project supported by the OpenAustralia Foundation
- Can be run locally, but also on cloud platform morph.io "Heroku for scrapers"- very convenient - works with GitHub
	requires a working install of git
	requires ruby or python + several libraries for local running
- For this reason, not chosen for this workshop
- Platform runs the scraper and also stores data
	Data can easily be shared, downloaded in structured format and accessed through dedicated API
- Demo if enough time


CHECK IF ENOUGH TIME FOR LEGAL ASPECTS



SCRAPE DATA FROM PDFs

Use Tabula
	- Free, open source software
	- Available for Mac, Windows, Linux
	- Runs in browser (much like OpenRefine)
	https://github.com/tabulapdf/tabula


This example uses a library that converts PDF to XML, then does the extraction.

http://www.bl.uk/reshelp/atyourdesk/docsupply/help/replycodes/dirlibcodes/
https://morph.io/ostephens/british_library_directory_of_library_codes
	British Library maintains list of library codes for its Document Supply service.
	List of codes in a PDF file -> UNSTRUCTURED
	
	

