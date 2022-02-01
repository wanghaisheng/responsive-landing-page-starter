---
title: "Create an emergency broadcast system with Vonage, Node, and MongoDB "
description: "Create an emergency broadcast system with Vonage, Node, and MongoDB "
author: cory-althoff
published: true
published_at: 2022-02-01T00:31:09.675Z
updated_at: 2022-02-01T00:31:09.718Z
category: tutorial
tags:
  - messages-api
  - node
  - sms
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
It is hard to get anything right during an emergency, especially if you are panicking! If you are in an emergency situation, something as simple as letting the right person know you need help may prove impossible. Today, we are going to solve this by building a web app that allows you to click a single button to notify your family that you are in the middle of an emergency via text, and also send your location. We will create this website using Node.js, Express, MongoDB, and the Vonage Messages API. Node is a popular web server and express is a framework that developers often use with it. MongoDB is a NoSQL database, and the Vonage Messages API lets you quickly send messages (such as SMS messages) programmatically. 

Your web app will look like this.

![](/content/blog/screen-shot-2022-01-31-at-3.25.22-pm.png)

You can add contacts, see them, and when you press ALERT it sends an SMS message to all of your contacts, letting them know your location and that you are in an emergency. 

Ready to get started? Let’s build an emergency broadcast app!
Prerequisites

### Prerequisites

To follow along with this tutorial, you need Node (if you don’t have it you can download it here). You also need a Vonage account, which you can register for here. You will also need a https://account.mongodb.com/account/register.

After you register for your MongoDB account, you need to set up a database, which you can learn to do in MongoDB’s documentation.  

Once you’ve created your Vonage account, you need to do some initial setup.  

```
Install the Vonage CLI globally with this command:
npm install @vonage/cli -g
```

Next, configure the CLI with your Vonage API key and secret. You can find this information in the Developer Dashboard.

```
vonage config:set --apiKey=VONAGE_API_KEY --apiSecret=VONAGE_API_SECRET
```

Create a new directory for your project and CD into it:

```
mkdir my_project
CD my_project
```

Now, use the CLI to create a Vonage application with this command:


```
vonage apps:create
```

Scroll to SMS, click the spacebar, and it hit enter. 
Now you need a number so you can receive calls. You can rent one by using the following command (replacing the country code with your code). For example, if you are in the USA, replace GB with US:

```
vonage numbers:search US
vonage numbers:buy \[NUMBER] \[COUNTRYCODE]
```

Now link the number to your app:

```
vonage apps:link --number=VONAGE_NUMBER APP_ID
```

You can find all the code for this project in [this GitHub repository](https://github.com/calthoff). 

### Building the Broadcast System

To get started, you need to install the JavaScript libraries you will use for this project like this: 


```
npm install ​​express body-parser dotenv firebase mongodb mongoose nexmo@beta
```

Now, create a file called `app.js` and import these libraries:

```
require('dotenv/config')
require('mongodb')
const express = require('express')
const Vonage = require('@vonage/server-sdk')
const bodyParser = require('body-parser')
const mongoose = require('mongoose') 
```

Let’s start by creating a `"Hello, World!"` Express app. Here is how: 

```
const app = express()
const port = 3000

app.get('/', (req, res) => {
  res.send('Hello World!')
})

app.listen(port, () => {
  console.log(`Example app listening on port ${port}`)
}) 
```

When you run this code on your local server on port 3000 (http://127.0.0.1:3000), and go to the homepage, it should say `Hello, World! `

Now, add this code underneath` const app = express()`:

```
app.use( bodyParser.json() );
app.use(bodyParser.urlencoded({ extended: true }));
app.use(express.static("public"));
```

This helps you accept POST requests using Express and lets Express know to serve static files from a directory called `public`. 

It is time to add code to connect to your MongoDB database! Add this code to your app but replace `mongodb+srv://…` with the link to your MongoDB database. 

```
mongoose.connect("mongodb+srv://…", { useNewUrlParser: true, useUnifiedTopology: true });
const contactsSchema = new mongoose.Schema({
   name: String,
   number: Number
});
const Contacts = mongoose.model('Contacts', contactsSchema);
```

The code above uses mongoose to connect to your MongoDB database, creates a schema for your contacts (the people you will send an SMS message to), and creates a new model called `Contacts` you will use to add and get the contacts from your database. 

Next, add this code to help send SMS messages using the Vonage messages API.

```
const vonage = new Vonage({
   apiKey: process.env.API_KEY,
   apiSecret: process.env.API_SECRET,
   applicationId: process.env.APPLICATION_ID,
   privateKey: process.env.PRIVATE_KEY
})
```

The code above creates a new Vonage object and passes in your API key, secret, application ID, and the private key from your app. It gets this information from a .env file in your project, so create a `.env` file and add it to your project, and add the following variables:

```
API_KEY=your_vonage_api_key
API_SECRET=your_vonage_secret
APPLICATION_ID=your_vonage_application_id
PRIVATE_KEY=your_vonage_private_key
FROM_NUMBER=your_vonage_number
PORT=5000
```

Make sure to replace everything after each equals sign for each variable with the information from your Vonage account. You can now replace the line of code `const port = 3000` from earlier with  `process.env.PORT`, so your app gets the port from an environment variable (instead of hard coding it). 

Now, let’s create a few endpoints. Add the following code to `app.js`: 

```
app.post('/contacts',  function (req, res) {
   const contact = new Contacts({ name: req.body.name});
   contact.save();
   res.redirect('/')
})
```

Now, when you send a POST request to `/contacts`, this code creates a new contact in your MongoDB and redirects the user to the homepage. 

Let’s create an endpoint to get all of the contacts from your database. Add the following code to `app.js`: 

```
app.get('/contacts', function(req, res){
   Contacts.find({}, function(err, contacts){
       if(err){
           console.log(err);
       }
       else {
           res.json(contacts);
       }
   });
})
```

This code gets all of the contacts in your database and returns it as JSON when you send a GET request to /contacts. 

Now, let’s define an endpoint to send an SMS message. Add this code to app.js: 

app.post('/alert', function(req, res){
   let long = req.body\['coordinates']
   let lat = req.body\['coordinates']
   let contacts = req.body\['contacts']
   for (let i = 0; i <= contacts.length; i++) {
       vonage.channel.send(
           { "type": "sms", "number": contacts\[i].number},
           { "type": "sms", "number": process.env.FROM_NUMBER},
           {
               "content": {
                   "type": "text",
                   "text": `SOS! Your friend is in an emergency! Their latitude is ${lat} and` +
                       `their longitude is ${long}!`
               }
           },
           (err, data) => {
               if (err) {
                   console.error(err);
               } else {
                   console.log(data.message_uuid);
               }
           }
       );
   }
})

This endpoint accepts a POST request with JSON containing the user’s latitude, longitude, and a list of numbers to send an SMS message to. 
let long = req.body\['coordinates']
let lat = req.body\['coordinates']
let contacts = req.body\['contacts']

Then, it loops through the contacts and uses the Vonage Messages API to send a message to each number. 
   for (let i = 0; i <= contacts.length; i++) {
       vonage.channel.send(
           { "type": "sms", "number": contacts\[i].number},
           { "type": "sms", "number": process.env.FROM_NUMBER},
           {
               "content": {
                   "type": "text",
                   "text": `SOS! Your friend is in an emergency! Their latitude is ${lat} and` +
                       `their longitude is ${long}!`
               }
           },

Finally, let’s update our hompeage endpoint to handle when users go to our web app’s homepage. Change this code from earlier:

app.get('/', (req, res) => {
  res.send('Hello World!')
})

To this:

app.get('/', function(req, res){
   res.sendFile('index.html')
})

Now, your homepage will serve the file index.html, which you are about to create. 

Go ahead and create a new file called index.html and add the following code: 

<!DOCTYPE html>

<html lang="en">
<head>
   <meta charset="UTF-8">
   <title>Emergency Broadcast</title>
   <link rel="stylesheet" href="style.css">
</head>
<body>
   <h1 id="top">Add Contact</h1>
   <form action="/contacts" method="POST">
       <div>
           <input id="name" class="forms" type="text" name="name" placeholder="name"> <br>
           <input id="number" class="forms" type="text" name="number" placeholder="number">
       </div>
       <div>
           <input type="submit" value="Add" class="tons" id="add">
       </div>
   </form>
   <br> <br>
   <h1>Your Contacts</h1>
   <div></div>
   <div id="contacts"></div>
   <button onclick="alert_them()" class="tons" id="alert">ALERT</button>

The HTML above creates a form that allows you to enter a person’s name and number and press an “Add” button. When you press “Add” the form sends a POST request to /contacts, which creates a new contact in your database. Your HTML also has an ALERT button. When you press it, it sends a POST request to /alert, which sends an SMS message to all of the contacts in your database. 

Finally, you need to add some JavaScript to this HTML to display the user’s list of contacts and prepare the data to send to /alert. 

After the ALERT button in your HTML, add a script tag and define an object called data.

<script>
let data = {}
</script>

We will use data to store the data to send to /alert. 

Next, call a function called httpPostAsync and pass in ‘/contacts’ and create_contacts:

<script>
let data = {}
httpPostAsync('/contacts', create_contacts)
</script>

Now, define create_contacts:

function create_contacts(contacts) {
   data\['contacts'] = \[]
   for (let i = 0; i < contacts.length; i++) {
       let contact = contacts\[i]
       data\['contacts'].push(contact)
       const newDiv = document.createElement("div");
       newDiv.className = 'left'
       const newContent = document.createTextNode(contact.name);
       newDiv.appendChild(newContent);
       const currentDiv = document.getElementById("contacts");
       document.body.insertBefore(newDiv, currentDiv);
   }
}

This is a callback function httpPostAsync will call when it gets the contact data from the server. It accepts contacts as a parameter (the contact data). First, this function adds an array to data\['contacts']. Then, it loops through the contacts from the server and adds each contact to the array and creates a new HTML div with each contact’s name. 

Now, you need to define httpPostAsync: 
function httpPostAsync(theUrl, callback) {
   let xmlHttp = new XMLHttpRequest();
   xmlHttp.onreadystatechange = function() {
       if (xmlHttp.readyState == 4 && xmlHttp.status == 200)
           callback(JSON.parse(xmlHttp.responseText));
   }
   xmlHttp.open("GET", theUrl, true); // true for asynchronous
   xmlHttp.send(null);
}

This code sends a GET request to a URL and passes in the JSON it receives in the response to a callback function. 

Finally, you need to define a function that responds when a user clicks the ALERT button.
function alert_them(){
   function success(position) {
       data\['coordinates'] = {}
       data\['coordinates'] = position.coords.latitude
       data\['coordinates'] = position.coords.longitude
       let xmlHttp = new XMLHttpRequest();
       xmlHttp.open("POST", '/alert', true);
       xmlHttp.setRequestHeader("Content-Type", "application/json");
       xmlHttp.send(JSON.stringify(data));
   }

   function error(){
       console.log('error')
   }
   if(!navigator.geolocation) {
       console.log('Geolocation is not supported by your browser');
   } else {
       navigator.geolocation.getCurrentPosition(success, error);
   }
}

This function checks to see if navigator.geolocation is true. Navigator.geolocation lets you get the user’s location. You have to check if it is true because some browser versions do not support it. If it is true, the function above calls navigator.geolocation.getCurrentPosition(success, error) and passes in two functions: one that handles what happens if the browser successfully gets the user’s location and one that handles errors.

The success function adds the user’s coordinates to data\['coordinates'] and then sends a POST request to /alert with the data. 
function success(position) {
       data\['coordinates'] = {}
       data\['coordinates'] = position.coords.latitude
       data\['coordinates'] = position.coords.longitude
       let xmlHttp = new XMLHttpRequest();
       xmlHttp.open("POST", '/alert', true);
       xmlHttp.setRequestHeader("Content-Type", "application/json");
       xmlHttp.send(JSON.stringify(data));
   }

Your /alert endpoint then sends an SMS message to all of the contacts in the database letting them know they are in trouble along with the user’s latitude and longitude. 
Now all you need is to style your app with some css. 
Create a new file in public called style.css and add the following code:
body {
   background-color: lavenderblush;
   text-align: center
}

h1 { color: #111; font-family: 'Helvetica Neue', sans-serif; font-size: 30px; font-weight: bold; letter-spacing: -1px; line-height: 1; text-align: center; }

.forms{
   height: 2.5em;
   width: 30%;
}

.tons {
   margin-top: 1em;
   background-color: #4CAF50; / *Green* /
   border: none;
   color: #f8f8ff;
   padding: 15px 32px;
   text-align: center;
   text-decoration: none;
   display: inline-block;
   font-size: 22px;
   font-family: Arial, serif;
}

.custom-field input {
   border: none;
   -webkit-appearance: none;
   -ms-appearance: none;
   -moz-appearance: none;
   appearance: none;
   background: #f2f2f2;
   padding: 12px;
   border-radius: 3px;
   width: 250px;
   font-size: 14px;
}

\#alert {
   margin-bottom: 5em;
   margin-top:1em;
   background-color:red;
}

\#add{
   background-color:black;
}

\#top{
   margin-top: 5em;
   margin-bottom: 1.5em;
}

.cons{
   font-size: 22px;
}

\#number{
   margin-top: 1em;
}

Now when you go to your website’s homepage, you should see a website like the preview at the begining of this article. 

Add yourself as a contact and press alert. 

You should receive a text containing your longitude and lttitude letting you know you are in an emergency.\
Final Thoughts
This demo is only a starting point! 
There are a ton of features you can add to it. 
For example, if you were using this in production, you would want to create a feature to handle different users (logging in with a username and password, etc.). 
You could also add features like editing contacts and adding options for what type of emergency is happening. 
If you decide to build on this demo, I would love to see what you do: make sure to send me what you build on Twitter. 
Also, make sure to join the Vonage developer community for more awesome content and tutorials. 
You can follow Vonage on Twitter here and join our Slack channel here.
I hope you enjoyed this tutorial!
Thanks for reading!