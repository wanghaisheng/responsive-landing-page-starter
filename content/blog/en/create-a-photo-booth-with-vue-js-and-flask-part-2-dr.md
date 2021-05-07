---
title: "Create a Photo Booth with Vue.js and Flask: Part 2"
description: Connect a Flask back end to your Vue.js photo booth app and add the
  ability to apply filters and send an SMS with a link to your photos.
thumbnail: /content/blog/create-a-photo-booth-with-vue-js-and-flask-part-2-dr/Part-1-2.png
author: diana-rodriguez
published: true
published_at: 2020-06-26T14:17:20.000Z
updated_at: 2021-05-04T15:58:03.632Z
category: tutorial
tags:
  - flask
  - vuejs
  - sms-api
comments: true
redirect: ""
canonical: ""
---
In [Part 1](https://www.nexmo.com/blog/2020/06/25/create-a-photobooth-with-vue-js-and-flask-part-1-dr) of this tutorial, we looked at how to use [Vue.js](https://vuejs.org/) and the [Vonage Video API](https://tokbox.com/account/user/signup) to create a photo booth app, complete with smile detection powered by the [Azure Face API](https://azure.microsoft.com/en-us/services/cognitive-services/face/). In this post, you'll learn how to add filters to your snaps, how to download all the images you've created, and how to send an SMS message to share a link to your images. Along the way, we'll build out a [Flask](https://flask.palletsprojects.com/en/1.1.x/) back end to serve up the application and handle the SMS messaging.

**Source:** <https://github.com/opentok-community/opentok-photobooth/tree/python-backend>

**Deployed app:** [https://opentok-nexmo.azurewebsites.net](https://github.com/opentok-community/opentok-photobooth/tree/python-backend)

<sign-up number></sign-up>

## Filters

An important feature for our photo booth is the capability to apply filters (effects) to a selected snap. Once you transform your captured photo from the camera into a canvas, you can manipulate the image as you wish. For our app, we'll have four filters: Grayscale, Sepia, Green, and Blue.
In terms of functionality, in the left drawer, we are going to add a **switch** component for filters. This **switch** component will be linked with the `filters` variable (`false` by default). When the switch component changes from `false` to `true`, a **watcher** executes the method in charge of generating the filtered images and adding them to the drawer on the right.

Let's start by adding the switch component. In the left drawer section, paste the following code after the `manual` switch:

```html
<v-img :src="require('./assets/filter.png')" class="title" contain />
<v-switch v-model="filters"></v-switch>
```

For filter generation, we will define a method called `imgwfilter` as follows:

```javascript
    imgwfilter(img, filter, density) {
      //console.log(img)
      let r = (filter.r * density + 255 * (100 - density)) / 25500;
      let g = (filter.g * density + 255 * (100 - density)) / 25500;
      let b = (filter.b * density + 255 * (100 - density)) / 25500;
      var canvas = document.createElement("canvas");
      //canvas.width = img.width;
      //canvas.height = img.height;
      canvas.width = 640;
      canvas.height = 480;
      var ctx = canvas.getContext("2d");
      ctx.drawImage(img, 0, 0);
      //Change pixel color tone
      var imageData = ctx.getImageData(0, 0, 640, 480);
      var data = imageData.data;
      for (var i = 0; i < data.length; i += 4) {
        var luma = 0.299 * data[i] + 0.587 * data[i + 1] + 0.114 * data[i + 2];
        data[i] = Math.round(r * luma);
        data[i + 1] = Math.round(g * luma);
        data[i + 2] = Math.round(b * luma);
      }
      //Rewrite data in canvas
      ctx.putImageData(imageData, 0, 0);
      //Add filtered images to document
      this.filteredImages.push({
        id: this.filteredImages.length + 1,
        dataurl: canvas.toDataURL()
      });
    }
```

This method receives two parameters: the first is the color pattern (the filter to apply), and the second is the density. Using the selected image, the method creates a new canvas, then loops through each pixel in the image applying the filter and adding the new image to the `filteredImages` array.

This method should be called when the `filters` switch changes to `true`. To capture this specific moment, we are going to create a new watcher for the `filters` variable. After the `manual` watcher, add the following code:

```javascript
    filters(val) {
      if (val) {
        //If filters are enabled, generate 4 images with effects
        this.filteredImages = [];
        if (this.images.length > 0) {
          //Grayscale
          this.imgwfilter(
            document.getElementById("snap_1"),
            { r: 0xff, g: 0xff, b: 0xff },
            50
          );
          //Sepia
          this.imgwfilter(
            document.getElementById("snap_1"),
            { r: 0xac, g: 0x7a, b: 0x33 },
            30
          );
          //Green
          this.imgwfilter(
            document.getElementById("snap_1"),
            { r: 0x19, g: 0xc9, b: 0x19 },
            30
          );
          //Blue
          this.imgwfilter(
            document.getElementById("snap_1"),
            { r: 0x1d, g: 0x35, b: 0xea },
            30
          );
        }
      }
    },
```

This watcher evaluates the moment the value changes to `true`. Once this happens, the watcher checks if an image exists to apply these filters to. If an image exists, the application generates four filtered images using the original image as a base. 

The next step is rendering the filtered images in the template section. For this, we are going to use the right-side drawer. Add the following code right after the images loop:

```html
<img
  v-for="filteredImage in filteredImages"
  :key="'key_'+filteredImage.id"
  style="width:135px; height:auto; cursor:pointer;"
  :id="'filtered_' + filteredImage.id"
  :src="filteredImage.dataurl"
  @click="forceFileDownload(filteredImage.id,'filtered')"
/>
```

This code will be renders each filtered image in the drawer on the right. When the user clicks on an image, it will trigger a download. 

In the last part of the tutorial, `forceFileDownload` received a single parameter. We need to modify it to reuse it for the filtered images too. After modifications, we will have something like this:

```javascript
    forceFileDownload(index, place) {
      let imgs = null;
      if (place == undefined) imgs = this.images;
      else imgs = this.filteredImages;
      let image_file = this.dataURItoBlob(imgs[index - 1].dataurl);
      const url = window.URL.createObjectURL(image_file);
      const link = document.createElement("a");
      link.href = url;
      link.setAttribute("download", "snap_" + index + ".png"); //or any other extension
      link.click();
    },
```

We've modified several of the lines so that if the `place` parameter is not provided the application is going to download an image from the `images` array, and if the `place` parameter is equal to `filtered` then the method downloads a single image from the `filteredImages` array.

## Save All Images at Once

A bonus option we can include in the configuration section (the left section) is to download all images by clicking a single button. For this, we are going to add the following code inside the left section just right after the filters switch:

```html
<br />
<br />
<v-btn
  @click="downloadImages()"
  style="background-color:inherit !important; padding: 0 !important;"
>
  <img src="/images/download.png" height="40px" />
</v-btn>
```

There isn't a lot to say about this code, other than we added a single button and assigned the `downloadImages` method to the click event. Now we define this function in the `methods` section as follows:

```javascript
    downloadImages() {
      for (let i = 0; i < this.images.length; i++) {
        this.forceFileDownload(this.images[i].id);
      }
      for (let f = 0; f < this.filteredImages.length; f++) {
        this.forceFileDownload(this.filteredImages[f].id, "filtered");
      }
    },
```

The `downloadImages` method loops through the images array (original images), and the filtered images array (images with effects), then downloads every single image reusing the `forceFileDownload` method.

## Send a Link to Images through SMS

The only thing left is to provide a user interface to share these snaps with others using the Vonage Messaging API. First, we need to define additional variables in the `data` section to control certain states:

```javascript
data: () => ({
  ......
  ......
  nexmo_dialog: false,
  phone: "",
  sel2next: null,
  self2nextAlias: "",
  stripedimage: ""
  ......
  ......
})
```

Next, we create a button on the left-side drawer:

```html
<v-btn
  @click="nexmo_dialog=true; getStripImage()"
  style="background-color:inherit !important; padding: 0 !important; text-align:center;"
>
  <img src="/images/sms.png" height="40px" />
</v-btn>
```

This button is going to show us a dialog with the form UI to send our message, and at the same time it is going to create the image strip (using a method that merges all filtered images into a single image). The method is called `getStripImage`.

At the end of the `methods` section add:

```javascript
    getStripImage() {
      let x_increment = 640;
      let y = 0;
      let x = -640;
      let pic_number = 4;
      var canvas = document.createElement("canvas");
      canvas.width = 640 * pic_number;
      canvas.height = 480;
      var ctx = canvas.getContext("2d");
      for (let f = 0; f < this.filteredImages.length; f++) {
        x = x + x_increment;
        let img = document.getElementById(
          "filtered_" + this.filteredImages[f].id
        );
        ctx.drawImage(img, x, y);
      }
      this.stripedimage = canvas.toDataURL();
    }
```

Using the default width and height of a single OpenTok `publisher`, this method creates a horizontal canvas with these dimensions: (width*4,height). From there, it merges the images next to each other before the final result is stored in the `stripedimage` variable.

Now let's add our invisible dialog to the Vue template. We are going to place it right after the end of the `v-content` component:

```html
<v-dialog v-model="nexmo_dialog" persistent max-width="400">
  <v-card>
    <v-card-title class="headline">
      <v-img
        :src="require('./assets/sms-title.png')"
        class="title"
        contain
        height="50"
      />
    </v-card-title>
    <v-card-text>
      <v-text-field
        v-model="phone"
        label="Enter your phone number: Eg: 12023334455"
        required
      ></v-text-field>
      <p>Select one of the following images</p>
      <img
        v-for="image in images"
        :key="'snap_key_'+image.id"
        style="cursor:pointer;width:100px; height:auto;"
        :id="'snap_preview_' + image.id"
        :src="image.dataurl"
        :class="'image-selection'+((('snap_preview_' + image.id) == self2nextAlias)?' choosenone':'')"
        @click="selected2Nexmo(image.id); self2nextAlias='snap_preview_' + image.id;"
      />
      <img
        v-for="filteredImage in filteredImages"
        :key="'snap_filtered_key_'+filteredImage.id"
        style="cursor:pointer;width:100px; height:auto;"
        :id="'snapfiltered_preview_' + filteredImage.id"
        :src="filteredImage.dataurl"
        :class="'image-selection'+((('snapfiltered_preview_' + filteredImage.id) == self2nextAlias)?' choosenone':'')"
        @click="selected2Nexmo(filteredImage.id, 'filtered'); self2nextAlias='snapfiltered_preview_' + filteredImage.id;"
      />
      <img
        style="cursor:pointer;width:300px; height:auto;"
        id="strip_image"
        :src="stripedimage"
        :class="'image-selection'+((('striped') == self2nextAlias)?' choosenone':'')"
        @click="selected2Nexmo('striped', 'striped'); self2nextAlias='striped';"
      />
    </v-card-text>
    <v-card-actions>
      <v-spacer></v-spacer>
      <v-btn color="green darken-1" text @click="nexmo_dialog=false"
        >Cancel</v-btn
      >
      <v-btn color="green darken-1" text @click="sendMMS()">Send</v-btn>
    </v-card-actions>
  </v-card>
</v-dialog>
```

This form allows users to indicate the phone number where the selected image will be sent. When the user selects the image, `selected2Nexmo` is executed. This method refreshes the value of the variable `sel2next` that represents the selected image. Both the phone number and the image are required. If you don't provide one or the other, you will see a warning.

To define the `selected2Nexmo` method, add the following code at the end of the `methods` section:

```javascript
  selected2Nexmo(imgid, filtered) {
      if (filtered == undefined) this.sel2next = this.images[imgid - 1].dataurl;
      else if (filtered == "striped") this.sel2next = this.stripedimage;
      else this.sel2next = this.filteredImages[imgid - 1].dataurl;
  },
```

This method receives two parameters: the id of the image (`imgid`) and the category (`filtered)`. Depending on the category (striped, filtered, or undefined), the application decides where to search. In the case of striped, the `imgid` is not needed because `stripedimage` is not an array.
When the user clicks the `sendMMS` button the form is sent. 

Add the `sendMMS` method to the `methods` section as follows:

```javascript
    sendMMS() {
      if (this.phone == "" || this.self2nextAlias == "") {
        this.snackbar_message =
          "Fields required. Please ensure to fill the phone and select an image.";
        this.snackbar = true;
      } else {
        //Send MMS
          axios
          .post(this.site_url + "/send-mms", {
            phone: this.phone,
            image: this.sel2next
          })
          .then(response => {
            console.log(response);
            if (response.data.status == "success") {
              this.snackbar_message = "Your message was sent successfully";
              this.snackbar = true;
              this.nexmo_dialog = false;
            } else {
              this.snackbar_message = "Error: " + response.data.message;
              this.snackbar = true;
            }
          })
          .catch(error => {
            console.log(error);
            this.snackbar_message = error;
            this.snackbar = true;
          });
      }
    }
```

The `sendMMS` function validates the required fields, and if there are empty fields the application is going to show us the `snackbar` with any error messages.

If data was provided, then the application performs a post request to our backend endpoint using Axios. As we mentioned at the beginning of this tutorial, our backend is going to serve the static site (built using the vue-cli) and also will be serving as an endpoint to send the SMS (`this.site_url + "/send-mms"` represent this endpoint).

We provide the data to send (in this case the phone number and the image), and then we wait for the response from the service using Axios promises. The `catch` statement is triggered when an error occurs, and a `snackbar` will show the details related to this error. We use the `then` statement to evaluate if the response succeeds—if so, the application shows a `snackbar` indicating the message was sent successfully.

## Build a Back End

In this section, we are going to configure a Flask server. This back-end application is going to serve the static front end (built with vue-cli) and defines the `/send-mms` endpoint to send the SMS with the selected image.

***Note**: Compatible Python versions (Python >= 3.6, pip3)*

### Install Dependencies

To install required modules, use this `requirements.txt` file as a reference. This one was created with `pip freeze` and contains the modules you need to install. Copy it and save it as `requirements.txt`.

```bash
certifi==2020.4.5.1
cffi==1.14.0
chardet==3.0.4
click==7.1.2
cryptography==2.9.2
Flask==1.1.2
Flask-Cors==3.0.8
idna==2.9
itsdangerous==1.1.0
Jinja2==2.11.2
MarkupSafe==1.1.1
nexmo==2.4.0
pkg-resources==0.0.0
pycparser==2.20
PyJWT==1.7.1
python-dotenv==0.13.0
pytz==2020.1
requests==2.23.0
six==1.14.0
urllib3==1.25.9
Werkzeug==1.0.1
gunicorn
```

Then execute:

```bash
pip install -r requirements.txt
```

### Create the Environment

Once dependencies have been installed by pip, create your `backend` directory (we recommend you create this folder alongside the Vue application directory).

```bash
mkdir backend
cd backend
```

Once inside the `backend` directory, create the `.env` file. We are going to define some environment variables here related to the Vonage account credentials:

```javascript
NEXMO_API_KEY = ''
NEXMO_API_SECRET = ''
NEXMO_NUMBER = ''
SITE_URL = ''
```

Replace the empty strings with your Vonage account credentials, which you can find on [the dashboard](https://dashboard.nexmo.com/). Be sure to include your virtual number and the `SITE_URL` (the URL where your application is running, e.g. http://localhost).

### Create the Server

Create the file `server.py` and add the following code:

```python
#re module is for regular expression ops
import nexmo, re, base64
from flask import Flask, request, jsonify, send_from_directory
from flask_cors import CORS
from dotenv import load_dotenv
from os import environ, makedirs
from os.path import join, dirname, abspath, exists
from datetime import datetime
```

The code above imports all the modules that our Flask application needs to work correctly. Among the most important we have:

* nexmo: for sending an SMS
* flask: for creating our backend application
* flask_cors: to make our `/send-mms` endpoint accessible

We can init our Flask application by adding the following:

```python
app = Flask(__name__, static_url_path='PATH_TO_YOUR_STATIC', static_folder = 'dist')
CORS(app)
#10mb size is allowed
app.config['MAX_CONTENT_LENGTH'] = 10 * 1024 * 1024
```

***Note**: Pay attention to the `app.config['MAX_CONTENT_LENGTH'] = 10 * 1024 * 1024` line. Here we are telling Flask to allow big files to be uploaded. This is because we will be sending the selected image in the Axios request, and these images tend can be quite big.*

The next step is to initialize the `nexmo` client. As you can see, first we load the environment file to be able to extract the credentials. Then we create our client:

```python
#Get env vars from file
envpath = join(dirname(__file__),"./.env")
load_dotenv(envpath)
#Needed for upload files
basedir = abspath(dirname(__file__))
client = nexmo.Client(key=environ.get("NEXMO_API_KEY"), secret=environ.get("NEXMO_API_SECRET"))
```

***Note**: We define the `nexmo` client as a global variable to be available wherever it is needed.*

Next, we check to see if the `snap` directory exists inside the `dist` directory. Using the `basedir`, the application verifies if this directory exists, and if not, it creates it. When the client sends the selected image, the Flask application is going to save the image inside this directory.

```python
#Verify if snaps directory exists, if not then created it
if not exists("{}/dist/snaps".format(basedir)):
    makedirs("{}/dist/snaps".format(basedir))
```

With all these defined, we can proceed to work on our endpoints. The first, and the most important, is the one that serves our application:

```python
@app.route('/', methods=['GET'])
def home():
  return send_from_directory('{}/dist/'.format(basedir), 'index.html')
```

To achieve this, Flask uses `send_from_directory` to read inside a directory and serves all the available HTML static content.

Next, we define the `/send-mms` endpoint. The front-end application is going use this to share a link to the picture, through SMS:

```python
@app.route('/send-mms', methods=['POST'])
def send_mms():
    params = request.get_json() or request.form or request.args
    if 'phone' and 'image' in params:
        #Get the base64 image
        image = re.sub(r"^data:image\/png;base64,","" ,params['image'])
        image = bytes(image, 'utf-8')
        phone = params['phone']
        #Get the current timestamp
        filename_prefix = datetime.utcnow().isoformat()
        filename = "{phone}-{prefix}.png".format(phone=phone, prefix=filename_prefix)
        #Save image
        imagedir = "{}/dist/snaps".format(basedir)
        #Create the binary file in the snaps directory
        imagefile = open("{imagedir}/{filename}".format(imagedir=imagedir, filename=filename), "wb")
        #Write the bytes
        imagefile.write(base64.decodebytes(image))
        #Close file
        imagefile.close()
        #Send sms
        response = client.send_message(
            {
                'from': environ.get("NEXMO_NUMBER"),
                'to': phone,
                'text': "Opentok-Nexmo, Your snap is ready: {site_url}/snaps/{filename}"
                        .format(
                            site_url=environ.get("SITE_URL"),
                            filename=filename
                        )
            }
        )
        if response["messages"][0]["status"] == "0":
            return jsonify({ "status":"success", "message": "All OK" }), 200
        else:
            return jsonify({ "status":"error", "message": "Message failed with error: "+ response['messages'][0]['error-text'] }), 200
    else:
        return jsonify({ "status":"error", "message": "Required params (phone, image) not provided" }), 200
```

This endpoint validates if `phone` and `image` were provided. If not,  the endpoint returns a JSON object with an error. If the required data is there, then the endpoint generates a binary version of the image and saves it into the `snaps` folder. But first, using the phone number and the current date-time, it assigns a unique name to the images.

Our app uses the `nexmo` client to send an SMS with the URL of the image. If the request succeeds then it returns a message with the `success` status. If it fails, our endpoint returns a JSON with the error details.

## Local Deployment

**Front End**

Go to the main directory of your Vue application and run:

```bash
npm run build
```

This will generate the `dist` directory with all the static files for the frontend application.

**Back End**

To start serving your application, go to the main directory of your `backend` application and run:

```bash
gunicorn -b 0.0.0.0:80 server:app
```

And that's it! Enjoy your new photo booth app, and reach out to us on [Twitter](https://twitter.com/VonageDev) or the [Vonage Developer Community Slack](https://vonage-community.slack.com) if you have any questions.