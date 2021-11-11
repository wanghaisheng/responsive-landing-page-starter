---
title: Auto Layout for Vonage Video Application
description: Using opentok-layout-js we can easily render vonage video call
  participants on the screen. This saves time and makes it super simple to build
  video application using vonage video sdk.
author: mofi-rahman
published: false
published_at: 2021-11-11T15:40:53.771Z
updated_at: 2021-11-11T15:40:53.817Z
category: tutorial
tags:
  - video-api
comments: false
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
Vonage Video API makes it very easy to create our very own video chat application. With as little as 15 lines of code we can get a working video application. But to make it look and feel like a proper video chat app we have to do some work. One of these challenges is to properly place the participants in the screen. We are all kind of familiar with how a video chat app looks. Apps like Zoom, Teams, Google Meet or Webex all have their distinct look. But fundamentally they all accomplish the same goal: place participants in the screen in some form of grid and rearrange them in different screen sizes. Thats what we will attempt to do for our Vonage Video Application today.

This is what we will have at the end of the tutorial.

[![demo9a78f3a979365eb2.gif](https://s9.gifyu.com/images/demo9a78f3a979365eb2.gif)](https://gifyu.com/image/S22vG)

## Prerequisite

- Text editor (VS Code)
- Browser

## Getting started

The source code of this demo is in [This repo](https://github.com/moficodes/opentok-layout-demo). 

The main ingredient to our auto layout is [Opentok Layout JS](https://github.com/aullman/opentok-layout-js). This is an Open Source library created by Adam Ullman. 

## Code Deep Dive

The code for this demo is fairly short. There are 4 files in total. `index.html`, `style.css`, `script.js` and `opentok-layout.js` which is downloaded from [Opentok Layout JS](https://github.com/aullman/opentok-layout-js) repository. 

### index.html

In `<head>` of our html file we add reference to Opentok Client Library used for getting access to `OT` which allows us to get camera access. We also include `opentok-layout.js` and our `style.css`.

```html
    <script src="https://static.opentok.com/v2/js/opentok.min.js"></script>
    <script src="js/opentok-layout.js"></script>
    <link rel="stylesheet" href="css/style.css" />
```

In the `<body>` we add an empty `div` with `id="layout"`. This is the container where we will place all our participant videos. We also have two buttons used for adding and removing video from our layout.

```html
    <div id="layout"></div>
    <div id="buttons">
      <input
        type="button"
        name="add"
        value="Add"
        id="add"
        onclick="addElement()"
      />
      <input
        type="button"
        name="remove"
        value="Remove"
        id="remove"
        onclick="removeElement()"
      />
    </div>
``` 




