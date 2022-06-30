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
Since 2004 Ruby on Rails has been beloved by developers and especially startup founders for it's ability to quickly build full scale web applications. Kerry Doyle perfectly summed up the "Convention Over Configuration" philosophy that underpins Rails, "(Ruby on Rails) attempts to decrease the number of decisions that a developer using the framework is required to make without necessarily losing flexibility and don't repeat yourself (DRY) principles". This philosophy makes Rails ideal for the [LEAN](https://en.wikipedia.org/wiki/Lean_software_development) mindest.

Last year Vonage released Video Express. This Javascript library which sits ontop of Vonage's Video API is like the "Convention Over Configuration" extension to make building Video Call Applications easier and faster. Any developer can now spin up powerful, robust Video Meetings without knowing all the nuts and bolts.

In this tutorial, I'll show you how to combine these powerful technologies to build a modern, fullstack application with all the video conferencing features that users have come to expect. I'll use Ruby on Rails, Vonage Video Express, and Vonage's new UI Toolkit VIVID.

This post is inspired by [Create a Party With Ruby on Rails and the Vonage Video API](https://developer.vonage.com/blog/20/05/12/create-a-party-with-ruby-on-rails-and-the-vonage-video-api-part-1-building-the-backend-dr). I will largely recreate the functionality and afterwards you can see just how much much less code was required!

## What The App Will Do
Back in 2020 Ben Greenberg built an app to allow his son to watch movies with his friends. This app will be essentially the same. Only I don't watch movies online with friends. But since moving across the world, it's been hard to find fans of my favorite ice hockey team, the Blues. So instead of watching movies with my friends, I'd love to watch the Blues or sports in general with friends from back home. Functionally, the app is the same and borrows heavily from Ben's movie app. This will make it easy to compare the pros/cons between using Video Express and writing in native Vonage Video API.

This app will have two pages; a landing page with a login form and a "party" page. When users login they will be taken to a "Chill Zone" to hang out. Here they will be displayed in equal size. Then when the game is beginning, the "moderator" who owns the session will trigger a screenshare to display the game. In the "Watch Mode", the shared screen of the moderator will be dominant and take up most of the video call. Because the best part of watching sports with friends is the banter, I will still see and hear the other participants. But if my friends are being too loud, I still want the ability to mute them. We'll also allow participants to mute themselves, turn off their video camera, and select their video/audio inputs and outputs.

**Yalla! Let's go!**

## Requirements
- [ruby v3.0.0+](https://www.ruby-lang.org/en/)
- [Rails v6.1.6+] (https://rubyonrails.org/)
- [Vonage Video API](https://tokbox.com/account/user/signup)
- [Vonage Video Express v1.2.3+](https://tokbox.com/developer/video-express/)
- [VIVID v2.34.0+](https://vivid.vonage.com/)


## Vonage Video API Setup

It is free to create a Vonage Video API account. You need to do so in order to obtain your API key and secret, which are essential to making the app functional.

### Credentials
After you have [created an account with the Vonage Video API](https://tokbox.com/account/user/signup), you will see a dashboard interface. The first step in obtaining API credentials is to create a new project.

- Open the **Projects** tab from the left sidebar.
- Click the **Create New Project** option.
- Select **Vonage Video API** when asked what kind of project to create; *Create Custom Project*
- Provide any name for the project name
- Choose the VP8 codec option. (Details on the difference between VP8 and H.264 are detailed [here](https://tokbox.com/developer/guides/codecs/))

- You now have access to your project's API key and secret. Keep them somewhere safe, we will be using them soon.



### Enable Video Express
To use Video Express, be sure to activate the Video Express add-on for your account.


- Go to your [Video API Account](https://tokbox.com/account/) and click **Account Settings** in the left-hand menu.
- In the list of **Account add-ons**, find **Video Express** and click **Add to account**. Then follow the remaining instructions to enable the add-on.

## Setting Up The Rails App
Before we can add the frontend magic with Video Express and Vivid, we need to do a bit of work to get our Rails app created and ready with Vonage Video API on the backend.

### Installation
Create a new rails app using postgresql as the database.

`rails new video-express-rails –database=postgresql`

Move into the project:
`cd video-express`

Now open up the project with your favorite text editor.

### Gem Dependencies
We'll need to add two gems: [Vonage Video API Ruby SDK](https://github.com/opentok/OpenTok-Ruby-SDK) (formerly known as OpenTok) and [dotenv-rails](https://github.com/bkeepers/dotenv) to handle our environment variables.


Open the `Gemfile` and add the gems:

```
gem 'opentok'
gem 'dotenv-rails'
```
Once that is done, we can run bundle install from the command line to install our dependencies.

### Node Modules
We'll be using two 
`yarn add @vonage/video-express @vonage/vivid`

`rails g model WatchParty session_id:string expired:boolean`

1. Update in migration session_id -> null: false, expired -> default false\
   in db/migrate directory:  TIMESTAMP_create_watch_parties.rb

   ```
   class CreateWatchParties < ActiveRecord::Migration[6.1]
     def change
       create_table :watch_parties do |t|
         t.string :session_id, null:false
         t.boolean :expired, default: false

         t.timestamps
       end
     end
   end
   ```
2. Add logic in WatchParty model
3. 1. Require opentok

      `require 'opentok'`
   2. Pass our env variables to opentok

      `@opentok = OpenTok::OpenTok.new ENV['OPENTOK_API_KEY'], ENV['OPENTOK_API_SECRET']`
   3. Def self.create_new_session

      ```
        def self.create_new_session
          session = @opentok.create_session
          record = WatchParty.new
          record.session_id = session.session_id
          record.save
          @session_id = session.session_id
          @session_id
        end
      ```
   4. Def self.create_or_load_session_id

      ```
        def self.create_or_load_session_id
          if WatchParty.any?
            last_session = WatchParty.last
            if last_session && last_session.expired == false
              @session_id = last_session.session_id
              @session_id
            elsif (last_session && last_session.expired == true) || !last_session
              @session_id = create_new_session
            else
              raise 'Something went wrong with the session creation!'
            end
          else
            @session_id = create_new_session
          end
        end
      ```
   5. Def self.create_token

      ```
        def self.create_token(user_name, moderator_name, session_id)
          @token = user_name == moderator_name ? @opentok.generate_token(session_id, { role: :moderator }) : @opentok.generate_token(session_id)
        end
      ```

* Ask about moderator role here

  Full WatchParty looks like:

  ```
  class WatchParty < ApplicationRecord
    require 'opentok'

    @opentok = OpenTok::OpenTok.new ENV['OPENTOK_API_KEY'], ENV['OPENTOK_API_SECRET']

    def self.create_or_load_session_id
      if WatchParty.any?
        last_session = WatchParty.last
        if last_session && last_session.expired == false
          @session_id = last_session.session_id
          @session_id
        elsif (last_session && last_session.expired == true) || !last_session
          @session_id = create_new_session
        else
          raise 'Something went wrong with the session creation!'
        end
      else
        @session_id = create_new_session
      end
    end

    def self.create_new_session
      session = @opentok.create_session
      record = WatchParty.new
      record.session_id = session.session_id
      record.save
      @session_id = session.session_id
      @session_id
    end

    def self.create_token(user_name, moderator_name, session_id)
      @token = user_name == moderator_name ? @opentok.generate_token(session_id, { role: :moderator }) : @opentok.generate_token(session_id)
    end
  end
  ```

10. Touch .env and Add our .env variables

    ```
    OPENTOK_API_KEY=''
    OPENTOK_API_SECRET=''
    MODERATOR_NAME=''
    PARTY_PASSWORD=''
    ```
11. rails db:create db:migrate

    `rails db:create db:migrate`
12. rails routes

    ```
    Rails.application.routes.draw do
      get '/', to: 'watch_party#home'
      get '/party', to: 'watch_party#party'
      post 'login', to: 'watch_party#login'
    end
    ```
13. rails g controller WatchParty home login party

    `rails g controller WatchParty home login party`

* Delete extra routes

14. Home page

* A bit rails magic with some vivid magic (add html to home.html.erb)

  ```
  <div class="card-wrapper">
    <vwc-card>
      <div slot="main" id="box">
        <vwc-text font-face="subtitle-1" >Big Game Watch Party
          <br>
          <span><vwc-text font-face="body-1-code">Built With Vonage Video Express on Rails</vwc-text></span></vwc-text>

        <%= form_with(url: "/login", method: "post") do %>
          <vwc-textfield name="name" label="Enter Your Name" icon="user" outlined="">
          </vwc-textfield>
          <vwc-textfield name="password" label="Enter Team Passcode" icon="lock" type="password" outlined="">
          </vwc-textfield>
          <div class="controls">
            <vwc-button layout="outlined" type="reset" outlined="">
              Reset
            </vwc-button>
            <vwc-button layout="filled" type="submit" unelevated="">
              Submit
            </vwc-button>
          </div>
        <% end %>
      </div>
    </vwc-card>
  </div>
  ```
* Uh oh, doesn't work
* import '@vonage/vivid'; in application.js

  `import '@vonage/vivid';`
* Works but ugly
* Add some css in watch_party.scss

  ```
  // Home Page Styles

  .card-wrapper{
    display: flex;
   }
   vwc-card{
     margin: auto;
     padding: 10%;
   }

   #box{
     padding: 50px 100px;
   }

  form {
    display: grid;
    gap: 20px;
   }
   .controls {
     display: flex;
     justify-content: flex-end;
     gap: 10px;
   }
  ```
* Before move on, let's add the beautiful Vonage background in application.scss

  ```
  body{
    background: linear-gradient(90deg, #9DD2FE 4.86%, #8728FB 96.11%);
    margin: 0px;
  }
  ```

15. Login page logic

* Add simple redirect logic

  ```
  class WatchPartyController < ApplicationController
    def home
    end

    def login
      @name = login_params[:name]
      if login_params[:password] == ENV['PARTY_PASSWORD']
        redirect_to party_path(name: @name)
      else
        redirect_to('/', flash: { error: 'Incorrect password' })
      end
    end

    def party
    end

    private

    def login_params
      params.permit(:name, :password, :authenticity_token, :commit)
    end
  end
  ```
* Need to capture name and password from form: login_params
* Don't forget to delete the view
* Ok now we're here on the page, what do we want to do?

16. Build structure

* Header
* Main
* For now removed the chat. Will update post with chat component from vivid

  ```
  <header>
    <%= render partial: 'header' %>
  </header>

  <main class="app">
    <!-- Video Chat Will Go Here -->
    <toolbar class="toolbar-wrapper">
      <%= render partial: 'toolbar' %>
    </toolbar>
  </main>
  ```
* Header and toolbar will hold the Vivid UI to control Video Express logic. Create empty partials

  `touch app/views/watch_party/_header.html.erb`

  `touch app/views/watch_party/_toolbox.html.erb`
* So all that remains on Party Page is Video Express. Lets build that now

17. Video Express

* Require library

  `<script src="https://static.opentok.com/v1/js/video-express.js"></script>`
* Add new VideoExpress.Room from documentation
* Note passing participant name. Video Express is lightweight but comes with some options, explore the docs

  ```
  <script>
    const room = new VideoExpress.Room({
     apiKey: '<%= @api_key %>', // add your OpenTok API key
     sessionId: '<%= @session_id %>', // add your OpenTok Session ID
     token: '<%= @token %>', // add your OpenTok token
     roomContainer: 'roomContainer',
     participantName: '<%= @name %>'
   });
   room.join();
  </script>
  ```
* Can see thatr roomContainer is targetting the roomContainer id, so need to do that and final html will look like:

  ```
  <script src="https://static.opentok.com/v1/js/video-express.js"></script>
  <header>
    <%= render partial: 'header' %>
  </header>

  <main class="app">
    <div id="roomContainer"></div>
    <!--  -->
    <toolbar class="toolbar-wrapper">
      <%= render partial: 'toolbar' %>
    </toolbar>
  </main>

  <script>
    const room = new VideoExpress.Room({
     apiKey: '<%= @api_key %>', // add your OpenTok API key
     sessionId: '<%= @session_id %>', // add your OpenTok Session ID
     token: '<%= @token %>', // add your OpenTok token
     roomContainer: 'roomContainer',
     participantName: '<%= @name %>'
   });
   room.join();
  </script>
  ```
* Rails s and….. Nothing!
* We haven't used any of that OpenTok logic to send to Video Express
* Need to set_opentok_vars in WatchParty Controller
* ```
    def set_opentok_vars
      @api_key = ENV['OPENTOK_API_KEY']
      @api_secret = ENV['OPENTOK_API_SECRET']
      @session_id = WatchParty.create_or_load_session_id
      @moderator_name = ENV['MODERATOR_NAME']
      @name ||= params[:name]
      @token = WatchParty.create_token(@name, @moderator_name, @session_id)
    end
  ```

  * Why before_action, and why verify_authenticity token?

  ```
    skip_before_action :verify_authenticity_token
    before_action :set_opentok_vars
  ```

  Full controller looks like:

Full Controller Looks Like:

```
class WatchPartyController < ApplicationController
  skip_before_action :verify_authenticity_token
  before_action :set_opentok_vars

  def set_opentok_vars
    @api_key = ENV['OPENTOK_API_KEY']
    @api_secret = ENV['OPENTOK_API_SECRET']
    @session_id = WatchParty.create_or_load_session_id
    @moderator_name = ENV['MODERATOR_NAME']
    @name ||= params[:name]
    @token = WatchParty.create_token(@name, @moderator_name, @session_id)
  end

  def home
  end

  def login
    @name = login_params[:name]
    if login_params[:password] == ENV['PARTY_PASSWORD']
      redirect_to party_path(name: @name)
    else
      redirect_to('/', flash: { error: 'Incorrect password' })
    end
  end

  def party
  end

  private

  def login_params
    params.permit(:name, :password, :authenticity_token, :commit)
  end
end
```

18. Now if we refresh…..we see that our camera turns on and audio goes all weird. But nothing on the screen. Why?! Video Express is working and we are connected to a video session in opentok. But haven't given the video screen any room

* Add some css for the video screen. Targeting #roomContainer.

  ```
  // Video Express Styles
  #roomContainer {
    width: 100vw;
    height: calc(100vh - 130px);
    position: relative;
  }
  #roomContainer > .OT_publisher {
    top: 25px;
    right: 25px;
    position: absolute;
    border-radius: 10px;
  }
  #roomContainer > .OT_screenshare {
    top: 25px;
    left: 25px;
    position: absolute;
    border-radius: 10px;
  }
  ```
* Boom now we have a video session.
* Try joining from multiple tabs/different names!
* Boom! You have video conferencing in Rails!
* Now let's add some of the functionality you expect in modern video conferencing:

  * screensharing: will be limited so that only our moderator can use it
  * Mute the room
  * Mute self
  * Hide self
  * Select Inputs: mic and camera
  * Select Audio Output
  * List other capabilities of Video Express to explore

**Rest of Tutorial Will be in JS/HTML**

19. Top - Bottom: Let's build that header
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
* * 3 times we'll need to toggle hide/show, let's build a small function toggleButtonView
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
  * Lasatly, Vivid gives us tooltips out of the box! Let's help our users understand the toolbar with tooltips!
