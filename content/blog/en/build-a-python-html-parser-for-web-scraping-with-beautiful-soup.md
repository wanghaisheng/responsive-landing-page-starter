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
The internet contains the largest collection of data in human history. As a programmer, all that data is available to you if you learn how to build a web scraper. A web scraper is a piece of software that collects data from websites. It is a powerful tool you can use to feed data to your programs. For example, you could scrape data from a lyrics website and use it to create word clouds from the day’s top 10 most popular songs. Or you could analyze headlines and use their sentiment to trade stocks. In this tutorial, you will learn how to scrape data from the web using Python and Beautiful Soup (a Python library). To follow along, you should have a basic understanding of programming in Python. I do not assume you have experience as a web developer, so I will explain all of the web basics you need to keep up. By the end of this tutorial, you will have a functioning web scraper that collects data from a website. Let's get started! 

## How Web Scraping Works

When you visit a website, your web browser sends an HTTP request to the website’s server asking for the resources your browser needs to display the site. The server might respond with files containing HTML, CSS, JavaScript, and anything else your browser needs to display the site. HTML gives a website its structure, CSS gives it style, and JavaScript makes it interactive.

When you build a web scraper, you write code that sends the HTTP request for you, without you having to go to the website using your web browser. Your program then extracts the data it needs and processes it. 

So a web scraper:

1. Makes a request to a website.
2. Gets the website’s HTML.
3. Searches the HTML for what it needs. 
4. Uses the data to do something.

## Web Scraping Problems

Although web scraping is useful in many situations, it does have some problems. One problem is web scrapers often break. Web scrapers rely on a website’s content staying the same, so when a developer updates a site content, it can break your scraper and you have to make changes to fix it. 

Web scraping can also be against the terms of service of certain websites, so it is important to read a website's terms of service before scraping data from it. 

If a website allows you to scrape data from it in their terms of service, scraping a website’s data still costs money for the website (because you are consuming its resources), so here are a few things to keep in mind as best practices when you are scraping data: 

1. Don’t scrape data more often than you need to. 
2. Cache data when you can. 

## Web Scraping VS. APIs

Before you build a web scraper, you may want to investigate if the data source you are scraping from has an API. An API is an application programming interface, and it allows two programs to talk to each other. For example, at Vonage, we have an SMS API that allows you to programmatically send an SMS message. We also have a video API that lets you easily add video streaming to your website and a bunch of other communication APIs. 

Many data sources offer APIs that give you access to their information without having to write a web scraper. For example, Rotten Tomatoes has an API you can use to get their movie ratings. There are several advantages to using Rotten Tomatoes API instead of scraping the data from their website yourself. The first is that you don’t have to worry about violating their terms of service. It is also much faster to get data from an API than to scrape it. Finally, Rotten Tomatoes API is managed by a team of developers, so you don’t have to worry about it breaking like a web scraper could.

## What is HTML? 

Before we get any further, here is a quick primer on HTML (if you are already familiar with web development feel free to skip this section). HTML stands for hyper-text markup language and is a markup language that gives websites their structure. For example, this is what HTML looks like:![](https://lh6.googleusercontent.com/PwRNMCaDayPIg3daAZR1c3zNBqqAMNvOK8NxJLZNZgpFTTfxU7yJPuCe72akCzWMN447Hg_M5h42qSW_R3KFSJHvlM-GfIGtpkjB7I6Iwc0r6PH3VD7uqfdqRi5AWfzmBwjlUgkP=s0)As you can see, HTML consists of tags that tell your web browser to do something. In this case, your HTML tells your web browser to create a webpage that says “Example Domain” as a header (which means it is displayed in bold in large font). Underneath “Example Domain” there is text inside of <p> tags. A <p> tag tells your browser to create a paragraph of text. Finally, there is information inside an <a> tag that tells your browser to create a link to https://www.iana.org/domains/example.

This is what your web browser produces when it displays the HTML in the image above. 

![Example.com site](/content/blog/build-a-python-html-parser-for-web-scraping-with-beautiful-soup/example.com.png)

To see this website live, you can head to [www.example.com](http://www.example.com). To see its HTML you can press Ctrl+U in your browser or Cmd+Option+U on a Mac (Cmd+U if you are using Firefox).

## Downloading a Website's HTML

Alright, it is time to start building our web scraper! In this tutorial, we will scrape all the data from [www.example.com](http://www.example.com). You can use Python's request library to send an HTTP request. An HTTP request is how your browser gets the HTML, JavaScript, CSS, and anything else it needs from a website’s server. There are many different types of HTTP requests to accomplish various tasks, but you can use a GET request to ask a website's server to send you its resources. Here is how to use the request library to send a GET request and print example.com's HTML.

```python
import request



print(request.get('example.com').content)
```



When you run this code, you should see example.com's HTML, which should look something like this:

## Parsing HTML with Beautiful Soup

We can now use the Beautifulsoup library to parse example.com's HTML. To do this, we are going to import the BeautifulSoup library and use it to create a BeautifulSoup object like this: 

```python
import requests

from bs4 import BeautifulSoup



URL = "https://example.com"

page = requests.get(URL)

soup = BeautifulSoup(page.content, "html.parser")
```

Now you can use your BeautifulSoup object’s find method to search for different tags in the HTML. The find method accepts the name of a tag as a parameter and returns all the tags that match.   



print(soup.find('p'))



In this case, you searched the HTML for ‘p’ tags, which stands for paragraph and BeautifulSoup returned the first result.