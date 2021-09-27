---
title: The How To Guide -  Sending an SMS With Python, FastAPI and Vonage
description: This is a how to guide on sending an SMS with Python, FastAPI and
  Vonage. Buckle up and come on a fun adventure to learn how to send a text
  message that could save your life!
author: tonya-sims-1
published: false
published_at: 2021-09-27T00:50:14.818Z
updated_at: 2021-09-27T00:50:14.858Z
category: tutorial
tags:
  - python
  - fastapi
  - sms
comments: false
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
Imagine you’re stranded on an island with only your computer and WiFi. The ocean waters are starting to creep up and you’re worried you’ll be underwater if you don’t act quickly. 

The only way out is for you to quickly build a Python website and send a text message to someone. Luckily, you’re a coder and have recently been tinkering with the new FastAPI web framework. You’ve also been trying out the Vonage Messages API to send SMS messages. 

Equipped with all this new knowledge, you whip out your laptop to get started. 

## Using the Vonage CLI

The first thing you do is install the new **[Vonage CLI ](https://learn.vonage.com/blog/2021/09/21/vonage-cli-is-v1-0-0/)**which will allow you to quickly create the dashboard application you need to interact with the Messages API. From your Mac terminal, you run these commands and talk yourself through them during the installation:

You have NodeJS and npm installed so the first command should work depending on whether or not your $PATH is set properly. 

```shell
% npm install -g @vonage/cli
```

Bam! That worked! You now have the Vonage CLI installed on your machine. 

You want to make sure the installation was successful so you type:

```shell
% vonage
```

![Vonage CLI](/content/blog/the-how-to-guide-sending-an-sms-with-python-fastapi-and-vonage/cli_1.png)

Next, you go to the [dashboard](https://dashboard.nexmo.com/) to grab your API Key and API Secret. You’ve already registered so all you have to do is sign in. 

Then you set your keys by doing the following:

```shell
% vonage config:set --apiKey=12345 --apiSecret=abcde
```

Success.

You got this but in case you forget something you can always turn to the help flag:

```shell
% vonage --help
```

Now for the fun part. You have to create your application so you run this command:

```shell
% vonage apps:create
```

You give it an **Application Name** of **send sms** and press Return.

![](/content/blog/the-how-to-guide-sending-an-sms-with-python-fastapi-and-vonage/cli_2.png)

Then under the option **Select App Capabilities** you choose **Messages**.

![](/content/blog/the-how-to-guide-sending-an-sms-with-python-fastapi-and-vonage/cli_3.png)

Now you create your webhooks: inbound and status. 

You choose **“Y”** for **Create Message Webhooks**.

![](/content/blog/the-how-to-guide-sending-an-sms-with-python-fastapi-and-vonage/cli_4.png)

For the Webhook URLs, you use the examples from the [Vonage documentation](https://developer.nexmo.com/messages/code-snippets/configure-webhooks) but understand these are configurable and can be tested with [ngrok](https://developer.nexmo.com/tools/ngrok).

You proceed.

For the **Inbound Message Webhook,** you provide this URL: <https://www.example.com/webhooks/inbound-message>. In the next option select **POST** for the **Inbound Message Method**.

![](/content/blog/the-how-to-guide-sending-an-sms-with-python-fastapi-and-vonage/cli_5.png)

For the **Status Message Webhook**, you give it this URL: <https://www.example.com/webhooks/message-status> and also select **POST** for the **Status Message Method**.

![](/content/blog/the-how-to-guide-sending-an-sms-with-python-fastapi-and-vonage/cli_6.png)

The next prompt asks you if you want to **Allow use of data for AI training? Read data collection disclosure** and you say **“No”**.

![](/content/blog/the-how-to-guide-sending-an-sms-with-python-fastapi-and-vonage/cli_7.png)

Your application has been created and you’re pretty darn happy about that. You’ll be able to binge-watch Loki in no time.

![](/content/blog/the-how-to-guide-sending-an-sms-with-python-fastapi-and-vonage/cli_8.png)

You go back to the [dashboard](https://dashboard.nexmo.com/) because you want to check if your application was created. You click on **Your Applications** and you see it. 

![](/content/blog/the-how-to-guide-sending-an-sms-with-python-fastapi-and-vonage/cli_9.png)

You’d like to check if the Messages API option is toggled on and if your webhooks made it, so you select to edit. 

![](/content/blog/the-how-to-guide-sending-an-sms-with-python-fastapi-and-vonage/cli_10.png)

![](/content/blog/the-how-to-guide-sending-an-sms-with-python-fastapi-and-vonage/cli_11.png)

Everything looks great! You also realize that you don’t need to do this step every time as it’s only a sanity check.

The water is rising past your knees and your biggest fear is you’ll be left alone on this island with a volleyball.

You hated that movie. 

Now it’s time for you to write your FastAPI application so you can send your SMS.

What you’ve learned about FastAPI so far is that it offers an amazing developer experience and speeds up coding time. It’s also super fast in performance because of its asynchronous nature. 

Perfect.

## Creating the Python Virtual Environment

The first thing you do is navigate to or **cd** into the directory where you want to create your Python project. 

Then you create a new folder called **send_sms** by running this command from your directory:

```shell
% mkdir send_sms
```

You change into that directory by doing this:

```shell
% cd send_sms
```

You think to yourself that this would be a good time to create a virtual environment, so you run:

```shell
% python3 -m venv venv
```

To verify this new virtual environment was created you should see one called “venv”, so you type:

```shell
% ls
```

Walla! There it is.

Now it’s time to activate it so you can install FastAPI and your other packages, so you do:

```shell
% source venv/bin/activate
```

You see that **(venv)** is at the beginning of your user in your terminal, so you know it’s been activated. 

Gosh, the sun is beating down on your face and there’s a glare on your computer screen. You can barely see anything and wish you had your sunglasses. You remembered that right before leaving home and getting stranded, your dog ate them! 

## Installing FastAPI

Ok, now it’s time for you to install FastAPI.

You remembered the last time you installed it and had to upgrade pip first like this:

```shell
% pip install --upgrade pip
```

Then because you’re using **zsh** you installed FastAPI like so:

```shell
% pip install 'fastapi[all]'
```

When you used bash you’d install it without the **‘ ‘** around fastapi\[all] like this:

```shell
% pip install fastapi[all]
```

Anyways, you now have FastAPI installed and are thrilled you installed it with the **\[all]** option because it gives you all the dependencies and features like async-generators, the requests module, JSON, Jinja2 for templating in HTML, Pydantic, etc.

## FastAPI Hello World

You want to get a Hello World example up and running fast, so you can test if your install worked. You create a main.py file in your project directory.

##### **main.py**

```python
from fastapi import FastAPI

app = FastAPI() 

@app.get("/")
async def home(): 
  return {"hello": "world"}
```

The line **`from fastapi import FastAPI`** just imports FastAPI as a Python class

**`app = FastAPI()`** creates an instance of FastAPI called app.

Here **`@app.get("/")`** you create a route operation. Route refers to the URL or where you want to direct yourself when you hit your endpoint. In your case, you're going to the root of the page or localhost (<http://127.0.0.1:8000>). The operation refers to the HTTP method, in this case, you’re doing a GET operation using the decorator **`@app.get`**, which tells users to read the data and go to the route, which is (“/”) or the root page.

This is an async function **`async def home():`**. They allow you to have asynchronous functions which can process other requests before others have been completed, so they run in parallel, which is pretty sweet because it makes things much faster than running synchronously, or in order. You can also define a normal function here with just: **`def home():`** if you don’t care about asynchronous code.

This line `return {"hello": "world"}` returns a dictionary to the browser.

Now you run your code in development mode by doing the following:

```shell
% uvicorn main:app --reload
```

You think of uvicorn as being a super fast ASGI (Asynchronous Server Gateway Interface) server implementation. In **main:app**, main is the name of your file **main.py** and **app** is the name of your FastAPI instance from above. The **\--reload** flag using hot reloading which allows you to make live code changes, so you don’t have to keep restarting your server every time.

You see in the terminal you need to go to your localhost <http://127.0.0.1:8000/> in the browser and you see **{"hello": "world"}**. Perfect!

## FastAPI Hello World

Now it’s time to write your code to send your SMS. 

The sky is dark and the wind is heavy. A tornado is brewing.

You need to hurry!

In your main.py you replace your Hello World code with this:

```python
from fastapi import FastAPI, Request
from fastapi.templating import Jinja2Templates
from fastapi.responses import HTMLResponse

app = FastAPI()

templates = Jinja2Templates(directory="templates")

@app.get("/", response_class=HTMLResponse)
async def get_message(request: Request):
  return templates.TemplateResponse("index.html", {"request": request})
```

Here you are also importing Request **`from fastapi import FastAPI, Request.`** Request allows you to get incoming details or requests to your function.

Import Jinja so you can use its templating engine **`from fastapi.templating import Jinja2Templates`**.

In this line, **`from fastapi.responses import HTMLResponse`** we need to allow you to get back an HTMLResponse.

Here you mount the templates folder (you’ll create one in a bit) and tell it to hold all your HTML files in a directory called **templates**. Your line of code looks like this: **`templates = Jinja2Templates(directory="templates")`**

In the route decorator **`@app.get("/", response_class=HTMLResponse)`**. The HTMLResponse indicates that the response you get back contains HTML.

This is another async function `async def get_message(request: Request):`. Declaring a route operation function with a parameter of type Request will tell your application to pass the Request in that parameter.

Lastly, **`return templates.TemplateResponse("index.html", {"request": request})`** renders your template or your response. It takes as arguments the HTML file (index.html) and the context, which keeps track of the data we get from our request

You just created the route that does the **GET** operation. You’ll also have to create a **POST** because you’re submitted data to a form. But before that, you decide to create the “templates” folder in your project directory to hold your HTML files. You create two HTML files inside of templates: index.html and sent_sms.html. 

Then in your **index.html** you include this markup:

```html
<!DOCTYPE html>

<html lang="en">
<head>
   <meta charset="UTF-8">
   <title>Title</title>
</head>
<body>
<h1>Send a Text Message</h1>
  <form action="/sent_sms" method="POST" novalidate>
      <input type="text" placeholder="Enter number to text" name="to_number">
      <button type=submit">Send Text</button>
  </form>
</body>
</html>
```

This line is crucial: `<form action="/sent_sms" method="POST" novalidate>`. The **method** attribute tells you how to send form data, in this case, you’re doing a POST. The **`action`** attribute specifies which page to send the form data to. You’re sending it to “sent_sms.html”. Note that the POST method here does not display the data in the URL like a GET would, instead it appends the data inside the body of the HTTP request.

Here  `<input type="text" placeholder="Enter number to text" name="to_number" >` you define an input element of type text and give it some placeholder text which will display inside of the textbox. Next, you provide a name attribute called **to_number** which specifies the name of the input element. This will be important later on when you reference this attribute to get the number in which you’re sending the SMS.

This line`<button type=submit">Send Text</button>` we define a button with **`type=”submit”`**, so when the user clicks the button, the text will be sent. 

Next, you code up the page since it’s very simple.

```html
<!DOCTYPE html>

<html lang="en">
<head>
   <meta charset="UTF-8">
   <title>Title</title>
</head>
<body>
  <h1>Send a Text Message</h1>
  <h3>Thank you {{ number }}!</h3>
</body>
</html>
```

If the SMS is sent successfully, you will see this page. The only sort of tricky thing here is this: **`{{ number }}`**. But it’s not that tricky at all! This is the Jinja language and it’s pulling in the phone number you put in the form. The number you want to send the SMS to. You’re about to write the POST route and will see how it works.

You’re feeling pretty good now because you’re in the home stretch. But it’s raining now, and you’re worried your laptop is going to get messed up. So your fingers get to coding. 

You continue in the **main.py** file by adding this POST method:

```python
from fastapi import FastAPI, Request, Form
from base64 import b64encode
import requests
import json

@app.post("/send_sms", response_class=HTMLResponse)
async def send_message(request: Request, to_number: str = Form(...)):

  payload = {

      "to": {
          "type": "sms",
          "number": to_number
      },

      "from": {
          "type": "sms",
          "number": [YOUR_VONAGE_NUMBER]
      },

      "message": {
          "content": {
              "type": "text",
              "text": "Help me! I need to watch Loki!"
          }
      }
  }

key = 'abcde'
secret = '12345'
encoded_credentials = b64encode(bytes(f'{key}:{secret}',

                                     encoding='ascii')).decode('ascii')

auth_header = f'Basic {encoded_credentials}'
headers = {"content-type": "application/json", "Authorization": auth_header}
response = requests.post("https://api.nexmo.com/v0.1/messages",
                        auth=(key, secret),
                        headers=headers,
                        data=json.dumps(payload))

return templates.TemplateResponse("send.html", {"request": request, "number": to_number})
```

Here **`from fastapi import FastAPI, Request, Form`** you're importing **`Form`** , which you need to receive the form field data

You need this line, `from base64 import b64encode` to encode the API key and API secret. 

You import requests so you can send HTTP requests and import json because you have to do some things with JSON.

This line should look a little familiar `@app.post("/send_sms", response_class=HTMLResponse)`. Here you have an **`@app.post`** route operation and pass in an HTML Response. 

You have your async function again **`async def send_message(request: Request, to_number: str = Form(...)):`** This time you define form parameters as a type hint and read the form in by doing `Form(...)`.

This is the payload or body of data you’ll send to your request:

```python
 payload = {

      "to": {
          "type": "sms",
          "number": to_number
      },
      "from": {
          "type": "sms",
          "number": \[YOUR_VONAGE_NUMBER]
      },
      "message": {
          "content": {
              "type": "text",
              "text": "Help me! I need to watch Loki!"
          }
      }
  }
```

A few things to note about this key/value pair in the payload: **`"number": to_number`**.  to_number is the same value as in our index.html with the name attribute set to **to_number.** To use it you’ll have to use its key: number.

```python
 "to": {
          "type": "sms",
          "number": to_number
      },
```

Another thing to notice in the payload is the **number: \[YOUR_VONAGE_NUMBER]**, which will be your [Vonage phone number you buy here](https://dashboard.nexmo.com/buy-numbers).

```python
"from": {
          "type": "sms",
          "number": \[YOUR_VONAGE_NUMBER]
      },
```

Lastly, in the payload you’ll leave the type set to text like this **"type": "text"** and provide a message for your text like so **"text": "Help me! I need to watch Loki!"**.

```python
"message": {
          "content": {
              "type": "text",
              "text": "Help me! I need to watch Loki!"
          }
```

Next, you define the headers for the request which indicates that the body request format is JSON:

```python
headers = {"content-type": "application/json"}
```

You pause here for a second and remember for the next line of code you’ll need to use authentication. You can choose between using a [JWT or Basic Authentication](https://developer.nexmo.com/concepts/guides/authentication#header-based-api-key-and-secret-authentication) and you choose the latter. 

You store your API key and secret in these variables:

```python
key = 'abcde'
secret = '12345'
```

You then create a variable called encoded_credentials and proceed to do the Base64 encoding by using an f-string and passing in your key and secret.

```python
encoded_credentials = b64encode(bytes(f'{key}:{secret}',
                                     encoding='ascii')).decode('ascii')
```

Here you create your authorization header which is a key/value pair including your Base64 encoded username and password. This pair authenticates your requests and allows you to access the API. 

```python
auth_header = f'Basic {encoded_credentials}'
```

Next, you pass in the authorization header:

```python
headers = {"content-type": "application/json", "Authorization": auth_header}
```

Now the fun part! Here you are using the requests module and sending a post request requests.post to the Vonage API. You pass in the API URL (<https://api.nexmo.com/v0.1/messages>) and use HTTP Basic Auth from the requests module. The auth keyword provides a shortcut and allows you to do [Basic Authentication](https://docs.python-requests.org/en/latest/user/authentication/#basic-authentication). Then you pass in your headers **`headers=headers`**. Lastly, you pass in the request body and convert the Python dictionary object into a JSON string **`data=json.dumps(payload)`**.

```python
response = requests.post("https://api.nexmo.com/v0.1/messages",
                        auth=(key, secret),
                        headers=headers,
                        data=json.dumps(payload))
```

The last step is to render the template and pass in that will be shown when the SMS is successful (send.html), the request, and the context. The context **`"number": to_number`** will display the number on send.html.

```python
return templates.TemplateResponse("send.html", {"request": request, "number": to_number})
```

Ok. Here’s do or die time. 

You start your server:

% uvicorn main:app --reload

You navigate to your localhost <http://127.0.0.1:8000/>

You see where to enter a phone number to send an SMS to your friend, and you do so.

![Test](/content/blog/the-how-to-guide-sending-an-sms-with-python-fastapi-and-vonage/sms_form_10.png)

You’re super nervous about if they’ll receive the message.

Great news! They got the text message!

![](/content/blog/the-how-to-guide-sending-an-sms-with-python-fastapi-and-vonage/img-7473.jpg)

![](/content/blog/the-how-to-guide-sending-an-sms-with-python-fastapi-and-vonage/screen-shot-2021-09-26-at-9.17.22-pm.png)

You see a boat approaching and realize that it’s for you. 

You hop on and they bring you to safety.

Later that evening you lay in bed watching Loki, thinking to yourself: Thank goodness for Python.

The end.