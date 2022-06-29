---
title: Vonage Video Express with Ruby on Rails
description: A ruby on rails tutorial that implements the Video Express
  javascript library for fast and easy WebRTC video conferencing applications.
author: benjamin-aronov
published: true
published_at: 2022-06-29T21:18:46.378Z
updated_at: 2022-06-29T21:18:46.409Z
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




1. rails new video-express-rails –database=postgresql
2. cd video-express
3. gem install opentok dotenv-rails
4. Open gemfile and add gems in gemfile
5. bundle install
6. yarn add @vonage/video-express @vonage/vivid 
7. rails g model WatchParty session_id:string expired:boolean
8. Update in migration session_id -> null: false, expired -> default false
9. Add logic in WathcParty model
10. 1. Require opentok
    2. Pass our env variables to opentok
    3. Def self.create_new_session
    4. Def self.create_or_load_session_id
    5. Def self.create_token

* Ask about moderator role here

10. Touch .env and Add our .env variables
11. rails db:create db:migrate
12. rails routes
13. rails g controller WatchParty home login party 

* Delete extra routes

14. Home page 

* A bit rails magic with some vivid magic (add html to home.html.erb)
* Uh oh, doesn’t work
* import '@vonage/vivid'; in application.js 
* Works but ugly
* Add some css in video.scss
* Before move on, let’s add the beautiful Vonage background in application.scss



15. Login page logic

* Add simple redirect logic
* Need to capture name and password from form: login_params
* Don’t forget to delete the view 
* Ok now we’re here on the page, what do we want to do?

16. Build structure

* Header 
* Main 
* For now removed the chat. Will update post with chat component from vivid
* Header and toolbar will hold the Vivid UI to control Video Express logic. Create empty partials
* So all that remains on Party Page is Video Express. Lets build that now

17. Video Express

* Require library
* Add script tag
* Note passing participant name. Video Express is lightweight but comes with some options, explore the docs
* Rails s and….. Nothing!
* We haven’t used any of that OpenTok logic to send to Video Express
* Need to set_opentok_vars in Video Controller
* Why before_action, and why verify_authenticity token?

18. Now if we refresh…..we see that our camera turns on and audio goes all weird. But nothing on the screen. Why?! Video Express is working and we are connected to a video session in opentok. But haven’t given the video screen any room

* Add some css for the video screen. Targeting #roomContainer. 
* Boom now we have a video session. 
* Try joining from multiple tabs/different names!
* Boom! You have video conferencing in Rails!
* Now let’s add some of the functionality you expect in modern video conferencing

19. Top - Bottom: Let’s build that header
20. 1. Great news! Vivid comes with some powerful toolbars
    2. Concept of slots
    3. Add javascript, because most of our javascript is responding to user actions, add defer: true to javascript pack tag in application.html.erb
    4. Create components folder in javascript, in header.js add the header javascript code
21. Toolbar

* Go left to right with each vivid element build the structure of html/css
* * First create the box around the elements
  * Mute all / unmute all
  * Mute self / unmute self
  * Disable camera / enable camera
  * Select Audio Output
  * Tooltips
* Add JS in toolbar.js
* * 3 times we’ll need to toggle hide/show, let’s build a small function toggleButtonView
  * Lets combine toggling the view with toggling the functionality of Video Express: 
  * * toggleMuteAllButton
    * toggleInputButton - audio/video input devices on/off
    * listenForToggle lets us listen for both states of the button
  * Moving on to the device selectors: input (audio/video) and output (audio)
  * * Video Express handles input and output a tiny bit differently
    * Inputs come together:
    * * First receive in getDeviceInputs
      * Listen to changes in selected inputs and update the room
    * Audio Output alone
    * * getAudioOutputDevices()
      * setudioOutPtDevice
  * Lasatly, Vivid gives us tooltips out of the box! Let’s help our users understand the toolbar with tooltips!