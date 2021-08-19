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

1. A free [Vonage API account](https://dashboard.nexmo.com/sign-up).

   <sign-up number></sign-up>
2. A Vonage application. You can [follow this guide](https://developer.nexmo.com/messages/code-snippets/create-an-application) to create an application on your Vonage dashboard.
3. Python (version 3.6 or a later). You can download Python from the [official website](https://www.python.org/).
4. The Python package manager pip. You can find [instructions for installing pip here](https://pip.pypa.io/en/stable/installing/).
5. The Python tool virtualenv for creating isolated [virtual environments for Python projects](https://pypi.org/project/virtualenv/).

## Setup and Installation

You will start with setting up the project dependencies and installing the modules you need with 'pip.' Next, you will create the Django project for the tutorial.

### Install Dependencies

First, create a new directory and a virtual environment. Then, activate the newly created virtual environment:

```
mkdir test_project && cd test_project
python -m venv env
source env/bin/activate
```

The commands above installed the following packages: 

1. `Django`: the Django framework package.
2. `djangorestframework`: the Django REST framework for creating APIs in Django.
3. `django-cors-headers`: Allows our API to make cross-origin requests to other servers.
4. `vonage`: the Vonage Python server SDK.

### Create a Django project

Now, use the `django-admin` utility to create a Django project called `vonage_project`:

```
django-admin startproject vonage_project
```

Next, you need to configure the `Django-cors-headers` for the application. That way, other origins and frontend applications can make a request to your Django application. Go to the `MIDDLEWARE` in the `settings.py` file and add the following middleware classes:

```python
MIDDLEWARE = [
    ...
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.common.CommonMiddleware',
    ...
]
```

Next, create a Django app called `myapp` to host our bulk SMS functionality:

```
cd vonage
django-admin startapp myapp
```

## Create Bulk SMS functionality

In this section, you will set up the bulk SMS feature with the Vonage SMS API. You will also implement a throttling feature.

### Initialize the Vonage Library

Usually, you would need to initialize the Vonage library to use the Vonage API for sending messages. However, the new Vonage Messages API is in beta, and Python is not supported yet. However, we can still use the Messages API. Follow along the tutorial to see how.

First, add the following code to the `views.py` file:

```python
import base64

vonageCredentials = 'API_KEY:API_SECRET'
encodedData = vonageCredentials.encode("utf-8")
b64value = b64encode(encodedData).decode("ascii")
```

In the above code, replace the `API_KEY` and `API_SECRET` values with the values from your Vonage dashboard. The `vonageCredentials` variable takes your Vonage credentials: your API key and secret key in the form `'API_KEY: API_SECRET'`. You then encode and decode the string with your credentials in base64 form to pass them as [](https://en.wikipedia.org/wiki/ASCII)ASCII standard characters.

### Create a View to Send SMS message

Now, create a view for sending the SMS messages in `views.py` like this:

```python
from vonage import Sms
from django.views.decorators.csrf import csrf_exempt
from rest_framework.decorators import parser_classes
from rest_framework.parsers import JSONParser
import json
from django.http.response import JsonResponse
import time

vonageCredentials = 'API_KEY:API_SECRET'
encodedData = vonageCredentials.encode("utf-8")
b64value = base64.b64encode(encodedData).decode("ascii")

@ csrf_exempt
@ parser_classes(\[JSONParser])
def sendMessage(self, request):
    pass
```

In the code above, you imported the `csrf_exempt`, `parser_classes`, and `JSONParser` class to enable you to define decorators for the view. You also imported the `json`,  `JsonResponse`, and `time` modules. Then, you created the `sendMessage` function, where you will put the logic for sending the message.

Next, you will add code inside the `sendMessage` view to accept requests and user inputs. Modify `view.py` as follows:

```python
def sendMessage(self, request):
    if request.method == 'POST':
        body_unicode = request.body.decode('utf-8')
        body_data = json.loads(body_unicode)

        sender = body_data['sender']
        recipients = body_data['recipients']
        message_string = body_data['message_string']
        batch_size = body_data['batch_size']
        delay_period = body_data['delay_period']
```

In the above code, you specified the view accepts POST requests in the line: `request.method == 'POST'`. Then, you decoded the request body in JSON format. After that, you stripped the body of the request to the items it contains. The items are the following inputs received from the user:

1. `sender`: a variable that contains the information about the message sender.
2. `recipients`: a list of the phone numbers of the SMS recipients.
3. `message_string`: the text you will send in the bulk SMS campaign. 
4. `batch_size`: stipulates the number of recipients to send an SMS to at once.
5. `delay_period`: the time frame in between sending SMS batches (measured in seconds).

Now, you will create a function to split the list of the recipients' phone numbers into batches. Add the following code outside the `sendMessage` function in the `views.py` file:

```python
def batch(recipients, batch_size=1):
    for i in range(0, len(recipients), batch_size):
        yield recipients\[i:min(i + batch_size, len(recipients))]
```

1. You defined a function called `batch`. It accepts two parameters: the `recipients` list and the `batch_size` integer that represents how many recipients you want to send a message at once.
2. A range of phone numbers. You use the yield keyword to return a batch. You repeat this until you've sent a message to all your recipients. 

You can read more about the [range](https://docs.python.org/3/library/stdtypes.html#typesseq-range) and [yield](https://www.geeksforgeeks.org/python-yield-keyword/) keywords work.

Now, you will implement the logic to enable sending messages. Modify the `sendMessage` view as shown below:

```python
def sendMessage(self, request):
    if request.method == 'POST':
        body_unicode = request.body.decode('utf-8')
        body_data = json.loads(body_unicode)

        sender = body_data['sender']
        recipients = body_data['recipients']
        message_string = body_data['message_string']
        batch_size = body_data['batch_size']
        delay_period = body_data['delay_period']

        for eachBatch in batch(recipients, batch_size):
            for number in eachBatch:
                response = requests.post('https://api.nexmo.com/v0.1/messages',
                     headers={
                         "Authorization": "Basic %s" % b64value,
                         "Content-type": "application/json",
                         "Accept": "application/json"},
                     json={
                         "to": {
                             "type": "sms",
                             "number": number
                         },
                         "from": {
                             "type": "sms",
                             "number": sender
                         },
                         "message": {
                             "content": {
                                 "type": "text",
                                 "text": message_string
                             }
                         }
                     })
                print("message sent to ", number)
            time.sleep(delay_period)
        try:
            return JsonResponse("OK", status=200, safe=False)

        except Exception as e:
            return JsonResponse({'the error is': str(e)}, status=403)
```

In the code above, the first "for" statement that was added loops through batches of phone numbers using the `batch` function created earlier. 

The inner "for" loop takes each phone number in a batch and makes an HTTP request to the Vonage Messages API. The request header contains the base64 encoded credentials that we derived earlier as a "b64value." It also includes a JSON payload to be delivered to the Messages API. The JSON payload consists of the receiver details in the `to` dictionary, the sender's details in "from," and the message content in 'message.'

* the `send_message` function takes the `from` as the sender of the SMS campaign and `to` as the SMS receiver. It also takes the `text` as the SMS message. 
* the value of the `sender` input taken from the user of your application will be shown to the receivers in the text received as the sender of the message.
* the phone numbers of the recipients that will receive a message at a particular time, based on the `batch_size` are fed into the `to` variable.
* the `delay_period` delays the execution after each batch of SMS is sent. The delay is executed by the `time.sleep(delay_period)` line. The delay period is in seconds.
* a `JsonResponse` is returned if the SMS sending is successful or an error statement if it is unsuccessful.

The full `views.py` code is as follows:

```python
from django.views.decorators.csrf import csrf_exempt
from rest_framework.decorators import parser_classes
from rest_framework.parsers import JSONParser
import json
import requests
import base64
import time
from django.http.response import JsonResponse


vonageCredentials = 'API_KEY:API_SECRET'
encodedData = vonageCredentials.encode("utf-8")
b64value = base64.b64encode(encodedData).decode("ascii")


def batch(recipients, batch_size=1):
    for i in range(0, len(recipients), batch_size):
        yield recipients[i:min(i + batch_size, len(recipients))]


@ csrf_exempt
@ parser_classes([JSONParser])
def sendMessage(request):
    if request.method == "POST":
        body_unicode = request.body.decode('utf-8')
        body_data = json.loads(body_unicode)

        sender = body_data['sender']
        recipients = body_data['recipients']
        message_string = body_data['message_string']
        batch_size = body_data['batch_size']
        delay_period = body_data['delay_period']

        for eachBatch in batch(recipients, batch_size):
            for number in eachBatch:
                requests.post('https://api.nexmo.com/v0.1/messages',
                     headers={
                         "Authorization": "Basic %s" % b64value,
                         "Content-type": "application/json",
                         "Accept": "application/json"},
                     json={
                         "to": {
                             "type": "sms",
                             "number": number
                         },
                         "from": {
                             "type": "sms",
                             "number": sender
                         },
                         "message": {
                             "content": {
                                 "type": "text",
                                 "text": message_string
                             }
                         }
                     })
                print("message sent to ", number)
            time.sleep(delay_period)
```

### Define a URL path

Since you have created a view for receiving requests, you will need a corresponding URL for users to access the view to make requests. Therefore, you will add a path to the `urlpatterns` inside the `urls.py` file of the project. Navigate to the project subdirectory and add the following code:

```python
from django.urls import path
from myapp.views import sendMessage

urlpatterns = [
   ...
    path('message/', sendMessage),
]
```

As shown above, you imported `path` and the `sendMessage` view. Then, you added a path with the URL `message/` to the list of `urlpatterns`.

## Test Application API

You can test the functionality built above with the [Postman](https://www.postman.com) tool for simulating and documenting APIs. You can [sign up](https://identity.getpostman.com/signup) for a free Postman account.

To run this test, you also need to start Django's test server like this: 

```
Python manage.py runserver
```

Let's assume you intend to use the following details for your SMS bulk campaign. 

```json
{
    "sender": "Admin Team",
    "recipients": [2340000000000, 23411111111, 2342222222222, 2343333333333, 2344444444444, 2345555555555, 2346666666666, 2347777777777, 2348888888888, 2349999999999],
    "message_string": "COVID-19 Vaccine now available in your state",
    "batch_size": 3,
    "delay_period": 3600
}
```

You can input the above details into the body of a Postman request as JSON as shown in the following image:

![A Postman screenshot]()

Make sure you replace the dummy recipient phone numbers with real numbers before sending the request. Then, the messages will be delivered to the recipients, as shown in a smartphone screenshot below.

!\[A screenshot of the received message on a smartphone]()

## Conclusion

In this article, you implemented bulk SMS throttling using Vonage in a Django REST API. You can now integrate this solution into your projects and build more solutions with Vonage. You can review our \[authentication guide] to better understand authentication with [Vonage APIs](https://developer.nexmo.com/concepts/guides/authentication). If you want to learn more about the new Messages API, you can visit the [developer documentation](<https://www.vonage.com/communications-apis/messages/developer/)>).