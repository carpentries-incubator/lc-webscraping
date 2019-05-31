---
layout: reference
root: .
---

## Resources

* [W3Schools: JavaScript HTML DOM Navigation](http://www.w3schools.com/js/js_htmldom_navigation.asp)
* [Scraper extension tutorial](https://schoolofdata.org/handbook/recipes/scraper-extension-for-chrome/)
* [XPath Cheatsheet](/library-webscraping/extras/xpath-cheatsheet.md.pdf)
* [Scrapy documentation](https://doc.scrapy.org/en/latest/index.html)
* [Scrapy tutorial](https://doc.scrapy.org/en/latest/intro/tutorial.html)
* [How to crawl the web politely](https://blog.scrapinghub.com/2016/08/25/how-to-crawl-the-web-politely-with-scrapy/)
* [Additional resources on scraping by the School of Data](http://schoolofdata.org/handbook/courses/scraping/)

## Glossary

**CSS selectors**  
CSS selectors serve a similar function to XPath, in selecting parts of an HTML document, but were designed for web development (for applying styles such as colour to parts of a document). As such, they are more popular, but are limited in what they can express relative to XPath. Every CSS selector can be translated into an equivalent XPath expression, but not vice-versa. CSS selectors are constructed by specifying properties of the targets combined with properties of their context. CSS selectors can be evaluated using the `document.querySelectorAll()` function.

**Denial of service attack**  
If someone sends too many requests over a short span of time, they can prevent other “normal” users from accessing the site during that time, or even cause the server to run out of resources and crash. In fact, this is such an efficient way to disrupt a web site that hackers are often doing it on purpose. Modern web servers include measures to ward off such illegitimate use of their resources and their first line of defense often involves refusing any further requests coming from this IP address. Since, scraping tools’ usually have developed measures to avoid inadvertently launching such an attack, the risks of causing trouble is limited.

**Visual scrapers**  
Visual scrapers are tools in which the user can visually select the elements to extract, and the logical order to follow in performing a sequence of extractions. They require little or no code, and assist in designing XPath or CSS selectors. These tools vary in how flexible they are, how easy to use, to what extent they help you identify and debug scraping problems, how easy it is to keep and transfer your scraper to another service, and how costly the service is. 

**Web scraping**  
Web scraping is a technique for targeted, automated extraction of information from websites. Similar extraction can be done manually but it is usually faster, more efficient and less error-prone to automate the task. Web scraping allows you to acquire non-tabular or poorly structured data from websites and convert it into a usable, structured format, such as a .csv file or spreadsheet.

**Web scraping code of conduct**
+ Ask nicely. 
+ Don’t download copies of documents that are clearly not public. 
+ Check your local legislation. 
+ Don’t share downloaded content illegally. 
+ Share what you can. 
+ Don’t break the Internet. 
+ Publish your own data in a reusable way. 

**XPath**    
These specify parts of a tree-structured document, be it XML or HTML. They can be very specific about which nodes to include or exclude.
