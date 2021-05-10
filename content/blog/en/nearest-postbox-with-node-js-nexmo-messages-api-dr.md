---
title: Locate Your Nearest Postbox with Node.js and the Messages API
description: Locate your nearest postbox. In this tutorial we query an external
  API and send the results via Facebook messenger with the Nexmo messages API
thumbnail: /content/blog/nearest-postbox-with-node-js-nexmo-messages-api-dr/christmas-card-dash.png
author: marklewin
published: true
published_at: 2018-12-06T17:29:19.000Z
updated_at: 2021-05-10T04:10:27.538Z
category: tutorial
tags:
  - javascript
  - messages-api
comments: true
redirect: ""
canonical: ""
---
So you've built a database for this year's Christmas Card list, printed the cards with custom [xkcd](https://xkcd.com/) cartoons based on each recipient's profession and mail-merged the lot. That's a boring job automated and taken care of, with hopefully some fun technical challenges along the way.

Except you have now had a backpack full of Christmas cards for over a week and it's getting perilously close to the last posting date. Worse still, you're attending a meetup in an unfamiliar place and have no idea where the nearest postbox is.

Of course, you could ask someone, but where's the fun in that? Plus, you anticipate that a least a few of your technical friends will have found themselves in the same predicament. Why not help them out, in the spirit of Christmas?

In this post, you'll build and configure a Node.js/Express application that uses the [Nexmo Messages API](https://www.nexmo.com/products/messages) to let users send their current location via Facebook Messenger and get back a map with the location of their nearest postbox.

In addition to the Messages API, we'll be using the following technologies:

* [Matthew Somerville's](https://dracos.co.uk) "Find your nearest postbox service". Details of every postbox in the UK was made available a few years ago in response to a Freedom of Information Request. Matt did the potentially expensive job of calculating the map coordinates of each postbox and making them [available online](http://dracos.co.uk/made/nearest-postbox/).
* [Facebook Messenger](https://www.messenger.com/). Users who have the mobile app will be able to share their location details with us. Those who are using the web client or don't want to share their actual location will be able to send their address instead.
* The [LocationIQ API](https://locationiq.com) to convert addresses to map coordinates in a process known as *geocoding*.

## Prerequisites

There's a couple of things you'll need if you want to get the Postbox Finder working:

* <sign-up></sign-up>
* A Facebook account, ideally with a business page already configured. If you don't have one already, we'll talk you through creating one.
* A [LocationIQ](https://locationiq.com/) account to geocode addresses. You can use their free tier for this.

## Create the Project

All the code for this project is available [on Github](https://github.com/nexmo-community/postbox-finder). Install it locally by performing the following steps:

Clone the repository from GitHub. For example:

```
cd MyProjects
git clone git@github.com:nexmo-community/postbox-finder.git
cd postbox-finder
```

Install the dependencies by running:

```
npm install
```

This application relies on the following packages:

* `express`: the web application framework.
* `body-parser`: for parsing `POST` requests.
* `dotenv`: for configuring the application using an external `.env` file.
* `nexmo ^2.4.2-beta-1`: a beta version of the Node.js REST client library. You need this to work with the Messages API, which is also still in beta.
* `request`: to make calls to the postbox locator and LocationIQ geoocoding APIs.

Have a look around. We'll highlight some of the more interesting bits later in this post.

However, there's a bit of setup you must do before you can get it working. Sorry about that. But hopefully it's more fun than traipsing around the streets in the freezing cold looking for a post box?

## Create a Nexmo Application

First, you need to create a Nexmo application. This is not an application in the same sense as the one you just downloaded, but instead a way for Nexmo to store authentication and other configuration information that you need to work with the Messages API. You can do this using the [Nexmo developer dashboard](https://dashboard.nexmo.com/) or the [Nexmo CLI](https://github.com/Nexmo/nexmo-cli). We'll use the CLI in this post.

Because the Messages API is still in beta, you'll need to install a beta version of the Nexmo CLI to create a Messages API application.

Install the Nexmo CLI beta with `npm`:

```javascript
npm install nexmo-cli@beta -g
```

Configure the Nexmo CLI with your API key and secret, which you can find in the developer dashboard:

```sh
nexmo setup
```

Create the Nexmo application by running `nexmo app:create`. You need to supply this command with:

* The `name` of the application. We'll use `Postbox Finder`.
* The `type` of application you want to create. Here, it's `messages`.
* The webhooks that the Nexmo Messages API will use to communicate with your application. These have to be publicly-available URLs. If you don't have a public web server, consider using [ngrok](https://ngrok.com), as shown in [this blog post](https://www.nexmo.com/blog/2017/07/04/local-development-nexmo-ngrok-tunnel-dr/).

> If you are using `ngrok` on the free plan, run it now using `ngrok http 3000` and leave it running. The free plan provides temporary URLs that change every time you run it, so if you restart it later you will need to reconfigure the application webhooks in the [Developer Dashboard](https://dashboard.nexmo.com/messages/your-applications)

A Messages API application needs the endpoints for two webhooks:

* One to notify us about incoming messages from Facebook Messenger. We'll use `/webhooks/inbound-messages`.
* Another to provide updates on the status of outgoing messages. We'll use `/webhooks/message-status`.
* The name of the file within your application directory where the Nexmo CLI will store your authentication details. In this example, it's `private.key`.

Execute `nexmo app:create`, replacing the sample URLs shown below with either the URL of your web server or the temporary URLs provided by `ngrok`:

```sh
nexmo app:create "Postbox Finder"
https://465d37a7.ngrok.io/webhooks/inbound-message
https://465d37a7.ngrok.io/webhooks/message-status
--keyfile=private.key --type=messages
```

Make a note of the application ID that the above command creates. It will be a GUID, such as `79f9c2dc-a9a8-4aa5-a3c4-c6910ef7b090`.

## Create Your Facebook Business Page

If you want to use the Messages API to send and receive Facebook messages, you must do so from a Facebook business page. If you already have such a page, then just make a note of the Page ID which appears in the page URL, e.g. for the following fictitious page: `https://www.facebook.com/AcmeInc-Postboxes-373699269870723/`, the page ID is `373699269870723`.

If you don't have a page for your business then you can [create one](https://www.facebook.com/business/learn/set-up-facebook-page). Make a note of your Page ID before continuing.

## Link Your Facebook Business Page and Nexmo Account

Now that you have a business page, you need to link it to your Nexmo account so that the Messages API can use it as a communication channel. (The Messages API supports a number of channels, such as Viber, WhatsApp and SMS.)

You need to authenticate this process using a JWT (JavaScript Web Token). This does not have to be application-specific, so if you already have a JWT for another Nexmo application - such as for the Voice API - you can use the same JWT for this step.

If you don't have a JWT, use the following Nexmo CLI command to create one, replacing `YOUR_NEXMO_APPLICATION_ID` with the application ID you generated earlier.

```
JWT="$(nexmo jwt:generate ./private.key application_id=YOUR_NEXMO_APPLICATION_ID"
```

Execute `echo $JWT` to view the generated JWT, then copy it to the clipboard.

Visit [this page](https://static.nexmo.com/messenger). Select your Facebook page from the "Select Page" drop-down list, paste your JWT into the "Provide a Valid JWT token" text box, and then click "Subscribe":

![Link your Facebook page to your Nexmo account](/content/blog/christmas-card-dash-locate-your-nearest-postbox-with-node-js-and-the-messages-api/link-facebook-page-1200x600.png "Link your Facebook page to your Nexmo account")

\[caption id="attachment_25789" align="alignnone" width="640"] Link your Facebook page to your Nexmo account\[/caption]

Your Facebook page and Nexmo account are now linked and you are almost ready to geek the heck out of this posting Christmas cards thing! However, you need to configure the Node.js application with all this information first.

## Configure the Postbox Finder App

Copy `example.env` to `.env`:

```sh
cp example.env .env
```

Replace the entries in `.env` with your own personal details:

```sh
NEXMO_API_KEY=
NEXMO_API_SECRET=
NEXMO_APPLICATION_ID=
NEXMO_APPLICATION_PRIVATE_KEY=private.key
LOCATIONIQ_API_KEY=
```

That's all the hard work done. Let's run this thing.

## Run the Postbox Finder App

If you are exposing your application to the public Internet via `ngrok`'s free plan and have restarted `ngrok` since you created the Nexmo application, the webhook URLs will have changed. [Edit the Message API application webhook URLs](https://dashboard.nexmo.com/messages/your-applications) before continuing.

Launch the Postbox finder application:

```javascript
node index.js
```

From a mobile device with the Facebook Messenger app installed, send your device location to your Facebook business page.

From within the Facebook Messenger app, tap the Location icon or tap the More icon and then select Location.

If everything is working properly you should receive a message with a link.

![The map link received in Facebook Messenger](/content/blog/christmas-card-dash-locate-your-nearest-postbox-with-node-js-and-the-messages-api/fb-messenger-maplink-359x600.png "The map link received in Facebook Messenger")

\[caption id="attachment_25783" align="alignnone" width="359"] The map link received in Facebook Messenger\[/caption]

Clicking on the link shows a map with a marker that represents your closest postbox.

!["The postbox location on the map](/content/blog/christmas-card-dash-locate-your-nearest-postbox-with-node-js-and-the-messages-api/fb-messenger-map-359x600.png "\\\"The postbox location on the map")

\[caption id="attachment_25780" align="alignnone" width="359"] The postbox location on the map\[/caption]

Try sending your address instead. You can do this from either the Facebook Messenger app on your mobile device or the Facebook web site as shown below:

![Sending address details through the Facebook web client](/content/blog/christmas-card-dash-locate-your-nearest-postbox-with-node-js-and-the-messages-api/fb-messenger-web-1200x600.png "Sending address details through the Facebook web client")

\[caption id="attachment_25786" align="alignnone" width="640"] Sending address details through the Facebook web client\[/caption]

## How it Works

All the application logic is in `index.js`. Nothing happens until an incoming Facebook message causes Nexmo's APIs to make a `POST` request to the `/webhooks/inbound-message` webhook.

### Handling the Incoming Message

The request body contains a `message` property which is the bit we are interested in.

If `message` has a `type` of `location` then you know that Facebook Messenger sent the user's current location, which is represented as a latitude/longitude coordinate pair:

```json
{
   "content": {
      "type": "location",
      "location": {
         "url": "https://l.facebook.com/l.php?u=...",
         "lat": 51.522778,
         "long": -0.0851741
      }
   }
}
```

If, on the other hand, the user sent a standard message containing their address, this appears in the `text` property:

```json
{
   "content": {
      "type": "text",
      "text": "99 Ditton Court Road, Westcliff-on-sea, Essex SSO 7HG"
   }
}
```

The webhook handler logic routes the request appropriately. If we already have the user's geographic coordinates we can use them to look up their nearest postcode.

If we only have their address then we need to *geocode* that address to find out the map coordinates for that location. The more address information the user provides, the better the result is likely to be.

```javascript
    // User has sent location from FB Messenger mobile app
    if (content.type === 'location') {
        getPostboxLocations(content.location.lat, content.location.long).then((results) => {
            processCSV(results).then((nearest) => {
                sendMessage(from, to, generateMapLink(nearest));
            }, (error) => {
                sendMessage(from, to, error);
            });
        }, (error) => {
            sendMessage(from, to, error);
        });
    // User has sent address which needs geocoding
    } else if (content.type === 'text') {
        geocodeAddress(content.text).then((address) => {
            getPostboxLocations(address.latitude, address.longitude).then((results) => {
                processCSV(results).then((nearest) => {
                    sendMessage(from, to, generateMapLink(nearest));
                }, (error) => {
                    sendMessage(from, to, error);
                });
            }, (error) => {
                sendMessage(from, to, error);
            });
        }, (error) => {
            sendMessage(from, to, error);
        });
    }
```

In either case we need to do a bit of processing to extract the coordinates of the user's nearest postbox because the postcode locator service returns CSV. That's a bit of a shame: JSON would be easier to handle. Then, finally we can generate a map with a marker that shows that location.

This workflow is implemented as a chain of promises with some very basic error handling.

### Geocoding the User's Address

The postbox locator service expects a latitude/longitude map coordinate. If the user only provided an address, we don't have one. So we need to turn that address into a map coordinate. This process is called *geocoding*. There are a bunch of APIs you can use for this, but we're going to use [LocationIQ](https://locationiq.com/) which has a pretty generous [free tier](https://locationiq.com/pricing) that allows up to 10,000 requests per day. That should be ample for our purposes!

The parameters we need to include in our `GET` request to the service are:

* `countrycodes`: The postbox locator service only contains details about UK postboxes, so we can save the geocoding service some time by limiting lookups to `gb` addresses.
* `limit`: We're only interested in the closest match, so we'll set `limit=1` to ignore any others and shortcut the search process.
* `q`: This is the URL-encoded address we want to query. (You could replace this parameter with `postalcode` if you want a postcode-only lookup.)

```javascript
const geocodeAddress = (addressToGeocode) => {
    return new Promise((resolve, reject) => {
        const geocodeUrl = 'https://eu1.locationiq.com/v1/search.php?countrycodes=gb&limit=1&format=json';
        const queryAddress = encodeURI(addressToGeocode);
        request(`${geocodeUrl}&key=${process.env.LOCATIONIQ_API_KEY}&q=${queryAddress}`, (error, response, body) => {
            if (error) {
                // Something went wrong in the call to the geocoding servie
                reject(error);
            } else {
                const obj = JSON.parse(body);
                // No results returned
                if (obj.error) {
                    reject('Unable to find that address!');
                    // Retrieve map coordinates from the results
                } else {
                    let lat = obj[0].lat;
                    let lon = obj[0].lon;
                    resolve({
                        latitude: lat,
                        longitude: lon
                    });
                }
            }
        });
    });
};
```

If the user supplied a valid address, we will have a map coordinate we can pass to the postbox service.

### Locating the Nearest Postbox

The postbox locator service is very simple. We just pass it a `lat` and `lon` and it returns the five nearest postboxes plus some information about collection times in CSV format:

```csv
Ref,Latitude,Longitude,Distance (km),Last M-F,Last Sat
EC2 2011,51.5235064107165,-0.0845384502633335,0.07,18:30,12:00
EC2 201,51.5234136,-0.0844097,0.07,18:30,12:00
EC2 202,51.523428,-0.0844062,0.07,18:30,
EC1 125,51.5222668,-0.0871634,0.16,18:30,12:00
EC1 1251,51.522202173471,-0.087274363366587,0.17,18:30,12:00
```

Here's the call to the postbox locator service:

```javascript
const getPostboxLocations = (lat, lon) => {
    return new Promise((resolve, reject) => {
        const postboxUrl = 'http://dracos.co.uk/made/locating-postboxes/nearest/?format=csv';
        request(`${postboxUrl}&amp;lat=${lat}&lon=${lon}`, (error, response, body) => {
            if (error) {
                reject('Unable to locate postboxes!');
            } else {
                resolve(body);
            }
        });
    });
};
```

Because the output is in CSV format, we need to pull out the bits we are interested in:

```javascript
const processCSV = (csv) => {
    return new Promise((resolve, reject) => {
        if (csv.length == 0) {
            reject('No results found!');
        } else {
            const content = csv.split('\n');
            const firstRow = content[1].split(',');
            const result = {
                latitude: firstRow[1],
                longitude: firstRow[2],
            };
            resolve(result);
        }
    });
};
```

### Creating the Map Link

Fortunately, it's super-easy to create a link to a Google Map with a marker. All you do is pass the latitude and longitude of the marker to the Google Maps Places API endpoint:

```javascript
const generateMapLink = (nearest) => {
    let msg = '';
    const mapUrl = 'https://www.google.com/maps/search/?api=1';

    // Check to see if lat/lng pair was generated
    if (typeof nearest.latitude === 'undefined') {
        msg = 'No location provided!';
    } else {
        msg = `Your nearest postcode is here: ${mapUrl}&query=${nearest.latitude},${nearest.longitude}`;
    }
    return msg;
};
```

### Sending a Facebook Message with the Map Link

Finally, we want send our user a Facebook message containing the results.

In the `/webhooks/inbound-messages` webhook we capture the message's `from` and `to` properties and reverse them so that we can respond to the message:

```javascript
app.post('/webhooks/inbound-message', (req, res) => {
    // swap 'to' and 'from' to respond to the message
    const from = req.body.to.id;
    const to = req.body.from.id;
    ...
```

And that's all there is to it! You should now know where you can post your Christmas cards. (If you can automate the actual posting process then there is nothing I can teach you!)

## What Else You Could Do

If you want to extend the application there are a couple of things I'd like to suggest.

Firstly, we're not using all the details returned in the response to the postbox locator service. There's info there about distances, collection times and, of course, alternative postboxes in the vicinity. Your app could make use of those.

However, if you do include that information then you will probably want to improve your map. Perhaps you'd like to customize the marker icon (a postbox?) or show multiple markers (maybe the user's location in blue and their other nearby postboxes in a different colour?) Or show a pop-up with collection times? Those things are hard to hand-craft URLs for and you'll find it much easier to use [Google's Places API JavaScript Library](https://developers.google.com/maps/documentation/javascript/places).

And of course this workflow lends itself well to other "find my nearest thing" scenarios using any dataset with real-world locations. If you don't have map coordinates then you'll probably want to batch-geocode them (LocationIQ can help with that). Then, in order to find out which items are nearest to a given point you might want to look into a spatial indexing package such as [geokdbush](https://github.com/mourner/geokdbush).

However you proceed, have a very happy Christmas from all of us in the [Developer Relations team](https://developer.nexmo.com/team) at Nexmo.

## Find Out More

* [Messages API product overview](https://www.nexmo.com/products/messages)
* [Messages API documentation](https://developer.nexmo.com/messages/overview)
* [Introducing Nexmo Messages and Dispatch API blog post](https://www.nexmo.com/blog/2018/03/14/developer-preview-nexmo-messages-api-workflows-api/)