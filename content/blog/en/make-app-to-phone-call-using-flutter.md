---
title: Make app-to-phone call using Flutter
description: "Build Android application using Flutter and utilise Vonage Client
  SDK to make a call from mobile application to the phone. "
author: igor-wojda
published: false
published_at: 2021-03-02T13:08:11.401Z
updated_at: 2021-03-02T13:08:11.447Z
category: tutorial
tags:
  - Android
  - Flutter
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
Today you will build an Android application using Flutter and utilize Vonage Client SDK to make a call from a mobile application to the phone. 

First, you need to deal with few prerequisites:

* Crete NCCO
* Install Nexmo CLI
* Setup Vonage application
* Install Flutter SDK
* Create Flutter project

# Prerequisites

## Vonage application

### Create NCCO

A Nexmo Call Control Object (NCCO) is a JSON array that you use to control the flow of a Voice API call. More information on NCCO can be found here [here](https://developer.nexmo.com/voice/voice-api/ncco-reference).

The NCCO must be public and accessible by the internet. To accomplish that, you will be using [GitHub Gist](https://gist.github.com/) which provides a convenient way to host the configuration. Let's add a new configuration:

1. Go to <https://gist.github.com/> (you have to be logged in into Github)
2. Create a new gist with `ncco.json` the filename
3. Copy and paste the following JSON object into the gist:
4. Replace `PHONE_NUMBER` with your phone number (Vonage numbers are in E.164 format <https://developer.nexmo.com/concepts/guides/glossary#e-164-format>, '+' and '-' are not valid. Make sure you specify your country code when entering your number, for example, US: 14155550100 and UK: 447700900001)

```json
[
    {
        "action": "talk",
        "text": "Please wait while we connect you."
    },
    {
        "action": "connect",
        "endpoint": [
            {
                "type": "phone",
                "number": "PHONE_NUMBER"
            }
        ]
    }
]
```

5. Click the `Create secret gist` button
6. Click the `Raw` button
7. Take note of the URL shown in your browser, you will be using it in the next step

### Install Nexmo CLI

The Nexmo CLI allows you to carry out many operations on the command line. If you want to carry out tasks such as creating applications, purchasing Vonage numbers and so on, you will need to install the Nexmo CLI. 

Nexmo CLI requires `node.js`, so you will need to install node.js first using [these instructions](https://nodejs.org/en/download/).

To install the Beta version of the CLI with NPM run this command:

`npm install nexmo-cli@beta -g`

Set up the Nexmo CLI to use your Vonage API Key and API Secret. You can get these from the [settings page](https://dashboard.nexmo.com/settings) in the Dashboard.

Run the following command in a terminal, while replacing api_key and api_secret with your own:

`nexmo setup api_key api_secret`

### Setup Vonage application

1. Create your project directory if you've not already done so:

`mkdir vonage-tutorial`

2. Change into the project directory:

`cd vonage-tutorial`

3. Create a Vonage application by copying and pasting the command below into terminal Make sure to change the value of `--voice-answer-url` argument by replacing `GIST-URL` with the gist URL from the previous step.

```
nexmo app:create "App to Phone Tutorial" --capabilities=voice --keyfile=private.key --voice-event-url=https://example.com/ --voice-answer-url=GIST-URL
```

Make a note of the Application ID.

> NOTE: A hidden file named `.nexmo-app` is created in your project directory and contains the newly created Vonage Application ID and the private key. A private key file named `private.key` is also created.

### Create User

Each participant is represented by a [User](https://developer.nexmo.com/conversation/concepts/user) object and must be authenticated by the Client SDK. In a production application, you would typically store this user information in a database.

Execute the following command to create a user called `Alice`

`nexmo user:create name="Alice"`

### Generate JWT

The JWT is used to authenticate the user. Execute the following command in the terminal to generate a JWT for the user `Alice`.

In the following command replace the `APPLICATION_ID` with the ID of your application:

```
nexmo jwt:generate sub=Alice exp=$(($(date +%s)+86400)) acl='{"paths":{"/*/users/**":{},"/*/conversations/**":{},"/*/sessions/**":{},"/*/devices/**":{},"/*/image/**":{},"/*/media/**":{},"/*/applications/**":{},"/*/push/**":{},"/*/knocking/**":{},"/*/legs/**":{}}}'
```

The command above sets the expiry of the JWT to one day from now, which is the maximum.

Make a note of the JWT you generated for `Alice`.

> NOTE: In a production environment, your application should expose an endpoint that generates a JWT for each client request.

## Flutter

### Install Flutter SDK

Download and install flutter SDK.

This step will vary on MacOS, Win, and Linux, but in general, it boils down to downloading flutter SDK to a given OS, extracting it, and adding the `sdk\bin` folder to the system PATH variable. Detailed instruction can be found here: <https://flutter.dev/docs/get-started/install>

Fortunately, flutter comes with a tool that allows us to verify if SDK and all required "components" are present and configured correctly. Run this command:

`flutter doctor`

Flutter Doctor will verify if Flutter SDK is installed correctly and check if your IDE had flutter and Dart plugins installed.

### Install Flutter Plugin

Open [Android Studio](https://developer.android.com/studio), go to Preferences and plugins. Install Flutter and Dart plugins from the marketplace. 

# Building the Android app

## Create Flutter project

You will create a Flutter project using Android Studio. 

* Run Android Studio
* On the Android Studio welcome screen select `Create New Flutter project`
* Select `Flutter Application` and click `Next`
* Enter `app_to_phone_flutter` as project name, enter `Flutter SDK path` and click `Next`
* Select `Include Kotlin support for Android code` and click `Finish`

Connect Android device or emulator and run the app to verify that everything works as expected.

## Two-way Flutter/Android communication

Currently, Client SDK is not available as a Flutter package, so we have to use [Android native Client SDK](https://developer.nexmo.com/client-sdk/setup/add-sdk-to-your-app/android) and communicate between Android and Flutter using methodChannel (https://api.flutter.dev/flutter/services/MethodChannel-class.html). Flutter will call methods defined in ANdroid code (MainActivity.kt) and Android will invoke `updateState` method to notify Flutter about SDK state updates. 

## Building the Flutter part

Flutter applications are using [Dart programming language](https://dart.dev/).

Open `lib\main.dart` file, remove its content, and paste bellow snippet:

```
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      home: CallWidget(title: 'app-to-phone-flutter'),
    );
  }
}

class CallWidget extends StatefulWidget {
  CallWidget({Key key, this.title}) : super(key: key);
  final String title;

  @override
  _CallWidgetState createState() => _CallWidgetState();
}

class _CallWidgetState extends State<CallWidget> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            SizedBox(height: 64),
            _buildConnectionButtons()
          ],
        ),
      ),
    );
  }
}

enum SdkState {
  LOGGED_OUT,
  LOGGED_IN,
  WAIT,
  ON_CALL,
  ERROR
}
```

The above code contains our custom `CallWidget` a widget that will be responsible for logging the user and managing the call. Code also contains `SdkState` enum that represents possible states of Vonage Client SDK.

The code does not compile yet, because we still have to add a few missing pieces.

### Logged out state

Update body of the `_CallWidgetState` class:

```
class _CallWidgetState extends State<CallWidget> {
  SdkState _sdkState = SdkState.LOGGED_OUT;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            SizedBox(height: 64),
            _updateView()
          ],
        ),
      ),
    );
  }

  Widget _updateView() {
    if (_sdkState == SdkState.LOGGED_OUT) {
      return ElevatedButton(
          onPressed: () {  },
          child: Text("LOGIN AS ALICE")
      );
    }
  }
}
```