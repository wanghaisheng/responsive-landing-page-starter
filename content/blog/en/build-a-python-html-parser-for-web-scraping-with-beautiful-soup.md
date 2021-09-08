---
title: Build a Python HTML Parser for Web Scraping with Beautiful Soup
description: "Learn how to parse HTML by building a web scraper using Beautiful
  Soup and Python. "
author: cory-althoff
published: true
published_at: 2021-09-08T01:08:28.326Z
updated_at: 2021-09-08T01:08:28.361Z
category: tutorial
tags:
  - python
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
The internet contains the most extensive collection of data in human history. All that data is available to you if you learn how to build a web scraper. A web scraper is a piece of software that collects data from websites. It is a powerful tool you can use to feed data to your programs. For example, you could scrape data from a lyrics website and use it to create word clouds from the day’s top 10 most popular songs. Or you could analyze headlines and use their sentiment to trade stocks. In this tutorial, you will learn how to scrape data from the web using Python and Beautiful Soup (a Python library). To follow along, you should have a basic understanding of programming in Python. I do not assume you have experience as a web developer, so I will explain all of the web basics you need to keep up. By the end of this tutorial, you will have a functioning web scraper that collects data from a website. Let's get started! 

## How Web Scraping Works

When you visit a website, your web browser sends an HTTP request to the website’s server, asking for the resources your browser needs to display the site. The server might respond with files containing HTML, CSS, JavaScript, and anything else your browser needs to display the site. HTML gives a website its structure, CSS gives it style, and JavaScript makes it interactive.

When you build a web scraper, you write code that sends the HTTP request for you and uses the data without you having to go the website using your web browser. So a web scraper:

1. Makes a request to a website.
2. Gets the website’s HTML.
3. Searches the HTML for what it needs. 
4. Uses the data to do something.

## Web Scraping Problems

Although web scraping is helpful in many situations, it does have some problems. One problem is web scrapers often break. Web scrapers rely on a website’s HTML staying the same, so when a developer updates a site, it can break your scraper, and you have to make changes to fix it. 

Web scraping can also be against the terms of service of certain websites, so it is important to read a website's terms of service before scraping data from it. 

If a website allows you to scrape data from it in their terms of service, scraping a website’s data still costs money for the website (because you are consuming its resources), so here are a few things to keep in mind as best practices when you are scraping data: 

1. Don’t scrape data more often than you need to. 
2. Cache data when you can. 

## Web Scraping VS. APIs

Before you build a web scraper, you may want to investigate if the data source you are scraping from has an API. An API is an application programming interface, and it allows two programs to talk to each other. For example, at Vonage, we have an SMS API that allows you to send an SMS message programmatically. We also have a video API that lets you easily add video streaming to your website and a bunch of [other communication APIs](https://www.vonage.com/communications-apis/). 

Many data sources offer APIs that give you access to their information without having to write a web scraper. For example, [IMDB has an API](https://developer.imdb.com/documentation) you can use to get their movie ratings. There are several advantages to using IMDB's API instead of scraping the data from their website yourself. The first is that you don’t have to worry about violating their terms of service. It is also much faster to get data from an API than to scrape it. Finally, a team of developers manages the IMDB API, so you don’t have to worry about it breaking like a web scraper could.

## What is HTML? 

Before we get any further, here is a quick primer on HTML (if you are already familiar with web development, feel free to skip this section). HTML stands for hyper-text markup language and is a markup language that gives websites their structure. For example, this is what HTML looks like:![](https://lh6.googleusercontent.com/PwRNMCaDayPIg3daAZR1c3zNBqqAMNvOK8NxJLZNZgpFTTfxU7yJPuCe72akCzWMN447Hg_M5h42qSW_R3KFSJHvlM-GfIGtpkjB7I6Iwc0r6PH3VD7uqfdqRi5AWfzmBwjlUgkP=s0)As you can see, HTML consists of tags that tell your web browser to do something. In this case, your HTML tells your web browser to create a webpage that says “Example Domain” as a header (which means it is in a  large font). Underneath “Example Domain,” there is text inside of `<p>` tags. A `<p>` tag tells your browser to create a paragraph of text. Finally, the information inside the `<a>` tag tells your browser to create a link to https://www.iana.org/domains/example.

This image is what your web browser produces when it displays the HTML in the image above. 

![Example.com site](/content/blog/build-a-python-html-parser-for-web-scraping-with-beautiful-soup/example.com.png)

To see this website live, you can head to [www.example.com](http://www.example.com). To see its HTML, you can press Ctrl+U in your browser or Cmd+Option+U on a Mac (Cmd+U if you are using Firefox).

## Downloading a Website's HTML

Alright, it is time to start building our web scraper! In this tutorial, we will scrape all the data from [www.example.com](http://www.example.com). The first thing we need to do is get its HTML. You can get a website's HTML by sending an HTTP request. An HTTP request is how your browser gets the HTML, JavaScript, CSS, and anything else it needs from a website’s server. There are many types of HTTP requests to accomplish various tasks, but you can use a GET request to ask a website's server to send you its resources. Python has a built-in library called `request` that lets you easily send an HTTP request. Here is how to use the `request` library to send a GET request and print example.com's HTML.

```python
import request



print(request.get('example.com').content)
```



When you run this code, you should see example.com's HTML.



## Parsing HTML with Beautiful Soup

We can now use Python's `BeautifulSoup` library to parse example.com's HTML. To do this, we are going to import the `BeautifulSoup` library and use it to create a `BeautifulSoup` object like this: 

```python
import requests

from bs4 import BeautifulSoup



URL = "https://example.com"

page = requests.get(URL)

soup = BeautifulSoup(page.content, "html.parser")
```

Now you can use your `BeautifulSoup` object’s find method to search for different tags in the HTML. The find method accepts the name of a tag as a parameter and returns all the tags that match.   

```python
print(soup.find('p'))
```

In this case, you searched the HTML for `p` tags, which stands for paragraph, and BeautifulSoup returned the first result.

![HTML example](/content/blog/build-a-python-html-parser-for-web-scraping-with-beautiful-soup/html.png)

The text “This domain is for use…” is in a `<p>` tag, so in this case, your code returns that text. 

The part of the example.com website that says “Example Domain” is in a `<h1>` tag. To scrape “Example Domain,” you can pass in `h1` to find_all instead of `p`.

```python
print(soup.find(“h1”))
```

Now, your code should print this: `[<h1>Example Domain</h1>]`.

The last piece of information on example.com is the link at the end that says “More information…” to grab this final piece of information, you need to search for an `a` tag. 

```python
print(soup.find(“a”))
```

Now when you run your code, it should return the link like this: 

`[<a href="https://www.iana.org/domains/example">More information...</a>]`

## Scrape More Data

Let’s take a look at how to scrape even more data from a website. 

When you are using your web browser, and you have multiple tabs open, each tab has the website's name.

Web developers define a website’s title in a `<title>` tag. You can get a website’s title like this: 

```python
import requests

from bs4 import BeautifulSoup



URL = "https://example.com"

page = requests.get(URL)

soup = BeautifulSoup(page.content, "html.parser")

print (soup.title.get_text())
```

When you run this code, Python should print “Example Domain.”

## Regular Expressions

If you want to get fancier with your web scraping, you can use regular expressions. A regular expression is a sequence of characters that define a search pattern. 

Python has a built-in library called `re` you can use to define them. For example, you can define a regular expression that searches for numbers in a string. Here is how to use re to search for numbers in a string.

```python
import re



print(re.findall('\d+', 'hello 1 hello 2 hello 3'))



>> \[‘1’, ‘2’, ‘3’]
```



As you can see, this regular expression returned all of the numbers in the string `'hello 1 hello 2 hello 3'`. The regular expression that looks for digits is `'\d+'`. 

Regular expressions are flexible: you can write regular expressions to match everything from broad patterns to specific ones. For example, here is how to match a regular expression that only matches strings that start with "The" and end with "brown." 

```python
import re



print(re.findall('^The.*brown$', 'The fox is brown'))
```

In this case, Python prints the string because it starts with The and ends with brown.

But if you changer the string to end with green, Python does not find a match: 

```python
import re



print(re.findall('^The.*brown$', 'The fox is green))
```

You can use regular expressions when you are scraping data from websites. Here is how: 

```python
import re

import requests

from bs4 import BeautifulSoup



URL = "https://example.com"

page = requests.get(URL)

soup = BeautifulSoup(page.content, "html.parser")

result = soup.find_all(re.compile("(head|div)"))

print(result)
```



This code uses a regular expression to return anything either in a `<head>` tag or a `<div>` tag.

## Final Thoughts

Congratulations! You know how to scrape a website! All of the public data from the web is now at your fingertips. Of course, as we discussed earlier, web scraping does have some problems, so before you commit to scraping data, it is best to see if the data source provides an API first. If they do not provide one, and their terms of service allow it, you can now scrape any data you need. 

Of course, there is more to web scraping than the basics I covered in this tutorial. If you want to learn more about web scraping, you can try a free Coursera course like [Using Python to Access Web Data](https://www.coursera.org/learn/python-network-data). You can also read through BeautifulSoup’s documentation.

 Finally, you might want to check out Scrapy, a popular Python framework for web scraping.

``