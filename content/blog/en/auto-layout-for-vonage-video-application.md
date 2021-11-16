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
Vonage Video API makes it easier to create our own video chat application with as little as 15 lines of code. To make it look and feel like a proper video chat app we have to do some more work. One challenge is to properly place the participants on the screen. Apps like Zoom, Teams, Google Meet, and Webex all have their own distinct looks. Fundamentally they all accomplish the same goal: place participants on the screen in some form of a grid and rearrange them based on different screen sizes. That's what we will do for our Vonage Video Application today.

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

Finally we add reference to our `script.js` file.

### style.css

Most of the css file is used to place the buttons in the bottom of the screen and leave the rest of the screen real estate for `layout` container. There is some css transition set to make the transition from one view to another look smooth.

### script.js

This is where we initialize and make use of `opentok-layout-js`. We first get access to our `layoutEl` and create a global variable of `layout` function which we will call every time we need to reorganize the screen.

```js
var layoutEl = document.getElementById('layout');
var layout;
```

Then we create a function that initializes the layout container and set the `layout` function.

```js
function updateLayoutValues() {
  const opts = {
    maxRatio: 3 / 2,
    minRatio: 9 / 16,
    fixedRatio: false,
    alignItems: 'center',
    bigPercentage: 0.8,
    bigFixedRatio: false,
    bigMaxRatio: 3 / 2,
    bigMinRatio: 9 / 16,
    bigFirst: true,
    scaleLastRow: true,
    smallMaxWidth: Infinity,
    smallMaxHeight: Infinity,
    bigMaxWidth: Infinity,
    bigMaxHeight: Infinity,
    bigAlignItems: 'center',
    smallAlignItems: 'center',
  };
  layout = initLayoutContainer(layoutEl, opts).layout;
}
updateLayoutValues();
```

You can read about what all the possible fields in `opts` are and what they mean [here](https://github.com/aullman/opentok-layout-js#usage). 

At this point we have `layout` ready to be called when we need to reorganize the participants. We also write functions for adding and removing elements. And finally a window resize event listener to call `layout` on window resize.

## Conclusion

This was a quick demo to showcase how you can get easy auto layout for your Vonage Video Application. For creating a quick, easy and streamlined multi party Video Application using Vonage API you can make use of [Vonage Video Express](https://tokbox.com/developer/video-express/). Video Express actually uses `opentok-layout-js` and wraps our OpenTok client sdk to make building Multiparty Video application easier. You can get started with Vonage Video Express with [this article by Enrico](https://learn.vonage.com/blog/2021/09/27/create-a-multiparty-video-app-with-the-new-video-express/).

You can reach out to us via [Twitter](https://twitter.com/vonagedev) or [Slack](https://developer.nexmo.com/community/slack). 


