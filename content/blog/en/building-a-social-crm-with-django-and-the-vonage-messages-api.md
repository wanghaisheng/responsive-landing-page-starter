---
title: Building a Social CRM with Django and the Vonage Messages API
description: Learn how to build a social CRM using Django and the Vonage Messages API
author: cory-althoff
published: true
published_at: 2022-01-04T20:12:13.200Z
updated_at: 2022-01-04T20:12:13.243Z
category: tutorial
tags:
  - messages-api
  - django
  - CRM
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
One of the most used tools in the business world today is Customer Relationship Management (CRM) applications. Every business needs to connect and communicate with its customers in the best way possible. Social media is one of the most effective mediums for businesses to reach their customers conveniently.

In this article, you will learn how to build the core feature of a social CRM using Django and the Vonage Messages API. Our social CRM will help sales agents and the customer support team communicate with potential customers directly on Facebook in real-time. Let's call it Sales Fox.

## Pre-requisites

1. You need a Vonage account. If you do not have one yet, Go to the [Vonage Sign-up](<1. https://dashboard.nexmo.com/sign-up>) page to create an account. Then, come back here to continue with the next steps.
2. Now, login to your [Vonage accoun](https://dashboard.nexmo.com/sign-in)t. Click on \`Your applications\` in the menu tab and select \`Create a new application\`. (picture-1)
3. Fill in the form provided to create a new application. Under the capabilities section, turn on the **Messages** capabilities.
4. You will find two fields in Messages capabilities: the inbound URL field and the status URL field. These URLs are endpoints to web-hooks that help the Vonage Messages API communicate with your application. For now, Fill in a dummy URL or maybe the link to your GitHub account. (picture-2)
5. The next thing is to authorize Vonage to access your Facebook business page. You can create a demo business Facebook page for the sake of this tutorial. Go to [Connect Facebook Page](<1. https://dashboard.nexmo.com/messages/social-channels/facebook-connect>) and follow the steps outlined on the page.
6. Afterward, Go to your application page and link your already authorized social media page to the application you created in Step 3. (picture-3)
7. Install Redis - If you're using Linux or Mac, follow the instructions [here](<1. https://redis.io/topics/quickstart#installing-redis>). If you're using Windows, follow the instructions [here](<1. https://redis.com/blog/redis-on-windows-10/>).
8. Install Ngrok. Ngrok provides you with a publicly available URL that forward requests to your locally running application. Go to the [Ngrok download page](https://ngrok.com/download) and follow the instructions to set Ngrok up on your computer.

Now that you have the pre-requisites completed. You need to set up your development environment for the tutorial. You need Python >= 3.6, Django 3. Also, I recommend using Visual Studio as your IDE.

## Project Set-up

### 1. Create and Activate Your Virtual Environment

Create a directory for your project if you haven't done that already. Change your working directory to the directory you just created. Then, run the following commands to create and activate a virtual environment for your project.

```
python3 -m venv sales-env
source sales-env/bin/activate
```

### 2. Install the Required Packages

It would be time-consuming to install our packages one after the other because we have a lot of them. Hence, copy all the packages we need in a .txt file and install everything with one command. Create a \`requirements.txt\` file in the directory created in step 1. Copy and paste the code snippet below in your \`requirements.txt\` file.

```
    aioredis==1.3.1
	asgiref==3.3.4
	async-timeout==3.0.1
	attrs==21.2.0
	autobahn==21.3.1
	Automat==20.2.0
	certifi==2021.10.8
	cffi==1.14.6
	channels==2.4.0
	channels-redis==2.4.2
	charset-normalizer==2.0.7
	constantly==15.1.0
	cryptography==3.4.7
	daphne==2.5.0
	Django==3.2.2
	djangorestframework==3.12.4
	hiredis==2.0.0
	hyperlink==21.0.0
	idna==3.2
	incremental==21.3.0
	msgpack==0.6.2
	Pillow==8.2.0
	pyasn1==0.4.8
	pyasn1-modules==0.2.8
	pycparser==2.20
	pyOpenSSL==20.0.1
	python-dotenv==0.19.2
	pytz==2021.1
	requests==2.26.0
	service-identity==21.1.0
	six==1.16.0
	sqlparse==0.4.1
	Twisted==21.7.0
	txaio==21.2.1
	typing-extensions==3.10.0.0
	urllib3==1.26.7
	zope.interface==5.4.0
```

```
Now, install all the packages in `requirements.txt` by running the command below in your terminal.

`pip install -r requirements.txt`
```

### 3. Create your Django project

Create your Django project for Sales Fox with the django-admin command. Remember that we already installed Django in requirements.txt. It means the "django-admin" command is available in our virtual environment. Run the following to create project `sales_fox`

```
	`django-admin startproject sales_fox`
```

We will create two apps in our `sales_fox`: The `lead_manager` app to manage leads and the `conversation` app for sales agents to communicate with potential customers (known as leads). Our focus will be on the `conversation` app. Now, let's create our two apps by running these commands.

```
	`python manage.py startapp lead_manager`

	`python manage.py startapp conversation`
```

Take note that in this tutorial,

* I'll be using the words - "leads" and "customers" interchangeably. Leads are potential customers, so it won't hurt to regard them as customers where convenient.
* I will use the term `Project Directory` to refer to the directory where you have `settings.py`. This directory is created when you ran `django-admin startproject sales_fox`.
* I will use the term `Overall Directory` to refer to the directory you created at the beginning of the tutorial. It contains your virtual environment folder, the app directories, and your project directory

### 4. Let's get SalesFox ready to use Vonage.

* Create a `.env` file in your overall directory. Define `FACEBOOK_ID`, `VONAGE_API_KEY`, and `VONAGE_API_SECRET`. Your .env file should look like this:
  		`FACEBOOK_ID=YOUR-LINKED-FACEBOOK-ID
  		VONAGE_API_KEY=YOUR-VONAGE-API-KEY
  		VONAGE_API_SECRET=YOUR-VONAGE-API-SECRET`

  ```
  	You can find your Vonage API key and API secret in your [Vonage settings page](https://dashboard.nexmo.com/settings).

  	And your Facebook ID can be found in the `Link social channels` tab on your application page.
  ```

  In your project directory, Go to `settings.py`, load the variables in your .env file using `python-dotenv` installed from `requirements.txt`. Add the following snippet in `settings.py` to load the .env file:

  ```
  from  dotenv  import  load_dotenv	
  import  os
  load_dotenv()
  ```

  load_dotenv loads all variables in our .env file as environment variable.

Now, define `FACEBOOK_ID`, `VONAGE_API_KEY`, `VONAGE_API_SECRET`, `VONAGE_MESSAGES_ENDPOINT` in your `settings.py` file. Simply copy and paste the snippet below. 

````
	 ```
	FACEBOOK_ID = os.getenv("FACEBOOK_ID")
 
	VONAGE_API_KEY = os.getenv("VONAGE_API_KEY")
	VONAGE_API_SECRET = os.getenv("VONAGE_API_SECRET")
	VONAGE_MESSAGES_ENDPOINT = "https://api.nexmo.com/v0.1/messages"
	```
````