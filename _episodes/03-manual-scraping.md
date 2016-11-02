---
title: "IN DEVELOPMENT: Manually scrape data using browser extensions"
teaching: 45
exercises: 20
questions:
- "Key question"
objectives:
- "First objective."
keypoints:
- "First key point."
---

Say we want a spreadsheet with all British parliament members. How to extract this data?


There are different options to scrape data:

* Manually (copy paste). Sometimes the best/fastest way. Humans are good at structuring information. 
  We know what an address looks like, etc.
	Good for relatively small amount of data, sometimes only solution if no structural clues.

* Semi-manually, using tools such as browser extensions to indicate what data needs to be scraped.
	Good when data is somehow structured, e.g. table rows, titles in bold, etc.
	
	Examples: Chrome Scraper extension
		(try ffscrap in Firefox)
	
	Going back to British parliament example:
	- Select first row of table
	- Right click -> Scrape similar
	
	Uses XPath selectors -> cf XPath lesson
	
	Export to Google Docs or clipboard/TSV.
		Can be cleaned up further using OpenRefine (e.g. separate first, last name, party)
	
Sometimes not as clear cut. Addresses of Canadian MPs:
	http://www.parl.gc.ca/Parliamentarians/en/members/addresses
	
	- Select 1st MP, Scrape similar
	- Not what we want

Useful to inspect web page to find structure.

	- Chrome or Firefox -> inspect element
	- MPs are inside ULs
	
	Try changing Xpath with //body/div[1]/div/ul
	Click "Scrape"
	
	Better, but still unstructured. We see that name, addresses separated by LI elements.
	
	In Columns, replace . by ./li and rename column.
	
	Add new columns:
		./li[2] -> Hill Office
		./li[3] -> Constituency
		
	Etc. Then export and clean up using OpenRefine.
	
> ## Export the members of parliament for the Province of Ontario
>
> * Install Chrome Scraper extension
> * Choose a province (e.g. Ontario), find list of MPPs
> * Scrape them
> * Open in OpenRefine, separate first name, last name, political party, constituency
>	TIP: use copy-paste to import into OR
>
{: .challenge}