---
title: Vonage Video Express with Ruby on Rails Part 2
description: A ruby on rails tutorial that implements the Video Express
  javascript library for fast and easy WebRTC video conferencing applications.
author: benjamin-aronov
published: true
published_at: 2022-07-04T05:14:04.412Z
updated_at: 2022-07-04T05:14:04.447Z
category: tutorial
tags:
  - video-api
  - video-express
  - ruby-on-rails
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
This is the second part of a two-part series on creating a video watch party application using Ruby on Rails with Vonage Video API and the Video Express library.

In the [Part 1](<>), we went through the steps of building the Rails app, showed how to use a few Vivid components, and got the Video Express video chat to run. If you have not read that post yet, it would be a good place to start.

Once we are done, we will have a watch party app that we can use to chat with our friends and watch sports or videos together!

## What The App Will Do

A quick reminder, we are building a video-conferencing application that gives a toolbar to users for different audio/video controls. Additionally, the application gives the moderator the ability to send the Watch Party into different viewing modes.

At this point, we have a working Video Express [Room](https://tokbox.com/developer/video-express/reference/room.html). This object gives us the ability to call different functions to trigger the actions in our toolbar. We want to give the user a way to trigger this functionality, we'll do that with Vivid components. We will organize both our HTML and JS into components. With Webpack, we'll then  `import` our Modules and `require` our components into `application.js` which will expose our Javascript in the client-side.

## Building Out Helper Components

The rest of this tutorial will be building the components to users control their Video Express room. Each component will follow a similar structure: HTML with Vivid components and Javascript to trigger Video Express functions.

### Organizing the HTML

Let's build out our partials where the HTML will live. From the command line, run:

`touch app/views/watch_party/header.html.erb`
`touch app/views/watch_party/_toolbox.html.erb`

And update the `party.html.erb` file to render the partials:

```
<header>
  <%= render partial: 'components/header' %>
</header>

<main class="app">
  <div id="roomContainer"></div>
  <!--  -->
  <toolbar>
    <%= render partial: 'components/toolbar' %>
  </toolbar>
```

### Organizing the Javascript

Just as we have a components folder in our Views, let's create a components folder in our Javascript folder to house our corresponding component logic.

`mkdir app/javascript/components`

Here we'll add our component files:
`touch app/javascript/components/header.js`
`touch app/javascript/components/header.js`

And we'll need to tell Webpack to require them for our clientside. So add the following lines in Application.js belowe our module imports.

```
require("components/header");
require("components/toolbar");
```

Now we're ready to build out our components.

## Building the Header

### Building the HTML

A reminder of the header we want to build:

![The Header in Moderator View](/content/blog/vonage-video-express-with-ruby-on-rails-part-2/screen-shot-2022-07-01-at-13.25.36.png "The Header in Moderator View")

Going top to bottom, let's build that header! Some great news is that Vivid has exactly what we need, a [Top App Bar](https://vivid.vonage.com/?path=/story/components-top-app-bar-fixed--dense&args=dense:) component.
The top app bar comes with a few slot options but two that we care about: `title` and `actionItems`. The title is great for a logo or in our case title. And the `actionitems` can be used as the content of the app bar. This is where we will add the toggler for the moderator to change modes between chill mode and party mode. We can accomplish this with the `vwc-switch` component.

Inside `app/view/watch_party/_header.html.erb` will look like this:

```
<vwc-top-app-bar-fixed alternate="true">
  <span slot="title" id="title">Big Game Chill Zone</span>
  <% if @name == @moderator_name  %>
    <span slot="actionItems" id="mode-name">Chill Mode</span>
    <vwc-switch slot="actionItems"></vwc-switch>
  <% else %>
    <span slot="actionItems"><%= @moderator_name %> is the host</span>
  <% end %>
</vwc-top-app-bar-fixed>
```

### Building The Header Javascript

Inside the `components/header.js` file we'll have two parts: an event listener that will toggle the Video Express ScreenSharing and a function to iterate through all participants in the room to update their display.

First, let's look at the `toggleParticipants`. It toggles through each participant's window to update the layout. This is important to note because the `room` is unique to each participant, so updates must be done to each user.

```
let toggleParticipants = (participants, state) => {
    const title = document.querySelector('#title');
    const mode_name = document.querySelector('#mode-name');
    Object.entries(participants).forEach(participant => {
      if (state === "chill"){
        title.innerHTML = "Big Game Live!";
        mode_name.innerHTML = "Watch Mode"
        room.setLayoutMode("active-speaker");
      } else if (state === "watch") {
        title.innerHTML = "Big Game Chill Zone";
        mode_name.innerHTML = "Chill Mode";
        room.setLayoutMode("grid")
      } else {
        console.log("Error in state of toggleParticipants")
      }
    })
  }
```

We will call the `toggleParticpants` only when the `vwc-switch` is toggled. Notice that this listener is scoped to only listen in the moderators session. Additionally, the Video Express functions `.startScreensharing()` and `.stopScreensharing()` are call here on `room`.

An optional targetElement can be passed to change the location of the screenShare. Read more [here](https://tokbox.com/developer/video-express/reference/room.html).

```
if (document.querySelector('vwc-switch') !== null){
  const switch_btn = document.querySelector('vwc-switch');
  switch_btn.addEventListener('change', (event) => {
    if (event.target.checked){
      room.startScreensharing();
      toggleParticipants(room.participants, "chill");
    }
    else if (!event.target.checked){
      room.stopScreensharing();
      toggleParticipants(room.participants, "watch");
    }
    else{
      console.log("Error in Switch Button Listener");
    }
  });
}
```

Because our Javascript is responding to user actions in the DOM, we want to make sure that Javascript is loaded by Rails after the DOM is loaded. So we need to make a small addition and `defer:true` to the `javascript_pack_tag` in `application.html.erb`:

`<%= javascript_pack_tag 'application', 'data-turbolinks-track': 'reload', defer: true  %>`

### Building The Toolbar HTML

Now we can add the last and most complicated component we have: the toolbar. But it won't be so bad, just building out the HTML with Vivid components and then adding some javascript to trigger Video Express functions. You know the drill!

A reminder of the toolbar we want to build:

![The Toolbar To Control The Video Call](/content/blog/vonage-video-express-with-ruby-on-rails/screen-shot-2022-07-01-at-13.25.36.png "The Toolbar To Control The Video Call")

#### Building The Toggle Buttons

We can see in the toolbar that there are 3 groups of buttons that will toggle on/off some functionality in the room: mute/unmute all, disable/enable microphone, and disable/enable video camera. For all three we will use two buttons from Vivd and then use Javascript to hide the inactive button.

```
<!-- Mute all / Unmute all -->
<vwc-icon-button icon="audio-max-solid" shape="circled" layout="filled" id="mute-all" class="white-border"></vwc-icon-button>
<vwc-icon-button icon="audio-off-solid" shape="circled" layout="filled" id="unmute-all" class="hidden white-border"></vwc-icon-button>
```

```
<!-- Mute self / Unmute self -->
<vwc-icon-button icon="mic-mute-solid" shape="circled" layout="ghost" id="mute-self" class="vvd-scheme-alternate" ></vwc-icon-button>
<vwc-icon-button icon="microphone-2-solid" shape="circled" layout="ghost" id="unmute-self" class="hidden vvd-scheme-alternate"></vwc-icon-button>
```

```
<!-- Disable camera / Enable camera -->
<vwc-icon-button icon="video-off-solid" shape="circled" layout="ghost" id="hide-self" class="vvd-scheme-alternate" ></vwc-icon-button>
<vwc-icon-button icon="video-solid" shape="circled" layout="ghost" id="unhide-self" class="hidden vvd-scheme-alternate"></vwc-icon-button>
```

#### Building DropDown Selects

We can see that the second and third sets of buttons are a little different though. They also should be accompanied by a dropdown which will allow the user to select the associated input; microphone or camera. This is possible with Vivid's [`<vwc-action-group` element](https://vivid.vonage.com/?path=/story/alpha-components-actiongroup--split-button). The left side of the action group comes straight from documentation with a button and a separator. On the right side, we'll make use of the Vivid `vwc-select` component to generate a `select` element which we can target with the different options we receive from VideoExpress. We also pass the `vwc-select` two options: selected and disabled to tell it to show and disable the default `vwc-list-item` which will act as a label.

Our two action groups with buttons now look like this:

```
<!-- Mute Self / Unmute Self -->
<!-- Select Mic Input -->
<vwc-action-group layout="outlined" shape="pill" class="vvd-scheme-alternate">
  <vwc-icon-button icon="mic-mute-solid" shape="circled" layout="ghost" id="mute-self" class="vvd-scheme-alternate" ></vwc-icon-button>
  <vwc-icon-button icon="microphone-2-solid" shape="circled" layout="ghost" id="unmute-self" class="hidden vvd-scheme-alternate"></vwc-icon-button>
  <span role="separator"></span>
  <vwc-select appearance="ghost" id="audio-input" class="select-max-width">
    <vwc-list-item
      disabled
      selected
    >
    Mic
    </vwc-list-item>
  </vwc-select>
</vwc-action-group>
```

```
<!-- Disable Camera / Enable Camera -->
<!-- Select Camera Input -->
<vwc-action-group layout="outlined" shape="pill" class="vvd-scheme-alternate">
  <vwc-icon-button icon="video-off-solid" shape="circled" layout="ghost" id="hide-self" class="vvd-scheme-alternate" ></vwc-icon-button>
  <vwc-icon-button icon="video-solid" shape="circled" layout="ghost" id="unhide-self" class="hidden vvd-scheme-alternate"></vwc-icon-button>
  <span role="separator"></span>
  <vwc-select appearance="ghost" class="select-max-width" id="video-input">
    <vwc-list-item
      disabled
      selected
    >
    Camera
    </vwc-list-item>
  </vwc-select>
</vwc-action-group>
```

We can see that we have a third action group: the audio inputs. We can use the same structure:

```
<!-- Select Audio Output -->
<vwc-action-group layout="outlined" shape="pill" class="vvd-scheme-alternate" id="audio-output-target">
  <vwc-icon-button icon="headset-solid" shape="circled" layout="ghost" class="vvd-scheme-alternate"></vwc-icon-button>
  <span role="separator"></span>
  <vwc-select appearance="ghost" id="audio-output" class="select-max-width">
    <vwc-list-item
      disabled
      selected
    >
    Audio
    </vwc-list-item>
  </vwc-select>
</vwc-action-group>
```

#### Building The ToolTips HTML

Let's give our users a little bit more information about these components. We can do so elegantly using [Vivid's tooltips](https://vivid.vonage.com/?path=/story/alpha-components-tooltip--introduction).

There are three parts to the tooltips we care about: the `anchor`, the `corner`, and the `id`. The anchor tells the tooltip which HTML element to hook onto. The [corner](https://vivid.vonage.com/?path=/story/alpha-components-tooltip--introduction) tells the tooltip in which direction to display, in relation to the anchor. And the `id` is something we will use in the Javascript to trigger displaying the tooltip when a user hovers over the anchor.

With the Tooltips added, our full `_toolbar.html.erb` looks like this:

```
<!-- Mute all / Unmute all -->
<vwc-icon-button icon="audio-max-solid" shape="circled" layout="filled" id="mute-all" class="white-border"></vwc-icon-button>
<vwc-icon-button icon="audio-off-solid" shape="circled" layout="filled" id="unmute-all" class="hidden white-border"></vwc-icon-button>
<vwc-tooltip anchor="mute-all" text="Mute All" corner="top" id="mute-all-tooltip"></vwc-tooltip>
<vwc-tooltip anchor="unmute-all" text="Un-Mute All" corner="top" id="unmute-all-tooltip"></vwc-tooltip>

<!-- Mute Self / Unmute Self -->
<!-- Select Mic Input -->
<vwc-action-group layout="outlined" shape="pill" class="vvd-scheme-alternate">
  <vwc-icon-button icon="mic-mute-solid" shape="circled" layout="ghost" id="mute-self" class="vvd-scheme-alternate" ></vwc-icon-button>
  <vwc-icon-button icon="microphone-2-solid" shape="circled" layout="ghost" id="unmute-self" class="hidden vvd-scheme-alternate"></vwc-icon-button>
  <vwc-tooltip anchor="mute-self" text="Disable Mic" corner="top" id="mute-self-tooltip"></vwc-tooltip>
  <vwc-tooltip anchor="unmute-self" text="Enable Mic" corner="top" id="unmute-self-tooltip"></vwc-tooltip>
  <span role="separator"></span>
  <vwc-select appearance="ghost" id="audio-input" class="select-max-width">
    <vwc-list-item
      disabled
      selected
    >
    Mic
    </vwc-list-item>
  </vwc-select>
</vwc-action-group>



<!-- Disable Camera / Enable Camera -->
<!-- Select Camera Input -->
<vwc-action-group layout="outlined" shape="pill" class="vvd-scheme-alternate">
  <vwc-icon-button icon="video-off-solid" shape="circled" layout="ghost" id="hide-self" class="vvd-scheme-alternate" ></vwc-icon-button>
  <vwc-icon-button icon="video-solid" shape="circled" layout="ghost" id="unhide-self" class="hidden vvd-scheme-alternate"></vwc-icon-button>
  <vwc-tooltip anchor="hide-self" text="Disable Camera" corner="top" id="hide-self-tooltip"></vwc-tooltip>
  <vwc-tooltip anchor="unhide-self" text="Enable Camera" corner="top" id="unhide-self-tooltip"></vwc-tooltip>
  <span role="separator"></span>
  <vwc-select appearance="ghost" class="select-max-width" id="video-input">
    <vwc-list-item
      disabled
      selected
    >
    Camera
    </vwc-list-item>
  </vwc-select>
</vwc-action-group>

<!-- Select Audio Output -->
<vwc-action-group layout="outlined" shape="pill" class="vvd-scheme-alternate" id="audio-output-target">
  <vwc-tooltip anchor="audio-output-target" text="Select Audio Output" corner="top" id="audio-output-tooltip"></vwc-tooltip>
  <vwc-icon-button icon="headset-solid" shape="circled" layout="ghost" class="vvd-scheme-alternate"></vwc-icon-button>
  <span role="separator"></span>
  <vwc-select appearance="ghost" id="audio-output" class="select-max-width">
    <vwc-list-item
      disabled
      selected
    >
    Audio
    </vwc-list-item>
  </vwc-select>
</vwc-action-group>


#### Styling The Toolbar
```

Before we build out the Javascript for the components, let's make the toolbar look like our mockup:

```
// toolbar styles
toolbar{
  display: flex;
  justify-content: space-around;
  margin: 0 auto;
  width: 650px;
  background-color: var(--vvd-color-primary);
  padding: 10px;
  border-radius: 8px 8px 0 0;
}

.hidden{
  display: none;
}

.white-border{
  border: 2px solid white;
  border-radius: 50%;
}

.select-max-width{
  max-width: 130px;
}

vwc-tooltip {
  --tooltip-inline-size: 100px;
  text-align: center;
}
```

## Building The Toolbar Javascript

Going left to the right, the first component we need to build is the "Mute All" button. We want this to be a toggler which toggles on and off the audio of all other participants. But how can we have a toggle button? We really need two buttons: a "mute all" and an "unmute all" button. This pattern will be replicated several times in the toolbar so let's write a helper function for it.

```
// toggle hide/display of buttons
let toggleButtonView = (buttonToHide, buttonToShow) => {
  buttonToHide.style.display = "none";
  buttonToShow.style.display = "block";
}
```

### Building The Mute Others Button

To mute all the participants we need to listen for the user to trigger the action and then cycle through all the participants in this user's instance of the room. This is important to note that the room is local to each user. So do actions on the room is only local to their own browser.

So to disable the audio, we need to iterate through all the participants in the room and use the `.camera.disable()` function. You'll notice that we call this on participant\[1] because the participant object returns an array; [`id`, participantObject]. Additionally room.participants does not include the local user, only all the other participants. **this isn't true. Lets try to make this work with the other function** So we 
**Try to clean this up to use listenForToggle below, don't think need to do room.camera.disableAudio()** 

```
// toggle Mute All / Unmute All
let toggleMuteAllButton = (button, state, participants) =>{
  button.addEventListener("click", function(){
    Object.entries(participants).forEach(participant => {
      if (state === "mute"){
        toggleButtonView(mute_all_btn, unmute_all_btn)
        // why need both here???
        room.camera.disableAudio();
        participant[1].camera.disableAudio();
      } else if (state === "unmute") {
        toggleButtonView(unmute_all_btn, mute_all_btn)
        room.camera.enableAudio();
        participant[1].camera.enableAudio();
      } else {
        console.log("Error in toggleMuteAll")
      }
    })
  })
}
```

```
const mute_all_btn = document.querySelector('#mute-all');
const unmute_all_btn = document.querySelector('#unmute-all');
```

```
toggleMuteAllButton(mute_all_btn, "mute", room.participants);
toggleMuteAllButton(unmute_all_btn, "unmute", room.participants);
```

### Building The Mute Self And Disable Camera Buttons

The Mute/Unmute Self and Disable/Enable Camera buttons follow the same logic as the MuteAll button. They will listen for a user action, check a state and trigger an action in VideoExpress. However they are much simpler because VideoExpress gives us functions to check the state. The two functions are `room.camera.isVideoEnabled` and `room.camera.isAudioEnabled`. Also we only need to trigger the action on a single user.

We can create this function `toggleInputButton` which will check the accept a condition, the boolean we receive from our Video Express functions, and call the corresponding Video Express action. It will also update the view with `toggleButtonView`.

```
// toggle button (display and functionality) of any audio and video input devices
let toggleInputButton = (condition, defaultBtn, altBtn, action) => {
  if (condition()){
    toggleButtonView(defaultBtn, altBtn);
    action();
  } else if (!condition()){
    toggleButtonView(altBtn, defaultBtn);
    action();
  } else {
    console.log(`Error in toggleInputButton. Condition: ${condition}`);
  }
}

The toggle will need to be triggered on a user click, so we can wrap this in the `listenForToggle` function.

// listen for clicks to trigger toggle of input buttons
let listenForToggle = (condition, defaultBtn, altBtn, defaultAction, altAction) => {
    defaultBtn.addEventListener("click", function(){
      console.log("inside listenfortoggle listener")
      toggleInputButton(condition, defaultBtn, altBtn, defaultAction)
    })
    altBtn.addEventListener("click", function(){
      toggleInputButton(condition, defaultBtn, altBtn, altAction)
    })
}
```

We'll need the specific DOM elements to listen for and pass `listenForToggle`, so we create some query selectors.

```
const mute_self_btn = document.querySelector('#mute-self');
const unmute_self_btn = document.querySelector('#unmute-self');

const hide_self_btn = document.querySelector('#hide-self');
const unhide_self_btn = document.querySelector('#unhide-self');
```

Finally all together we can call our code by passing the conditions, the buttons, and the actions: 

```
listenForToggle(room.camera.isVideoEnabled, hide_self_btn, unhide_self_btn, room.camera.disableVideo, room.camera.enableVideo);
listenForToggle(room.camera.isAudioEnabled, mute_self_btn, unmute_self_btn, room.camera.disableAudio, room.camera.enableAudio);
```

### Building The Select Device Inputs

Our select dropdowns for audio and video inputs are currently empty. Let's fill them up with each user's devices. Video Express gives us this functionality out of the box with `VideoExpress.getDevices()`. This returns an array of both audio and video inputs for the local VideoExpress instance. 

We'll need to retrieve this list and then sort it so we can add audio devices to the microphone input and video devices to the camera input. We do so with this asynchronous function, which waits for the Promise to return from querying VideoExpress and then iterates on the devices and appends them to the select element.

```
// retrieve available input devices from VideoExpress
// add retrieved input devices to select options
async function getDeviceInputs(audioTarget, videoTarget){
  const audio = document.querySelector(`${audioTarget}`);
  const video = document.querySelector(`${videoTarget}`);
  const availableDevices = VideoExpress.getDevices();
  availableDevices.then(devices=> {
    devices.forEach(device => {
      if (device.kind === "audioInput"){
        let opt = document.createElement('vwc-list-item');
        opt.value = device.deviceId;
        opt.innerHTML = device.label;
        audio.appendChild(opt);
      }
      else if (device.kind === "videoInput"){
        let opt = document.createElement('vwc-list-item');
        opt.value = device.deviceId;
        opt.innerHTML = device.label;
        video.appendChild(opt);
      }
      else{
        console.log("Error in retrieveDevices");
      }
    })
  })
}
```

We'll also want to update the room when a user changes their selection. We can trigger this by listening to changes in the select menus and then calling either `room.camera.setAudioDevice()` or `room.camera.setVideoDevice()`.

```
// // listen for changes to selected audio/video inputs
// // update room when inputs are changed
let listenInputChange = (target) => {
  const targetSelect = document.querySelector(`${target}`);
  targetSelect.addEventListener('change', (inputOption) => {
    if (target === "vwc-select#audio-input"){
      room.camera.setAudioDevice(inputOption.target.value);
    }
    else if (target === "vwc-select#video-input"){
      room.camera.setVideoDevice(inputOption.target.value);
    }
    else{
      console.log("Error in listenInputChange");
    }
  })
}
```

Putting it all together, calling our functions looks like this:

```
getDeviceInputs("vwc-select#audio-input", "vwc-select#video-input");
listenInputChange("vwc-select#audio-input");
listenInputChange("vwc-select#video-input");
```

### Building The Select Audio Output

Retrieving the list of Audio Outputs from VideoExpress will look almost identical to the inputs. However, here we don't have any devices for Video Output so our code is almost identical except without the filtering. Just like we had `VideoExpress.getDevices()`, now we have `VideoExpress.getAudioOutputDevices()`. And similar to `room.camera.setAudioDevice()` or `room.camera.setVideoDevice()`,  now we have `VideoExpress.setAudioOutputDevice()`.

Because we are only listening for a single change, we can tie it all up nicely in a single function:

```
// retrieve lists of auidoOutput
// add audioOutputs to select menu
// On user select new option, update audio input
async function audioOutputs() {
  var audioOutputs = await VideoExpress.getAudioOutputDevices();
  const audioOutputSelect = document.querySelector('vwc-select#audio-output');
  audioOutputs.forEach(output => {
    let opt = document.createElement('vwc-list-item');
    opt.value = output.deviceId;
    opt.innerHTML = output.label;
    audioOutputSelect.appendChild(opt);
  })

  audioOutputSelect.addEventListener('change', (audioOutputOption) => {
    VideoExpress.setAudioOutputDevice(audioOutputOption.target.value);
  });
}
```

And don't forget to call it in your code!

```
audioOutputs();
```

### Building The Tooltips

Our very last step in the toolbar is to add the behaviour of tooltips to appear and disappear. We do this with vanilla.js by targeting the anchor which we'll call the `target` and the associated tooltip which we call `targertToolTip`. The `mouseover` and `mouseout` allow us to listen when a user hovers on an anchor and when they stop.

Our `addToolTipListeners` looks like this:

```
// toggle tooltips on hover
let addToolTipListeners = (toolTipsToListen) => {
  toolTipsToListen.forEach(toolTipToListen => {
    const target = document.querySelector(`#${toolTipToListen.targetId}`);
    const targetToolTip = document.querySelector(`#${toolTipToListen.toolTipId}`);
    target.addEventListener('mouseover', (event) => targetToolTip.open = !targetToolTip.open);
    target.addEventListener('mouseout', (event) => targetToolTip.open = !targetToolTip.open);
  })
}
```

Then we'll define our tooltips to listen on:

```
const toolTipsToListen = [
  {targetId: "hide-self", toolTipId:"hide-self-tooltip"},
  {targetId: "unhide-self", toolTipId: "unhide-self-tooltip"},
  {targetId: "mute-self", toolTipId: "mute-self-tooltip"},
  {targetId: "unmute-self", toolTipId: "unmute-self-tooltip"},
  {targetId: "mute-all", toolTipId: "mute-all-tooltip"},
  {targetId: "unmute-all", toolTipId: "unmute-all-tooltip"},
  {targetId: "audio-output-target", toolTipId: "audio-output-tooltip"},
]
```

And lastly, call our code.

```
addToolTipListeners(toolTipsToListen);
```

Our final code for `toolbar.js` looks like this:

```
// Start of Toolbar Code

// toggle hide/display of buttons
let toggleButtonView = (buttonToHide, buttonToShow) => {
  buttonToHide.style.display = "none";
  buttonToShow.style.display = "block";
}

// toggle Mute All / Unmute All
let toggleMuteAllButton = (button, state, participants) =>{
  button.addEventListener("click", function(){
    Object.entries(participants).forEach(participant => {
      if (state === "mute"){
        toggleButtonView(mute_all_btn, unmute_all_btn)
        // why need both here???
        participant[1].camera.disableAudio();
      } else if (state === "unmute") {
        toggleButtonView(unmute_all_btn, mute_all_btn)
        participant[1].camera.enableAudio();
      } else {
        console.log("Error in toggleMuteAll")
      }
    })
  })
}



// toggle button (display and functionality) of any audio and video input devices
let toggleInputButton = (condition, defaultBtn, altBtn, action) => {
  if (condition()){
    toggleButtonView(defaultBtn, altBtn);
    action();
  } else if (!condition()){
    toggleButtonView(altBtn, defaultBtn);
    action();
  } else {
    console.log(`Error in toggleInputButton. Condition: ${condition}`);
  }
}

// listen for clicks to trigger toggle of input buttons
let listenForToggle = (condition, defaultBtn, altBtn, defaultAction, altAction) => {
    defaultBtn.addEventListener("click", function(){
      toggleInputButton(condition, defaultBtn, altBtn, defaultAction)
    })
    altBtn.addEventListener("click", function(){
      toggleInputButton(condition, defaultBtn, altBtn, altAction)
    })
}

// retrieve available input devices from VideoExpress
// add retrieved input devices to select options
async function getDeviceInputs(audioTarget, videoTarget){
  const audio = document.querySelector(`${audioTarget}`);
  const video = document.querySelector(`${videoTarget}`);
  const availableDevices = VideoExpress.getDevices();
  availableDevices.then(devices=> {
    devices.forEach(device => {
      if (device.kind === "audioInput"){
        let opt = document.createElement('vwc-list-item');
        opt.value = device.deviceId;
        opt.innerHTML = device.label;
        audio.appendChild(opt);
      }
      else if (device.kind === "videoInput"){
        let opt = document.createElement('vwc-list-item');
        opt.value = device.deviceId;
        opt.innerHTML = device.label;
        video.appendChild(opt);
      }
      else{
        console.log("Error in retrieveDevices");
      }
    })
  })
}


// // listen for changes to selected audio/video inputs
// // update room when inputs are changed
let listenInputChange = (target) => {
  const targetSelect = document.querySelector(`${target}`);
  targetSelect.addEventListener('change', (inputOption) => {
    if (target === "vwc-select#audio-input"){
      room.camera.setAudioDevice(inputOption.target.value);
    }
    else if (target === "vwc-select#video-input"){
      room.camera.setVideoDevice(inputOption.target.value);
    }
    else{
      console.log("Error in listenInputChange");
    }
  })
}


// retrieve lists of auidoOutput
// add audioOutputs to select menu
// On user select new option, update audio input
async function audioOutputs() {
  var audioOutputs = await VideoExpress.getAudioOutputDevices();
  const audioOutputSelect = document.querySelector('vwc-select#audio-output');
  audioOutputs.forEach(output => {
    let opt = document.createElement('vwc-list-item');
    opt.value = output.deviceId;
    opt.innerHTML = output.label;
    audioOutputSelect.appendChild(opt);
  })

  audioOutputSelect.addEventListener('change', (audioOutputOption) => {
    VideoExpress.setAudioOutputDevice(audioOutputOption.target.value);
  });
}


// toggle tooltips on hover
let addToolTipListeners = (toolTipsToListen) => {
  toolTipsToListen.forEach(toolTipToListen => {
    const target = document.querySelector(`#${toolTipToListen.targetId}`);
    const targetToolTip = document.querySelector(`#${toolTipToListen.toolTipId}`);
    target.addEventListener('mouseover', (event) => targetToolTip.open = !targetToolTip.open);
    target.addEventListener('mouseout', (event) => targetToolTip.open = !targetToolTip.open);
  })
}


// define all our DOM elements which we'll want to listen to
const mute_all_btn = document.querySelector('#mute-all');
const unmute_all_btn = document.querySelector('#unmute-all');

const mute_self_btn = document.querySelector('#mute-self');
const unmute_self_btn = document.querySelector('#unmute-self');

const hide_self_btn = document.querySelector('#hide-self');
const unhide_self_btn = document.querySelector('#unhide-self');

const toolTipsToListen = [
  {targetId: "hide-self", toolTipId:"hide-self-tooltip"},
  {targetId: "unhide-self", toolTipId: "unhide-self-tooltip"},
  {targetId: "mute-self", toolTipId: "mute-self-tooltip"},
  {targetId: "unmute-self", toolTipId: "unmute-self-tooltip"},
  {targetId: "mute-all", toolTipId: "mute-all-tooltip"},
  {targetId: "unmute-all", toolTipId: "unmute-all-tooltip"},
  {targetId: "audio-output-target", toolTipId: "audio-output-tooltip"},
]


// Call all of our functions!
toggleMuteAllButton(mute_all_btn, "mute", room.participants);
toggleMuteAllButton(unmute_all_btn, "unmute", room.participants);
listenForToggle(room.camera.isVideoEnabled, hide_self_btn, unhide_self_btn, room.camera.disableVideo, room.camera.enableVideo);
listenForToggle(room.camera.isAudioEnabled, mute_self_btn, unmute_self_btn, room.camera.disableAudio, room.camera.enableAudio);
getDeviceInputs("vwc-select#audio-input", "vwc-select#video-input");
listenInputChange("vwc-select#audio-input");
listenInputChange("vwc-select#video-input");
audioOutputs();
addToolTipListeners(toolTipsToListen);
```

## Finale

And....that's it! With a little bit of Ruby and a bit more Javascript manipulation, you have a full video conferencing application.

We can run the app and try it out with some friends!

### Running ngrok

The easiest way is with ngrok. [Install ngrok](https://developer.vonage.com/tools/ngrok) if you don't have it.

Then use ngrok to run a publicly accessible server. From the command line run:
`ngrok http 3000`

In your terminal you'll see a window that looks like this:

![ngrok server screenshot](/content/blog/vonage-video-express-with-ruby-on-rails/screen-shot-2022-07-01-at-16.32.41.png "ngrok server screenshot")

You'll want to copy the line that ends in ngrok.io. This is the temporarily accessible URL that ngrok will forward your Rails server to. We need Rails to give permission to ngrok to be a host. In our \`config/environments/development.rb\` file, inside the `Rails.application.configure do` we need to add the following line:

`config.hosts << "[ngrok url]"`

For example, in the above instance of running ngrok I would add:

`config.hosts << "c3b9-146-185-57-50.ngrok.io"`

Now we can run our rails server:

`rails s`

And now our app is up and running and you can invite friends to join your watch party by sending them the ngrok URL. And the password!

### Get In Touch

How did you like this tutorial? Have questions or feedback about Video Express or Vivid? Join us on the [Vonage Developer Slack](https://developer.vonage.com/community/slack). And follow us on [Twitter](https://twitter.com/VonageDev) to keep up with the latest Vonage Developer updates.