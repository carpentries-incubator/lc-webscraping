---
title: "Selecting content on a web page with XPath and XQuery"
teaching: 20
exercises: 10
questions:
- "Key question"
objectives:
- "First objective."
keypoints:
- "First key point."
---
Before we delve into web scraping proper, we will first spend some time introducing
some of the techniques that are required to indicate exactly what should be
extracted from the web pages we aim to scrape.

The material in this section was adapted from the [XPath and XQuery Tutorial](https://github.com/code4libtoronto/2016-07-28-librarycarpentrylessons/blob/master/xpath-xquery/lesson.md)
written by Kim Pham for the July 2016 [Library Carpentry workshop](https://code4libtoronto.github.io/2016-07-28-librarycarpentry/) in Toronto.

# Introduction
XPath (which stands for XML Path Language) is an _expression language_ used to specify parts of an XML document.
XPath is rarely used on its own, rather it is used within software and languages that are aimed at manipulating
XML documents, such as XSLT, XQuery or the web scraping tools that will be introduced later in this lesson.
XPath can also be used in documents with a structure that is similar to XML, like HTML.

## Markup Languages
XML and HTML are _markup languages_. This means that they use a set of tags or rules to organise and provide
information about the data they contain. This structure helps to automate processing, editing, formatting, 
displaying, printing, etc. that information.

XML documents stores data in plain text format. This provides a software- and hardware-independent way of storing,
transporting, and sharing data. XML format is an open format, meant to be software agnostic. You can
open an XML document in any text editor and the data it contains will be shown as it is meant to be represented. 
This allows for exchange between incompatible systems and easier conversion of data.


> ## XML and HTML
>
> Note that HTML and XML have a very similar structure, which is why XPath can be used almost interchangeably to
> navigate both HTML and XML documents. In fact, starting with HTML5, HTML documents are fully-formed XML documents.
> In a sense, HTML is like a particular dialect of XML. 
> 
> For the sake of brevity, we will refer to XML only for the remainder of this section, but unless stated otherwise,
> the information below applies both to XML and HTML documents.
>
{: .callout}

XML document follows basic syntax rules:

* An XML document is structured using _nodes_, which include element nodes, attribute nodes and text nodes
* XML element nodes must have an opening and closing tag, e.g. `<catfood>` opening tag and `</catfood>` closing tag
* XML tags are case sensitive, e.g. `<catfood>` does not equal `<catFood>`
* XML elements must be properly nested:

```
<catfood>
  <manufacturer>Purina</manufacturer>
    <address> 12 Cat Way, Boise, Idaho, 21341</address>
  <date>2019-10-01</date>
</catfood>
```
* Text nodes (data) are contained inside the opening and closing tags
* XML attribute nodes contain values that must be quoted, e.g.
``` <catfood type="basic"></catfood> ```

# XPath Expressions

XPath is written using expressions, and when these expressions are evaluated on XML documents they return an object
containing the node(s) that you aim to select. Contrary to a flat text document, XML data is structured, as it is
organized in nodes and subnodes.
Therefore, when using XPath, we are not querying raw text or data values like we would so using Regular Expressions,
for example. Instead, XPath makes use of the fact that XML documents are structured and instead navigates through the
node structure to select the data that we are looking for.

XPath is typically used to select and compare nodes, not edit them. To manipulate or edit nodes, another language such
as XQuery would be used instead.

> ## XPath assumes _structured_ data
> 
> We can think of using XPath as similar to search a library catalogue using the advanced search function.
> In a catalogue, we can take advantage of the fact that bibliographic information has been properly structured
> in the database by specifying which metadata fields we want to query. For example, if we are looking for books
> about Shakespeare but not those written by him, we can use the advanced search function to look for that name
> in the "title" field only.
>
> Contrary to Regular Expressions, this means that we don't have to know in advance what the data we are looking for
> looks like, we just need to know in which node(s) (or fields) it resides.
>
{: .callout}

Now let's start using XPath.

## Navigating through a webpage with XPath using a browser console

We will use the HTML code that describes this very page you are reading as an example. By default, a web browser
interprets the HTML code to determine what markup to apply to the various elements of a document, and the code is
invisible. To make the underlying code visible, all browsers have a function to display the raw HTML content of
a web page.

> ## Display the source of this page
> Using your favourite browser, display the HTML source code of this page.
>
> Tip: in most browsers, all you have to do is do a right-click anywhere on the page and select the "View Page Source"
> option ("Show Page Source" in Safari).
>
> Another tab should open with the raw HTML that makes this page. See if you can locate its various elements, and
> this challenge box in particular.
>
{: .challenge}


> ## Using the Safari browser
>
> If you are using Safari, you must first turn on the "Develop" menu in order to view the page source, and use the
> functions that we will use later in this section. To do so, navigate to Safari > Preferences and in the Advanced tab
> select the "Show Develop in menu bar" option.
>
{: .callout}

FIXME starting here

## Selecting Nodes

In BaseX, bring up the tree visualization of your document to explore the tree and hover over your nodes.

XPath understands XML as a tree data structure. XPath expressions are written in a way that represents searching and traversing through this tree. In other words, to select a node in XPath, you write the expression as if you are following a path or a number of steps to get to that node.

![XML Node Tree](http://www.w3schools.com/xml/nodetree.gif)

Here are some common terminologies to describe the different parts of the XML document tree:

**node** - A node is a unit in the document. A node can be an element node, an attribute node, a text node.

**element** - XML trees are composed of connected XML elements in a hierarchy = nodes of the tree. An XML element is everything from (including) the element's start tag to (including) the element's end tag.

**level** - a level represents a hierarchical connection from one node to another

**path** - the sequence of connections from node to node

**root** - referring to the top of the hierarchy of your XML tree structure

**child** - a node connected directly under the node you're talking about (one level below current node)

**parent** - converse of child, the node connected above the element you're talking about  (one level above current node)

**sibling** - nodes with same parent (same level as current node)

## Abbreviated Syntax

XPath uses the slash (/shell/fruit/seed) to denote traversal of the structure of your document in a path, in the same way as URLs or unix directories.

In XPath, all of your expressions are evaluated based on a context node. The context node is the node that you're starting your path expression from. The default context is your root node, indicated by a single slash (/).

The most useful path expressions are listed below:

| Expression   | Description |
|-----------------|:-------------|
| ```nodename```| Select all nodes with the name "nodename"   |
| ```/```  | A beginning single slash indicates a select from the root node, subsequent slashes indicate selecting a child node from current node  |
| ```//``` | Select direct and indirect child nodes in the document from the current node - this gives you the ability to "skip levels" |
| ```.```       | Select the current context node   |
|```..```  | Select the parent of the context node|
|```@```   |Select attributes|
|`text()`| Select the value of an element|
| &#124;|Pipe chains expressions and brings back results from either expression, think of a set union |


### Examples

| Path Expression   | Expression Result |
|-----------------|:-------------|
|```catalog```|	Select all nodes with the name "catalog"|
|```/catalog```|	Select the root element catalog. Note: If the path starts with a slash ( / ) it always represents an absolute path to an element! Your absolute path is the same as your context in this case.|
|```//author```| Select all nodes with the name "author"|
|```//author/..``` |Selects all of the parents of the nodes with the name "author"|
|```/catalog/book/@id```|Select all book node id attributes|


### Exercises

Now you try XPath using a different context. In BaseX, if you go to the tree view and double-click on a book node, your context will start at that book node. From your current context:

| Path Expression   | Expression Result |
|-----------------|:-------------|
|| Select the context book node|
|| Select the context's parent node|
|| Select the context's publish date value|
|| Select the author node|

Did you try /author? Why doesn't it work? How do you select the author node irregardless of context - the absolute path?

## Operators

Operators are used to compare nodes. There are mathematical operators, boolean operators. Operators can give you boolean (true/false values) as a result. Here are some useful ones:

| Operator   | Explanation |
|-----------------|:-------------|
|```=```|Equivalent comparison, can be used for numeric or text values|
|```!=```|Is not equivalent comparison|
|```>, >=```|Greater than, greater than or equal to|
|```<, <=```|Less than, less than or equal to|
|```or```|Boolean or|
|```and```|Boolean and|
|```not```|Boolean not|

### Examples

| Path Expression   | Expression Result |
|-----------------|:-------------|
|`/catalog/book/@category="WESTERN"`|Are any of the most popular books Westerns (category WESTERN)?|
|`/catalog/book/price>80`|Are there any books over $80?|


### Exercises

| Expression   | Result |
|-----------------|:-------------|
||Are any of the most popular books Sci-Fi (category SCIFI)?|
||Are there any books in the Computer genre that are over $10 but under $50?|
||I want to see the categories of all of the books but also the publish date. Hint: use the pipe|

## Predicates

Predicates are used to find a specific node or a node that contains a specific value.

Predicates are always embedded in square brackets, and are meant to provide additional filtering information to bring back nodes. You can filter on a node by using operators or functions.

### Examples

| Expression   | Result |
|-----------------|:-------------|
| ```[1]```  |Select the first node|
| ```[last()]```  |Select the last node|
| ```[last()-1]```  |Select the last but one node (also known as the second last node)|
|```[position()<3]```|Select the first two nodes, note the first position starts at 1, not = |
|```[@lang]```|	Select nodes that have attribute 'lang'|
|```[@lang='en']```|	Select all the nodes that have a "attribute" attribute with a value of "en"|
|```[price>15.00]```|	Select all nodes that have a price node with a value greater than 15.00|

Note: '!=' != 'not', for instance in this snippet

```xml
<book id='bk101'></book>
<book></book>
<book type='paperback'></book>
<book id='bk102'></book>
<book id='bk103'></book>
```

The ```[@id != 'bk101']``` will bring back

```xml
<book id='bk102'></book>
<book id='bk103'></book>
```

While ```[not(@id='bk101')]``` will bring back

```xml
<book></book>
<book type='paperback'></book>
<book id='bk102'></book>
<book id='bk103'></book>
```

This is because != indicates the existence of an @id, whie the not() expression expresses to bring back everything except @id='bk101'


##Wildcards

XPath wildcards can be used to select unknown XML nodes.

|Wildcard	|Description|
|-----------------|:-------------|
|```*```	|Matches any element node|
|```@*```|	Matches any attribute node|
|```node()```|	Matches any node of any kind|



### Examples

|Path Expression|	Result|
|-----------------|:-------------|
|```/catalog/*```	|Select all the child element nodes of the catalog element|
|```//*```	|Select all elements in the document|
|```//title[@*]```	|Select all title elements which have at least one attribute of any kind|


### Exercises

|Path Expression|	Result|
|-----------------|:-------------|
|   ```//@*```    |What do you expect as your result?|
|  |For all nodes that have the language attribute, select the immediate parent|
||Select book nodes where the id attribute corresponds to "bk101"|
||Select books that are either romance or fiction and are less than ten dollars. Hint: use `and` and `or`|
||Select all the title elements of the book elements of the catalog element that have a price element with a value greater than 15.00|
|```//title[@lang!="en"]```|What do you expect as your result?|
|```//title[not(@lang="en")]```|What do you expect as your result?|
|```/catalog/book[title[not(@lang='en')]][position()<4]/author/text()```|What do you expect as your result?|



## In-text search

XPath can do in-text searching using functions and also supports regex with its matches() function. Note: in-text searching is case-sensitive!

|Path Expression|	Result|
|-----------------|:-------------|
|```//author[contains(.,"Matt")]```| Matches on all author nodes, in current node contains Matt (case-sensitive)|
|```//author[starts-with(.,"G")]```|Matches on all author nodes, in current node starts with G (case-sensitive)|
|```//author[ends-with(.,"w")]```|Matches on all author nodes, in current node ends with w (case-sensitive)|
|```//author[matches(.,"Matt.*")]```| regular expressions match 2.0 |

### Exercises

| Expression   | Result |
|-----------------|:-------------|
||Match all publish date nodes, from January to May (Note: all dates are in yyyy-mm-dd format)|

### Exercises

Now try XPath with `xpath-xquery/data-menu/menu.xml`

| Expression   | Result |
|-----------------|:-------------|
||I want to know all of the foods with a nutritional value of 800 calories and less than or equal to 200 grams of sugar|
||I want to view the reviews of all foods with a review rating below 5 or are under $10|
||I want to compare the corporate and review description of these foods|
||I want to compare the cost of a food, the amount of sugar, and its review rating|
||I want to find all foods that have Waffle in its name|


## Complete syntax: XPath Axes

XPath Axes fuller syntax of how to use XPath. Provides all of the different ways to specify the path by describing more fully the relationships between nodes and their connections. The XPath specification describes 13 different axes:

* self ‐‐ the context node itself
* child ‐‐ the children of the context node
* descendant ‐‐ all descendants (children+)
* parent ‐‐ the parent (empty if at the root)
* ancestor ‐‐ all ancestors from the parent to the root
* descendant‐or‐self ‐‐ the union of descendant and self • ancestor‐or‐self ‐‐ the union of ancestor and self
* following‐sibling ‐‐ siblings to the right
* preceding‐sibling ‐‐ siblings to the left
* following ‐‐ all following nodes in the document, excluding descendants
* preceding ‐‐ all preceding nodes in the document, excluding ancestors • attribute ‐‐ the attributes of the context node


|Path Expression|	Result|
|-----------------|:-------------|
|```/descendant::book```|Select all book element nodes|
|```//attribute::lang```|Select all lang attribute nodes|



# XQuery

## Introduction

XQuery is a language used to query XML data. Single files, collection of files, or databases.

Below are a few examples of how XQuery can be used:

* Querying XML data to return XML documents
* Extracting and searching through XML data
* Combining, grouping, sorting, aggregating XML data
* Transforming XML documents into other XML, HTML or markup-based documents
* Cleaning XML data
* Publishing data from databases onto the web or in applications

## Functions
We've been using functions like contains, matches. For simplicity, BaseX hides the complete query but we were using the fn:doc function everytime we were querying a document!

A function is an action or set of actions that you can reuse. In XPath/XQuery, a function takes in a parameter or set of parameters to bring back a new result.

Functions on strings and numeric types, recognize dates and can do comparisons. For a list of functions, visit the [W3C Function specification.](https://www.w3.org/TR/xpath-functions/#func-replace)

Some useful string functions:

* fn:replace('query', 'input', 'replacementvalue') - can use regex with this function
* fn:upper-case('query')
* fn:lower-case('query')
* fn:normalize-space('query')

It's also possible to write your own reusable functions!

## FLWOR statements

XQuery statements are written with FLWOR clauses. If you are familiar with SQL, you structure your queries using clauses like SELECT, FROM, WHERE clauses and similarly can be used to join documents. In XQuery:

`:=` indicates creation of a variable creation and variable assignment in one fell swoop

**For** - the 'for' clause basically states: "for every item in a set of items, do something."

The 'for' clause in XQuery iteratively assigns a variable to every item in a set of items selected using XPath.

e.g. if you were to copy and paste this query into the Editor window:

```
for $titleitem in fn:doc("books.xml")/catalog/book/title
return $titleitem
```

You would get all of the titles of each book back.

**Let** - the 'let' clause basically declares a value without iteration. In XQuery, you assign a single result to a variable using let, which can be a single value, element or a whole set of elements. Your subsequent statements will act on an aggregate set of items.

e.g.

```
let $cost := 300
return $cost
```

e.g.

```
let $prices := fn:doc("books.xml")/catalog/book/price
let $sum := sum($prices)
return $sum
```

e.g.

```
for $author in fn:doc("books.xml")//author
  let $secondname := $author//secondaryname
return $secondname
```

**Where** - the 'where' clause is a boolean (true/false) statement that filters out nodes that do not satisfy this statement

```
for $author in fn:doc("books.xml")//author
  let $secondname := $author//secondaryname
  where $secondname[starts-with(.,"C")]
return $secondname
```

**Order By** - the default order of the results of an XPath/XQuery expression is the document order. the 'order' clause will sort or group items together based on this expression

e.g.
```
for $title in fn:doc("books.xml")/catalog/book/title
order by $title descending
return $title
```

**Return** - the 'return' clause will bring back specified the nodes

You can use as many for and let statements as you want and in any order.


### Exercises


Generate a report of all Fantasy books alphabetically sorted by author

```
<report><title>Book Report Stats</title> {
for $item in fn:doc("books.xml")/catalog/book
let $author := $item/author/text()
let $title := $item/title/text()
where fn:contains($item/genre, "Fantasy")
order by $title
return<object><id>{$author}</id><title>{$title}</title></object> }</report>
```

## Querying Databases

Up until now we've been querying single documents, but XPath and XQuery lets you query multiple documents using fn:collection("databasename") or in BaseX you can just use collection("databasename"). To specify a single document use fn:doc("file") or in a database in BaseX use db:open("database", "file").

XPath example:
```
fn:collection("data-collection-plants")/CATALOG/PLANT/COMMON
```

## XQuery Update

XQuery has an extension called [XQuery Update Facility](https://www.w3.org/TR/xquery-update-10/) that lets you directly modify the current XML document or database that you're working with.

In SQL there is the UPDATE clause which is used to update records in a table. In XQuery, the XQuery Update Facility (XQUF) provides five basic operations acting upon XML nodes:

* **insert node:** insert one or several nodes inside/after/before a specified node
* **delete node:** delete one or several nodes
* **replace node:** replace a node (and all its descendants if it is an element) by a sequence of nodes.
* **replace value of node:** replace the contents (children) of a node with a sequence of nodes, or the value of a node with a string value.
* **rename node:** rename a node (applicable to elements, attributes and processing instructions) without affecting its contents or attributes.

BaseX has a [complete implementation](http://docs.basex.org/wiki/XQuery_Update) of the XQuery Update specification that you'll want to take a look at to structure your queries. When you run these queries, they are not automatically saved unless you do so yourself. Nevertheless, it is still important to back up your work!

### Example

Running

```
for $zonenode in collection(data-collection-plants)//ZONE
return insert node $zonenode after $zonenode
```

Will find the nodes that match the xpath expression `//ZONE`, then insert an identical <ZONE /> node wherever that node occurs in the document.

### Exercises

* In `/data-menu-xquery`, clean up the prices so you can perform operations on those nodes as numeric values
```
for $text in db:open("data-menu-xquery","menuprices.xml")/breakfast_menu/food/price
return replace value of node $text with fn:replace($text, "\$", "")
```

* Rename the AVAILABILITY node to SERIALNO in the entire plant database.

```
for $availability in fn:collection(data-collection-plants)//AVAILABILITY
return rename node $availability as 'SERIALNO'
```

* Replace all occurrences of the word leaf with leaves in the entire plant database.

```
TRY IT OUT!
```

### Examples - bringing it all together



### Non-updating functions

There's also copy, modify, return. BaseX also has its own update function.

# Reference

## Setup

[Install basex](http://basex.org/products/download/)

Mac:
[Install a JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
You need to install [homebrew](http://brew.sh/)
Once that's installed, run
```bash
brew install basex
```

Windows: use installer
Linux: use the distribution package

In your terminal, run

```$ basexgui```
