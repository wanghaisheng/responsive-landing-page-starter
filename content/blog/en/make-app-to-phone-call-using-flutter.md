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
Today you will build an Android application using [Flutter](https://flutter.dev/) and utilise Vonage Client SDK to make a call from a mobile application to the phone.  The application will have 3 screen (3 UI states):

![](/content/blog/make-app-to-phone-call-using-flutter/ui-states.png)

The source code is available on [GitHub](https://github.com/nexmo-community/client-sdk-voice-app-to-phone-flutter).

Firstly, you need to deal with few prerequisites:

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

```cmd
npm install nexmo-cli@beta -g
```

Set up the Nexmo CLI to use your Vonage API Key and API Secret. You can get these from the [settings page](https://dashboard.nexmo.com/settings) in the Dashboard.

Run the following command in a terminal, while replacing api_key and api_secret with your own:

```cmd
nexmo setup api_key api_secret
```

### Setup Vonage application

1. Create your project directory if you've not already done so:

```cmd
mkdir vonage-tutorial
```

2. Change into the project directory:

```cmd
cd vonage-tutorial
```

3. Create a Vonage application by copying and pasting the command below into terminal Make sure to change the value of `--voice-answer-url` argument by replacing `GIST-URL` with the gist URL from the previous step.

```
nexmo app:create "App to Phone Tutorial" --capabilities=voice --keyfile=private.key --voice-event-url=https://example.com/ --voice-answer-url=GIST-URL
```

Make a note of the Application ID.

> NOTE: A hidden file named `.nexmo-app` is created in your project directory and contains the newly created Vonage Application ID and the private key. A private key file named `private.key` is also created.

### Create User

Each participant is represented by a [User](https://developer.nexmo.com/conversation/concepts/user) object and must be authenticated by the Client SDK. In a production application, you would typically store this user information in a database.

Execute the following command to create a user called `Alice`

```cmd
nexmo user:create name="Alice"
```

### Generate JWT

The JWT is used to authenticate the user. Execute the following command in the terminal to generate a JWT for the user `Alice`.

In the following command replace the `APPLICATION_ID` with the ID of your application:

```
nexmo jwt:generate sub=Alice exp=$(($(date +%s)+86400)) acl='{"paths":{"/*/users/**":{},"/*/conversations/**":{},"/*/sessions/**":{},"/*/devices/**":{},"/*/image/**":{},"/*/media/**":{},"/*/applications/**":{},"/*/push/**":{},"/*/knocking/**":{},"/*/legs/**":{}}}'
```

The command above sets the expiry of the JWT to one day from now, which is the maximum.

Make a note of the JWT you generated for `Alice`.

> NOTE: In a production environment, your application should expose an endpoint that generates a JWT for each client request.

## Install Android Studio

Download and install [Android Studio](https://developer.android.com/studio).

## Flutter setup

### Install Flutter SDK

Download and install flutter SDK.

This step will vary on MacOS, Win, and Linux, but in general, it boils down to downloading flutter SDK for a given OS, extracting the SDK file, and adding the `sdk\bin` folder to the system PATH variable. Detailed instruction can be found [here](https://flutter.dev/docs/get-started/install).

Fortunately, flutter comes with a tool that allows us to verify if SDK and all required "components" are present and configured correctly. Run this command:

```cmd
flutter doctor
```

Flutter Doctor will verify if Flutter SDK is installed and other components are installed and configured correctly.

### Install Flutter Plugin

Open [Android Studio](https://developer.android.com/studio), go to `Preferences | plugins` and Install Flutter and Dart plugins from the marketplace. 

Flutter plugin will add new toolbar that allows to run and debug Flutter application:

![](/content/blog/make-app-to-phone-call-using-flutter/flutter-plugin-ui.png)

# Building the Android app

## Create Flutter project

You will create a Flutter project using Android Studio. 

* Run Android Studio
* On the Android Studio welcome screen select `Create New Flutter project`

![](/content/blog/make-app-to-phone-call-using-flutter/create-new-flutter-project.png)

* Select `Flutter Application` and click `Next`
* Enter `app_to_phone_flutter` as project name, enter `Flutter SDK path` and click `Next`
* Select `Include Kotlin support for Android code` and click `Finish`

Connect Android device or emulator and run the app to verify that everything works as expected.

## Two-way Flutter/Android communication

Currently, Client SDK is not available as a Flutter package, so you have to use [Android native Client SDK](https://developer.nexmo.com/client-sdk/setup/add-sdk-to-your-app/android) and communicate between Android and Flutter using [MethodChannel](https://api.flutter.dev/flutter/services/MethodChannel-class.html) - Flutter will call android methods, Android will call Flutter methods. 

Flutter code will be stored in the `main.dart` file, while Android native code will be stored in the `MainActivity.kt` file.

## Building the Flutter part

Flutter applications are build with [Dart programming language](https://dart.dev/).

Open `lib\main.dart` file, remove its content, and paste bellow snippet:

```dart
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

  Future<void> _loginUser() async {
      // Login user
  }

  Future<void> _makeCall() async {
      // Make call
  }

  Future<void> _endCall() async {
      // End call
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

The above code contains custom `CallWidget` which will be responsible for managing application state (logging the user and managing the call). The `SdkState` enum that represents possible states of Vonage Client SDK. This enum will be defined twice - one for Flutter using Dart and one for Android using Kotlin.

Update body of the `_CallWidgetState` class:

```dart
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

The initial state of Flutter application is `SdkState.LOGGED_OUT`. 

Run the application you should see `Login Alice` button:

![](/content/blog/make-app-to-phone-call-using-flutter/loggedout.png)

### Login the user

Button is disable so now add `onPressed` handler to the `ElevatedButton` to allow logging in:

```dart
Widget _updateView() {
    if (_sdkState == SdkState.LOGGED_OUT) {
      return ElevatedButton(
          onPressed: () { _loginUser(); },
          child: Text("LOGIN AS ALICE")
      );
    }
  }
```

Update body of `_loginUser` method:

```dart
Future<void> _loginUser() async {
    String token = "ALICE_TOKEN";

    try {
      await platformMethodChannel.invokeMethod('loginUser', <String, dynamic>{'token': token});
    } on PlatformException catch (e) {
      print(e);
    }
  }
```

Replace the `ALICE_TOKEN` with the token, you obtained previously from Vonage CLI. Flutter will call `loginUser` method and pass the `token` as argument. The `loginUser` method defined in `MainActivity` class (you will get there in a moment). To call this method from Flutter you have to define a `MethodChannel`. Add `platformMethodChannel` the field at the top of `_CallWidgetState` class:

```dart
class _CallWidgetState extends State<CallWidget> {
  SdkState _sdkState = SdkState.LOGGED_OUT;
  static const platformMethodChannel = const MethodChannel('com.vonage');
```

The `com.vonage` string represents the unique channel id that you will also refer on the native Android code (`MainActivity` class). Now you need to handle this method call on the native Android side. 

Open `MainActivity` class. Note that Flutter plugin displays a hint to open this Android project in the separate instance of Android Studio (another window). Do so to have better code completion for Android project:

![](/content/blog/make-app-to-phone-call-using-flutter/openinas.png)

> NOTE: This happens because the Flutter project consists of the Android project and the iOS project.

To listen for method calls originating from Flutter add `addFlutterChannelListener` method call inside `configureFlutterEngine` method:

```kotlin
override fun configureFlutterEngine(@NonNull flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)

        addFlutterChannelListener()
    }
```

Now add `addFlutterChannelListener` and `login` methods inside `MainActivity` class (same level as above `configureFlutterEngine` method):

```kotlin
private fun addFlutterChannelListener() {
        MethodChannel(flutterEngine?.dartExecutor?.binaryMessenger, CHANNEL).setMethodCallHandler { call, result ->

            when (call.method) {
                "loginUser" -> {
                    val token = requireNotNull(call.argument<String>("token"))
                    loginUser(token)
                    result.success("")
                }
                else -> {
                    result.notImplemented()
                }
            }
        }
    }

private fun loginUser(token: String) {
        Log.d("TAG", "login with token: $token")
}
```

After running the application you should see `login with token...` message at Android Logcat. Now it's time to create missing `client`. 

### Add Client SDK dependency

Add a custom Maven URL repository to your Gradle configuration. Add the following maven block inside the `allprojects` block within the project-level `build.gradle.kts` file:

```groovy
allprojects {
    repositories {
        google()
        jcenter()

        maven {
            url "https://artifactory.ess-dev.com/artifactory/gradle-dev-local"
        }
    }
}
```

Now add the Client SDK dependency to the project in the `app\build.gradle` file:

```groovy
dependencies {
    // ...

    implementation 'com.nexmo.android:client-sdk:2.8.1'
}
```

In the same file set min Android SDK version to `23`:

```groovy
minSdkVersion 23
```

Run `Sync project with Gradle` command in Android Studio:

![](/content/blog/make-app-to-phone-call-using-flutter/sync-projct-with-gradle.png)

### Initialize Client

Open `MainActivity` class and `client` property that will hold reference to Nexmo client.

```kotlin
private lateinit var client: NexmoClient
```

Now add `initClient` method:

```kotlin
private fun initClient() {
        client = NexmoClient.Builder().build(this)
    }
```

Add code to call `initClient` method from existing `configureFlutterEngine` method:

```kotlin
override fun configureFlutterEngine(@NonNull flutterEngine: FlutterEngine) {
     super.configureFlutterEngine(flutterEngine)

    initClient()
    addFlutterChannelListener()
}
```

### Login the user

Modify `login` method body to call `login` on the client instance:

```kotlin
private fun login(token: String) {
        client.login(token)
    }
```

This will allow us to login the user (`Alice`) using Client SDK.

### Notify flutter about SDK state change

You will add enum to represent states of the client SDK (you have already added equivalent `SdkState` enum in the `main.dart` file). Add `SdkState` enum at the bottom of the `MainActivity.kt` file:

```kotlin
enum class SdkState {
    LOGGED_OUT,
    LOGGED_IN,
    WAIT,
    ON_CALL,
    ERROR
}
```

You will now add the connection listener and map some of the SDK states to `SdkState` enum. Modify body of `initClient` method:

```kotlin
private fun initClient() {
        client = NexmoClient.Builder().build(this)

        client.setConnectionListener { connectionStatus, _ ->
            when (connectionStatus) {
                ConnectionStatus.CONNECTED -> notifyFlutter(SdkState.LOGGED_IN)
                ConnectionStatus.DISCONNECTED -> notifyFlutter(SdkState.LOGGED_OUT)
                ConnectionStatus.CONNECTING -> notifyFlutter(SdkState.WAIT)
                ConnectionStatus.UNKNOWN -> notifyFlutter(SdkState.ERROR)
            }
        }
    }
```

To send these states to flutter you need to add `notifyFlutter` method in the `MainActivity` class:

```kotlin
private fun notifyFlutter(state: SdkState) {
        Handler(Looper.getMainLooper()).post {
            MethodChannel(flutterEngine?.dartExecutor?.binaryMessenger, "com.vonage")
                .invokeMethod("updateState", state.toString())
        }
    }
```

Notice that we store state in enum, but we are sending it as a string. Communication with Flutter happens on the `main` thread, so you need to use `Handler` to switch threads. The `MethodChannel` will call `updateState` method defined in the `main.dart` file. 

### Retrieve SDK state by Flutter

To retrieve state updates in Flutter you have to listen for method channel updates. Add these two methods inside `_CallWidgetState` class:

```dart
_CallWidgetState() {
    platformMethodChannel.setMethodCallHandler(methodCallHandler);
  }

Future<dynamic> methodCallHandler(MethodCall methodCall) async {
    switch (methodCall.method) {
      case 'updateState':
        {
          setState(() {
            var arguments = 'SdkState.${methodCall.arguments}';
            _sdkState = SdkState.values.firstWhere((v) {return v.toString() == arguments;}
            );
          });
        }
        break;
      default:
        throw MissingPluginException('notImplemented');
    }
  }
```
Method receives "signal" from Android and converts it to an emum. Now update body of `_updateView` method to support `SdkState.WAIT` and `SdkState.LOGGED_IN` states:

```dart
Widget _updateView() {
    if (_sdkState == SdkState.LOGGED_OUT) {
      return ElevatedButton(
          onPressed: () { _loginUser(); },
          child: Text("LOGIN AS ALICE")
      );
    }  else if (_sdkState == SdkState.WAIT) {
      return Center(
        child: CircularProgressIndicator(),
      );
    } else if (_sdkState == SdkState.LOGGED_IN) {
      return ElevatedButton(
          onPressed: () { _makeCall(); },
          child: Text("MAKE PHONE CALL")
      );
    }
  }
```

During `SdkState.WAIT` progress bar will be displayed. After succesfull login application will show `MAKE PHONE CALL` button.

> NOTE: While modyfying Android native code Flutter hot reload will not work. You have to stop the application and run it again.

![](/content/blog/make-app-to-phone-call-using-flutter/flutter-plugin-ui.png)

Run the app. Click `LOGIN AS ALICE` button. You should see `MAKE PHONE CALL` button (another state of the Flutter app based on the `SdkState` enum`):

![](/content/blog/make-app-to-phone-call-using-flutter/makeaphonecall.png)

### Make a call

To make a phone call open `main.dart` file and update body of `_makeCall` method:

```dart
Future<void> _makeCall() async {
    try {
      await requestPermissions();

      await platformMethodChannel
          .invokeMethod('makeCall');
    } on PlatformException catch (e) {
      print(e);
    }
  }
```

The above method will communicate with Android so you have to update code in `MainActivity` class as well. Add `makeCall` clausule to `when` statement inside `addFlutterChannelListener` method:

```kotlin
private fun addFlutterChannelListener() {
        MethodChannel(flutterEngine?.dartExecutor?.binaryMessenger, "com.vonage").setMethodCallHandler { call, result ->

            when (call.method) {
                "loginUser" -> {
                    val token = requireNotNull(call.argument<String>("token"))
                    login(token)
                    result.success("")
                }
                "makeCall" -> {
                    makeCall()
                    result.success("")
                }
                else -> {
                    result.notImplemented()
                }
            }
        }
    }
```

Now in the same file add `onGoingCall` property:

```kotlin
private var onGoingCall: NexmoCall? = null
```

> NOTE: Currently Client SDK does not store ongoing call reference, so you have to store it in `MainActivity` class. You will use it later to end the call.

Now in the same file add `makeCall` method:

```kotlin
@SuppressLint("MissingPermission")
    private fun makeCall() {
        notifyFlutter(SdkState.WAIT)

        // Callee number is ignored because it is specified in NCCO config
        client.call("IGNORED_NUMBER", NexmoCallHandler.SERVER, object : NexmoRequestListener<NexmoCall> {
            override fun onSuccess(call: NexmoCall?) {
                onGoingCall = call
                notifyFlutter(SdkState.ON_CALL)
            }

            override fun onError(apiError: NexmoApiError) {
                notifyFlutter(SdkState.ERROR)
            }
        })
    }
```

The above method sets the state of the Flutter app to `SdkState.WAIT` and waits for the Client SDK response (error or success). Now you need to add support for both states (`SdkState.ON_CALL` and `SdkState.ERROR`) inside `main.dart` file (Fluttter). Update body of the `_updateView` method:

```dart
Widget _updateView() {
    if (_sdkState == SdkState.LOGGED_OUT) {
      return ElevatedButton(
          onPressed: () { _loginUser(); },
          child: Text("LOGIN AS ALICE")
      );
    } else if (_sdkState == SdkState.WAIT) {
      return Center(
        child: CircularProgressIndicator(),
      );
    } else if (_sdkState == SdkState.LOGGED_IN) {
      return ElevatedButton(
          onPressed: () { _makeCall(); },
          child: Text("MAKE PHONE CALL")
      );
    } else if (_sdkState == SdkState.ON_CALL) {
      return ElevatedButton(
          onPressed: () { _endCall(); },
          child: Text("END CALL")
      );
    } else {
      return Center(
        child: Text("ERROR")
      );
    }
  }
```

Each state change will result in UI modification. Before making a call the application needs specific permission.

### Request permissions

The application needs to be able to access the microphone, so you have to request Android `android.permission.RECORD_AUDIO` permission (Flutter calls it `Permission.microphone`). 

First you need to add the [permission_handler](https://pub.dev/packages/permission_handler) package. Open `pubspec.yaml` file and add `permission_handler: ^5.1.0+2` dependency under `sdk: flutter`:

```yaml
dependencies:
  flutter:
    sdk: flutter

  permission_handler: ^5.1.0+2
```

> NOTICE: Intention matters in `yaml` files, so make sure `permission_handler` is at the same indention level as the `flutter:` item.

Run the below command in the terminal to download the newly added Flutter package:

```cmd
flutter pub get
```

Add package import at the top of the `main.dart` file:

```dart
import 'package:permission_handler/permission_handler.dart';
```

Add this method inside `_CallWidgetState` class defined in the `main.dart` file to request permission:

```dart
Future<void> requestPermissions() async {
    Map<Permission, PermissionStatus> statuses = await [
      Permission.microphone
    ].request();
  }
```

Finally you need add two permissions (`uses-permission` tags) inside `app/src/main/AndroidManifest.xml` file, over the `application` tag:

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />

<application
...
```

> NOTE: `android.permission.INTERNET` permission is granted implicitly by the Android, so we don't have to request it in Flutter explicitly.

Run the app and click `MAKE PHONE CALL` to start a call. Permissions dialog will appear and after granting the permissions the Call will start.

> Remainder: You define the phone number earlier in NCCO 

The state of the application will be updated to `SdkState.ON_CALL` and UI will be updated:

![](/content/blog/make-app-to-phone-call-using-flutter/oncall.png)

### End call

Update body of the `_endCall` method inside 

```dart
Future<void> _endCall() async {
    try {
      await platformMethodChannel.invokeMethod('endCall');
    } on PlatformException catch (e) {}
  }
```

The above method will call communicate with Android so you have to update code in `MainActivity` class. Add `endCall` clausule to `when` statement inside `addFlutterChannelListener` method:

```kotlin
when (call.method) {
                "loginUser" -> {
                    val token = requireNotNull(call.argument<String>("token"))
                    login(token)
                    result.success("")
                }
                "makeCall" -> {
                    makeCall()
                    result.success("")
                }
                "endCall" -> {
                    endCall()
                    result.success("")
                }
                else -> {
                    result.notImplemented()
                }
            }
```

Now in the same file add `endCall` method:

```kotlin
private fun endCall() {
        onGoingCall?.hangup(object : NexmoRequestListener<NexmoCall> {
            override fun onSuccess(call: NexmoCall?) {
                onGoingCall = null
                notifyFlutter(SdkState.LOGGED_IN)
            }

            override fun onError(apiError: NexmoApiError) {
                notifyFlutter(SdkState.ERROR)
            }
        })
    }
```

The above method sets the state of the Flutter app to `SdkState.WAIT` and waits for the Client SDK response (error or success). Both UI states are already supported in the Flutter application.

You have handled ending the call by pressing `END CALL` button in the Flutter application UI, however, the call can also end outside of the Flutter app e.g. the call will be rejected or answered and later ended by the callee (on the real phone). 

To support this cases you have to add `NexmoCallEventListener` listener to the call instance and listen for call specific events. 

Define `callEventListener` property at the top of the `MainActivity` class:

```kotlin
private val callEventListener = object : NexmoCallEventListener {
        override fun onMemberStatusUpdated(nexmoCallStatus: NexmoCallMemberStatus, callMember: NexmoCallMember) {
            if (nexmoCallStatus == NexmoCallMemberStatus.COMPLETED || nexmoCallStatus == NexmoCallMemberStatus.CANCELLED) {
                onGoingCall = null
            }
        }

        override fun onMuteChanged(nexmoMediaActionState: NexmoMediaActionState, callMember: NexmoCallMember) {}

        override fun onEarmuffChanged(nexmoMediaActionState: NexmoMediaActionState, callMember: NexmoCallMember) {}

        override fun onDTMF(dtmf: String, callMember: NexmoCallMember) {}
    }
```

The `onMemberStatusUpdated` callback informs you about call end.

To register above listener modify `onSuccess` callback inside `makeCall` method: 

```kotlin
onGoingCall = call
onGoingCall?.addCallEventListener(callEventListener)
```

Finally modify `endCall` method to unregister the `callEventListener` listener inside `onSuccess` callback:

```kotlin
onGoingCall?.removeCallEventListener(callEventListener)
onGoingCall = null
```

Run the app and test if everything is working as expected.

# Summary

You have successfully build the application. By doing so you have learned how to make a phone call from mobile application to the phone using Vonage Client SDK. For the complete version please see this project on [GitHub](https://github.com/nexmo-community/client-sdk-voice-app-to-phone-flutter).

To familiarize yourself with other use cases please check [other tutorial](https://developer.vonage.com/client-sdk/tutorials) and [Vonage developer center](https://developer.vonage.com/).

# References

* [Vonage developer center](https://developer.vonage.com/)
* [Write your first flutter app](https://flutter.dev/docs/get-started/codelab)
* [Flutter Plaftorm chanels](https://flutter.dev/docs/development/platform-integration/platform-channels)