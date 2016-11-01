---
layout: page
title: Setup
permalink: /setup/
---
In order to follow this lesson, you will need to make sure the following software is installed on your computer.

## Part one: Manually scrape data using browser extensions

For the first half of the lesson, we will use a Chrome browser extension to get started with web scraping. 

1. Please ensure you have a working copy of the [Chrome browser](https://www.google.com/intl/en/chrome/browser/).
2. Using Chrome, download and enable the [Scraper extension](https://chrome.google.com/webstore/detail/scraper/mbigbapnjcgaffohmbkdlecaccepngjd).

## Part two: Write Python programs to automatically scrape data

### Shell and Python
The second part of the lesson requires the Python programming language and access to a command-line interface (shell) on your computer.
Please refer to [the Software Carpentry setup instructions](http://swcarpentry.github.io/workshop-template/#setup) for
*the Bash shell* and *Python* if you need guidance.

> ## Prerequisites
> This part of the lesson requires some prior knowledge of Python and how to use a shell.
> If you need help getting started on those topics, we suggest going through the following
> lessons first (during a workshop or on your own):
> 
> * [The Unix Shell](http://swcarpentry.github.io/shell-novice/)
> * [Programming with Python](http://swcarpentry.github.io/python-novice-inflammation/)
>
{: .prereq}

### Scrapy

Once you have a working installation of Python, the next step is to install [Scrapy](https://scrapy.org/).

If you have installed Python using the Anaconda framework as suggested by the Software Carpentry setup instructions,
you can easilly install Scrapy by doing the following:

1. Open a new shell (e.g. Terminal on Mac, or the Anaconda command-line tool on Windows)
2. Type the following:

> conda install -c scrapinghub scrapy
>
{: .source}

Alternatively, if you have another distribution of Python, you can try using pip:

> pip install Scrapy
>
{: .source}

If you run into issues while installing Scrapy, refer to the 
[official Scrapy install guide](https://doc.scrapy.org/en/latest/intro/install.html#intro-install)
or get in touch with your lesson instructor.
