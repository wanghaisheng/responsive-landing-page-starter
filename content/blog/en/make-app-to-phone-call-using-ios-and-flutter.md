---
title: Make app-to-phone call using iOS and Flutter
description: "Build iOS application using Flutter and utilise Vonage Client SDK
  to make a call from mobile application to the phone. "
author: igor-wojda
published: false
published_at: 2021-03-10T06:59:40.028Z
updated_at: 2021-03-10T06:59:40.058Z
category: tutorial
tags:
  - iOS
  - Flutter
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
Today you will build an iOS application using [Flutter](https://flutter.dev/) and utilize Vonage Client SDK to make a call from a mobile application to the phone.  The application will have 3 screens (3 UI states):

![](/content/blog/make-app-to-phone-call-using-flutter/ui-states.png)

The source code is available on [GitHub](https://github.com/nexmo-community/client-sdk-voice-app-to-phone-flutter).

efore we begin building the application for our iOS device, you'll need to prepare with the following prerequisites:

* Create a Call Control Object ([NCCO](https://developer.nexmo.com/voice/voice-api/guides/ncco))
* Install Nexmo CLI
* Setup Vonage application
* Install Flutter SDK
* Create Flutter project

## Prerequisites

## Vonage application

### Create an NCCO

A Call Control Object (NCCO) is a JSON array that you use to control the flow of a Voice API call. More information on NCCO can be found here [here](https://developer.nexmo.com/voice/voice-api/ncco-reference).

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

3. Create a Vonage application by copying and pasting the command below into the terminal Make sure to change the value of `--voice-answer-url` argument by replacing `GIST-URL` with the gist URL from the previous step.

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

## Install Xcode

Open AppStore and install [Xcode](https://developer.apple.com/xcode/).

## Flutter setup

### Install Flutter SDK

Download and install flutter SDK.

This step will vary on MacOS, Win, and Linux, but in general, it boils down to downloading flutter SDK for a given OS, extracting the SDK file, and adding the `sdk\bin` folder to the system PATH variable. Detailed instruction can be found [here](https://flutter.dev/docs/get-started/install).

Fortunately, flutter comes with a tool that allows us to verify if SDK and all required "components" are present and configured correctly. Run this command:

```cmd
flutter doctor
```

Flutter Doctor will verify if Flutter SDK is installed and other components are installed and configured correctly.

## Create Flutter project

You will create a Flutter project using the terminal:

```cmd
flutter create app_to_phone_flutter
```

> Notice that `app_to_phone_flutter` folder (flutter project) contains`ios` folder containing the OS project and `ios` folder containing the iOS project.

Enter the `app_to_phone_flutter`, open `pubspec.yaml` and add `permission_handler` dependency (just below `sdk: flutter`):

```yaml
dependencies:
  flutter:
    sdk: flutter
  
  permission_handler: ^6.0.1+1
```

> NOTICE: Intention matters in `yaml` files, so make sure `permission_handler` is at the same indention level as the `flutter:` item.

Now run this command to download the above dependency:

```cmd
flutter pub get
```

The above command will also create `Podfile` in `ios` subfolder. Open `ios\Podfile` uncomment `platform` line and update the platform version to `11`:

```
platform :ios, '11.0'
```

At the end of the same file add `pod 'NexmoClient'`:

```
target 'Runner' do
  use_frameworks!
  use_modular_headers!
  pod 'NexmoClient'
```

Open `app_to_phone_flutter/ios` folder in the termnal and instal pods:

```cmd
pod install
```

Open `Runner.xcworkspace` in Xcode and run the app to verify that everything works as expected.

## Two-way Flutter/iOS communication

Currently, Client SDK is not available as a Flutter package, so you have to use [iOS native Client SDK](https://developer.nexmo.com/client-sdk/setup/add-sdk-to-your-app/ios) and communicate between iOS and Flutter using [MethodChannel](https://api.flutter.dev/flutter/services/MethodChannel-class.html) - Flutter will call iOS methods, iOS will call Flutter methods. 

Flutter code will be stored in the `lib/main.dart` file, while iOS native code will be stored in the `ios/Runner/AppDelegate.swift` file.

## Init Flutter application

Flutter applications are built with [Dart](https://dart.dev/) programming language.

Open `lib/main.dart` file, remove its content, and paste bellow snippet:

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

The above code contains custom `CallWidget` which will be responsible for managing the application state (logging the user and managing the call). The `SdkState` enum represents possible states of Vonage Client SDK. This enum will be defined twice - one for Flutter using Dart and one for iOS using Kotlin.

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

The initial state of the Flutter application is `SdkState.LOGGED_OUT`. 

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

Replace the `ALICE_TOKEN` with the token, you obtained previously from Vonage CLI. Flutter will call `loginUser` method and pass the `token` as argument. The `loginUser` method defined in `AppDelegate` class (you will get there in a moment). To call this method from Flutter you have to define a `MethodChannel`. Add `platformMethodChannel` field at the top of `_CallWidgetState` class:

```dart
class _CallWidgetState extends State<CallWidget> {
  SdkState _sdkState = SdkState.LOGGED_OUT;
  static const platformMethodChannel = const MethodChannel('com.vonage');
```

The `com.vonage` string represents the unique channel id that you will also refer on the native iOS code (`AppDelegate` class). Now you need to handle this method call on the native iOS side. 

Open `AppDelegate` class. To listen for method calls originating from Flutter add `addFlutterChannelListener` method call inside `application` method:

```swift
override func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {
        addFlutterChannelListener()
        
        GeneratedPluginRegistrant.register(with: self)
        return super.application(application, didFinishLaunchingWithOptions: launchOptions)
    }
```

Open `AppDelegate` class and `vonageChannel` property that will hold the reference to the `FlutterMethodChannel`:

```swift
var vonageChannel: FlutterMethodChannel?
```

Now add `addFlutterChannelListener` and `login` methods inside `AppDelegate` class (same level as above `application` method):

```swift
func addFlutterChannelListener() {
        let controller = window?.rootViewController as! FlutterViewController
        
        vonageChannel = FlutterMethodChannel(name: "com.vonage",
                                             binaryMessenger: controller.binaryMessenger)
        vonageChannel?.setMethodCallHandler({ [weak self]
            (call: FlutterMethodCall, result: @escaping FlutterResult) -> Void in
            guard let self = self else { return }
            
            switch(call.method) {
            case "loginUser":
                if let arguments = call.arguments as? [String: String],
                   let token = arguments["token"] {
                    self.loginUser(token: token)
                }
                result("")
            default:
                result(FlutterMethodNotImplemented)
            }
        })
    }
    
    func loginUser(token: String) {
        
    }
```

### Initialize Client

Open `AppDelegate` class and add NexmoClient import at the top of the file:

```swift
import NexmoClient
```

In the same file add `client` property that will hold reference to Nexmo client.

```swift
let client = NXMClient.shared
```

Now add `initClient` method:

```swift
func initClient() {
        client.setDelegate(self)
    }
```

Add code to call `initClient` method from existing `application` method:

```swift
override func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {
        initClient()
        addFlutterChannelListener()
        
        GeneratedPluginRegistrant.register(with: self)
        return super.application(application, didFinishLaunchingWithOptions: launchOptions)
    }
```

IN the `AppDelegate` file add delegate to listen for Client SDK connection state changes:

```swift
extension AppDelegate: NXMClientDelegate {
    func client(_ client: NXMClient, didChange status: NXMConnectionStatus, reason: NXMConnectionStatusReason) {
        switch status {
        case .connected:
            notifyFlutter(state: .loggedIn)
        case .disconnected:
            notifyFlutter(state: .loggedOut)
        case .connecting:
            notifyFlutter(state: .wait)
        @unknown default:
            notifyFlutter(state: .error)
        }
    }
    
    func client(_ client: NXMClient, didReceiveError error: Error) {
        notifyFlutter(state: .error)
    }
}
```

### Login the user

Modify `login` method body to call `login` on the client instance:

```swift
func loginUser(token: String) {
        self.client.login(withAuthToken: token)
    }
```

This will allow us to login the user (`Alice`) using Client SDK.

### Notify flutter about SDK state change

You will add enum to represent states of the client SDK (you have already added equivalent `SdkState` enum in the `main.dart` file). Add `SdkState` enum inside `AppDelegate` class (at the top):

```swift
enum SdkState: String {
        case loggedOut = "LOGGED_OUT"
        case loggedIn = "LOGGED_IN"
        case wait = "WAIT"
        case onCall = "ON_CALL"
        case error = "ERROR"
    }
```

To send these states to Flutter (from above delegate) you need to add `notifyFlutter` method in the `AppDelegate` class:

```swift
func notifyFlutter(state: SdkState) {
        vonageChannel?.invokeMethod("updateState", arguments: state.rawValue)
    }
```

Notice that you store the state in the enum, but you are sending it as a string.

### Retrieve SDK state by Flutter

To retrieve state updates in Flutter you have to listen for method channel updates. Open `main.dart` file and add these two methods inside `_CallWidgetState` class:

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

Method receives "signal" from iOS and converts it to an emum. Now update body of `_updateView` method to support `SdkState.WAIT` and `SdkState.LOGGED_IN` states:

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

> NOTE: While modyfying iOS native code Flutter hot reload will not work. You have to stop the application and run it again.

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

The above method will communicate with iOS so you have to update code in `AppDelegate` class as well. Add `makeCall` clausule to `when` statement inside `addFlutterChannelListener` method:

```swift
func addFlutterChannelListener() {
        let controller = window?.rootViewController as! FlutterViewController
        
        vonageChannel = FlutterMethodChannel(name: "com.vonage",
                                             binaryMessenger: controller.binaryMessenger)
        vonageChannel?.setMethodCallHandler({ [weak self]
            (call: FlutterMethodCall, result: @escaping FlutterResult) -> Void in
            guard let self = self else { return }
            
            switch(call.method) {
            case "loginUser":
                if let arguments = call.arguments as? [String: String],
                   let token = arguments["token"] {
                    self.loginUser(token: token)
                }
                result("")
            case "makeCall":
                self.makeCall()
                result("")
            case "endCall":
                self.endCall()
                result("")
            default:
                result(FlutterMethodNotImplemented)
            }
        })
    }
```

Now in the same file add `onGoingCall` property:

```swift
var onGoingCall: NXMCall?
```

> NOTE: Currently Client SDK does not store ongoing call reference, so you have to store it in `AppDelegate` class. You will use it later to end the call.

Now in the same file add `makeCall` method:

```swift
func makeCall() {
        client.call("IGNORED_NUMBER", callHandler: .server) { [weak self] (error, call) in
            guard let self = self else { return }
            
            if error != nil {
                self.notifyFlutter(state: .error)
                return
            }
            
            self.onGoingCall = call
            self.notifyFlutter(state: .onCall)
        }
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

The application needs to be able to access the microphone, so you have to request access to the microphone (Flutter calls it `Permission.microphone`). 

You already added the [permission_handler](https://pub.dev/packages/permission_handler) package to Flutter. Open `ios/Runner/info.plist` file and add `Privacy - Microphone Usage Description` hey with `Make a call` value:

Add this method inside `_CallWidgetState` the class defined in the `main.dart` file to request permission:

```dart
Future<void> requestPermissions() async {
    Map<Permission, PermissionStatus> statuses = await [
      Permission.microphone
    ].request();
  }
```

Run the app and click `MAKE PHONE CALL` to start a call. Permissions dialog will appear and after granting the permissions the Call will start.

> Remainder: You define the phone number earlier in NCCO 

The state of the application will be updated to `SdkState.ON_CALL` and UI will be updated:

![](/content/blog/make-app-to-phone-call-using-flutter/oncall.png)

### End call

To end the call you need to trigger the method on the native iOS application using `platformMethodChannel`. Inside `main.dart` file update body of the `_endCall` method:

```dart
Future<void> _endCall() async {
    try {
      await platformMethodChannel.invokeMethod('endCall');
    } on PlatformException catch (e) {}
  }
```

The above method will communicate with iOS so you have to update code in the `AppDelegate` class s well. Add `endCall` clausule to `switch` statement inside the `addFlutterChannelListener` method:


```swift
func addFlutterChannelListener() {
        let controller = window?.rootViewController as! FlutterViewController
        
        vonageChannel = FlutterMethodChannel(name: "com.vonage",
                                             binaryMessenger: controller.binaryMessenger)
        vonageChannel?.setMethodCallHandler({ [weak self]
            (call: FlutterMethodCall, result: @escaping FlutterResult) -> Void in
            guard let self = self else { return }
            
            switch(call.method) {
            case "loginUser":
                if let arguments = call.arguments as? [String: String],
                   let token = arguments["token"] {
                    self.loginUser(token: token)
                }
                result("")
            case "makeCall":
                self.makeCall()
                result("")
            case "endCall":
                self.endCall()
                result("")
            default:
                result(FlutterMethodNotImplemented)
            }
        })
    }
```

Now in the same file add the `endCall` method:

```swift
func endCall() {
        onGoingCall?.hangup()
        onGoingCall = nil
        notifyFlutter(state: .loggedIn)
    }
```

The above method sets the state of the Flutter app to `SdkState.WAIT` and waits for the response from the Client SDK, which can be either error or success. Both UI states are already supported in the Flutter application.

You have handled ending the call by pressing `END CALL` button in the Flutter application UI, however, the call can also end outside of the Flutter app e.g. the call will be rejected or answered and later ended by the callee (on the real phone). 

To support these cases you have to add `NexmoCallEventListener` listener to the call instance and listen for call-specific events. 

In the `AppDelegares.swift` file add `NXMCallDelegate`:

```swift
extension AppDelegate: NXMCallDelegate {
    func call(_ call: NXMCall, didUpdate callMember: NXMCallMember, with status: NXMCallMemberStatus) {
        if (status == .completed || status == .cancelled) {
            onGoingCall = nil
            notifyFlutter(state: .loggedIn)
        }
    }
    
    func call(_ call: NXMCall, didUpdate callMember: NXMCallMember, isMuted muted: Bool) {
        
    }
    
    func call(_ call: NXMCall, didReceive error: Error) {
        notifyFlutter(state: .error)
    }
}
```

To register above listener modify `onSuccess` callback inside `makeCall` method: 

```swift
self.onGoingCall = call
self.onGoingCall?.setDelegate(self)
self.notifyFlutter(state: .onCall)
```

Run the app and if you've followed through this tutorial step by step, you'll be able to make a phone call from your mobile application to a physical phone number.

# Summary

You have successfully built the application. By doing so you have learned how to make a phone call from a mobile application to the phone using Vonage Client SDK. For the complete version please see this project on [GitHub](https://github.com/nexmo-community/client-sdk-voice-app-to-phone-flutter).

To familiarize yourself with other use cases please check [other tutorials](https://developer.vonage.com/client-sdk/tutorials) and [Vonage developer center](https://developer.vonage.com/).

# References

* [Vonage developer center](https://developer.vonage.com/)
* [Write your first flutter app](https://flutter.dev/docs/get-started/codelab)
* [Flutter Plaftorm chanels](https://flutter.dev/docs/development/platform-integration/platform-channels)