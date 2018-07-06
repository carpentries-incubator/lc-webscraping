---
title: "Scrape data using browser extensions"
teaching: 30
exercises: 30
questions:
- "How can I get started scraping data off the web?"
- "How can I use XPath to more accurately select what data to scrape?"
objectives:
- "Introduce the Chrome Scraper extension."
- "Practice scraping data that is well structured."
- "Practice scraping data that is poorly structured."
- "Use XPath queries to refine what needs to be scraped when data is less structured."
keypoints:
- "Data that is relatively well structured (in a table) is relatively easy to scrape."
- "More often than not, web scraping tools need to be told what to scrape."
- "XPath can be used to define what information to scrape, and how to structure it."
- "More advanced data cleaning operations are best done in a subsequent step."

---
# XPath and the Scraper Chrome Extension

Let us get back to where we were in the introduction. 

We proposed the following scenario:

> Let's go to the list of [UK House of Commons members](https://www.parliament.uk/mps-lords-and-offices/mps/). Backup link: [ https://perma-archives.org/warc/M8BC-JUJ8/https://www.parliament.uk/mps-lords-and-offices/mps/](https://perma-archives.org/warc/M8BC-JUJ8/https://www.parliament.uk/mps-lords-and-offices/mps/). We are interested in downloading this list to a spreadsheet, with columns for names and
constituencies. 

But our problem was the horrible whitespace in the clipboard and wondering about what that "XPath" thing was. Now let's unpack that a bit more. 

![Screenshot of the Scraper main window]({{ page.root }}/fig/scraper-ukparl-01.png)

## XPath and the Document Object Model

Note the "Selector" on the left column. It should read:

~~~
//tbody/tr[td]
~~~

![A good output of the scraper]({{ page.root }}/fig/goodManualScrape.png)


What happens if you don't highlight the full row and right click scrape similar? We see something like:

~~~
//tr[2]/td
~~~
![A bad output of the scraper]({{ page.root }}/fig/badManualScrape.png)

The difference in these two "XPath Queries" is where they're telling the scrape extension to look within the "Document Object Model" of the webpage. We will be exploring these in a few minutes, but right now, we need to get this data to a usable "spreadsheet" format.





## Cleaning scraped Data

One characteristic of scraped data is that it is very seldom formatted for clean computer consumption. If we choose "Copy to Clipboard" and we paste the data copied from Scraper into a text document, we see something like this:

~~~
Name  Constituency
A back to top
                                 Abbott, Ms Diane                                 (Labour)                              Hackney North and Stoke Newington
                                 Abrahams, Debbie                                 (Labour)                              Oldham East and Saddleworth
~~~
{: .output}

This is because there are a lot of unnecessary white spaces in the HTML that's behind that table, which
are being captured by Scraper. We can however tweak the XPath column selectors to take advantage of the
`normalize-space` XPath function:

~~~
normalize-space(*[1])
normalize-space(*[2])
~~~

![Screenshot of the Scraper window showing the Column selectors]({{ page.root }}/fig/scraper-ukparl-02.png)

We now need to tell Scraper to scrape the data again by using our new selectors, this is done by clicking
on the "Scrape" button. The preview will not noticeably change, but if we now copy again the results
and paste them in our text editor, we should see:

~~~
Name  Constituency
A back to top
Abbott, Ms Diane (Labour) Hackney North and Stoke Newington
Abrahams, Debbie (Labour) Oldham East and Saddleworth
Adams, Nigel (Conservative) Selby and Ainsty
~~~
{: .output}

which is a bit cleaner.



## Custom XPath queries

Sometimes, however, we do have to do a bit of work to get Scraper to select the data elements
that we are interested in.

Going back to the example of the Canadian Parliament we saw in the introduction,
there is a page on the same website that [lists the mailing addresses](http://www.ourcommons.ca/Parliamentarians/en/members/addresses) With Backup link [https://perma-archives.org/warc/ZBM9-4VRE/http://www.ourcommons.ca/Parliamentarians/en/members/addresses](https://perma-archives.org/warc/ZBM9-4VRE/http://www.ourcommons.ca/Parliamentarians/en/members/addresses) of all
parliamentarians. We are interested in scraping those addresses.

If we select the addresses for the first MP and try the "Scrape similar" function...

![Screenshot of the Scraper context menu being used on an address block]({{ page.root }}/fig/scraper-canparl-01.png)

Scraper produces this:

![Screenshot of the Scraper window trying to scrape addresses]({{ page.root }}/fig/scraper-canparl-02.png)

which does a nice job separating the address elements, but what if instead we want a table of
the addresses of all MPs? Selecting multiple addresses instead does not help. Remember what we said
about computers not being smart about structuring information? This is a good example. We humans
know what the different blocks of texts on the screen mean, but the computer will need some help from
us to make sense of it.

We need to tell Scraper what exactly to scrape, using XPath.

If we look at the HTML source code of that page, we see that individual MPs are all within `ul`
elements:

~~~
(...)
<ul>
   <li><h3>Aboultaif, Ziad</h3></li>
   <li>
      <span class="addresstype">Hill Office</span>
      <span>Telephone:</span>
      <span>613-992-0946</span>
      <span>Fax:</span>            
      <span>613-992-0973</span>
   </li>
   <li>
         <ul>       
            <li><span class="addresstype">Constituency Office(s)</span></li>
            <li>                            
               <span>8119 - 160 Avenue (Main Office)</span>
               <span>Suite 204A</span>
               <span>Edmonton, Alberta</span>
               <span>T5Z 0G3</span>
               <span>Telephone:</span> <span>780-822-1540</span>
               <span>Fax:</span> <span>780-822-1544</span>                                    
               <span class="spacer"></span>
            </li>                         
      </ul>
   </li>                                   
</ul>   
(...)
~~~
{: .language-html}

So let's try changing the Selector XPath in Scraper to

~~~
//body/div[@class='container']/div/div/ul
~~~
{: .source}

and hit "Scrape". We get something that is closer to what we want, with one line per MP, but
the addresses are still all in one block of unstructured text:

![Screenshot of the Scraper window trying to scrape addresses]({{ page.root }}/fig/scraper-canparl-03.png)

Looking closer at the HTML source, we see that name and addresses are separated by `li` elements
within those `ul` elements. So let's add a few columns based on those elements:

~~~
./li[1] -> Name
./li[2] -> Hill Office
./li[3] -> Constituency
~~~
{: .source}

This produces the following result:

![Screenshot of the Scraper window scraping addresses]({{ page.root }}/fig/scraper-canparl-04.png)

The addresses are still one big block of text each, but at least we now have a table for all MPs
and the addresses are separated.


> ## Scrape the Canadian MPs' phone numbers
> Keep working on the example above to add a column for the Hill Office phone number
> and fax number for each MP.
>
>
> > ## Solution
> > 
> > Add columns with the XPath query
> > 
> > ~~~
> > ./li[2]/span[3] -> Hill Office Phone
> > ./li[2]/span[5] -> Hill Office Fax
> > ~~~
> > {: .source}
> >
> > ![Screenshot of the Scraper window on scraping MP phone numbers]({{ page.root }}/fig/scraper-canparl-05.png)
> >
> {: .solution}
{: .challenge}


## Getting attribute data from elements.

Let us explore the UK parliamentary website for a moment. Can we get the profile URIs of the members?

This isn't something that can be solved by being careful with our selection criteria. We'll have to add another column in the scraper. 

First, we must identify where the data is that we want. 
Open up the inspector on the first link (currently Ms Diane Abbot) and we see:

![Diane Abbot in the inspector view]({{page.root}}/fig/diane.png)

Note the `<a ... href="http://www.parliament.uk/biographies/commons/ms-diane-abbott/172">Abbott, Ms Diane</a>`

This means that the link, or `href` does indeed contain their profile URL. Now we need to get the scraper to extract that data into a new column.

Currently, "Surname, First name" is extracted via `*[1]` into the first column.


The syntax to select the value of an attribute of the type `<element attribute="value">` is `element/@attribute`.

Let us add a new column by clicking the green `+`. Let us now explore relative XPaths some more. If we add `td/a` to column 3 and click scrape, we see: 

![Adding td/a gives us name without affiliation]({{page.root}}/fig/col3.png)

Note how this shows us the *text* inside the first element called `a` in the first table cell `td`. We can extract this, because this text is *inside* the container `a`. It is far more tedious to extract their party affiliation sans name, without using a programming language. But, we're not interested in the text here. We are interested in the attribute of the element called `href`. Therefore, let us add `/@href`. 

We see:
![Adding td/a/@href gives us url]({{page.root}}/fig/href.png)

## Exercises: scrape the list of Australian members of parliament, their page URLs, their districts, their parties, and their twitter handles.

Let us remind ourselves of the handy set of XPath expressions referenced in the prior section.


| Expression   | Description |
|-----------------|:-------------|
| ```nodename```| Select all nodes with the name "nodename"   |
| ```/```  | A beginning single slash indicates a select from the root node, subsequent slashes indicate selecting a child node from current node  |
| ```//``` | Select direct and indirect child nodes in the document from the current node - this gives us the ability to "skip levels" |
| ```.```       | Select the current context node   |
|```..```  | Select the parent of the context node|
|`element/@attribute`| The syntax to select the value of an attribute of the type `<element attribute="value">`|
|```[@attribute = 'value']```   |Select nodes with a particular attribute value|
|`text()`| Select the text content of a node|
|`normalize-space()`| Remove all of the whitespace around text|
| &#124;|Pipe chains expressions and brings back results from either expression, think of a set union |






> ## Part 1: Get scraped list by element
> 
> Use Scraper to export the list of the first twelve [Members of the Australian Parliment](https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=&mem=1&par=-1&gen=0&ps=0). Backup link: [https://perma-archives.org/warc/8ATF-RT3Q/https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=](https://perma-archives.org/warc/8ATF-RT3Q/https://www.aph.gov.au/Senators_and_Members/Parliamentarian_Search_Results?q=).
> 
> We want to get a spreadsheet of three columns: their name, their profile URL, and their district. 
> * Explore highlighting different parts of each member's `div` until you get a list of results by name.
> * Use the inspector to find the elements that contain the data you want.
> * Use XPath relative paths to indicate those elements into their own columns.
> Tips:
> 
> We first need to find the HTML tag that "holds" each row. Or we can play around with highlighting     
> 
> * To add another column in Scraper, use the little green "+" icon in the columns list.
> * Look at the source code and try out XPath queries in the console until you find what
>   you are looking for.
> * The syntax to select the value of an attribute of the type `<element attribute="value">`
>   is `element/@attribute`.> 
> 
> > ## Solution
> > 
> > `//div[2]/div[2]/div[1]/div[3]/div` worked for the selected XPath. 
> > 
> > My three columns were:
> > 
> > |--------|:--------|
> > | `normalize-space(*[2]/h4)` | Name |
> > | `normalize-space(*[2]/dl/dd)` | District |
> > | `*[2]/h4/a/@href` | Link |
> > 
> > Which produced:
> > 
> > ~~~
> > Name    District    Link
Hon Tony Abbott MP  Warringah, New South Wales  /Senators_and_Members/Parliamentarian?MPID=EZ5
> > Hon Anthony Albanese MP Grayndler, New South Wales  /Senators_and_Members/Parliamentarian?MPID=R36
> > Mr John Alexander OAM, MP   Bennelong, New South Wales  /Senators_and_Members/Parliamentarian?MPID=M3M
> > Dr Anne Aly MP  Cowan, Western Australia    /Senators_and_Members/Parliamentarian?MPID=13050
> > Hon Karen Andrews MP    McPherson, Queensland   /Senators_and_Members/Parliamentarian?MPID=230886
> > Hon Kevin Andrews MP    Menzies, Victoria   /Senators_and_Members/Parliamentarian?MPID=HK5
> > Mr Adam Bandt MP    Melbourne, Victoria /Senators_and_Members/Parliamentarian?MPID=M3C
> > Ms Julia Banks MP   Chisholm, Victoria  /Senators_and_Members/Parliamentarian?MPID=18661
> > Hon Sharon Bird MP  Cunningham, New South Wales /Senators_and_Members/Parliamentarian?MPID=DZP
> > Hon Julie Bishop MP Curtin, Western Australia   /Senators_and_Members/Parliamentarian?MPID=83P
> > Hon Chris Bowen MP  McMahon, New South Wales    /Senators_and_Members/Parliamentarian?MPID=DZS
> > Mr Andrew Broad MP  Mallee, Victoria    /Senators_and_Members/Parliamentarian?MPID=30379
> > ~~~
> > 
> {: .solution}
{: .challenge}


> # Discussion
> 
>  Question for the workshop. How can we use what we've learned so far in our own personal research?
{: .discussion}


## Part 2: Get scraped list by class (Advanced)


> # Discussion
> 
>  Do we want to explore the *techniques* of scraping more deeply, focusing on how to get tricky data out of a webpage, or do we want to focus on the *tools* more deeply and figuring out how to automate this with python? 
>  
>  If the latter, proceed to the [next section](/04-scrapy)
{: .discussion}

Modern "responsive" web-pages make pesudo-tables out of divs. Usually, these tables have specific "class" stylings to make them behave like tables. We can take advantage of this fact. 


We need to introduce two new functions in this exercise: `text()` and the idea of `following-sibling::`. 

We can interrogate the content of an element inside an XPath selector by saying `[text()='foo']` which will return to us every element containing exactly the text "foo". When webpage designers don't put their data in container-elements, searching by `text()` still allows us to automate searching through the data.

We can test for "containing" with the `element[contains(haystack, needle)]` function. 

Therefore we need to explore some more [axes of XPath](https://www.w3schools.com/xml/XPath_axes.asp). Specifically, we need to explore the way of saying: "get me the tag after this tag." In this instance, this is the axis of `following-sibling::`. 


We now need to build the XPath cheatsheet out more by adding `contains()` and `following-sibling`. 

| Expression   | Description |
|-----------------|:-------------|
| ```nodename```| Select all nodes with the name "nodename"   |
| ```/```  | A beginning single slash indicates a select from the root node, subsequent slashes indicate selecting a child node from current node  |
| ```//``` | Select direct and indirect child nodes in the document from the current node - this gives us the ability to "skip levels" |
| ```.```       | Select the current context node   |
|```..```  | Select the parent of the context node|
|`element/@attribute`| The syntax to select the value of an attribute of the type `<element attribute="value">`|
|```[@attribute = 'value']```   |Select nodes with a particular attribute value|
|`text()`| Select the text content of a node|
|`normalize-space()`| Remove all of the whitespace around text|
| `&#124;` |Pipe chains expressions and brings back results from either expression, think of a set union |
|`element[contains(haystack, needle)]`| Matches elements which have the needle string in their haystack (which can be, among other things, @class or text(). )|
|`following-sibling::element`| Finds the element which is a *sibling* of the parent instead of a *child* of the parent.|



> ## Exercise Part 2: get scraped list by class
> In the inspector find a unique class holding each record. In this instance, we also need to find the classes of the container holding each (hint) row. 
> 
> While there are many `class="row"`, there is only one repeating element `//div[@class='row border-bottom padding-top']`
> 
> Here, let us get their twitter handle along with their name and party.
> 
> [It's a trap!](https://www.youtube.com/watch?v=4F4qzPbcFiA)
> 
> If we use `*/dl/dd`, it works for Tony, but the moment someone has a position, our data becomes garbage. We need to be more creative. We know that the "Definition list" has the word "party" in its `dt`, 
> 
> > ## Solution
> > 
> > `//div[@class='row border-bottom padding-top']` worked for the selected XPath. 
> > 
> > My three columns were:
> > 
> > |--------|:--------|
> > | `normalize-space(*/h4)` | Name |
> > | `*/dl/dt[text()='Party']/following-sibling::dd` | Party |
> > | `*/dl/dd/a[contains(@class, 'twitter')]/@href` | Twitter |
> > 
> > Which produced:
> > 
> > ~~~
> > Name    Party   Twitter
Hon Tony Abbott MP  Liberal Party of Australia  http://twitter.com/TonyAbbottMHR
Hon Anthony Albanese MP Australian Labor Party  http://twitter.com/AlboMP
Mr John Alexander OAM, MP   Liberal Party of Australia  http://twitter.com/JohnAlexanderMP
Dr Anne Aly MP  Australian Labor Party  
Hon Karen Andrews MP    Liberal Party of Australia  http://twitter.com/karenandrewsmp
Hon Kevin Andrews MP    Liberal Party of Australia  http://twitter.com/kevinandrewsmp
Mr Adam Bandt MP    Australian Greens   http://twitter.com/adambandt
Ms Julia Banks MP   Liberal Party of Australia  
Hon Sharon Bird MP  Australian Labor Party  http://twitter.com/SharonBirdMP
Hon Julie Bishop MP Liberal Party of Australia  http://twitter.com/JulieBishopMP
Hon Chris Bowen MP  Australian Labor Party  http://twitter.com/Bowenchris
Mr Andrew Broad MP  The Nationals   https://twitter.com/broad4mallee
> > ~~~
> > 
> > Let us break down the XPaths used here.
> > 
> > `//div[@class='row border-bottom padding-top']` matches the specific row styling of the results. We can't use `class='row'` because that matches other `div` elements. This method is slightly more robust than the relative path example used in part 1 because even if they move elements around or change headers, this primary "search result box" will be consistent. This page, however, is designed for output rendering without any semantic coding, so while this is slightly more robust, it's not *that* much stronger. It is, however, easier to debug.
> > 
> > `normalize-space(*/h4)` is the same pattern as before.
> > 
> > `*/dl/dt[text()='Party']/following-sibling::dd` is the first "evil line." By asking for "Party" instead of "Location", I removed our ability to say `/dl/dd[2].` As that will match party for some and position for others. Instead, we need a way of articulating which row of the "definition list" (the `dl/dt/dd` thing) we are interested in. We do that with `*/dl/dt[text()='Party']`. Unfortunately, that alone produces a column of "Party". While everyone attending a party is enjoyable, it is not what we are after. 
> > 
> > By using the `following-sibling::` axis, we can say "Give us the `<dd>` element which follows this `<dt>` element." This took a fair amount of googling on my part, but will be a very useful pattern for you to know. 
> > 
> > `*/dl/dd/a[contains(@class, 'twitter')]/@href` follows a similar idea. "Say what we want to search for, then get it". Here, we don't need to search for the word "connect" because only items in connect have `<a>` elements. However, we need to search for the word twitter in the various class items of those elements with `a[contains(@class, 'twitter')]`. Once we *find* the link, we then need to get the `href` attribute of the link with `/@href`.
> {: .solution}
> 
> A mistake (I, the author) kept making while creating this exercise was to confuse the part of the XPath where I was "searching" for something with the part of the XPath telling me what it is I wanted. The difficult part of finding the twitter url was `a[contains(@class, 'twitter')]`.
> 
> 
{: .challenge}

This page, unfortunately doesn't have any useful elements delineated by id. As a final takeaway, most of these XPath queries have been well answered on stack overflow. It is worth searching on [Questions tagged "XPath"](https://stackoverflow.com/questions/tagged/XPath) when exploring a query you are trying to write. In the process of writing this lesson, I had to refer to three separate answers.

> ## Discussion
> 
> Why couldn't we use relative positioning to get party affiliation or twitter handle?
> 
> * A. There were too many elements.
> * B. There was no semantic markup. 
> * C. `*/dl/dd[2]` produced garbage data.
> * D. The elements were not consistent relative to each other. Sometimes they were the second, sometimes they were the third.
> 
> > ## Solution
> > 
> >  D. Because the data we wanted "moved around" for each `<div>` we were looking in, we need to be more specific in how we find the data. Semantic markup *would* have allowed easier addressing, but it also isn't involved in relative positioning. 
> {: .solution}
{: .discussion}

# References

* [The w3schools XPath tutorial is a great cheat sheet for the syntax](https://www.w3schools.com/xml/XPath_intro.asp)
* [Questions tagged "XPath" on Stack Overflow](https://stackoverflow.com/questions/tagged/XPath)