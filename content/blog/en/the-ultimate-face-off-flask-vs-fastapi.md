---
title: "The Ultimate Face-off: Flask vs. FastAPI"
description: When you need to choose between web frameworks, our face-off
  between FastAPI and Flask helps lead you to victory.
author: tonya-sims-1
published: true
published_at: 2021-07-30T21:31:23.996Z
updated_at: 2021-07-30T21:31:24.022Z
category: tutorial
tags:
  - python
  - flask
  - fastapi
comments: false
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
Imagine this scenario. 

You’re attending a wedding tonight and going back and forth between which outfit you’ll wear. You want to impress because your ex will be there. The only problem is that your ex is the one getting married. Ouch!

It’s kind of like choosing between two web frameworks. You’re in this vicious cycle of which one to choose, kind of like the wedding outfits. You want to impress your customers while making your competition jealous.

I feel your pain.

You may be wondering what this has to do with our Flask vs. FastAPI faceoff? 

A lot!

Choosing a framework is not easy so that’s why I’m here to help you get rid of the headache.

So, why should we even compare Flask and FastAPI?

Both are similar. They’re both Python microframeworks, which are stripped down without all the bloated bells and whistles, which means faster development time and more flexibility. Both are also used to build out APIs and web applications.

They are also different. Flask is more battle-tested therefore slightly more reliable and it’s used by many people. FastAPI is a newer, more modern framework known for its speed with lots of built-in support like Pydantic and SwaggerUI.

So now that you have a better understanding of each framework. Let our faceoff begin!

# **Installation**

Sometimes the hardest part of learning something new is actually getting started. So that’s why we’ll start with Installation.

It’s relatively straightforward to install both Flask and FastAPI using Python’s favorite installer, pip. It’s also recommended to install both inside a virtual environment, which is an isolated environment for each of your Python projects that eliminates collision errors.   

#### **Flask**

```python
$ pip install flask
```

#### FastAPI

```python
$ pip install fastapi uvicorn
```

**Conclusion**: Notice with FastAPI you install it with Uvicorn. Think of Uvicorn as a lightning-fast server that allows your applications to perform faster. 

# Hello World Application

If you’ve only written one line of code in your entire life, I bet it was something like this:

`print(“Hello World”)`

It’s kind of like if you were learning another language, let’s say Mandarin. There’s a system called Pinyin which transcribes Chinese characters to English so people can pronounce them. It’s designed to get you up and running quickly, just like a Hello World application. 

Let’s see what a hello world application looks like in both Flask and FastAPI.

#### Flask < 2.0

```python
\# inside of a Python .py file

from flask import Flask

app = Flask(\_\_name\_\_)

@app.route("/", methods=\[“GET”])
def home():
    return {"Hello": "World"}

if \_\_name\_\_ == "\_\_main\_\_":

    app.run()
```

#### Flask 2.0

```python
from flask import Flask

app = Flask(\_\_name\_\_)

@app.get("/")
def home():
    return {"Hello": "World"}

if \_\_name\_\_ == "\_\_main\_\_":

    app.run()
```

#### **FastAPI**

```python
\# inside of a Python .py file

import uvicorn

from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def home():
    return {"Hello": "World"}

if \_\_name\_\_ == "\_\_main\_\_":

    uvicorn.run("main:app")
```

**Conclusion**: In the newer versions of Flask you can use the @app.get() and @app.post() decorators as shortcuts for routing. The previous way using @app.route() you had to pass in your HTTP verbs to a methods list like so: methods=\[“GET”, “POST”]. Note in Flask it does a GET by default so you don’t need to specify it in the methods list. These methods also come with FastAPI with support for the following decorated routes for each HTTP method including:

* @app.get()
* @app.post()
* @app.put()
* @app.delete()

# Running in Development

Once you have your Hello World app written you’ll want to run it in development, or locally on your machine first before putting it out in production for the whole world to see. If your application doesn’t work as expected, people will definitely freak out. You want to minimize the freakout.

So from your terminal run in Flask and FastAPI run these commands:

#### Flask

```python
$ export FLASK_APP=app.py

$ export FLASK_ENV=development

$ flask run
```



#### FastAPI

```python
$ uvicorn main:app --reload
```



**Conclusion**: FastAPI uses Hot Reloading, which keeps the app running when you’re making code changes so you don’t have to keep restarting the development server. With Flask you have an extra terminal command export FLASK_ENV=development which allows you to make code changes without restarting your development server. 

# HTTP Methods

In the Hello World example we saw what a GET looks like in Flask and FastAPI, so now let’s take a closer look at a POST method.

#### Flask < 2.0

```python
@app.route("/teams", methods=\["POST"])
def create_team():
    team = {

        "team_name": "Phoenix Suns",
        "players": [

            {

                  "name": "Chris Paul",
                  "age": 36

            }

        ]

    }

    teams.append(team)
    return (jsonify(teams))
```



#### Flask 2.0

```python
@app.post("/teams")
def create_team():
    team = {

        "team_name": "Phoenix Suns",
        "players": [

            {

                  "name": "Chris Paul",
                  "age": 36

            }

        ]

    }

    teams.append(team)
    return (jsonify(teams))
```



#### FastAPI

```python
@app.post("/teams")
def create_team():
    team = {

        "team_name": "Phoenix Suns",
        "players": [

            {

                  "name": "Chris Paul",
                  "age": 36

            }

        ]

    }

    teams.append(team)
    return {'teams':teams}
```

**Conclusion**: Flask 2.0 and FastAPI look really similar when doing a POST method. The trick is seeing how you new data is created. With Flask you’ll have to use a tool called Postman which acts as a client so you can see your POST requests and the data you’ve created in json format. FastAPI come out of the box with something called Pydantic and SwaggerUI which allows you to use automatic documentation to interact with your requests from the browser including POST requests. Flask can also use automatic documentation but you’ll have to install it using flask-swagger and to make it work there’s lots of configuration. Let’s look at how to see your POST requests in FastAPI in the next section.

# Automatic Documentation

If you believe in magic you’ll most definitely love Automatic Documenation. FastAPI is based on Pydantic which comes out of the box so there’s no need to install it. Pydantic is a framework for easily modeling and validating objects. With Pydantic it takes the pain away because you don’t have to write constructors and you get all the magic methods. Pydantic also does Data validation which displays friendlier errors reducing debuggin time and uses python type hints. To access your automatic documentation make sure your development server is running and go to your localhost and the port in which your application is running:

    ` http://127.0.0.1:8000/docs`

You’ll see your POST request like the below example, but if you’re using other HTTP methods like GET, PUT, DELETE they’ll be visible as well.

![](https://lh4.googleusercontent.com/5faKZk_Rcq-dIjGcV6eNgcKRqfa7f5pJ-k2DMH-j0PxhJB_W3TnruP8r9oe30A6h0wyboXC3xNoXMFq-sskbcJ7aDT7rcco0Q4Q4n5b-nlPriI9PpeR4hptDmk5h90kbpSvBhUtI)

Let’s do something much more cooler so we can see the beauty of automatic documentation. Let’s say we have this code in FastAPI:

#### FastAPI

```python
from pydantic import BaseModel

app = FastAPI()

class Player(BaseModel):
    player_name: str
    player_team: str
    player_age: int

@app.post("/teams")
def create_team(request: Player):
    return {'teams':request}
```



Notice that in order to use Pydantic you have to import the BaseModel in which our Player class will inherit. We’re also declaring variables as type hints inside of our class and returning a dictionary in our POST request.

When you pull up you automatic documentation you’ll see a Schema, which is a skeleton for your model with variables where you can see which fields are required or optional.

![](https://lh4.googleusercontent.com/Ge-DOMuMRzEAZOiOLmfrNvBylmunW8j6Tq9AznT42sIJbV3LPnHwS8u7a-vF2KWUe0pF-fqpamLKAxRMhsSsipmAcP-gkyXbMxGv1FYT1fZpiC2LdvXkWSjtgppHbGUWRm55DWV4)

You can also “Try it out” and test your API endpoint by passing in values for the variables. Here for example we’re passing in “Michael Jordan” for the variable player_name of type String.

![](https://lh6.googleusercontent.com/o5zjizYBoPXJTj9UUPA7dkGWc8VQoa9sisspu0nguEhvITJNl03s9j8iEJ9QYt7b6dbOJ79WgrrgCirXUhOEHKDDMdDY7rHyBxr4ouVvvH0_ooOlOEVfym8yME9SEyxSi7mbDKKR)

Then, when you click Execute it’ll give you the Response Body or what the server will receive for the POST request. So there’s no need to use an extra tool like Postman.

![](https://lh6.googleusercontent.com/khsqTWPxA9jHnQ3hs542QAC37EZ02jtgxRzOkcy-yp-ShsOgSx4rAHk0x61SxTiNsXYECsCWW8EM0JaFYcQVgDrDGWfndZnzp-3P-Ph_Kqsg0pzmNJq6Gglb9H6fFVK-QiL4Ao7S)

Your interactive documentation will also generate a curl command for you, so you don’t have to write one from scratch: 

![](https://lh5.googleusercontent.com/2AJ3gnhujq1HwdLHHR4LulHOXlSV_sqkKPM8PW5d6hR60ggedoKx19S3Zy0BmgFdcflLXk4rIotZ3aaFgT8MORV0pzVf8TOH6kdOGywan2mYMixkGgb8eS937PiVttBI6qoln9NV)

**Conclusion**: Since Automatic Documentation comes out of the box with FastAPI along with Pydantic and Swagger UI, these features will definitely speed up your development time because you don’t have to install any external tools to test your requests. 

# Data Validation

Since our lovely friend Pydantic comes with FastAPI upon installation, it will give you some pretty friendly error messages when you run into problems with you code.

#### FastAPI

```python
from pydantic import BaseModel

from typing import Optional

class Login(BaseModel):
    username: str
    password: str
    agree_to_terms: Optional\[bool]

@app.post("/login")
def login(request: Login):
    if request.username == "janedoe" and   request.password == "password12345":
        return {"message": "Success"}
    return {"message": "Authentication Failed"}
```



Here we’re once more creating a class Login that inherits from the Pydantic BaseModel with type hinted variables inside of it. In our function we are checking if the username is janedoe and the password is passworld12345 then it should return a success message otherwise a failure message will appear. 

We turn to automatic documentation and test our request body by passing in None to the username:

![](https://lh6.googleusercontent.com/KzV7vaKxI1fVMjzMnPTduBWhE6CXIrEwDQjXtaX7eUOtuPULKMf6DOWpr2CYjFujrspQ9OYI3cXM0nVQ4MC9v5hFQ8990CdoSsWbRdlQsXuPi2odFrhi_X0RG6q7neoyDw4_thy4)

Pydantic will work it’s magic and you’lll get a friendly message telling you exactly what the error is. In this case, it returns error *Expecting Value*, which is right on the money because we passed in None to the username.

![](https://lh6.googleusercontent.com/9biK0m7fzVbqdvTtw64PfK6cQRqK-HrQMVbW7gd0-3rDiYz69qD_35JtuUol7qt1ME3MFkJ9SH_fo3rR3R8XETkHhmYTpCuQ2-Vsq1J7ykC4f8Ex_85TNvz2YCWnt1WsmCCxiY53)

**Conclusion**: Flask does not have any inhouse data validation support. You can use the powerful Pydantic package for data validation by installing it with *Flask-Pydantic*.

# URL or Path Parameters

A path or URL parameter fetches one single item. Let’s say we want to get a single player. Whichever player has an id of what we pass into the URL, will get returned to the user. 

Let’s say we have a list of dictionaries and we want to get one player from this JSON file:

```python
players = [

    {

        "player_id": 1,
        "name": "Giannis"

    },

    {

        "player_id": 2,
        "name": "Luka"

    }

]
```

#### Flask

```python
@app.get('/players/<int:player_id>')

def get_player_details(player_id):
    for player in players:
        if player\["player_id"] == player_id:
            return jsonify(player)
```



Here we pass in our route to localhost on port 5000 with an id of 2 and we we get back the player with an id of 2.

![](https://lh4.googleusercontent.com/MMQ37ZRuC40c3ZnIvqv9XuN98ji0J0vAGlG4UtFE_JroLK9z1ZBG7vYBuqHCh7QFW7FqD5fsJSdSyMWRtF7kayerX-AG1M38bG3cRTm3KkctJ8EciBwwbcf3IzAucZ6EoY8B1QmE)

#### FastAPI

```python
@app.get("/player/{player_id}")

def get_player_details(player_id: int):
    for player in players:
        if player\['player_id'] == player_id:
            return {'player':player\['name']}
```

Here we pass in our route to localhost on port 8000 with an id of 1 and we get back the player with an id of 1.

![](https://lh3.googleusercontent.com/sdlIDlSJyDGCQt5Ps8sFfWNpny1uc5BEASoLFdOtOdnks6IqF0NGeEMbuc9NgUAPwUPbcUw2WgEjKPrfIVQO50i-H8PcT4IJSSpPFD_JRcip95bGKds8eD3C0OKusU2LVV0Xe9xA)

Conclusion: With FastAPi since it’s using Python type hinting you can port your code to other frameworks, like Django. With Flask it’s not portable because we’re using framework-specific type hinting, not Python hinting.

Templates Folder

The Templates Folder stores your HTML files when you’re building a web application in Flask or FastAPI and you have to use Jinja to display your variables in HTML. Jinja is a templating engine that allows you to write code similar to Python to display HTML. 

#### Flask

By default, Flask looks for templates in a "templates" folder so you’ll just create one in your file structure. 

![](https://lh6.googleusercontent.com/5x9KJw2aUIxEn8gglrQ88QEFfOj0HwIGhEHJuYSVsudt8HJKfOf42v1UmFlWfYxIif66pAY1su7Zu0EbMRjU-XmT25Ou-F_05NN6oGf9ac0xVmLxxnvYqt632M65fktbEI-GupPU)

![](https://lh5.googleusercontent.com/IQfD9C2YbAz2rdG6bJVj9N1dLkgAst75jOD2xBGPW9_CKI0bhqTpdGp-j1RPbpd0xKBmLoyP6d2EoHr5tAnPB9Lvn5FJPFsOXTE3ghT9vkg4uB7M8WxbjK7o5xXLTSwwKi_UntP2)

Then you can use Jinja to display your variables by surrounding them with double curly braces:

```html
<!DOCTYPE html>

<html lang="en">

<head>

    <meta charset="UTF-8">

    <meta http-equiv="X-UA-Compatible" content="IE=edge">

    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>Players</title>

</head>

<body>

    <h1>Display Players</h1>

    {{ player.name }}

    {{ player.jersey_number }}

</body>

</html>
```

**Conclusion**: Jijnja comes with Flask when it’s installed which is a huge plus. In FastAPi, you have to install Jinja and define the templates folder in your code.

# Production Server

At some point, you’ll want to deploy your application and show it to the world.

#### Flask

Flask uses a web server called WSGI, which stands for Web Server Gateway Interface and has been the Python standard for many years. The drawback is that it’s synchronous. This means if you have a bunch of requests, they have to wait in line for the others before it to complete.

#### FastAPI

FastAPI uses a web server called ASGI or Asynchronous Server Gateway Interface, which is lighting fast because it’s well, you guessed it Asynchronous. This means if you have a bunch of requests coming in, they don’t have to wait for the other ones to complete or finish before they start processing. 

Conclusion: ASGI makes for faster performance in your web applications because of the way they process requests asynchronously

Drumroll please.

The winner is...

This is how you can choose.

**Use Flask if you want:**

* A battle tested framework, as it’s been around for a long time
* To develop a quick prototype
* To do web application development

**Use FastAPI if you want:**

* Speed as in development time and performance
* To decrease the number of bugs and errors in your code
* To build APIs from scratch

Ok, so you’ve seen both Flask and FastAPI in action and now you have a better understanding of both and which one you’ll choose in your next project. You’ve chosen your outfit for the wedding and no longer care about making your ex jealous. 

So which framework did you choose? Tweet us @VonageDev or @tonyasims.