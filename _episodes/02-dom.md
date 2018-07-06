---
title: "Querying the Document Object Model with XPath"
teaching: 60
exercises: 30
questions:
- "How is a webpage organised from a code perspective?"
- "What is the DOM?"
objectives:
- "Introduce the Document Object Model"
- "Explain the structure of an HTML document"
- "Explain how to view the underlying HTML content of a web page in a browser"
- "Visualise a page's DOM in a browser"
- "Build a sample XPath query base on the DOM"

keypoints:
- "HTML is a markup language. It provides structure to documents."
- "HTML documents are made out of nodes, which form a hierarchy."
- "HTML uses the Document Object Model to build a webpage."
- "We need to be able to find the content we want to scrape in the page's DOM"
- "The hierarchy of nodes inside a document is called the Document Object Model."
- "Relationships between nodes are: parent, child, sibling."

---

Before we delve into web scraping proper, we will first spend some time introducing
some of the techniques that are required to indicate exactly what should be
extracted from the web pages we aim to scrape.

The material in this section was adapted from the [XPath and XQuery Tutorial](https://github.com/code4libtoronto/2016-07-28-librarycarpentrylessons/blob/master/xpath-xquery/lesson.md)
written by [Kim Pham](https://github.com/kimpham54) ([@tolloid](https://twitter.com/tolloid))
for the July 2016 [Library Carpentry workshop](https://code4libtoronto.github.io/2016-07-28-librarycarpentry/) in Toronto.

# Introduction
XPath (which stands for XML Path Language) is an _expression language_ used to specify parts of an XML document.
XPath is rarely used on its own, rather it is used within software and languages that are aimed at manipulating XML documents, such as XSLT, XQuery or the web scraping tools that will be introduced later in this lesson.



> ## XML and HTML
>
> Modern HTML has many similiarities to XML. We can therefore use "XPath" to query HTML documents. Moreover, all of the discussion in this section can also be applied to other XML documents.
>
{: .callout}



## Markup Languages
XML and HTML are _markup languages_. This means that they use a set of tags or rules to organise and provide information about the data they contain. This structure helps to automate processing, editing, formatting, displaying, printing, etc. that information.

XML-based documents store data in plain text format. This provides a software- and hardware-independent way of storing, transporting, and sharing data. XML format is an open format, meant to be software agnostic. You can
open an XML document in any text editor and the data it contains will be shown as it is meant to be represented.
This allows for exchange between incompatible systems and easier conversion of data.


Any XML document follows basic syntax rules:

* An XML document is structured using _nodes_, which include element nodes, attribute nodes and text nodes
* XML element nodes must have an opening and closing tag, e.g. `<catfood>` opening tag and `</catfood>` closing tag
* XML tags are case sensitive, e.g. `<catfood>` does not equal `<catFood>`
* XML elements must be properly nested:

~~~
<catfood>
  <manufacturer>Purina</manufacturer>
    <address> 12 Cat Way, Boise, Idaho, 21341</address>
  <date>2019-10-01</date>
</catfood>
~~~
{: .language-html}

* Text nodes (data) are contained inside the opening and closing tags
* XML attribute nodes contain values that must be quoted, e.g.
``` <catfood type="basic"></catfood> ```

# The Document Object Model

This lesson cannot really teach HTML and Javascript from scratch. However, we need to make sure we review some things about HTML. For a good discussion on the Document Object Model, see this <a href="https://www.w3schools.com/Js/js_htmldom.asp">W3Schools tutorial</a>.

The <a href="https://www.w3.org/DOM/">W3C defines</a> the DOM as:

> The Document Object Model is a platform- and language-neutral interface that will allow programs and scripts to dynamically access and update the content, structure and style of documents. The document can be further processed and the results of that processing can be incorporated back into the presented page.

This idea of a living software "object" being an HTML "page" is what allows us to have our dynamic and interactive web today. 

Every HTML page that we scrape is a container of containers which contain... containers and eventually text. We can, using an XPath expression, address any container or any pattern of containers. Before we continue, let us explore editing a page's DOM.


## Playing with the DOM

We will use the HTML code that describes this very page you are reading as an example. By default, a web browser
interprets the HTML code to determine what markup to apply to the various elements of a document, and the code is
invisible. To make the underlying code visible, all browsers have a function to display the raw HTML content of
a web page.



> ## A note on using the Safari browser
>
> If you are using Safari, you must first turn on the "Develop" menu in order to view the page source, and use the
> functions that we will use later in this section. To do so, navigate to Safari > Preferences and in the Advanced tab
> select the "Show Develop in menu bar" option. Note: in recent versions of Safari you must first turn on the "Develop" menu (in Preferences) and then navigate to `Develop > Show Javascript Console` and then click on the "Console" tab.
>
{: .callout}

The HTML structure of the page you are currently reading looks something like this (most text and elements have
been removed for clarity):

~~~
<!doctype html>
<html lang="en">
  <head>
    (...)
    <title>{{page.title}}</title>
  </head>
  <body>
   (...)
  </body>
</html>
~~~
{: .html}

We can see from the source code that the title of this page is in a `<title>` element that is itself inside the
`<head>` element, which is itself inside an `<html>` element that contains the entire content of the page.



> ## Exercise: Interactively edit this page
> Using your favourite browser, open up the browser "inspector" or developer console.
> 
> Edit the following sentence by removing the word "not":
>
> <p>You are <span style="color:red">not</span> ready to continue. </p>
>
> We first want to inspect the sentence above. Right click on the word "not", and choose "inspect."
>
> ![The developer console with the span highlighted]({{ page.root }}/fig/not.png)
> 
> Now, right click on the HTML `<span style...` and choose "delete element."
> 
> Observe how the page changes to:
>
> ![Ready to continue now]({{ page.root }}/fig/ready.png)
>
> Now, refresh the page, and play around with the text and HTML (change the colour of the word, for example) in this exercise box. Note how it returns to the original HTML when you refresh the page.
{: .challenge}


The page is a living document, with a skeleton we can inspect, manipulate, and query. We will now figure out how to walk through that skeleton.



> ## Discussion
>
> What does the inspect functionality of the browser allow us to do?
>
> * A. View the source of the webpage.
> * B. Instead of reading the code, we can point at the element we want and have it identify itself to us, live.
> * C. We can hack the webpage!
> * D. Quickly open the developer tools.
> 
> > ## Solution
> > B. The inspector allows us to interact with the page in front of us, using graphical manipulation to select elements. While D and A are kind of answers, choosing "view source" allows us to see the *original* source of the webpage (instead of the source of what is actually being displayed to us) and the function key <kbd>F12</kbd> is faster than choosing right-click, inspect, and then scrolling around. The inspect functionality allows us to precicely target elements in the *actual* web page we are looking at in front of us. 
> {: .solution}
> 
{: .discussion}


# XPath expressions

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

## Navigating through the HTML node tree using XPath

A popular way to represent the structure of an XML or HTML document is the _node tree_:

![HTML Node Tree](http://www.w3schools.com/js/pic_htmltree.gif)

In an HTML document, everything is a node:

* The entire document is a document node
* Every HTML element is an element node
* The text inside HTML elements are text nodes

The nodes in such a tree have a hierarchical relationship to each other. We use the terms _parent_, _child_ and
_sibling_ to describe these relationships:

* In a node tree, the top node is called the *root* (or *root node*)
* Every node has exactly one *parent*, except the root (which has no parent)
* A node can have zero, one or several *children*
* *Siblings* are nodes with the same parent
* The sequence of connections from node to node is called a *path*

![Node relationships](http://www.w3schools.com/js/pic_navigate.gif)

Paths in XPath are defined using slashes (`/`) to separate the steps in a node connection sequence, much like
URLs or Unix directories.

In XPath, all expressions are evaluated based on a *context node*. The context node is the node in which a path
starts from. The default context is the root node, indicated by a single `/`, as in the example above.

The most useful path expressions are listed below:

| Expression   | Description |
|-----------------|:-------------|
| ```nodename```| Select all nodes with the name "nodename"   |
| ```/```  | A beginning single slash indicates a select from the root node, subsequent slashes indicate selecting a child node from current node  |
| ```//``` | Select direct and indirect child nodes in the document from the current node - this gives us the ability to "skip levels". If we start out a query with `//`, this means we can find any child HTML element in the document without addressing its full path from `<html>`  |
| ```.```       | Select the current context node   |
|```..```  | Select the parent of the context node|
|```@```  | Select attributes of the context node|
|```[@attribute = 'value']```   |Select nodes with a particular attribute value|
|`text()`| Select the text content of a node|
|`normalize-space()`| Remove all of the whitespace around text|
| &#124;|Pipe chains expressions and brings back results from either expression, think of a set union |


> ## Discussion
>
> Is the DOM?
>
> * A. A way of finding data
> * B. HTML code
> * C. Something that a web browser has
> * D. A Hierarchy of elements which contain other elements
> 
> > ## Solution
> > D. The DOM is a hierarchy of nodes, called elements, which may be transversed to render or find data.
> {: .solution}
> 
{: .discussion}

## How do we use these?

There are many many web technologies which provide for: "Find me this specific element in the DOM." We are teaching XPath because it is fundamental to the DOM and the technology that all the others are based on. As you can imagine, all the others are more specific and more suited to their purposes. (If you want to learn more about this, take a look at JQuery.)

This next exercise will allow us to practice with an XPath directly. It can also be very useful for debugging XPaths when you are using them in other programs and scripts.


We can run XPath queries directly from python by calling a component of scrapy called the scrapy shell.

> ## Exercise: Run Scrapy Shell against this page
> 
> in the command prompt, run `scrapy shell "{{page.root}}"` and follow along with the following explorations.
> 
> > ## Quotes
> > It's always a good idea to enclose URLs in quotes in the command line as some of their characters do odd things to the shell
> {: .callout}
{: .challenge}

The syntax to run an XPath query within the scrapy shell is `response.xpath("XPath_QUERY")`, for example:

~~~
>>> response.xpath("/html/head/title/text()")
~~~
{: .source .language-python}

This should return something similar to

~~~
[<Selector xpath='/html/head/title/text()' data='Introduction to web scraping: Querying t'>]
~~~
{: .output}

Let's look closer at the XPath query used in the example above: `/html/head/title/text()`. The first `/` indicates
the _root_ of the document. With that query, we told scrapy shell to

|-----------------|:-------------|
| `/`| Start at the root of the document... |
| `html/`| ... navigate to the `HTML` node ... |
| `head/`| ... then to the `head` node  that's inside it... |
| `title/`| ... then to the `title` node that's inside it... |
| `text()`| and select the text node contained in that element |

Using this syntax, XPath thus allows us to determine the exact _path_ to a node.

> ## Select the "Introduction" title
> Write an XPath query that selects the "Introduction" title above and try running it in the console.
>
> Tip: if a query returns multiple elements, the syntax `element[1]` can be used. Note that
> XPath uses one-based indexing, therefore the first element has index 1, the second has index 2 etc.
>
> > ## Solution
> >
> > ~~~
> > >>> response.xpath("/html/body/div/article/h1[1]")
> > ~~~
> > {: .source .language-python}
> >
> > should produce something similar to
> >
> > ~~~
> > [<Selector xpath='/html/body/div/article/h1[1]' data='<h1 id="introduction">Introduction</h1>'>]
> > ~~~
> > {: .output}
> >
> {: .solution}
{: .challenge}

Before we look into other
ways to reach a specific HTML node using XPath, let's start by looking closer at how nodes are arranged
within a document and what their relationships with each others are.


For example, to select all the `blockquote` nodes of this page, we can write

~~~
>>> response.xpath("/html/body/div/article/blockquote")
~~~
{: .source .language-python}

This produces an array of objects:

~~~
[<Selector xpath='/html/body/div/article/blockquote' data='<blockquote class="objectives">\n  <h2>Ov'>, <Selector xpath='/html/body/div/article/blockquote' data='<blockquote class="callout">\n  <h2 id="x'>, <Selector xpath='/html/body/div/article/blockquote' data='<blockquote>\n  <p>The Document Object Mo'>, <Selector xpath='/html/body/div/article/blockquote' data='<blockquote class="callout">\n  <h2 id="a'>, <Selector xpath='/html/body/div/article/blockquote' data='<blockquote class="challenge">\n  <h2 id='>, <Selector xpath='/html/body/div/article/blockquote' data='<blockquote class="discussion">\n  <h2 id'>, <Selector xpath='/html/body/div/article/blockquote' data='<blockquote class="callout">\n  <h2 id="x'>, <Selector xpath='/html/body/div/article/blockquote' data='<blockquote class="discussion">\n  <h2 id'>, <Selector xpath='/html/body/div/article/blockquote' data='<blockquote class="callout">\n  <h2 id="e'>, <Selector xpath='/html/body/div/article/blockquote' data='<blockquote class="challenge">\n  <h2 id='>, <Selector xpath='/html/body/div/article/blockquote' data='<blockquote class="challenge">\n  <h2 id='>, <Selector xpath='/html/body/div/article/blockquote' data='<blockquote class="challenge">\n  <h2 id='>, <Selector xpath='/html/body/div/article/blockquote' data='<blockquote class="challenge">\n  <h2 id='>, <Selector xpath='/html/body/div/article/blockquote' data='<blockquote class="discussion">\n  <h2 id'>, <Selector xpath='/html/body/div/article/blockquote' data='<blockquote class="keypoints">\n  <h2>Key'>]

~~~
{: .output .language-html}

This selects all the `blockquote` elements that are under `html/body/div`. If we want instead to select _all_
`blockquote` elements in this document, we can use the `//` syntax instead:

~~~
>>> response.xpath("//blockquote")
~~~
{: .source .language-python}

This produces a longer array of objects:

~~~
[<Selector xpath='//blockquote' data='<blockquote class="objectives">\n  <h2>Ov'>, <Selector xpath='//blockquote' data='<blockquote class="callout">\n  <h2 id="x'>, <Selector xpath='//blockquote' data='<blockquote>\n  <p>The Document Object Mo'>, <Selector xpath='//blockquote' data='<blockquote class="callout">\n  <h2 id="a'>, <Selector xpath='//blockquote' data='<blockquote class="challenge">\n  <h2 id='>, <Selector xpath='//blockquote' data='<blockquote class="discussion">\n  <h2 id'>, <Selector xpath='//blockquote' data='<blockquote class="solution">\n    <h2 id'>, <Selector xpath='//blockquote' data='<blockquote class="callout">\n  <h2 id="x'>, <Selector xpath='//blockquote' data='<blockquote class="discussion">\n  <h2 id'>, <Selector xpath='//blockquote' data='<blockquote class="solution">\n    <h2 id'>, <Selector xpath='//blockquote' data='<blockquote class="callout">\n  <h2 id="e'>, <Selector xpath='//blockquote' data='<blockquote class="challenge">\n  <h2 id='>, <Selector xpath='//blockquote' data='<blockquote class="solution">\n    <h2 id'>, <Selector xpath='//blockquote' data='<blockquote class="challenge">\n  <h2 id='>, <Selector xpath='//blockquote' data='<blockquote class="challenge">\n  <h2 id='>, <Selector xpath='//blockquote' data='<blockquote class="solution">\n    <h2 id'>, <Selector xpath='//blockquote' data='<blockquote class="challenge">\n  <h2 id='>, <Selector xpath='//blockquote' data='<blockquote class="quotation">\n<p>Moral '>, <Selector xpath='//blockquote' data='<blockquote class="solution">\n    <h2 id'>, <Selector xpath='//blockquote' data='<blockquote class="discussion">\n  <h2 id'>, <Selector xpath='//blockquote' data='<blockquote class="solution">\n    <h2 id'>, <Selector xpath='//blockquote' data='<blockquote class="keypoints">\n  <h2>Key'>]

~~~
{: .output .language-html}

> ## Why is the second array longer?
> If you look closely into the array that is returned by the `response.xpath("//blockquote")` query above,
> you should see that it contains objects like `data='<blockquote class="solution">` that were not
> included in the results of the first query. Why is this so?
>
> Tip: look at the source code and see how the challenges and solutions elements are
> organised.
>
{: .challenge}

We can use the `class` attribute of certain elements to filter down results. For example, looking
at the list of `blockquote` elements returned by the previous query, and by looking at this page's
source, we can see that the blockquote elements on this page are of different classes
(challenge, solution, callout, etc.).

Let's have a look at the HTML code of this page, around this challenge box (using the "Inspect" option in our browser). The code looks something like this:
~~~
<!doctype html>
<html lang="en">
  <head>
    (...)
  </head>
  <body>
  <div class="container">
  (...)
    
<blockquote class="challenge">
  <h2 id="exercise-select-all-challenge-boxes-by-class">Exercise: Select all challenge boxes by class</h2>
  <p>Using an XPath query in the Javascript console of your browser, select the element that contains the text
you are currently reading on this page.</p>
      (...)
    </blockquote>
  (...)
  </div>
  </body>
</html>
~~~
{: .language-html}

We know that the `class` attribute is characteristic of all of the challenge boxes. This means we can create an Javascript query of XPath of 
~~~
>>> response.xpath("//blockquote[@class='challenge']");
~~~
{: .source .language-python}
This should return something like:
~~~
[<Selector xpath="//blockquote[@class='challenge']" data='<blockquote class="challenge">\n  <h2 id='>, <Selector xpath="//blockquote[@class='challenge']" data='<blockquote class="challenge">\n  <h2 id='>, <Selector xpath="//blockquote[@class='challenge']" data='<blockquote class="challenge">\n  <h2 id='>, <Selector xpath="//blockquote[@class='challenge']" data='<blockquote class="challenge">\n  <h2 id='>, <Selector xpath="//blockquote[@class='challenge']" data='<blockquote class="challenge">\n  <h2 id='>]

~~~
{: .output}

Let's walk through that syntax:
|-----------------|:-------------|
| `$x("`| This function tells the browser we want it to execute an XPath query. |
| `//`| Look anywhere in the document... |
| `blockquote`| ... for an blockquote element ... |
| `[@class = 'challenge']`| ... that has an `class` attribute set to `challenge`... |
| `");` | end of the Javascript function. |

By hovering on the object returned by your XPath query in the console, your browser should helpfully highlight
that object in the document, enabling you to make sure you got the right ones.



> ## Exercise: Select the "Introduction" title by ID
> In a previous challenge, we were able to select the "Introduction" title because we knew it was
> the first `h1` element on the page. But what if we didn't know how many such elements were on the
> page. In other words, is there a different attribute that allows us to uniquely identify that title
> element?
>
> Using the path expressions introduced above, rewrite your XPath query to select
> the "Introduction" title without using the `[1]` index notation.
>
>
> Tips:
>
> * In principle, `id` attributes in HTML are unique on a page. This means that if you know the `id`
>   of the element you are looking for, you should be able to construct an XPath that looks for this value
>   without having to worry about where in the node tree the target element is located.
> * The syntax for selecting an element like `<div id="mytarget">` is `div[@id = 'mytarget']`.
> * Remember that XPath queries are relative to a context node, and by default that node is the root node.
> * Use the `//` syntax to select for elements regardless of where they are in the tree.
> * The syntax to select the parent element relative to a context node is `..`
> * The `response.xpath(...)` Python syntax will always return an array of nodes, regardless of the number of
>   nodes returned by the query. Contrary to XPath, Python uses _zero based indexing_, so the syntax to get
>   the first element of that array is therefore `response.xpath(...)[0]`.
>
>
> > ## Solution
> >
> > ~~~
> > response.xpath("/html/body/div/article/h1[@id='introduction']")
> > ~~~
> > {: .source}
> >
> > should produce something similar to
> >
> > ~~~
> > [<Selector xpath="/html/body/div/article/h1[@id='introduction']" data='<h1 id="introduction">Introduction</h1>'>]
> > ~~~
> > {: .output}
> >
> {: .solution}
{: .challenge}





> ## Exercise: Combining the above
> 
> Unfortunately, on most websites, the data we want is seldom so easily extracted. 
> 
> Consider the following quotation from a book by [Neil Stephenson](https://www.nealstephenson.com/):
> 
> <blockquote class="quotation">
> <p>Moral reforms and deteriorations are moved by large forces, and they are mostly caused by reactions from the habits of a preceding period. Backwards and forwards swings the great pendulum, and its alterations are not determined by a few distinguished folk clinging to the end of it. </p>
> <p>Sir Charles Petrie, <a href="https://trove.nla.gov.au/work/5842140?q&versionId=12043210">The Victorians</a>. Epigraph from <a href="https://trove.nla.gov.au/work/15024537">The Diamond Age</a> by Neal Stephenson</p>
> </blockquote>
> 
> Write an XPath query which can get only titles of the books inside that quote.
> 
>  We know that we can select challenge blockquotes through `response.xpath("//blockquote[@class='challenge']");`. And we know we can follow elements into each other by using `/`. And we know we can get the text of an element through `text()`. How do we combine these ideas?
>  
> > ## Solution
> >  
> >  `response.xpath("//blockquote[@class='quotation']/p/a/text()")`
> >  
> >  We start with `class='quotation'` to find the container which has an identifiable class. If we don't restrict to `class='quotation'` then we get that trap link to Neil Stephenson I included above. 
> >  
> >  We then descend down through the `<p>`aragraph element, to the `<a>` elements. Since the preceeding paragraph doesn't have any hyperlinks, nothing in it is matched. We then use `text()` to get the text of those two links.
> >
> >  ~~~
>> [<Selector xpath="//blockquote[@class='quotation']/p/a/text()" data='The Victorians'>, <Selector xpath="//blockquote[@class='quotation']/p/a/text()" data='The Diamond Age'>]
> >  ~~~
> >  {: .output}
> {: .solution} 
>  
{: .challenge}



# Wrapping up XPath




> ## Discussion
>
> What are three ways of finding elements we want?
>
> * A. By path, by ID, and by class
> * B. By XPath, //, and, ${}
> * C. By path, element name, and relative position
> * D. View source, Ctrl-F, the element inspector
> 
> > ## Solution
> > A. We can address the elements relative to each other or the root, `/html/body/div/article/blockquote`, by a unique property they have (id): `//blockquote[@id = 'select-this-challenge-box']` or by a shared property we are interested in (class): `//blockquote[@class='challenge']` 
> {: .solution}
> 
{: .discussion}