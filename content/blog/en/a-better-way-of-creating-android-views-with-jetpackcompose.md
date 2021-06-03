---
title: A Better Way Of Creating Android Views with JetPackCompose
description: Modern way of building Android Views
author: igor-wojda
published: true
published_at: 2021-06-03T13:57:51.744Z
updated_at: 2021-06-03T13:57:51.764Z
category: inspiration
tags:
  - JetPack
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
# A Bit Of History

Users are interacting with the application via various screens usually composed of multiple Views. The way developers deal with these user interactions has been changing a lot during the Android platform lifetime by using multiple patterns. In the early days, developers were using Model-View-Controller then Model-View-Presenter (or Model-View-Intent), and finally, we moved to the Model-View-ViewModel pattern recommended by Google. The “view manager” has evolved (Controller/Presenter/ViewModel), but the “View” part itself hasn’t changed that much. The biggest change was the usage of Fragment as building blocks for UI instead of Activities like in the early days.

Through all this time we were mostly using XML to define layouts for the application views. We could define these views using code-only, but this approach has its downsides. Usually, applications may have few complex, dynamic views defined in code, but most of the application layouts are still defined in XML files nowadays.

In the meantime, Kotin language was introduced and had this cool feature that allowed to define views using custom Kotln DSL. This was a cool concept, but it has never gained enough attention from the Android developer community. It had its issues but the biggest ones were not supporting advanced use cases and lack official support from Android Studio.

## JetPack Compose

Some time ago Google decided to unify the way we develop Android applications. As a part of the JetPack suite, Google unified many aspects of Android application creation. From navigation, through database access to background jobs and much more… Google has provided a solid foundation to help developers follow best practices, reduce boilerplate code so that developers can focus on the code they care about. As a part of this family comes JetPack Compose – a new and cool way of dealing with UI. JetPack Compose utilizes Kotlin and custom DSL language that allows to easily configure screen layouts, define themes, manage view state and add UI animations. JetPack Compose follows a declarative approach to build UIs that is already widely spread in ReactNative, Flutter, and iOS apps.

## JetPack Compose In Practice

We have a little bit of background, so now let’s take a quick look at how we can use JetPack Compose and what we can do with this toolkit. Here are the highlights of the requirements:

* Use Android Studio Arctic Fox 2020.3.1 or newer
* Create a new Android project
* In the *Select a Project Template* window, select *Empty Compose Activity* and click *Next*.



![](/content/blog/a-better-way-of-creating-android-views-with-jetpackcompose/preview1.png)



![](/content/blog/a-better-way-of-creating-android-views-with-jetpackcompose/preview2.png)

![](/content/blog/a-better-way-of-creating-android-views-with-jetpackcompose/preview3.png)