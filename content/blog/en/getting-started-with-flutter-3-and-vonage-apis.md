---
title: Getting started with Flutter 3 and Vonage APIs
description: Flutter 3 has now been released! Lets take a look at how we can use
  the Vonage APIs within a flutter application.
thumbnail: /content/blog/getting-started-with-flutter-3-and-vonage-apis/flutter-3.png
author: zachary-powell-1
published: true
published_at: 2022-05-17T08:58:39.152Z
updated_at: 2022-05-17T08:58:39.171Z
category: tutorial
tags:
  - android
  - ios
  - flutter
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
With the release of Flutter 3.0 (which includes a range of [stability and performance improvements](https://medium.com/flutter/whats-new-in-flutter-3-8c74a5bc32d0)) now is a great time to take a look at how you can use communication APIs to improve your user experience and enhance your cross platform applications.

Thanks to Flutters ability to make use of native platforms SDKs and APIs we can easily use the Vonage Android and iOS SDKs within our Flutter applications. Let's take a look at how we can create a simple Flutter application that's able to make a voice phone call to a physical phone.

For this guide we will create a basic app from scratch but you could just as easily build the below into your own application.

## Vonage Setup

Before we get into the code there are a few things we need to do to setup the Vonage API and make use of it. 

### Account Signup

Start by signing up for a free Vonage Developer account. This can be done via the [Dashboard](https://dashboard.nexmo.com/sign-up?icid=tryitfree_api-developer-adp_nexmodashbdfreetrialsignup_nav#_ga=2.264088904.534069361.1652863684-651521337.1652863684), once signed up you will find your account API key and API secret. Take a note of these for future steps.

![Vonage dashboard home page showing API key and API secret location](/content/blog/getting-started-with-flutter-3-and-vonage-apis/dashboard.png)

### Install Vonage CLI

The [Vonage CLI](https://developer.nexmo.com/application/vonage-cli) allows you to carry out many operations on the command line. Examples include creating applications, purchasing numbers, and linking a number to an application all of which we will be doing today.

To install the CLI with NPM run:

```shell
npm install -g @vonage/cli
```

Set up the Vonage CLI to use your Vonage API Key and API Secret. You can get these from the [settings page](https://dashboard.nexmo.com/settings) in the Dashboard.

Run the following command in a terminal, while replacing `API_KEY` and `API_SECRET` with your own:

```shell
vonage config:set --apiKey=API_KEY --apiSecret=API_SECRET
```

### Buy a Vonage Number

Next we need a Vonage number that the application can use, this is the phone number that will show on the phone that we call from the application. 

You can purchase a number using the Vonage CLI. The following command purchases an available number in the US. Specify [an alternate two-character country code](https://www.iban.com/country-codes) to purchase a number in another country.

```
vonage numbers:search US
vonage numbers:buy 15555555555 US
```

### Create webhook server

When an inbound call is received, Vonage makes a request to a publicly accessible URL of your choice - we call this the `answer_url`. You need to create a webhook server that is capable of receiving this request and returning an [NCCO](https://developer.vonage.com/voice/voice-api/ncco-reference) containing a `connect` action that will forward the call to the [PSTN phone number](https://developer.vonage.com/concepts/guides/glossary#virtual-number). You do this by extracting the destination number from the `to` query parameter and returning it in your response.

On the command line create a new folder that will contain your webserver

```
mkdir app-to-phone-flutter
cd app-to-phone-flutter
```

Inside the folder, initialize a new Node.js project by running this command:

```
npm init -y
```

Next, install the required dependencies:

```
npm install express localtunnel --save
```

Inside your project folder, create a file named `server.js` and add the code as shown below - please make sure to replace `NUMBER` with your Vonage number (in [E.164](https://en.wikipedia.org/wiki/E.164) format), as well as `SUBDOMAIN` with an actual value. The value used will become part of the URLs you will set as webhooks in the next step.

```javascript
'use strict';

const subdomain = 'SUBDOMAIN';
const vonageNumber = 'NUMBER';

const express = require('express')
const app = express();
app.use(express.json());

app.get('/voice/answer', (req, res) => {
  console.log('NCCO request:');
  console.log(`  - callee: ${req.query.to}`);
  console.log('---');
  res.json([ 
    { 
      "action": "talk", 
      "text": "Please wait while we connect you."
    },
    { 
      "action": "connect",
      "from": vonageNumber,
      "endpoint": [ 
        { "type": "phone", "number": req.query.to } 
      ]
    }
  ]);
});

app.all('/voice/event', (req, res) => {
  console.log('EVENT:');
  console.dir(req.body);
  console.log('---');
  res.sendStatus(200);
});

if(subdomain == "SUBDOMAIN") {
  console.log('\n\t🚨🚨🚨 Please change the SUBDOMAIN value');
  return false;
}
if(vonageNumber == "NUMBER") {
  console.log('\n\t🚨🚨🚨 Please change the NUMBER value');
  return false;
}
app.listen(3000);

const localtunnel = require('localtunnel');
(async () => {
  const tunnel = await localtunnel({ 
      subdomain: subdomain, 
      port: 3000
    });
  console.log(`App available at: ${tunnel.url}`);
})();
```

You can now start the server by running, in the terminal, the following command:

```
node server.js
```

A notice will be displayed telling you the server is now available:

```
App available at: https://SUBDOMAIN.loca.lt
```

### Create a Vonage Application

In this step you will create a Vonage [Application](https://developer.vonage.com/conversation/concepts/application) capable of in-app voice communication use cases.

Open a new terminal and, if required, navigate to your project directory.

Create a Vonage application by copying and pasting the command below into terminal. Make sure to change the values of `--voice_answer_url` and `--voice_event_url` arguments, by replacing `SUBDOMAIN` with the actual value used in the previous step:

```
vonage apps:create "App to Phone Tutorial" --voice_answer_url=https://SUBDOMAIN.loca.lt/voice/answer --voice_event_url=https://SUBDOMAIN.loca.lt/voice/event 
```

A file named `vonage_app.json` is created/updated in your project directory and contains the newly created Vonage Application ID and the private key. A private key file named `app_to_phone_tutorial.key` is also created.

Make a note of the Application ID that is echoed in your terminal when your application is created:

![screenshot of the terminal with Application ID underlined](/content/blog/getting-started-with-flutter-3-and-vonage-apis/vonage-application-created.png)

### Link a Vonage number

Once you have a suitable number you can link it with your Vonage application. Replace `YOUR_VONAGE_NUMBER` with newly generated number, replace `APPLICATION_ID` with your application id and run this command:

```
vonage apps:link APPLICATION_ID --number=YOUR_VONAGE_NUMBER
```

### Create a User

[Users](https://developer.vonage.com/conversation/concepts/user) are a key concept when working with the Vonage Client SDKs. When a user authenticates with the Client SDK, the credentials provided identify them as a specific user. Each authenticated user will typically correspond to a single user in your users database.

To create a user named `Alice`, run the following command using the Vonage CLI:

```
vonage apps:users:create "Alice"
```

This will return a user ID similar to the following:

```
User ID: USR-aaaaaaaa-bbbb-cccc-dddd-0123456789ab
```

### Generate a JWT

The Client SDK uses [JWTs](https://developer.vonage.com/concepts/guides/authentication#json-web-tokens-jwt) for authentication. The JWT identifies the user name, the associated application ID and the permissions granted to the user. It is signed using your private key to prove that it is a valid token.

Run the following commands, remember to replace the `APPLICATION_ID` variable with id of your application and `PRIVATE_KEY` with the name of your private key file.

You are generating a JWT using the Vonage CLI by running the following command but remember to replace the `APP_ID` variable with your own value:

```
vonage jwt --app_id=APPLICATION_ID --subject=Alice --key_file=./PRIVATE_KEY --acl='{"paths":{"/*/users/**":{},"/*/conversations/**":{},"/*/sessions/**":{},"/*/devices/**":{},"/*/image/**":{},"/*/media/**":{},"/*/applications/**":{},"/*/push/**":{},"/*/knocking/**":{},"/*/legs/**":{}}}'
```

The above commands set the expiry of the JWT to one day from now, which is the maximum.

![terminal screenshot of a generated sample JWT](/content/blog/getting-started-with-flutter-3-and-vonage-apis/generated-jwt-key-vonage.png)

We now have everything we need to use the Vonage Voice API within a flutter application. Lets now get the application itself setup.

## Flutter setup

If you haven't already, start by downloading and installing Flutter and its dependencies. You can do this by following the [Install Guide](https://docs.flutter.dev/get-started/install). Once you have Flutter setup correctly the next thing you will need to do is configure your IDE, how to do this will depend on the IDE you wish to use but the [Set up an editor](https://docs.flutter.dev/get-started/editor) guide will help you with this. 

For this guide we will be using Android Studio. 

Once your IDE is setup follow the [test drive](https://docs.flutter.dev/get-started/test-drive) guide to setup a basic Flutter application with support for both Android and iOS. We will be using this base app as the start to this project, but of course if you already have a Flutter project you want to use you can do this as well.

## Installing SDKs

With the project now setup we can install the Vonage client SDK. Currently, the Client SDK is not available as a Flutter package, so we will have to use the [Android native Client SDK](https://developer.nexmo.com/client-sdk/setup/add-sdk-to-your-app/android) and the [iOS native Client SDK](https://developer.vonage.com/client-sdk/setup/add-sdk-to-your-app/ios) Communicate between Android/iOS and Flutter will use [MethodChannel](https://api.flutter.dev/flutter/services/MethodChannel-class.html) - this way, Flutter will call Android/iOS methods, Android/iOS will call Flutter methods.

### Android SDK

To install the Android SDK open your project level `build.gradle` file which can be found at `android/build.gradle` and add the following repository:

```
maven {
    url "https://artifactory.ess-dev.com/artifactory/gradle-dev-local"
}
```

So that your all projects repositories now looks like:

```
allprojects {
    repositories {
        google()
        mavenCentral()
        maven {
            url "https://artifactory.ess-dev.com/artifactory/gradle-dev-local"
        }
    }
}
```

Next open your app level `build.gradle` file which can be found at `android/app/build.gradle` and implment the vonage sdk like so:

```
dependencies {
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
    implementation "com.nexmo.android:client-sdk:4.1.0"
}
```

Finally make sure your `minSdkVersion` is set to at least `23`:

```
    defaultConfig {
        applicationId "com.vonage.tutorial.voice.app_to_phone"
        minSdkVersion 23
        targetSdkVersion flutter.targetSdkVersion
        versionCode flutterVersionCode.toInteger()
        versionName flutterVersionName
    }
```

The Android SDK is now setup and ready to be used for the Android build of the flutter application.

### iOS SDK

To install the iOS SDK start by generating the `PodFile` by opening a command line in the root of your Flutter project and then running the commands below:

```
cd ios/
pod init
```

This will generate the `PodFile`, open this file and add the below pod:

```
pod 'NexmoClient'
```

Make sure to also set the platform to at least ios 10

```
platform :ios, '10.0'
```

Your complete file should look something like:

```
platform :ios, '11.0'

target 'Runner' do
  use_frameworks!

  pod 'NexmoClient'
end
```

Next from the command line, again in the iOS directory run:

```
pod update
```

This will download and install the Vonage SDK and its dependencies.

Finally to link this to your Flutter project, from the root directory of your project run the below Flutter command. This will trigger an iOS build and generate the files needed to make use of the SDK.

```
flutter build ios
```

Once complete and successfully built your SDK is setup and ready to be used.

## Code