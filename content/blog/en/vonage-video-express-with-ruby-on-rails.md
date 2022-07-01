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

In this tutorial, I'll show you how to combine these powerful technologies to build a modern, fullstack application with all the video conferencing features that users have come to expect. I'll use Ruby on Rails, Vonage Video Express, and Vonage's new UI Toolkit Vivid.

This post is inspired by [Create a Party With Ruby on Rails and the Vonage Video API](https://developer.vonage.com/blog/20/05/12/create-a-party-with-ruby-on-rails-and-the-vonage-video-api-part-1-building-the-backend-dr). I will largely recreate the functionality and afterwards you can see just how much much less code was required!

## What The App Will Do

Back in 2020 Ben Greenberg built an app to allow his son to watch movies with his friends. This app will be essentially the same. Only I don't watch movies online with friends. But since moving across the world, it's been hard to find fans of my favorite ice hockey team, the Blues. So instead of watching movies with my friends, I'd love to watch the Blues or sports in general with friends from back home. Functionally, the app is the same and borrows heavily from Ben's movie app. This will make it easy to compare the pros/cons between using Video Express and writing in native Vonage Video API.

This app will have two pages; a landing page with a login form and a "party" page. When users login they will be taken to a "Chill Zone" to hang out. Here they will be displayed in equal size. Then when the game is beginning, the "moderator" who owns the session will trigger a screenshare to display the game. In the "Watch Mode", the shared screen of the moderator will be dominant and take up most of the video call. Because the best part of watching sports with friends is the banter, I will still see and hear the other participants. But if my friends are being too loud, I still want the ability to mute them. We'll also allow participants to mute themselves, turn off their video camera, and select their video/audio inputs and outputs.

**Yalla! Let's go!**

## Requirements

* [ruby v3.0.0+](https://www.ruby-lang.org/en/)
* [Rails v6.1.6+](https://rubyonrails.org/)
* [Vonage Video API](https://tokbox.com/account/user/signup)
* [Vonage Video Express v1.2.3+](https://tokbox.com/developer/video-express/)
* [Vivid v2.34.0+](https://vivid.vonage.com/)

## Vonage Video API Setup

It is free to create a Vonage Video API account. You need to do so in order to obtain your API key and secret, which are essential to making the app functional.

### Credentials

After you have [created an account with the Vonage Video API](https://tokbox.com/account/user/signup), you will see a dashboard interface. The first step in obtaining API credentials is to create a new project.

* Open the **Projects** tab from the left sidebar.
* Click the **Create New Project** option.
* Select **Vonage Video API** when asked what kind of project to create; *Create Custom Project*
* Provide any name for the project name
* Choose the VP8 codec option. (Details on the difference between VP8 and H.264 are detailed [here](https://tokbox.com/developer/guides/codecs/))
* You now have access to your project's API key and secret. Keep them somewhere safe, we will be using them soon.

### Enable Video Express

To use Video Express, be sure to activate the Video Express add-on for your account.

* Go to your [Video API Account](https://tokbox.com/account/) and click **Account Settings** in the left-hand menu.
* In the list of **Account add-ons**, find **Video Express** and click **Add to account**. Then follow the remaining instructions to enable the add-on.

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

We'll be using two Javascript libraries in the front end: Video Express and Vivid. Before we use them, let's get to know a little bit more about them.

#### Video Express

I promised that Video Express makes a developers life easier, but how? In primarily two ways: **Performance** and **Design**.

##### Performance

Video Express' **Quality Manager** continuously optimizes stream resolution, framerate, and rendering sizes. This is very important as the number of streams in a classic video conference session grows exponentially fast! For instance, 25 people in a video call mean that 625 streams are concurrently active.

The quality manager works to upgrade and downgrade resolution as networks and CPU allow, pause non-visible video streams and muted audio streams, and request smaller video streams from media servers when displayed videos becomes smaller.

This can result in **60% decreased bandwith** usage for 10 participant sessions and **80% decreased bandwith** usage for 25 participant sessions!

##### Design

Since the beginning of COVID, the world has become intimately familiar with video conferencing. Everyone at this point more or less knows what to expect from a video conference. And for most applications, there's no need to reinvent the wheel. Video Express does the heavy lifting with its **Layout Manager** and **Experience Manager**.

The Layout Manager handles the responsiveness of the video call, automatically adjusting the video windows as participants leave and join, screenshare, and optimizes video resolutions and frame rates based on the rendering sizes.

The Experience Manager dynamically sets speaker priority and auto-mutes joiners for larger meetings.

All of this heavy lifting means Video Express has a ton of features built that just need to be hooked up to a UI. Video Express gives you all of thes out of the box:

* Detecting when users do actions like join/leave, activate their cameras/audio
* Different layout options: grid vs active speaker
* Changing the camera and microphone used
* Setting the audio output device
* Creating a preview publisher
* Active speaker detection
* Accessing the screen-sharing publisher's audio/video
* Detecting when other clients publish screen-sharing streams
* Enabling and disabling a screen-sharing subscriber's audio and video 

#### Vivid

As I said Video Express has all the front-end functionality built out, it just needs a developer to build out a UI for the end user. Vonage Does That! We've been building a gorgeous UI Toolkit called Vivid which makes building applications with communications features much faster.

Vivid is built using Web Components so they will work in any framework or even vanilla HTML/JS like Rails. And they look great! And they're [web accessible](https://developer.vonage.com/blog/21/11/11/wcag-how-to-implement-web-accessibility-1)!

[Learn more about Vivid](https://github.com/Vonage/vivid)

### Node Modules Install

Now that I've convinced you that Video Express and Vivid are great, let's install them:

`yarn add @vonage/video-express @vonage/vivid`

### Model Generation

Next, we will generate a model to hold and manipulate the watch party information. In order to connect to users to the same video call, we will need to pass Vonage Video API the `session_id`. From the command line execute the following:

`rails g model WatchParty session_id:string`

Before we run the migration to create this data type in our database, we'll need to update in the migration to ensure that sessions don't default to null when need to retrieve them.

Open the `db/migrate` directory and find the file called:  TIMESTAMP_create_watch_parties.rb

```
//In TIMESTAMP_create_watch_parties.rb
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

You can now commit this database migration to the schema by running From the command line run:

`rails db:create db:migrate rake db:create`

This command will create the PostgreSQL database and the sessions table with the session_id column.

### Creating the Model Methods

Now let's implement our watch party logic will use the Vonage Video API to connect users to a video call session.

Each Vonage Video session has its own unique session ID. This session ID is what enables different participants to join the same video chat. Additionally, each participant in the video chat is granted a [token](https://tokbox.com/developer/guides/basics/#token) that enables them to participate. A token can be given special permissions, like moderation capabilities.

In the Session model we are going to create three class methods that will be used to either create a new session ID or load the previous one, and generate tokens for each participant.

Learn more about [Vonage Video API Sessions](https://tokbox.com/developer/guides/basics/#sessions).

Open `app/models/watch_party.rb`

First, we need to access our Vonage Video API functionality by instantiating an instance of the OpenTok Ruby SDK. We'll pass our API_KEY and API_SECRET from the credentials section above through ENV environment variables

```
require 'opentok'
@opentok = OpenTok::OpenTok.new ENV['OPENTOK_API_KEY'], ENV['OPENTOK_API_SECRET']
```

Now we can add those model methods. The Session#create_or_load_session_id method will check to see if there already is a session ID. If there is an ID, it will use that ID. If not, it will generate a new one.

```
def self.create_or_load_session_id
  if WatchParty.any?
    last_session = WatchParty.last
    if last_session
      @session_id = last_session.session_id
      @session_id
    elsif !last_session
      @session_id = create_new_session
    else
      raise 'Something went wrong with the session creation!'
    end
  else
    @session_id = create_new_session
  end
end
```

The above method also references an additional method we need to create called Session#create_new_session that does the work of creating a new session if one does not exist:

```def
  session = @opentok.create_session
  record = WatchParty.new
  record.session_id = session.session_id
  record.save
  @session_id = session.session_id
  @session_id
end
```

Lastly, we will create a method that will assign the right token for each participant:

```
def self.create_token(session_id)
  @token = @opentok.generate_token(session_id)
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

### Setting our ENV variables

We saw that our Video API logic requires the use of some secret environment variables. Let's set those now.

Create the `.env` file from the root of the `video-express` project:
`touch .env`

```
OPENTOK_API_KEY=''
OPENTOK_API_SECRET=''
MODERATOR_NAME=''
PARTY_PASSWORD=''
```

Here you will need to add your Video API credentials from above. In a real app you would want to store information about moderators and passwords in your database but for this demo, storing in an ENV variable does the trick!

Don't forget to add a `MODERATOR_NAME` and `PARTY_PASSWORD`to use in the login page.

### Defining The Routes

The user will see two pages: home and party. But we also need to catch the info from the login form to check that the password is correct and if the user is the moderator. So we will create two get requests and one post:

````
```
Rails.application.routes.draw do
  get '/', to: 'watch_party#home'
  get '/party', to: 'watch_party#party'
  post 'login', to: 'watch_party#login'
end
```
````

### Creating The Controller

With Rails "Convention over Configuration" we already know what actions we'll need for our controller, the same as the routes!

From the command line generate the WatchParty controller with home, login, and party actions"

`rails g controller WatchParty home login party`

* The rails generator will some extra stuff we don't want. So now go and delete the routes it generated in `app/config/routes.rb`. And completely delete login view file; `app/views/login.html.erb`

At this point with all the main config and setup done, I just like to go one action/view pair at a time until I finish an application. 

### Building The Home Page

The home page we want to build will look like this:

![Watch Party Home Page](/content/blog/vonage-video-express-with-ruby-on-rails/macbook-pro-16_-1-1-.png "Watch Party Home Page")

If we look at the Vivid documentation we see that we have all the components we need:

* [Card](https://vivid.vonage.com/?path=/story/components-card--introduction)
* [Text](https://vivid.vonage.com/?path=/story/alpha-components-text--introduction)
* [Forms](https://vivid.vonage.com/?path=/story/components-textfield--login-form)
* [Buttons](https://vivid.vonage.com/?path=/story/components-button--basic&args=label:Basic;layout:text)

So with bit rails magic with some vivid magic, we have everything we need. There are only two tricky things here. One, you'll notice that cards can have titles and subtitels in Vivid. So why do we need to use the vwc-text component? Because Vivid makes use of slots in their web components and when using the `slot="main"` to have content in the card, this overrides the slots of the title and subtitle.

Secondly, the form doesn't help us actually pass the data to the server. So we need to make use of the Rails helper `form_with`. Altogether it looks like this:

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

So now if we run our server we should have a beautiful home page like in the picture right? So try it! Run `rails s` from your command line and open `localhost:3000` in your browser.

**Uh oh, doesn't work!**

Because although we have Vivid in our project, we haven't actually imported it in the code to use it. So now we'll do that in our Javascript. In `app/javascript/packs/application.js` add this line under `import "channels`:

`import '@vonage/vivid';`

Now if you open the page it will work. But it's still pretty ugly. So let's style it. In `app/assets/stylesheets/watch_party.scss` let's add some css:

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

Before we move on, let's add the beautiful Vonage background in `app/assets/stylesheets/application.scss`

```
body{
  background: linear-gradient(90deg, #9DD2FE 4.86%, #8728FB 96.11%);
  margin: 0px;
}
```

#### Defining The Login

We have a form but we want to make sure only real fans aka our friends can join our watch party! So we'll add some Rails Strong Parameters logic to catch the `name` and `password` from the parameters sent in the form on the home page. 

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

Successful logins will redirect to the `party_path`. This path will lead users to the party page, the heart of the app where our Video Express room will live. Let's build it!

## Building The Party Page

First let's take a look at what we're building. This is the Party page in the "Chill Mode" as a non-moderator.

![Party Page: Chill Mode, Non Moderator](/content/blog/vonage-video-express-with-ruby-on-rails/macbook-pro-16_-10.png "Party Page: Chill Mode, Non Moderator")

And this is how it will look in the "Watch Mode" as a moderator.

![Party Page: Watch Mode, Moderator](/content/blog/vonage-video-express-with-ruby-on-rails/macbook-pro-16_-12.png "Party Page: Watch Mode, Moderator")

There's one thing to note here. In the header, only the moderator has a switch controller to trigger the change in modes. 

### Building The Structure

Let's start with the bones of the page, the HTML. We have 3 components; a header, the video call, and a toolbar. Add this to your `app/views/video/party.html.erb`:

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

The header and toolbar will hold the Vivid UI to control our Video Express logic. For now, build empty partials:

`touch app/views/watch_party/_header.html.erb`

`touch app/views/watch_party/_toolbox.html.erb`

So now all that remains on the Party Page is adding our video call with Video Express. Lets build that now.

### Building A Room With Video Express

The Vonage Video API gives developers full control of customizing their video layout by manipulating `publisher` and `subscriber` elements. In Video Express, these are replaced with the concept of a `room`. To add this is as easy as 1, 2, 3.

 **Do I still need this if I have Video Express installed and exposed in packs?** 

1. Include the library. Add this script at the top of the file:
   `<script src="https://static.opentok.com/v1/js/video-express.js"></script>`
    **Do I still need this if I have Video Express installed and exposed in packs?** 
2. Now run create the room with the sample code from the Video Express documentation. Note that we pass the additional parameter `participantName`. Video Express is lightweight but comes with some options, explore the docs!

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

3. We can see that the roomContainer is looking for the entry point in the html where to latch onto and embed the Room. So we need to create an element with an id of `roomContainter`. 

The final code of the `party.html.erb` looks like this:

```
<script src="https://static.opentok.com/v1/js/video-express.js"></script>
<header>
  <%= render partial: 'header' %>
</header>

<main class="app">
  <div id="roomContainer"></div>
  <!--  -->
  <toolbar>
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

So now we should be able to run our server and see a beautiful video call site, right? You should know the answer is no by now 😆. What are we missing? Well we haven't used any of that OpenTok logic from the Video API to send to Video Express.

So we need to set our OpenTok variables in the `WatchParty Controller` and pass them through to our frontend.

In the WatchParty Controller we need to add the `set_opentok_vars` action"

```
    def set_opentok_vars
      @api_key = ENV['OPENTOK_API_KEY']
      @api_secret = ENV['OPENTOK_API_SECRET']
      @session_id = WatchParty.create_or_load_session_id
      @moderator_name = ENV['MODERATOR_NAME']
      @name ||= params[:name]
      @token = WatchParty.create_token(@name, @moderator_name, @session_id)
    end
```

We will use the `before_action` which will call the method before entering the `party` action. So the full Controller looks like:

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

Now if we refresh… we see that our camera turns on and the audio goes all weird. But nothing shows on the screen. Why?! Video Express is working and we are connected to a video session through the Video API. But we haven't given the video screen any room to exist!

Let's add some CSS from the Video Express boilerplate for the video screen. We target the #roomContainer and give it full width and full height, minus the header. We also add a bit of CSS to tell Video Express where to put **What is the OT_publisher?** and the screenshare when it is triggered.

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

 **Boom!** now we have a video session. Try joining from multiple tabs/different names!  **Boom!** You have video conferencing in Rails!

Now let's add some of the functionality you expect in modern video conferencing:

* Screensharing: will be scoped so that only our moderator can use it
* Mute all other participants
* Mute yourself
* Hide yourself (turn off camera)
* Select Inputs: mic and camera
* Select Audio Output

## Building Out Helper Components

The rest of this tutorial will be building the components to users control their Video Express room. Each component will follow a similar structure: HTML with Vivid components and Javascript to trigger Video Express functions.

### Building The Header HTML

A reminder of the header we want to build:

![The Header in Moderator View](/content/blog/vonage-video-express-with-ruby-on-rails/header.png "The Header in Moderator View")

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

We'll need to add some Javascript. So let's create a compenents folder in `app/javascript` with the following command

`mkdir app/javascript/componenets`

And here we'll add the a file `header.js`, again from the command line:
`touch app/javascript/components/header.js`.

And we need to make sure that webpacker is serving our Javascript to the application. So inside of `app/javascript/packs/application.js`, we need to require the component:

```
require("components/header");
```

Inside the JS file we'll have two parts: an event listener that will toggle the Video Express ScreenSharing and a function to iterate through all participants in the room to update their display.

First, let's look at the `toggleParticipants`. It toggles through each participants window to update the layout. This is important to note because the `room` is unique to each participant, so updates must be done to each user.

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

So to disable the audio, we need to iterate through all the participants in the room and use the `.camera.disable()` function. You'll notice that we call this on participant\[1] because the participant object returns an array; [`id`, participantObject]. Additionally room.participants does not include the local user, only all the other participants. So we 
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