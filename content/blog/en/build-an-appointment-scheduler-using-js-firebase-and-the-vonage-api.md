---
title: Build an Appointment Scheduler using JS, Firebase and the Vonage API
description: Let's build an appointment scheduler using Node.js, Firebase,
  Express and the Vonage Messages API.
author: amanda-cavallaro
published: true
published_at: 2021-12-16T12:45:57.235Z
updated_at: 2021-12-16T12:45:57.260Z
category: tutorial
tags:
  - "#firebase"
  - "#messages-api"
  - "#javascript"
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
## Introduction

In this tutorial, we are going to build an appointment scheduler using Node.js, Firebase, Express, and the [Vonage Messages API](https://developer.vonage.com/messaging/sms/overview). The [GitHub repository for this project is also available, feel free to clone it](https://github.com/nexmo-community/appointment-scheduler).

## Setting Up Firebase

Let's create a new project from the [Firebase console](https://console.firebase.google.com/).

* Click to add a new project
* Give your project a name for instance `vonage appointment scheduler`
* Check if you like the unique identifier id for your project (it is used in your Realtime Database URL, Firebase Hosting subdomains, and more. It cannot be changed after project creation)
* Click on the button to continue 

![Console view with a text field to enter project and name and edit the project id](/content/blog/build-an-appointment-scheduler-using-js-firebase-and-the-vonage-api/1createproject.png "Console view with a text field to enter project and name and edit the project id")

* Select if you will use analytics, I won't in this tutorial.
* Click on the button to create the project
* Wait for the project to be created

![Project being created](/content/blog/build-an-appointment-scheduler-using-js-firebase-and-the-vonage-api/2projectbeingcreated.png "Project being created")

* Once the project is ready to click to continue, you will be taken to your project's console view
* Set the Billing type by clicking on the gear icon ⚙️, followed by Usage and Billing, then on the Details & Settings tab and modify the plan to use to Blaze. This Pay-as-you-go plan is required to use a third-party API.

### Install Firebase Tools CLI

From your terminal, install the Firebase tools with npm if you don't already have it installed by typing:
	`npm install -g firebase-tools`
Log in to Firebase using Firebase login by typing:
	`firebase login` 
The login process will open your browser for authentication that will either log you in automatically or ask you to add in your credentials. Once that's complete you now have the Firebase CLI installed.

### Create and Set Up a RealTime Database

Now it's time for us to create the NoSQL database instance that will hold the slots information. As the person interacting with the view picks an appointment date and time or adds in a code to cancel their appointment, that slot will be added or removed from the Firebase RealTime Database.

From the Firebase Console Menu, click on "Realtime Database" under Build, and let's create a new NoSQL database instance

![Button to create the database](/content/blog/build-an-appointment-scheduler-using-js-firebase-and-the-vonage-api/3createdatabase.png "Button to create the database")

* Click on "Create Database"
* Select the Realtime Database location where your data will be stored and click on next
* Select if you will use the database in locked or test mode
* Click enable

### Import the Database JSON File

Let's import an example database that already contains some slots allocated and where you'll be able to add and remove future slots.
You can create a file called `myAppointments.json` and from the console add import it.

```JSON
myAppointments.json
{
  "myAppointments": {
    "0": {
      "date": "2021-06-01T09:00",
      "userId": "1234abcd"
    },
    "new_activity_7kh3a3a3z": {
      "date": "2021-06-01T08:50",
      "userId": "_7kh3a3a3z"
    },
    "new_activity_etxen95x3": {
      "date": "2021-06-01T08:40",
      "userId": "_etxen95x3"
    }
  }
}
```

#### Add the Database Rules

The Firebase Realtime Database Security Rules determine who can access your database, the indexes, and how your data is structured. 

* From the Firebase console on the Realtime database view, you can see "Rules", click on that tab. You'll be taken to a screen that will allow you to edit your rules
* Copy and paste the rules from the below code snippet to your console in order to have your rules indexed on the date in addition to the read and write dates that will be accepted
* Click on Publish

```JSON
{
  "rules": {
    ".read": "now < 1643842800000",  // 2022-2-3
    ".write": "now < 1643842800000",  // 2022-2-3
    "myAppointments": {
      ".indexOn": ["date"]
    }
  }
}
```

## Create the Project Structure

By the end of this demonstration, this is roughly how your project structure will look like. In the following steps, we will create the files that will build up the content, appearance, functionalities, and handle the services we will use.

```
.
+-- _.env
+-- _firebaserc
+-- README.md
+-- package.json
+-- server.js
+-- index.html
+-- style.css

// TODO finish completing here
```

### Create the project

### Install Dependencies

## Create the HTML content

Did you know that the HTML input has a couple of types? For this tutorial, we will use `<input type="datetime-local">` it's not as good as a proper library as there can be some inconsistencies, but it works for this demonstration code example.

```HTML
  <!-- index.html -->
<!DOCTYPE html>
<html>
  <head>
    <title>Appointment Scheduler</title>

    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <link rel="stylesheet" href="styles/styles.css" />
  </head>
  <body>
    <main>
      <h1>Appointment Scheduler</h1>
      <!-- datepicker from html. Source: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/datetime-local -->
      <form action="/appointment" method="POST">
        <div>
          <label for="slot">Choose your slot: </label>
          <input
            id="slot"
            type="datetime-local"
            name="slotdate"
            min="2021-06-01T08:30"
            max="2021-10-30T16:30"
            step="300"
            required
          />
          <span class="validity"></span>
        </div>
        <div>
          <label for="phonenumber">Your phone number:</label>
          <input type="tel" id="phonenumber" name="phonenumber" required />
          <span class="validity"></span>
        </div>
        <div>
          <input type="submit" value="Book slot!" />
        </div>
      </form>
      <form action="/cancelAppointment" method="POST">
        <div>
          <input type="text" name="code" placeholder="code" />
          <input type="submit" value="Remove slot!" />
        </div>
      </form>
    </main>
  </body>
</html>
```

## Add some styling `style.css`

```CSS
body {
    margin: auto;
    width: 50%;
    padding: 10px;
}

div {
    margin-bottom: 10px;
    display: flex;
    align-items: center;
}

label {
  display: inline-block;
  width: 300px;
}

input:invalid+span:after {
    content: '✖';
    color: red;
    padding-left: 5px;
}

input:valid+span:after {
    content: '✓';
    color: green;
    padding-left: 5px;
}
```

## Create the JavaScript file `server.js`

Create a JavaScript file, for this example, we will create the  `server.js` and let's add the dependencies and import files.

```javascript
// server.js
require('dotenv').config();
const express = require('express');
const app = require('express')();
const port = 3000;
const admin = require('firebase-admin');
const Vonage = require('@vonage/server-sdk');

const serviceAccount = require('../serviceAccountKey.json');
```

### Initialize Firebase

We use `initializeApp` to create and initialize a Firebase app instance that will use the `/myAppointments` Firebase database instance.

```javascript
// Initializes firebase
admin.initializeApp({
  credential: admin.credential.cert(serviceAccount),
  databaseURL: `${process.env.FIREBASE_DATABASE_URL}`,
});

ref = admin.database().ref('/myAppointments');
```

### Initialize the Vonage API object

We create the instance of the Vonage client class, initializing it with the Vonage API Key and Secret.

```
const vonage = new Vonage({
  apiKey: process.env.VONAGE_API_KEY,
  apiSecret: process.env.VONAGE_API_SECRET,
});
```

### Create the `/appointment`

It's time to create the  `/appointment` endpoint linked to the POST verb. This endpoint will contain the information related to the slots appointments. It will hold three functions to check if the slot is available, to add the slot to the Firebase database, and a final one to Send an SMS back to the user's phone using the Vonage Messages API.

```javascript
app.post('/appointment', async (request, response) => {
  let phonenumber = request.body.phonenumber;
  let slot = request.body.slotdate;
  let [date, time] = getDateTime(slot);

  // Checks if a slot is available
  checkIfAvailable = async (slot) => {};
  
  // Adds to Database
  addToDatabase = () =>  {};
  
  // Sends an SMS back to the user's phone using the Vonage Messages API
  sendSMStoUser = async (code) => {};
```

### Check Slot Availability `/checkIfAvailable()`

This function checks if a slot is available by checking if the slot already exists in the database against the slot the user input.

```javascript
  // Checks if a slot is available
  checkIfAvailable = async (slot) => {
    let snapshot = await ref.orderByChild('date').once('value');

    let available = true;
	
    snapshot.forEach((data) => {
      let dataval = data.val();
      for (let key in dataval) {
        let datapoint = dataval[key];
        if (slot === datapoint) {
          available = false;
        }
      }
    });
    return available;
  };
```

#### Add the Slot to the Database `/addToDatabase`

The following function `addToDatabase()` adds the slot and a code to the Firebase database. The user can later add in this coder to cancel the appointment.

```javascript
  // Adds the slot to the database
  addToDatabase = () => {
    let code = Math.random().toString(36).substr(2, 9);

    ref.child(code).set({
      date: slot,
      userId: code,
    });

    return code;
  };
```

#### Send an SMS with the Appointment Information

Finally, once the slot is reserved, an SMS is sent back to the user with the message `Meeting booked at ${time} on date: ${date}. Please save this code: ${code} in case you'd like to cancel your appointment.` as you can see in the below function `sendSMStoUser()`.

```javascript
  // Sends an SMS back to the user's phone using the Vonage SMS API
  sendSMStoUser = async (code) => {
    const from = process.env.VONAGE_FROM_NUMBER;
    const to = phonenumber;
    const text = `Meeting booked at ${time} on date: ${date}. Please save this code: ${code} in case you'd like to cancel your appointment.`;
    const result = await new Promise((resolve, reject) => {
      vonage.message.sendSms(from, to, text, (err, responseData) => {
        if (err) {
          return reject(new Error(err));
        } else {
          if (responseData.messages[0].status === '0') {
            return resolve(
              `Message sent successfully: ${responseData.messages[0]['message-id']}`
            );
          } else {
            return reject(
              new Error(
                `Message failed with error: ${responseData.messages[0]['error-text']}`
              )
            );
          }
        }
      });
    });
  };
```

#### Put the Pieces Together

The piece of code below is responsible to call the previously created functions. If the slot is available, the user will have their slot added to the database and have the SMS sent back to them. Otherwise, they will be requested to choose a different time slot.

```javascript
let available = await checkIfAvailable(slot);
  if (available) {
    let code = addToDatabase();
    await sendSMStoUser(code);
    response.send(`This slot is available, booking it for you now: ${slot}`);
  } else {
    // Sends user error
    response.send(
      `Sorry, you'll need to choose a different slot.${slot} is already busy.`
    );
  }
});
```

### Cancel the Appointment `/cancelAppointment`

Let's create  `/cancelAppointment` endpoint linked to the POST verb. This endpoint will contain the information related to canceling the appointment slots. It will hold one function to remove the slot from the database by using a code provided by the user that they received upon scheduling their appointment.

```javascript
app.post('/cancelAppointment', async (request, response) => {
  let code = request.body.code;

  // Removes slot from the database
  removeSlotFromDB = (code) => {
    ref.child(code).remove();
  };
  removeSlotFromDB(code);

  response.send(`This slot has been removed.`);
});
```

### Listen to the port `port`

Finally, the app will be listening on a given port to this app that can be accessed locally on `https://localhost:${port}`. In this URL you can interact with the UI of this demo application and check the slots being added/ removed on the Firebase console webpage.

```javascript
app.listen(port, () => {
  console.log(`I run on port ${port}`);
});
```

## Conclusion and Next Steps

Today you saw how to build an appointment scheduler demo web app. Now you can go ahead and add fancier styling, other functionalities. You can take what you learned here to create many appointment schedulers may it be for a gym or for a vaccination slot - let the creativity flow!

[Reach out to us on Twitter](https://twitter.com/VonageDev) and [join our community on Slack](https://app.slack.com/client/T24SLSN21/C24QZH6E7).