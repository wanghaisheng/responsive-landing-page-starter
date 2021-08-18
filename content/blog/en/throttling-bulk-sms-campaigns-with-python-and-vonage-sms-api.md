---
title: Throttling Bulk SMS Campaigns with Python and Vonage SMS API
description: "A guide to creating a bulk SMS campaign that includes throttling
  and sending in batches. "
author: cory-althoff
published: true
published_at: 2021-08-18T21:49:37.989Z
updated_at: 2021-08-18T21:49:38.022Z
category: tutorial
tags:
  - sms-api
  - python
  - django
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
## Introduction

[Bulk SMS](https://www.bandwidth.com/glossary/bulk-sms/) comes in handy when marketing teams need to do promotional campaigns. Organizations can also deploy them to pass information to a large group of people. 

Problems can arise when you send a bulk SMS campaign meant to generate responses to a large audience. For instance, receiving thousands of inbound responses at once could overwhelm your team. One solution to this problem is to use throttling. For example, you can design a campaign to send the messages in batches, at specific periods, or both. 

This article will teach you how to implement bulk SMS throttling in Python using the Django REST framework and the Vonage Messages API. The web application we will build in this tutorial will allow you to message multiple users in batches at specified time intervals.

[Vonage Messages API](https://developer.nexmo.com/messages/overview) allows developers to develop SMS-based applications and implement messaging features in their apps for SMS, WhatsApp, Messenger, etc. [Django](https://www.djangoproject.com/) is a Python framework used for building web applications. [Django REST framework](https://www.django-rest-framework.org/) allows developers to build [RESTful APIs](https://en.wikipedia.org/wiki/Representational_state_transfer) with Django.

## Prerequisites

1. If you don't have a [Vonage API account](https://dashboard.nexmo.com/sign-up), you may \[create an account](https://dashboard.nexmo.com/sign-up) and get free credit.

   <sign-up number></sign-up>


2. Vonage application - you can follow this [guide](<1. <https://developer.nexmo.com/messages/code-snippets/create-an-application>>) to create an application on your Vonage dashboard.
3. Ensure you have installed version 3.6 or a later version of [](<1. https://www.python.org/>)Python on your development machine. You can download Python from the [official website](https://www.python.org/).
4. Have pip installed and working on your machine. See [instructions for installing pip here](https://pip.pypa.io/en/stable/installing/).
5. Virtualenv: Venv is useful for creating isolated [virtual environments for Python projects](https://pypi.org/project/virtualenv/).



## Setup and Installation

You will start with setting up the project dependencies and installing the modules you need with 'pip.' Next, you will create the Django project for the tutorial.

### Install Dependencies

Firstly, create a new directory and create a virtual environment. Then, activate the newly created virtual environment:

```
mkdir test_project && cd test_project
python -m venv env
source env/bin/activate
```

The packages installed above are:

1. 'Django': the Django framework package
2. 'djangorestframework': the Django REST framework for creating APIs in Django
3. 'django-cors-headers': the Django-cors-headers allow our API to make cross-origin requests to other servers
4. 'vonage': the Vonage Python server SDK

### Create a Django project

Now, use the `django-admin` utility to create a Django project called `vonage_project`:

```
django-admin startproject vonage_project
```

You need to configure the Django-cors-headers for the application. That way, other origins and frontend applications can make a request to your Django application. Go to the `MIDDLEWARE` in the `settings.py` file and add the following middleware classes:

"`python
...
MIDDLEWARE = \[
    ...
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.common.CommonMiddleware',
    ...
]
...

```

```

Next, create a Django app called 'myapp' to host our bulk SMS functionality:

"`bash
cd vonage
django-admin startapp myapp

```

```

## Create Bulk SMS functionality

In this section, you will set up the bulk SMS feature with the Vonage SMS API. You will also implement a throttling feature.