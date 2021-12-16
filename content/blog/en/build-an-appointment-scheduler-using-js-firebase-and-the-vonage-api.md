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
Let's build an appointment scheduler using Node.js, Firebase, Express and the [Vonage Messages API](https://developer.vonage.com/messaging/sms/overview). The [GitHub repository for this project is also available, feel free to clone it]()!
  
## Setting Up Firebase
Let's create a new project from the [Firebase console]() and give it a name
- Click continue 
- Select if you will use analytics, I won't in this tutorial.
- Wait for the project to be created
- Set the Billing type under the gear ⚙️ -> Usage and Billing -> Details & Settings to Blaze. 
- The Pay-as-you-go plan is required to use a third-party API. 
- Set the Google Cloud resource location in ⚙️ -> Project Settings.

### Install Firebase Tools**

From your terminal, install the Firebase tools with npm if you don't already have it installed by typing:
	`npm install -g firebase-tools`
Log in to Firebase using firebase login by typing:
	`firebase login`
The login process will open your browser for authentication.
 
## Create a Real Time Database
### GUI
### Import the database JSON file

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

### Database rules

```JSON
// database.rules.json
{
  "rules": {
    ".read": "now < 1626649200000", // 2021-7-19
    ".write": "now < 1626649200000", // 2021-7-19
    "myAppointments": {
      ".indexOn": ["date"]
    }
  }
}
``` 

## Let's do the HTML `index.html`
Did you know that the HTML input has a couple of types? For this tutorial we will use `<input type="datetime-local">` []() it's not as a good as a proper library as there can be some inconsistencies, but it works for as simple demo.  

## Let's add some styling `index.css`
```CSS


```

## `server.js`
Create a JavaScript file, for this example we will create the  `server.js` and let's add the dependencies and import files.
```
require('dotenv').config();
const express = require('express');
const app = require('express')();
const port = 3000;
const admin = require('firebase-admin');
const Vonage = require('@vonage/server-sdk');

const serviceAccount = require('../serviceAccountKey.json');

 ```

### We then initialize Firebase
```
// Initializes firebase
admin.initializeApp({
  credential: admin.credential.cert(serviceAccount),
  databaseURL: `${process.env.FIREBASE_DATABASE_URL}`,
});

ref = admin.database().ref('/myAppointments');
```

### We initialize the Vonage API object
```
// Sends SMS with Vonage API
const vonage = new Vonage({
  apiKey: process.env.VONAGE_API_KEY,
  apiSecret: process.env.VONAGE_API_SECRET,
});
```

### Let's create the `/appointment'`
```javascript
app.post('/appointment', async (request, response) => {
  let phonenumber = request.body.phonenumber;
  let slot = request.body.slotdate;
  let [date, time] = getDateTime(slot);

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

### Let's create the `/addToDatabase`
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
### Let's create the `/sendSMStoUser`
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
### Let's create the `/available`
  ```
let available = await checkIfAvailable(slot);
  if (available) {
    let code = addToDatabase();
    await sendSMStoUser(code);
    response.send(`This slot is available, booking it for you now: ${slot}`);
  } else {
    // Sends user error
    response.send(
      `Sorry, you'll need to choose a different slot.${slot} is alread busy.`
    );
  }
});
```

### Let's create the `/cancelAppointment'`
```
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
```
app.listen(port, () => {
  console.log(`I run on port ${port}`);
});

``` 

## Conclusion
Today you saw how to build an appointment scheduler demo web app. Now you can go ahead and take what you learned here to create many appointment schedulers may it be for a gym or for a vaccination slot - let the creativity flow!

Feel free to [reach out to me on twitter](https://twitter.com/amdcavallaro)!
// Add Vonage twitter and slack## Introduction
Let's build an appointment scheduler using Node.js, Firebase, Express and the [Vonage Messages API](https://developer.vonage.com/messaging/sms/overview). The [GitHub repository for this project is also available, feel free to clone it]()!
  
## Setting Up Firebase
Let's create a new project from the [Firebase console]() and give it a name
- Click continue 
- Select if you will use analytics, I won't in this tutorial.
- Wait for the project to be created
- Set the Billing type under the gear ⚙️ -> Usage and Billing -> Details & Settings to Blaze. 
- The Pay-as-you-go plan is required to use a third-party API. 
- Set the Google Cloud resource location in ⚙️ -> Project Settings.

### Install Firebase Tools**

From your terminal, install the Firebase tools with npm if you don't already have it installed by typing:
	`npm install -g firebase-tools`
Log in to Firebase using firebase login by typing:
	`firebase login`
The login process will open your browser for authentication.
 
## Create a Real Time Database
### GUI
### Import the database JSON file

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

### Database rules

```JSON
// database.rules.json
{
  "rules": {
    ".read": "now < 1626649200000", // 2021-7-19
    ".write": "now < 1626649200000", // 2021-7-19
    "myAppointments": {
      ".indexOn": ["date"]
    }
  }
}
``` 

## Let's do the HTML `index.html`
Did you know that the HTML input has a couple of types? For this tutorial we will use `<input type="datetime-local">` [https://lnkd.in/dbuRyzQ](https://lnkd.in/dbuRyzQ) it's not as a good as a proper library as there can be some inconsistencies, but it works for as simple demo.  

## Let's add some styling `index.css`
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

## `server.js`
Create a JavaScript file, for this example we will create the  `server.js` and let's add the dependencies and import files.
```
require('dotenv').config();
const express = require('express');
const app = require('express')();
const port = 3000;
const admin = require('firebase-admin');
const Vonage = require('@vonage/server-sdk');

const serviceAccount = require('../serviceAccountKey.json');

 ```

### We then initialize Firebase
```
// Initializes firebase
admin.initializeApp({
  credential: admin.credential.cert(serviceAccount),
  databaseURL: `${process.env.FIREBASE_DATABASE_URL}`,
});

ref = admin.database().ref('/myAppointments');
```

### We initialize the Vonage API object
```
// Sends SMS with Vonage API
const vonage = new Vonage({
  apiKey: process.env.VONAGE_API_KEY,
  apiSecret: process.env.VONAGE_API_SECRET,
});
```

### Let's create the `/appointment'`
```javascript
app.post('/appointment', async (request, response) => {
  let phonenumber = request.body.phonenumber;
  let slot = request.body.slotdate;
  let [date, time] = getDateTime(slot);

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

### Let's create the `/addToDatabase`
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
### Let's create the `/sendSMStoUser`
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
### Let's create the `/available`
  ```
let available = await checkIfAvailable(slot);
  if (available) {
    let code = addToDatabase();
    await sendSMStoUser(code);
    response.send(`This slot is available, booking it for you now: ${slot}`);
  } else {
    // Sends user error
    response.send(
      `Sorry, you'll need to choose a different slot.${slot} is alread busy.`
    );
  }
});
```

### Let's create the `/cancelAppointment'`
```
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
```
app.listen(port, () => {
  console.log(`I run on port ${port}`);
});

``` 

## Conclusion
Today you saw how to build an appointment scheduler demo web app. Now you can go ahead and take what you learned here to create many appointment schedulers may it be for a gym or for a vaccination slot - let the creativity flow!

Feel free to [reach out to me on twitter](https://twitter.com/amdcavallaro)!
// Add Vonage twitter and slack



