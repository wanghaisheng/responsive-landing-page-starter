---
title: "Create a Photo Booth with Vue.js and Flask: Part 1"
description: Build a Vue.js app that uses the Vonage Video API to take pictures
  from a video stream, including smile detection powered by the Azure Face API.
thumbnail: /content/blog/create-a-photobooth-with-vue-js-and-flask-part-1-dr/Part-1-1.png
author: diana-rodriguez
published: true
published_at: 2020-06-25T07:32:27.000Z
updated_at: 2021-05-04T15:49:50.456Z
category: tutorial
tags:
  - azure
  - video-api
  - photoboot
comments: true
redirect: ""
canonical: ""
---
**Smile! The web is your photo booth (Doofenshmirtz' Picture-Inator 2000)**

If you're a fan of Phineas and Ferb, then you will love this Dr. Doofenshmirtz themed photo booth application! We'll be splitting this tutorial into two parts—in the first, we'll show how the Vonage Video API can be used to capture a video stream from a webcam. This part includes how to set up an optional auto mode that uses the Azure Face API to detect the user's smile and take pictures automatically. The [second part of the tutorial](https://www.nexmo.com/blog/2020/06/26/create-a-photo-booth-with-vue-js-and-flask-part-2-dr) covers adding a "Filter-inator", which allows users to take pictures with filters, as well as how to use the Vonage Messaging API to send an SMS with a link to a selected file, so that users can share photos individually or as a photo booth film strip.

**Source:** [https://github.com/opentok-community/opentok-photobooth/tree/python-backend](https://github.com/opentok-community/opentok-photobooth/tree/python-backend)

**Deployed app:** [https://opentok-nexmo.azurewebsites.net](https://opentok-nexmo.azurewebsites.net)

## Objectives for Part 1

- Capture a stream into a canvas
- Add a **Snap Pic** button + counter to take a picture manually
- Add **Auto Mode** that enables pictures to be taken automatically
- Detect smile and snap!
- Add a hash to each canvas: Example 123456_0000-00-00.png
- Allow the images to be downloaded when clicked

## APIs Used

- [**Vonage Video API**](https://tokbox.com/account/user/signup) to stream the video from your webcam
- [**Azure Cognitive Services**](https://azure.microsoft.com/en-us/services/cognitive-services/face/), specifically the face recognition and sentiment APIs to detect emotions—in this case, smile/happiness

## Development Frameworks

- This part of the tutorial uses **[Vue.js](https://vuejs.org/)** to build out a front end.
- In [the next part](https://www.nexmo.com/blog/2020/06/26/create-a-photo-booth-with-vue-js-and-flask-part-2-dr), we show how to create a back end using **[Flask](https://flask.palletsprojects.com/en/1.1.x/)** to serve the **static** site (built for production) with an additional endpoint to send the snaps via SMS.

## Applications

A good starting point is to configure the applications/projects/services that we need to start development. Because each API mentioned here needs specific credentials to perform the operations required for the photo booth to work, let's go through the necessary steps for each API.

### Vonage Video API

To initialize a session (for streaming or starting a webcam), we need an `OPENTOK_API_KEY`, `OPENTOK_SESSION_ID`, and `OPENTOK_TOKEN`. Go to the [Video API Dashboard](https://tokbox.com/account/#/settings), authenticate, and create a new project.

Click on the sidebar and then click on *Projects > Create new project*. We are going to use the OpenTok API, so click on *Create custom project*, assign your project the name of your choice, and click the *Create* button. Once created, enter the project view, copy the API key, and proceed to generate the OpenTok session ID and token. For testing purposes, you can select an expiration time of one day—that's more than enough to start our application and our tests.

### Azure

For face/emotions detection, we are going to use Azure cognitive services. Go to the [Azure portal](portal.azure.com/) and authenticate.

- In the search bar write the following text: *cognitive services*
- In this section search for *Face*.
- Click on *Face* and then *Create*.
- After a few minutes, our face cognitive service will be created. Now from the service details, copy the **subscription key** and the **service endpoint**. We are going to use them to perform an HTTP request to evaluate what kind of emotions are detected from our snaps.


## Install Dependencies

We will be using **Vue.js** as our frontend framework of choice, but before we start developing the project, we need to install some packages using the [Node package manager (npm)](https://www.npmjs.com/).

Install **Vue CLI**. This is the core package—the Vue CLI is super useful to create new projects and add plugins to Vue applications.

```bash
npm install -g @vue/cli
```

Create a Vue application:

```bash
vue create opentok-nexmo
```

When the Vue CLI finishes this operation, the end result of our project will have the following structure:

```bash
- public/
- src/
- package.json
- package-lock.json
- babel.config.js
- vue.config.js
```

_**Note**: The most important directory here is `src/`, because this is where we will be adding our Vue files to create the front-end application._

Go to the application directory and add the **vuetify** plugin:

```bash
cd opentok-nexmo
vue add vuetify
```

Install other needed dependencies: Opentok to activate your webcam, and Axios to perform requests to the Azure API.

```bash
npm install opentok --save
npm install axios --save
```

For testing purposes, and to make sure there are no issues with the Vue project, run:

```bash
npm run serve
```

If your configuration is correct, you should be able to paste `http://localhost:8080/` in your browser and see the project's Vue.js placeholder page.

## Populate the Config File

It's time to start developing!! Remember the credentials we got at the beginning? It's time to use them. In the `src` directory, create a new file and name it `config.js`. Then add the following:

```javascript
module.exports['ENV'] = {
  OPENTOK_API_KEY: '',
  OPENTOK_SESSION_ID: '',
  OPENTOK_TOKEN: '',
  AZURE_FACE_API_SUBSCRIPTION_KEY: '',
  AZURE_FACE_API_ENDPOINT: '',
  SITE_URL: 'http://localhost:8080/',
}
```

Replace each of the empty strings with its corresponding value.

## Develop the App

We will develop our app in a single file. Go to `src` and look for the `App.vue` file, then delete the default content. 

Now before we start to add code, here is some useful information about Vue files.

A Vue file structure looks like:

```javascript
<template>
</template>

<script>
</script>

<stype>
</style>
```

As you can see, a Vue file has three sections:

- The **template section** contains all the HTML content that you need to structure your application (we will use some nice Vuetify components to reduce complexity).

- The **javascript** section contains all the JavaScript code that will be inserted, but be aware that you will be restricted to creating this code following the framework rules. This pertains to the lifecycle hooks, data section, components section, methods section and so on. For more information, you can check the official [Vue.js documentation](https://vuejs.org)

- The two sections mentioned above are the most important. But if you need to make style corrections you are free to add the third section (**style section**) where you can insert custom CSS.

Now back to our empty `App.vue`. Define the template section by adding the following code:

```html
<template>
  <v-app>
    <v-content>
      <v-row>
        <v-col cols="2">
          <v-navigation-drawer permanent class="options"> </v-navigation-drawer>
        </v-col>
        <v-col cols="8">
          <v-container> </v-container>
        </v-col>
        <v-col cols="2">
          <v-navigation-drawer permanent class="options"> </v-navigation-drawer>
        </v-col>
      </v-row>
    </v-content>
  </v-app>
</template>
```

You just defined the app layout! In the template section we use the **Vue grid system** (v-content, v-row, and v-col components) to define three other sections:

- The left section defines a drawer to customize our snaps.
- The right section defines a drawer too, where the captured snaps will be shown (with and without filters).
- The center section has a container that will show the user video stream detected by the camera.

We can now proceed to capturing video from the camera. Start by adding the following code to the center container:

```html
<v-container>
  <v-card class="mx-auto camera" max-width="500" outlined>
    <v-img
      :src="require('./assets/logo.png')"
      class="title"
      contain
      height="50"
    />

    <v-card-text>
      <div id="videos" align="center" justify="center">
        <div id="publisher"></div>
      </div>
      <img class="doof" src="/images/doof.png" />
    </v-card-text>
    <v-card-actions>
      <v-btn text class="analyze">
        <v-img :src="require('./assets/snap.png')" contain height="50" />
      </v-btn>
    </v-card-actions>
  </v-card>
</v-container>
```

We just added a card component to the container, but let's focus on the div with the id `publisher`. Inside this div, the Video API will display the video captured from the webcam.

Add the script section after the template section and insert the following JavaScript code:

```html
<script>
  import OT from "@opentok/client";
  import { ENV } from "./config";
  import axios from "axios";

  function handleError(error) {
    if (error) {
      console.log(error.message);
    }
  }

  export default {
    name: "App",

    components: {},

    data: () => ({
      //
      opentok_api_key: ENV.OPENTOK_API_KEY ? ENV.OPENTOK_API_KEY : "",
      opentok_session_id: ENV.OPENTOK_SESSION_ID ? ENV.OPENTOK_SESSION_ID : "",
      opentok_token: ENV.OPENTOK_TOKEN ? ENV.OPENTOK_TOKEN : "",
      publisher: null,
    }),
    mounted() {
      this.initializeSession();
    },
    methods: {
        async initializeSession() {
        var session = OT.initSession(
          this.opentok_api_key,
          this.opentok_session_id
        );

        // Create a publisher
        this.publisher = OT.initPublisher(
          "publisher",
          {
            insertMode: "append",
            width: 400,
            height: 300
          },
          handleError
        );

        // Connect to the session
        session.connect(this.opentok_token, error => {
          // If the connection is successful, initialize a publisher and publish to the session
          if (error) {
            handleError(error);
          } else {
            session.publish(this.publisher, handleError);
          }
        });
      }
    }
</script>
```

In the code above, we initialize the OpenTok session. Then all required packages are imported (OpenTok included), then a global/reusable function called `handleError` is defined (this function will be passed as a callback to the OpenTok methods just for error handling).

Maybe the most important part is `initializeSession`, defined in `methods`. Using the OpenTok credentials we create a session object. From there we  start the publisher (this will initialize the webcam and the browser will ask for permission to enable it). The last step of this method uses our session object to connect the publisher to the OpenTok session.

Then, in the Vue lifecycle hook called `mounted`, we call the `initializeSession` method. This is to start the camera when the document is ready. At this point you can reload your browser to confirm that your camera loads in the central section.

Now it's time to capture some frames from the video output and create a canvas image using the captured output! But remember that one of the objectives of this app is to also analyze the canvas and detect smiles. We'll also need to add a countdown to give users some time for posing.

Let's edit the data section in `App.vue`, because we need the Azure variables (containing the credentials) to make requests to the cognitive services:

```javascript
    data: () => ({
    opentok_api_key: ENV.OPENTOK_API_KEY ? ENV.OPENTOK_API_KEY : "",
    opentok_session_id: ENV.OPENTOK_SESSION_ID ? ENV.OPENTOK_SESSION_ID : "",
    opentok_token: ENV.OPENTOK_TOKEN ? ENV.OPENTOK_TOKEN : "",
    azure_face_api_subscription_key: ENV.AZURE_FACE_API_SUBSCRIPTION_KEY
      ? ENV.AZURE_FACE_API_SUBSCRIPTION_KEY
      : "",
    azure_face_api_endpoint: ENV.AZURE_FACE_API_ENDPOINT
      ? ENV.AZURE_FACE_API_ENDPOINT
      : "",
    site_url: ENV.SITE_URL ? ENV.SITE_URL : "",
    images: [],
    publisher: null,
    counter: 6,
    timerId: 0,
    manual: true,
    manual_label: "Manual",
    snackbar: false,
    snackbar_message: "",
    filters: false,
    filteredImages: [],
  }),
```

Our data section is growing nicely—we now have our Azure credentials in place for the requests, an image array to store the snaps, a counter to control the countdown, and a `timerId` to execute a `clearInterval` for resetting the countdown. We also have some variables that will be used to show a warning message indicating the status of the request, as well as the `manual` variable to switch between `manual` or `auto mode`. `filters` and `filteredImages` can be ignored for the moment.

### Manual Mode

Here we will create a method to analyze our shots, and if a smile is present then add the image to the drawer on the right side. In the `methods` section (after `initializeSession`), add the following methods:

```javascript
    dataURItoBlob(dataURI) {
      // convert base64/URLEncoded data component to raw binary data held in a string
      var byteString;
      if (dataURI.split(",")[0].indexOf("base64") >= 0)
        byteString = atob(dataURI.split(",")[1]);
      else byteString = unescape(dataURI.split(",")[1]);

      // separate out the mime component
      var mimeString = dataURI
        .split(",")[0]
        .split(":")[1]
        .split(";")[0];

      // write the bytes of the string to a typed array
      var ia = new Uint8Array(byteString.length);
      for (var i = 0; i < byteString.length; i++) {
        ia[i] = byteString.charCodeAt(i);
      }

      return new Blob([ia], { type: mimeString });
    },
    analyze() {
      this.images = []
      this.filteredImages = []
      this.filters = false

      this.timerId = setInterval(() => (this.counter -= 1), 1000);
      setTimeout(() => {
        clearInterval(this.timerId);
        this.counter = 6;

        let imageData = this.publisher.getImgData();
        let blob = this.dataURItoBlob("data:image/png;base64," + imageData);

        //Evaluates image emotion in azure cognitive face service
        const xhr = new XMLHttpRequest();
        xhr.open(
          "POST",
          `${this.azure_face_api_endpoint}face/v1.0/detect?returnFaceId=true&returnFaceLandmarks=false&returnFaceAttributes=emotion`
        );
        xhr.onreadystatechange = () => {
          let image = imageData;
          if (xhr.readyState === 4) {
            //Response from azure
            console.log(xhr.response);
            let response = xhr.response[0];
            if (response !== null && response !== undefined) {
              //Evalutes the face emotion happiness
              if (response.faceId !== null && response.faceId !== undefined) {
                console.log(response.faceId);
                if (
                  response.faceAttributes !== null &&
                  response.faceAttributes !== undefined
                ) {
                  if (
                    response.faceAttributes.emotion !== null &&
                    response.faceAttributes.emotion !== undefined
                  ) {
                    //Emotion is present, so we evaluate happiness factor (between 0 and 1) if happiness is > 0.5 we take the snap
                    if (
                      response.faceAttributes.emotion.happiness !== null &&
                      response.faceAttributes.emotion.happiness !== undefined
                    ) {
                      if (response.faceAttributes.emotion.happiness >= 0.5) {
                        //take the snap and put it in image array
                        this.images.push({
                          id: this.images.length + 1,
                          dataurl: "data:image/png;base64," + image
                        });
                      } else {
                        this.snackbar = true;
                        this.snackbar_message =
                          "Smiling is a requirement. Smile and we take your photo";
                      }
                    }
                  }
                }
              }
            } else {
              this.snackbar_message = "Connection error";
            }
          }
        };
        xhr.responseType = "json";
        xhr.setRequestHeader("Content-Type", "application/octet-stream");
        xhr.setRequestHeader(
          "Ocp-Apim-Subscription-Key",
          this.azure_face_api_subscription_key
        );
        xhr.send(blob);
      }, 6000);
    },
```

The methods defined above are fairly easy to explain. The first one (`dataURItoBlob`) analyses a base64 image data that comes from the OpenTok publisher (representing a single frame), converts this image data into a **blob** object and returns it. This image data conversion is important because Azure cognitive services requires blob objects.

The `analyze` method starts the countdown: `this.timerId = setInterval(() => (this.counter -= 1), 1000);`. When the counter gets to zero, the method cleans the interval and takes the snap from the camera using `let imageData = this.publisher.getImgData();`, then proceeds to transform this data into a blob `let blob = this.dataURItoBlob("data:image/png;base64," + imageData);`. After that, the method is ready to send the request to Azure to analyze the data.

An `XMLHttpRequest` is created, and from there we perform a **POST** request to the Azure endpoint (`${this.azure_face_api_endpoint}face/v1.0/detect?returnFaceId=true&returnFaceLandmarks=false&returnFaceAttributes=emotion`) in which we send the data `xhr.send(blob);`. Using `xhr.onreadystatechange`, we assign the callback to process the data when we have a response from Azure.

This callback validates the Azure response. Once validated, it evaluates if the `emotion` key is present and if the **happiness emotion** is greater than 0.5 (`if (response.faceAttributes.emotion.happiness >= 0.5)`). If the conditions are as expected, then the snap is pushed to the images array (this is a reactive action, which I will explain later on). If no smile is present (`response.faceAttributes.emotion.happiness < 0.5`) then we proceed to activate the `snackbar` (a Vue component for notifications that is activated by a reactive action) and we display a warning to the user letting them know that a smile is required.

What is a reactive action? A reactive action is when we change the state of a variable (for example a value) and this change is automatically reflected in the template where we are using this variable. We are not using this feature just yet, but we will on the next step. 

Let's modify our template. In the central section, below the publisher, we will be adding a button to take a picture. In the section on the right, we will loop through the array of **images** and then render each image inside the loop. (At the very beginning the `images` array is empty, which is why nothing is displayed on the right. When a user clicks on the take snap button a new image is added to the array and is rendered in the display in a **reactive way**.)

Let's edit the center section, and replace it with this:

```html
<v-container>
  <v-card class="mx-auto camera" max-width="500" outlined>
    <v-img
      :src="require('./assets/logo.png')"
      class="title"
      contain
      height="50"
    />

    <v-card-text>
      <div id="videos" align="center" justify="center">
        <div id="publisher">
          <v-overlay :absolute="true" :value="counter != 6">
            <div style="font-size:150px;">{{ counter }}</div>
          </v-overlay>
          <!--<div v-if="counter != 10" style="position:absolute;top:0px;font-size:150px;z-index:1000;">{{ counter }}</div>-->
        </div>
      </div>
      <img class="doof" src="/images/doof.png" />
    </v-card-text>
    <v-card-actions>
      <v-btn @click="analyze()" v-if="manual == true" text class="analyze">
        <v-img :src="require('./assets/snap.png')" contain height="50" />
      </v-btn>
    </v-card-actions>
  </v-card>
</v-container>
```

The `v-overlay` component is shown when the value is `true` and is hidden when the value is `false`. Because `counter` is reactive, while this variable is equal to six the component will be hidden. When we activate the countdown, the value of the counter decreases until zero then returns to six again (making this component invisible until the user takes another snap). This decrement is rendered in the `<div style="font-size:150px;">{{ counter }}</div>` statement. In other words, as the value decreases in JavaScript, it also decreases in the template.

We also assigned the `analyze()` method to the click event in the **v-btn** Vuetify component. The button has a `v-if="manual == true"`, indicating the button is going to be hidden if `automode` is enabled.

Now for the section on the right, let's add the following code:

```html
<v-navigation-drawer permanent class="options">
  <div style="padding: 10px">
    <v-row>
      <v-col cols="10" style="text-align:center;">
        <img src="/images/agents.png" height="22" />
      </v-col>
    </v-row>
    <v-divider></v-divider>
    <v-row style="margin-left:15px">
      <img
        v-for="image in images"
        :key="image.id"
        style="width:135px; height:auto; cursor:pointer;"
        :id="'snap_' + image.id"
        :src="image.dataurl"
        @click="forceFileDownload(image.id)"
      />
    </v-row>
  </div>
</v-navigation-drawer>
```

The above code, as previously explained, loops the **images** array and renders each image. An additional note to add here is the inclusion of the `forceFileDownload`, which is not defined yet. This is included so that when a user clicks on the image, the image gets downloaded.

So let's define the 'forceFileDownload' method. In the `methods` section, add this method definition after `analyze`:

```javascript
    forceFileDownload(index) {
      let imgs = null;
      imgs = this.images;
      let image_file = this.dataURItoBlob(imgs[index - 1].dataurl);
      const url = window.URL.createObjectURL(image_file);
      const link = document.createElement("a");
      link.href = url;
      link.setAttribute("download", "snap_" + index + ".png"); //or any other extension
      link.click();
    },
```

From the array index, we get the image we want to download. Then we transform the base64 text into a blob object (using `dataURItoBlob`). From there we create the object URL, we create a fake link (not added to body), and we assign the object URL to this link. Finally, we simulate a click with `link.click()`.

But where is the `snackbar` that will show the errors?. Let's create it. The `snackbar` is not going to be part of the left, right, or center because it is a reusable component. We will add it after the `v-content` component—the perfect place for a global element:

```html
    </v-content>
    <v-snackbar v-model="snackbar">
      {{ snackbar_message }}
      <v-btn color="pink" text @click="snackbar = false">Close</v-btn>
    </v-snackbar>
  </v-app>
```

When v-model is `false`, the component will be hidden. With `v-model="snackbar"` we can activate/deactivate the snack component by changing the value of the `snackbar` variable. Once the timeout ends (after a few seconds the snackbar disappears), the value of the snackbar variables return to `false`.

At this point, our app is capable of detecting a webcam, taking a picture, analyzing it, and if a smile is not detected, our `snackbar` shows a warning telling us that a smile is required. If a smile is detected, an image will be added in the right section. If you click the image, it should download.

### Auto Mode

When the user enables auto mode, the application takes several shots automatically and sends each photo to Azure to be analyZed. When a shot with a smile is detected, the `manual` state turns to `true` again (this disables auto mode). Next the application opens a dialog rendering all images with a smile in it. The user can choose the preferred image; the application adds the image to the right drawer and closes the dialog.

To achieve this we will add a switch component that allows us to "switch" between manual/automode. Let's add it to the left drawer:

```html
<v-navigation-drawer permanent class="options">
  <div style="padding: 10px">
    <v-img :src="require('./assets/manual.png')" class="title" contain />
    <v-switch v-model="manual" />
  </div>
</v-navigation-drawer>
```

The `v-model` from the switch component is linked with the `manual` variable, meaning when the user clicks the switch, the value of the variable is going to change (between true/false).

When auto mode is enabled, the *Take Snap* button disappears. Let's work on the dialog that loads the detected images. Add this code before Snackbar:

```html
<v-dialog v-model="dialog" persistent max-width="400">
  <v-card>
    <v-card-title class="headline">Do you like it?</v-card-title>
    <v-card-text>
      <p>
        A smile was detected. Select the image you like, when selecting the auto
        mode will be disabled to allow you work on the image.
      </p>
      <p>
        If you select No. This window is going to close and auto mode starts
        again
      </p>
      <img
        class="smile-images"
        v-for="image in images"
        :key="'snap_key_'+image.id"
        style="cursor:pointer;width:185px; height:auto;"
        :id="'snap_preview_' + image.id"
        :src="image.dataurl"
        @click="chooseImage(image.id)"
      />
    </v-card-text>
    <v-card-actions>
      <v-spacer></v-spacer>
      <v-btn
        color="green darken-1"
        text
        @click="dialog = false; manual = false; images = [];"
        >I dont like any</v-btn
      >
      <!--<v-btn color="green darken-1" text @click="dialog = false; manual = true; ">Yes</v-btn>-->
    </v-card-actions>
  </v-card>
</v-dialog>
```

As you can see, the code is pretty similar to the code used in the right drawer. This is because we are going to be reusing the images array to store the images that the application returns as valid. Then inside this dialog, we are going to render these images using a loop.

A couple of details to notice here are that when the user clicks on a single image, the application will execute the `chooseImage` method. This method adds the selected image to the section on the right, cleans the images array, and closes the dialog. To close the dialog we need to define the `dialog` variable which is assigned to the v-model of this component already.

Go to the data section and add the `dialog` field with false as a default value:

```javascript
  ......
  ......
    images: [],
    publisher: null,
    counter: 6,
    timerId: 0,
    manual: true,
    manual_label: "Manual",
    snackbar: false,
    snackbar_message: "",
    filters: false,
    filteredImages: [],
    dialog: false,
........
........
```

Then create the `chooseImage` method in the `methods` section immediately after `analyze`:

```javascript
    chooseImage(imgid) {
      this.manual = true;
      let image = this.images[imgid - 1];
      this.images = [];
      this.images.push({
        id: this.images.length + 1,
        dataurl: image.dataurl
      });
      this.dialog = false;
    },
```

This method enables the manual mode, assigns the selected images and closes the dialog.

But maybe you noticed there is some lost context here. When is this dialog going to be triggered? How will it be executed?. The key is the manual switch—**this dialog is only going to show when auto mode is enabled**. The **how** is the new method `analyzeAuto`, which sends an image (each second) to the Azure cognitive services to be analyZed.

It's time to define the `analyzeAuto` method. In the `methods` section, after the `analyze` definition, add the following code:

```javascript
    analyzeAuto() {
      this.filters = false
      //Each second an image is sent to azure to analyze if smile is present
      this.timerId = setInterval(() => {
        console.log("Intent");
        let imageData = this.publisher.getImgData();
        let blob = this.dataURItoBlob("data:image/png;base64," + imageData);
        //Evaluates image emotion in azure cognitive face service
        const xhr = new XMLHttpRequest();
        xhr.open(
          "POST",
          `${this.azure_face_api_endpoint}face/v1.0/detect?returnFaceId=true&returnFaceLandmarks=false&returnFaceAttributes=emotion`
        );
        xhr.onreadystatechange = () => {
          let image = imageData;
          if (xhr.readyState === 4) {
            //Response from azure
            console.log(xhr.response);
            let response = xhr.response[0];
            if (response !== null && response !== undefined) {
              //Evalutes the face emotion happiness
              if (response.faceId !== null && response.faceId !== undefined) {
                console.log(response.faceId);
                if (
                  response.faceAttributes !== null &&
                  response.faceAttributes !== undefined
                ) {
                  if (
                    response.faceAttributes.emotion !== null &&
                    response.faceAttributes.emotion !== undefined
                  ) {
                    //Emotion is present, so we evaluate happiness factor (between 0 and 1) if happiness is > 0.5 we take the snap
                    if (
                      response.faceAttributes.emotion.happiness !== null &&
                      response.faceAttributes.emotion.happiness !== undefined
                    ) {
                      if (response.faceAttributes.emotion.happiness >= 0.5) {
                        //take the snap and put it in image array
                        this.images.push({
                          id: this.images.length + 1,
                          dataurl: "data:image/png;base64," + image
                        });
                        this.dialog = true;
                        this.manual = true;
                      }
                    }
                  }
                }
              }
            } else {
              this.snackbar_message = "Connection error";
            }
          }
        };
        xhr.responseType = "json";
        xhr.setRequestHeader("Content-Type", "application/octet-stream");
        xhr.setRequestHeader(
          "Ocp-Apim-Subscription-Key",
          this.azure_face_api_subscription_key
        );
        xhr.send(blob);
      }, 1000);
    },
```

The differences between the `analyze` and `analyzeAuto` methods are minimal. The main difference is that `analyzeAuto` is taking captures from the camera each second and then creating a request for each captured image.

Now we need to define the **when**. As we mentioned before, the key is the `manual` variable. If `manual` is set to `false`, then auto mode is enabled. But how to detect the exact moment when this happens?. We will be using watchers—events that trigger when the state of variable changes.


Right after the `methods` section, add the following code:

```javascript
watch: {
  manual(val) {
      if (val) {
        //If Manual is selected stop the analyzeAuto function
        clearInterval(this.timerId);
      } else {
        this.analyzeAuto();
      }
  }
}
```

The explanation for this is brief. When the `manual` variable changes to `false`, the function `analyzeAuto` get executed. Then when `analyzeAuto` detects the first smile, `manual` changes to `false` and `clearInterval` is executed (these changes stop `analyze` automatically to prevent more requests).

At this point the application is capable of switching between manual snaps and auto mode. And that's all for the first part of this tutorial! Keep going with [Part 2](https://www.nexmo.com/blog/2020/06/26/create-a-photo-booth-with-vue-js-and-flask-part-2-dr), where we add filters to the images and allow users to send an SMS with a link to their photos.

