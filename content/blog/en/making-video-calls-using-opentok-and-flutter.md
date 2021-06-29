---
title: "Making Video Calls Using OpenTok And Flutter "
description: Let's tate a coser look as the OpenTok Flutter app that allows to
  make a video calls
author: igor-wojda
published: false
published_at: 2021-06-17T11:32:37.509Z
updated_at: 2021-06-17T11:32:37.550Z
category: announcement
tags:
  - Flutter
  - OpenTok
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
Flutter is gaining more and more popularity, so we decided to build a simple application that allows making video calls between two devices. Two technologies used to build the app are Flutter and OpenTok SDKs. Let's reap quickly these technologies:

* [Flutter](https://flutter.dev/) - open-source UI software development kit used to develop cross-platform applications for Android, iOS, Linux, Mac. The main programming language is [Dart](https://dart.dev/)
* [OpenTok](https://tokbox.com/developer/sdks/android/) - used to build video calls between various devices. The programming languages used are [Kotlin](https://kotlinlang.org/) for the Android platform and [Switf](https://www.swift.com/) for the iOS platform.

The application source code is available on [GitHub](https://github.com/opentok/opentok-flutter-basic-video-chat). This application is a Flutter equivalent of the Basic-Video-Chat application ([Basic-Video-Chat Android](https://github.com/opentok/opentok-android-sdk-samples/tree/main/Basic-Video-Chat), \[Basic-Video-Chat iOS](opentok-ios-sdk-samples/Basic-Video-Chat at main · opentok/opentok-ios-sdk-samples · GitHub)). Here are the main features of the application:

* Connect to an OpenTok session
* Publish an audio-video stream to the session
* Subscribe to another client's audio-video stream

Flutter is the main technology here. It is a foundation used to build a mobile application. It will be responsible for managing and displaying UI, and it will contain the application logic. This way application logic is only written once for both platforms.

Under the hood, this Flutter application will use [Android OpenTok SDK](https://tokbox.com/developer/sdks/android/) and iOS [OpenTok SDK](https://tokbox.com/developer/sdks/ios/) (Android/iOS native projects) that allows to log in the user and set up the video session to make a video call.  

![Flutter application under the hood](/content/blog/making-video-calls-using-opentok-and-flutter/method-channel.png)

Platform (Android, iOS) native code communicates with Flutter by using Flutter [MethodChannel](https://api.flutter.dev/flutter/services/MethodChannel-class.html) that uses method calls. MethodChannel serves as a bridge to send messages between Flutter and native code:

![Flutter application flow](/content/blog/making-video-calls-using-opentok-and-flutter/flutter-application.png)

Flutter can send messages to the native (Android / iOS) part of the app and the native part of the app can send a message back to Flutter eg. Flutter calls `initSession` method and passes the `apiKey`, `sessionId`, and `token` to native code to start OpenTok session. If the response natve code will inform Flutter part of the app about successful login (or error) and Flutter-sie code will update the UI.

> NOTE: Flutter app can be packaged as Android or iOS application, but never both at the same time. When the target platform is set to Android then MethodChannel communicates with Android native app code and when the target platform is set to iOS then MethodChannel communicates with iOS native app code.

To run the [application](https://github.com/opentok/opentok-flutter-basic-video-chat) you will have to install Flutter. This varies from platform to platform. Detailed instructions can be found [here](https://flutter.dev/docs/get-started/install).

> NOTE: Make sure to run `flutter doctor` to verify your local flutter setup.

To login into the OpenTok session, you will need an [TokBox account](https://tokbox.com/account/#/) and generate `initSession`, `apiKey`, and `sessionId`. You can get these values form in the [OpenTok Dashboard](https://tokbox.com/account/#/). Now open `main.dart` file and fill the retrieved values:

```
static String API_KEY = "";
static String SESSION_ID = "";
static String TOKEN = "";
```

Launch the mobile app to start the video call.

> NOTE: You can use [Developer playground](https://tokbox.com/developer/tools/playground/) to connect to the same session as the mobile device running the Flutter app.

## Summary

There are still a few [drawbacks](https://github.com/opentok/opentok-flutter-basic-video-chat#known-issues), but the overall Integration of Flutter and OpenTok is quite smooth. Even without a native Flutter package, it is possible to easily create build a fully-fledged Flutter app that utilizes OpenTok mobiles SDKs under the hood and runs on Android and iOS devices.