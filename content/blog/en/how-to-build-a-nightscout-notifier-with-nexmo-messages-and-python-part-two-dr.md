---
title: "How to Build a Nightscout Notifier with Nexmo Messages and Python: 2"
description: Monitor Nightscout, an open source dashboard for type 1 diabetics
  with Python, firebase, Google auth, and Nexmo Messages API
thumbnail: /content/blog/how-to-build-a-nightscout-notifier-with-nexmo-messages-and-python-part-two-dr/E_Nightscout-Notifier_Part2_1200x600.png
author: diana-rodriguez
published: true
published_at: 2020-02-27T16:37:10.000Z
updated_at: 2021-05-24T13:33:13.459Z
category: tutorial
tags:
  - python
  - flask
  - sms-api
comments: true
redirect: ""
canonical: ""
---
In the [first post](https://www.nexmo.com/blog/2020/02/24/nightscout-notification-nexmo-dr) in this series, we showed you how to get started building Scout, an application that can connect to a user's continuous glucose monitoring (CGM) device and call when it detects a blood glucose level outside the normal range. We showed how to set up a Flask app with a UI where a user can add, edit, and check the app settings, complete with Google auth for authentication and Firebase/Firestore for data storage.

In this post, we'll show how the app can be set up to make calls and send SMS messages. We'll create a Python thread to execute a function with a given frequency, making a call to the user's contact number to alert them when their blood glucose level is outside the ranges previously set for high/low. If the user does not respond, the application will send an SMS to the emergency contact registered by the user (and up to five additional numbers if the user does not respond). If the glucose level remains outside the allowed range for a defined period of time, the application repeats the process. Additionally, if the Nightscout service fails to respond to pings for one hour, the application will send an SMS to the user indicating the connection is unresponsive.

## Nexmo Setup

To get started, you'll need to [sign up](https://dashboard.nexmo.com/sign-up?utm_source=DEV_REL&utm_medium=github&utm_campaign=https://github.com/alphacentauri82/nexmo-scout) for a Nexmo account. Once you've signed up, you will see a Key and Secret on your Dashboard. Copy these values to `.env` as follows:

```
  NEXMO_API_KEY="YOUR API KEY HERE"
  NEXMO_API_SECRET="YOUR API SECRET HERE"
```

**Note**: An additional and very important detail is that this account is **free** and has a basic balance for testing. However, this balance will be limited to making outbound calls and SMS. In order to control the flow of *inbound* calls (incoming calls made by the client), you'll need to purchase a virtual number. It is recommended (but not strictly necessary for this application) to upgrade your account to have a virtual number if you'd like unlimited access to all Nexmo features!

## Create a Nexmo Application

In order to achieve our goal we need to make calls and send messages. We must create an application that uses the [Voice API](https://developer.nexmo.com/voice/voice-api/overview) and [Messages API](https://developer.nexmo.com/messages/overview).

To send messages using the Messages API, we need our API Key and API Secret. To use the Voice API, we need an Application ID and Private Key, which will be used to generate a JWT (JSON Web Token) to verify our identity and guarantee the transparent exchange of messages between our app and the Nexmo application.

In other words, we have to create a Nexmo application with Voice and Messages enabled:

- Go to your Nexmo dashboard. On the left sidebar click *Your applications*.

- Click *Create a New Application*.

- Enter the application details as requested.

- Enable *Voice*. When selecting *Voice*, the dashboard will request webhook URLs: *Event URL*, *Answer URL*, and *Fallback answer URL*. We'll only be using the *Event URL*, as the others are meant for inbound calls and our app only makes outbound calls. The *Event URL* is an endpoint on our app server to which Nexmo will send the tracking of all the events that occur in our Voice API. For example, when making a call, Nexmo will send the status of the call: ringing, answered, unanswered, and complete. For the application we are building, this is essential, as we need to know the status of the call made to a user, and one of our conditions is that if the user does not answer (`unanswered`) a message is sent to the other number(s) registered by the user. In our case, the *Event URL* should look like `https://domain.ext/webhooks/events`.

- Enable *Messages*: Two URLs are requested, but we won't need to define either, as we are only sending outbound SMS messages.

- Finally, click on *Generate New Application*.

To recap, we created and configured our Nexmo application. If your account is upgraded, there will be a link to buy a Nexmo virtual number (*Buy new numbers*). It is possible to link a Nexmo virtual number to our application so that the outbound calls do not appear as unknown numbers. It is very convenient since the contact can be stored as "Scout" or "Nightscout Alerts". If applicable/desired, you can purchase a virtual number and then link it to this application.

That's all we needed to do in order to create our Nexmo application. Now we proceed to obtain the `APPLICATION_ID` and `PRIVATE_KEY`. We can retrieve those values from our dashboard, going to *"*Application details*.

In order to display your `PRIVATE_KEY`, click on *Edit* and then click on *Generate public and private key*. This will trigger a file download containing your private key. Move the generated file to the root application folder. Rename it `nexmo_private.key`.

## Send Alerts with Nexmo

Let's edit our `.env` file and add the following:

```
NEXMO_APPLICATION_ID="NEXMO_APPLICATION_ID"
NEXMO_PRIVATE_KEY="./nexmo_private.key"
NEXMO_NUMBER="NEXMO_VIRTUAL_NUMBER"
```

Remember to replace these values with your information. If you don't have a virtual number, you can use `Nexmo` as a value.

Our application will ping a Nighscout dashboard every minute and get the user's blood glucose level. If it is not in the allowed range, the application will call the user to let them know. However, without an important modification, if the values remain out of range for a considerable period of time even after the user has taken the call, a call will be made every minute. To control this, we will define an additional variable called `WAIT_AFTER_CALL`. We'll also define another variable, `NEXMO_FAILED_PING_SMS`, to control the number of failed pings before an SMS will be sent to the user to alert them that their Nightscout service has been offline. The default number will be 60, which  is equivalent to one hour as a ping is made once a minute.

Let's add the following to the end of our `.env` file:

```
WAIT_AFTER_CALL="120"
NEXMO_FAILED_PING_SMS="60"
```

Before we go forward, let's look at the modules we need to include in this application. We need to create a thread to execute our job, which can be done with the native Python module `multiprocessing`. We will make some HTTP requests to obtain information from the Nightscout API and to use the Messages API to send SMS messages, we can use the `requests` module we installed previously. We'll also need a scheduler to execute a function periodically, which can be done with the `schedule` module, and finally we'll use the `nexmo` module for interacting with the Voice API.

Go to your terminal and run the following command:

```sh
pip install nexmo schedule
```

Now, go to `notifier.py`, and under the last import add the following:

```python

import nexmo, schedule, time, signal, uuid

from multiprocessing import Process

```

In addition to the modules mentioned above, we will use the native `signal` module to detect **SIGTERM** (CRTL + C), used to terminate an application from the console, and `uuid` to generate an unique identifier for our process running in the thread.

In the same file (`notifier.py`), before the definition of the `get_session` function, add the following lines:

```python

............

client = nexmo.Client(
    application_id = os.getenv('NEXMO_APPLICATION_ID'),
    private_key = os.getenv('NEXMO_PRIVATE_KEY')
)

active_scouts = nightscouts.get_all()

.............

def get_session(key):
```

With `client = nexmo.Client` we initialize the Nexmo client. As we will be using the Voice API, the values ​​that we will be passing are our `NEXMO_APPLICATION_ID` and the `NEXMO_PRIVATE_KEY`. These values are stored in our environment variable file (`.env`), so we simply get them ​​using `os.getenv` as we have been doing so far.

The line below `active_scouts = nightscouts.get_all()` gets the complete list of registered users so far. This list will be updated every hour to detect changes made by the client from the user interface that we previously created.

In `notifier.py`, after the endpoint definition `/logout`, we will add the functions responsible for notifications to users. We will start with the function that notifies the user indicating that the Nightscout service is not responding. For this we will use the Messages API, since we will be sending an SMS:

```python
nightscout_failed_pings = {}
def handle_nightscout_failed_pings(to,api_url,username):
    global client
    global nightscout_failed_pings
    if to not in nightscout_failed_pings:
        nightscout_failed_pings[to] = 1
    else:
        nightscout_failed_pings[to] += 1
    #print('Intent: {0} for {1}'.format(nightscout_failed_pings[to],to))
    if nightscout_failed_pings[to] == int(os.getenv("NEXMO_FAILED_PING_SMS")):
        response = requests.post(
            'https://api.nexmo.com/v0.1/messages',
            auth=HTTPBasicAuth(os.getenv("NEXMO_API_KEY"), os.getenv("NEXMO_API_SECRET")),
            headers={
                "Content-Type": "application/json",
                "Accept": "application/json"
            },
            json={
                "from": {
                    "type": "sms",
                    "number": os.getenv("NEXMO_NUMBER"),
                },
                "to": {
                    "type": "sms",
                    "number": to
                },
                "message":{
                    "content":{
                        "type": "text",
			"text": "Dear {0} the nexmo api url: {1} is not responding, please check the service".format(username,api_url)
                    }
                }
            }
        ).json()

        #Reset the variable
        nightscout_failed_pings[to] = 0

        if "message_uuid" in response:
            return True
    return False
```

Before defining the `handle_nightscout_failed_pings` function,`nightscout_failed_pings = {}` is a variable that will control the number of failed pings per user. In this case, the object's key (index) will be the user's phone number, and the value tracked is the failed ping count.

The `handle_nightscout_failed_pings` function receives as parameters `to`, `api_url`, and `username`. The parameter `to` is the telephone number of the user, `api_url` is the Nightscout api URL that is presenting the connection problem, and `username` is the name of the user to which the SMS will be sent to.

Initially, the function evaluates whether the `to` key is present in the `nightscout_failed_pings` object. If not, the key in the object is initialized with the value of one. If it exists, the value increases by one.

After the conditional `if nightscout_failed_pings[to] == int(os.getenv("NEXMO_FAILED_PING_SMS")):` is evaluated, indicating if the number of failed pings has reached the maximum as per our settings, an SMS is sent notifying the user.

The Messages API is in Beta, so the functions for this service have not yet been incorporated into the Nexmo Python SDK. To send the message we will use `requests` to directly make a call to the API endpoint.

The method to use when making the request will be `POST`. The first parameter, `https://api.nexmo.com/v0.1/messages`, corresponds to the URL of the Messages API. The second parameter handles authentication. In this case we are using basic authentication with the `NEXMO_API_KEY` and the `NEXMO_API_SECRET` obtained from our environment variables. In the header we define that the data will be sent in JSON format.

The JSON is structured in three sections: 

- `from`, where the channel we will use to send the message (e.g. Whatsapp, Messenger, SMS) is defined. In our case it is `sms`.
- `number`, the telephone number used to send the SMS (here we assign the value of the environment variable `NEXMO_NUMBER`).
- `to`: we also define `sms` as the `type`, and in `number` we assign the value of the `to` parameter that our function receives, which is the destination number where the `sms` will be sent. The `message` section is where the type of content we will send (image, video, text) is indicated. The notification `type` is `text` and `text` (message to be sent) receives the value of `"Dear {0} the Nexmo api url: {1} is not responding, please check the service".format(username, api_url)` where `{0}` is replaced by the value of `username` and`{1}`by the value of `api_url`.

Once the notification is sent, the failed pings count is reset to `0`. If in the response returned by Nexmo there is a `message_uuid` key, the function returns `True`, indicating that the message was sent. Every message or call made by/to `Nexmo` has its own `uuid`. If `Nexmo` has not generated the `message_uuid`, the function assumes that the message was not sent and `False` is returned.

At the end of our file let's add the following function, which will also send the user's glucose level to a number chosen by the user in the UI:

```python
def sms_glucose_alert(to, username, glucose):
    global client
    #We send our sms using the messages api not the sms api
    response = requests.post(
        'https://api.nexmo.com/v0.1/messages',
        auth=HTTPBasicAuth(os.getenv("NEXMO_API_KEY"), os.getenv("NEXMO_API_SECRET")),
        headers={
            "Content-Type": "application/json",
            "Accept": "application/json"
	},
        json={
            "from": {
                "type": "sms",
                "number": os.getenv("NEXMO_NUMBER"),
            },
            "to": {
                "type": "sms",
                "number": to
	    },
            "message":{
                "content":{
                    "type": "text",
		    "text": "Alert {username} Blood Glucose is {glucose}".format(username=username, glucose=glucose)
                }
            }
        }
    ).json()

    if "message_uuid" in response:
        return True
    else:
        return False
```

The `sms_glucose_alert` function is quite similar to `handle_nightscout_failed_pings` in the sense that exactly the same procedure is used for sending an SMS. The difference is that it will be indicating the **glucose level** instead of the **api url**. In this function, an additional comment has been added clarifying that sending SMS will be done using the Messages API, not the SMS API.

Then we add the function `call_glucose_alert`:

```python
last_call = {}
def call_glucose_alert(to, glucose):
    if to in last_call:
        if int(time.time()-last_call[to]) < int(os.getenv("WAIT_AFTER_CALL")):
            print("The number {0} was called recently.. Please wait a little longer: {1}".format(to,int(time.time()-last_call[to])))
            return False
    #print('Call {0} {1}'.format(to, glucose))
    last_call[to] = time.time()

    #We make our call using the voice API
    response = client.create_call(
        {
            "to": [{
                "type": "phone",
                "number": to
            }],
            "from": {
                "type": "phone",
                "number": os.getenv('NEXMO_NUMBER')
            },
            "ncco": [
                {
                    "action": "talk",
                    "text": "Alert Your Blood Glucose is {0}".format(glucose)
                }
            ],
            "eventUrl": [
                "{url_root}/webhooks/events".format(url_root=os.getenv("SITE_URL"))
            ]
        }
    )
    if "uuid" in response:
        return True
    else:
        return False
```

If we look carefully, we have added the variable `last_call`, which plays a role similar to `nightscout_failed_pings`. `last_call` is an object and its index will be the user's phone number and the value at the index will be the current time. At the beginning of the function, it is evaluated if the `to` key exists in `last_call`. If it does, we evaluate if the period between the current time and the time in which the last call of a particular user was made has not reached the value of our `WAIT_AFTER_CALL` environment variable. If this conditional returns `True`, a message is printed indicating that the number was recently called, and `False` is returned preventing the call from being made again too soon.

If `to` does not exist in `last_call`, we initialize it with the value `time.time()`, which reflects the current time in seconds, which in turn is a timestamp that we will be using to make calculations. For example: the time elapsed in _seconds_ since the last call is calculated in `int(time.time() - last_call[to])`, and we compare it with the value of `WAIT_AFTER_CALL`, which we measure in _seconds_.

To make the `call` using the Voice API, we will use the `create_call` function of the Nexmo module. Previously, we have initialized the `Nexmo` client using the `application_id` and the `private_key` indicating that we have all the preparations in place to use the Voice API.

The `create_call` method of `client` receives a single Python dictionary as an object. Its nomenclature is very similar to that of `JSON` objects, facilitating its manipulation since `JSON` is very well known and simple to implement.

Like the Messages API, the `create_call` function has the `to` and `from` sections. The only variant of these sections is the `type`, which in this case will be `phone`, indicating that a call will be made. In `from`, the value of `number` will be our `NEXMO_NUMBER` and in `to`, we will assign the parameter `to` that our function receives.

Subsequently, we have the section `NCCO`. An NCCO is a JSON object that indicates to Nexmo the flow of actions that will be performed on the call. We will pass the `NCCO` as an object directly to the`NCCO` section of `create_call`.

The `NCCO` section has the key `action` and its value is `talk`. This action tells Nexmo that when making the call, a message will be `speech` (read to the user over the call). It is worth mentioning that Nexmo supports the reproduction of speech in different languages!!. Subsequently, we have the key `text`, where the text that we want reproduced in the `call` is defined.

Next we define the `eventUrl`, which is a webhook for Nexmo to send information to our server. `eventUrl` is an endpoint on our server that will be receiving a request from Nexmo in JSON format with information on the different states of the call. This is very convenient for us since we are interested in capturing the `unanswered` status, to send our SMS to the emergency contact and additional numbers if applicable. For more information on status that will arrive at the `eventUrl` visit https://developer.nexmo.com/voice/voice-api/guides/call-flow.

Like the Messages API, in the Voice API Nexmo generates a unique `uuid` for each call made. If our answer does not have a `uuid` generated, the function returns `False`, indicating that the call was not made.

**Important note**: The `call_glucose_alert` function is the initial trigger of the Nexmo workflow.

Let's add our `event` webhook endpoint, at the end of `notifier.py`:

```python
@app.route('/webhooks/events',methods=["POST","GET"])
def events():
    global client
    req = request.get_json()
    #print(req)
    if "status" in req:
        if req["status"] == "unanswered":
            phone = req["to"]
            uscout = nightscouts.getby_personal_phone(phone)
            if uscout!=None:
                entries = requests.get(uscout["nightscout_api"]).json()
                glucose = entries[0]['sgv']
                sms_glucose_alert(uscout["emerg_contact"], uscout["username"],glucose)
                #print('sms simulation to: {0} {1} {2}'.format(uscout["emerg_contact"], uscout["username"], glucose))
                for phone in uscout["extra_contacts"]:
                    #print('sms simulation to: {0} {1} {2}'.format(phone, uscout["username"], glucose))
                    sms_glucose_alert(phone, uscout["username"],glucose)
    return "Event Received"
```

If the user's glucose level is not within the allowed range, the `call_glucose_alert` function that makes a call using the Voice API will be executed. At the time of making the call, Nexmo sends information to the event webhook to inform about the `started` state. When the phone starts to ring, Nexmo sends information to the event webhook indicating the status `ringing`, and so on with the rest of the call statuses.

To obtain the information that Nexmo sends to our event webhook, we will use `req = request.get_json()`. You can uncomment `print(req)` to see the structure of the `JSON` in your terminal.

Because we are interested in evaluating the status of the call, we make a conditional querying if `status` exists for `req`. There is a second conditional that assesses whether the value of the status is `unanswered`. If so, we capture the phone number where the call was not answered using `phone = req["to"]` and use our model to obtain the record related to this phone, `uscout =nightscouts.getby_personal_phone(phone)`.

If the record exists, we will recover the last glucose levels for the user, with `requests` and the `get` method, as follows: `entries = requests.get(uscout["nightscout_api"]).Json()`. From there, we get the last glucose level registered for the user: `glucose = entries[0]['sgv']` (the Nightscout API orders the entries in descending order, indicating that the most recent glucose level will be the first in the array). With the glucose level obtained, we use the `sms_glucose_alert` function to send an SMS indicating the user's glucose level to their emergency contact and any additional numbers registered.

With this we have finished the functions section for sending notifications!! 🍾

## Use a Scheduler to Send Automatic Notifications

In this section, we will add the code responsible for creating a parallel thread to our `Flask` application. This will be execute a scheduler to evaluate the glucose levels of the user, and it will execute the function `call_glucose_alert` if the glucose level is not in the allowed range. In addition, the scheduler is responsible for executing the `handle_nightscout_failed_pings` function that will evaluate the number of failed connection attempts to the Nightscout API and will send the notification when it has reached the maximum number of allowed attempts.

Let's add the following lines to the end of the `notifier.py`:

```python
thread = None

class ApplicationKilledException(Exception):
    pass

#Signal handler
def signal_handler(signum, frame):
    raise ApplicationKilledException("Killing signal detected")

```

In the previous lines we defined some functions and variables. `thread` is the variable that we will assign to our parallel thread, `ApplicationKilledException` is a custom class that inherits from `Exception`, created by us with the only task of triggering when the `signal_handler` function is executed. Then, we defined the `signal_handler` function that triggers our custom `Exception`. The idea of `signal_handler` is to give us a more control when the process is stopped by human intervention (by doing CTRL + C) or with the kill command.

By having greater control we can execute certain additional operations to stop our scheduler correctly.

```python
def refresh_scouts(id):
    global active_scouts
    active_scouts = nightscouts.get_all()
    print("Refresh Scouts Job " + id+ "")
```

`refresh_scouts` is a function that updates the value of the variable `active_scouts` during the life cycle of our application. It will be executed by a second scheduler within our thread every hour. The purpose of this function is to keep our Firebase Scout collection up to date. If a user has made changes through the UI, these changes will be available when `refresh_scouts` is executed.

Another interesting detail is that it receives the parameter `id`. This is because the scheduler, when executing a job, will have to identify it with a unique `id` in order to be able to edit the configuration of execution of a job in real time or to kill a particular job.

Let's continue to add the following lines of code at the end of our `notifier.py`:

```python
def job(id):
    #Calling nemo global client variable
    global client
    global active_scouts
    print("Alerts Job " + id+ "")
    if active_scouts != None:
        for active_scout in active_scouts:
            #print(active_scout["nightscout_api"])
            try:
                entries = requests.get(active_scout["nightscout_api"]).json()
                glucose = entries[0]['sgv']
                #We add a dynamic attribute called glucose to pass glucose info to events url
                if 70 <= glucose <= 240:
                    print("{0} is inside the range for {1}".format(glucose,active_scout["username"]))
                else:
                    print("Executing emergency call and loading sms NCCO if needed")
                    call_glucose_alert(active_scout["phone"],glucose)
            except:
                handle_nightscout_failed_pings(active_scout["phone"],active_scout["nightscout_api"],active_scout["username"])
                print("Server could not establish connection with "+active_scout["nightscout_api"])

```

The `job` function will be our main job, and it will be executed every minute by the scheduler. the function starts by reading the `active_scouts` variable, which will be updated every hour using the `refresh_scouts` function. It then proceeds to evaluate each of the users using `for active_scout in active_scouts:`.

Within `try`, we place the `request` that we will make to the `nightscout_api` to obtain the glucose level. If the connection to the API fails, the application will execute `except`. Within the `except` block, we print the URL of the Nightscout API and then execute `handle_nightscout_failed_pings`, a function that we previously defined.

If `requests` has obtained the glucose level correctly, we evaluate if it is between`70` and `240` (our allowed range). A good practice is to locate these limits in environment variables as we have done with other configurations of our application. If the glucose level is within the permitted range, a message is printed indicating that the glucose level is within the limit (`print (" {0} is inside the range for {1} ". Format (glucose, active_scout [" username "]))`), If not, the `call_glucose_alert` function will be executed and will make a call to the user's telephone number indicating the blood glucose level.

`call_glucose_alert` triggers the tracking of Nexmo which will be sending the `status` of the call for each of its phases. The states will be sent to our event webhook endpoint, where if there is an `unanswered status`, the function `sms_glucose_alert` is executed for the emergency contact(s) as describe previously.

In our `notifier.py` let's add the following lines:

```python
#Manage individual schedule Thread Loop
def run_schedule():
    global thread
    while True:
        try:
            schedule.run_pending()
            time.sleep(1)
        except ApplicationKilledException:
            print("Signal Detected: Killing Nightscout-Nexmo App.")
            #clean the schedule
            schedule.clear()
            return "Thread Killed"

if __name__ == "notifier":
    signal.signal(signal.SIGTERM, signal_handler)
    signal.signal(signal.SIGINT, signal_handler)
    #run job at second 30 of each minute
    schedule.every().minute.at(':30').do(job, str(uuid.uuid4()))
    #update scouts each our at minute 00
    schedule.every().hour.at(':00').do(refresh_scouts, str(uuid.uuid4()))
    thread = Process(target=run_schedule)
    thread.start()
    print("Nightscout-Nexmo Thread starts")
```

Observe the `run_schedule` function, which will initiate an infinite loop (within the parallel thread to the Flask app). In this loop, inside the `try` clause, we find `schedule.run_pending()`. This checks if there are jobs to be executed in the `schedule` and if so, it executes them on each iteration. After this, notice `time.sleep(1)`. This freezes the flow execution for 1 second before evaluating the next iteration.

In the `except` block we capture the exceptions of type `ApplicationKilledException`, the custom exception we created previously. If this exception is detected, our function will proceed to stop the scheduler and send the message "Signal Detected: Killing Nightscout-Nexmo App". The exception will be triggered when the user hits `CTRL+C` from the terminal after deploying the application.

The next lines evaluate whether the app that is running is `notifier` (the `notifier.py` script). If so, we assign the `signal_handler` function when `SIGTERM` `(CTRL+C)` and `SIGINT` (other types of interruptions) are detected.

The followinglines define the timing of the jobs:

- `schedule.every().minute.at(':30').do(job, str(uuid.uuid4()))` is easy to read and understand. Using the `schedule` module, every `minute` at `30` seconds it will execute the `job` function, passing the parameter `uuid.uuid4()`, which is a unique process `id` generated by the `uuid` module.

- `schedule.every().hour.at(':00').do(refresh_scouts, str(uuid.uuid4()))`: using the `schedule` module every `hour` at `00` minutes, execute the `refresh_scouts` function to which we pass the id `uuid.uuid4()` generated by the `uuid` module.

Once the programming of the jobs has been established in the scheduler, we define the thread where our scheduler will run. For this we will use `Process`, a class of the multiprocessing native module previously imported in `from multiprocessing import Process`.

`thread = Process(target = run_schedule)` indicates that a process will be created and that it will be executed in a separate `thread`. `Target` represents the name of the function that defines the infinite loop of the scheduler that will be executed in this thread.

The `thread.start()` line indicates the execution of the previously defined process. Once this is done, the thread will be created in the background, which can be confirmed with the `top` command in your terminal (two running Python processes should be detected).

Finally, we print the message `Nightscout-Nexmo Thread starts` to indicate that our application has started its execution (this message will be detected by any Python, gunicorn, or other server logs like nginx).

## Run the Application Locally

To run our app locally, head back to terminal, and from the app root directory run the following command:

```sh
gunicorn -b 0.0.0.0:80 notifier: app
```

Gunicorn will respond that the application has started, and the message `Nightscout-Nexmo Thread starts` will be displayed. In addition, in the `job` function which is executed every minute by `scheduler`, we observe`print("Alerts Job"+id+" ")`. Gunicorn will show on screen that the `Alerts Job XXXX-X-X-X-XXXXXXXX` was executed, as well as other messages found in some other functions of our application.

This way we can check that our thread is alive and running correctly.

**Additional tip:** If we want to deploy our app on another server different from the current one, we would need a list of modules required for our application to work. As it can be tedious to generate this list manually, we can execute the following command from our terminal:

```sh
pip freeze > requirements.txt
```

This command generates a text file called `requirements.txt` where we will have an updated list of all the app's required modules, so when we clone our repository to our external server we only have to run:

```sh
pip install -r requirements.txt
```

After running the above command, we are ready to deploy our application!!.

## Deploy the Application on Google Cloud Platform

In this section we will describe the procedure to deploy our application using **Google Cloud Platform**. Proceed to https://console.cloud.google.com and authenticate.

We have previously created a project, but in case you have more than one active project, click on *Select Project* and select the project associated with our app.

Click on the gcloud console icon (`>_`).This will display the console. In order to edit our files, select the pencil symbol. This opens a new window with an editor and our console loaded just below the editor.

In the Google Cloud console, let's clone our application:

```sh
git clone https://github.com/alphacentauri82/nexmo-scout.git
```

Init the virtual environment for Python3. This is very practical when using several versions of Python or when working on a project, as it allows us to work in isolation from the environment. For example, to generate the packages of our application with `pip freeze> requirements.txt`, if we have previously configured virtual env only the packages that we install inside the virtual env will be taken into account.

```sh
python3 -m venv env
source env/bin/activate
```

Let's open the editor and edit the `.env` file. We will configure all the necessary credentials, from our local directory and add all the private keys that are needed to the directory of our project (it is only necessary to drag and drop them and Google will upload them). The environment variable `SITE_URL` should have the structure`https://PROJECT_ID.appspot.com` where `PROJECT_ID` is the id that we set for our project.

Let's add an `app.yaml` file with the following content:

```yaml
runtime: python
env: flex
entrypoint: gunicorn -b :$PORT notifier:app

runtime_config:
  python_version: 3

manual_scaling:
  instances: 1
```

This file is used by Google Cloud to deploy. It contains general details of our application, such as the programming language, the necessary command to initialize our app, and even infrastructure details (number of instances, CPU, ram memory, disk size, etc).

Let's head back to the Google Cloud Console, switch to our app directory, and deploy the application:

```sh
cd nexmo-scout
gcloud app deploy
```

- `gcloud app deploy` deploys our app. It creates a container for our application, installs the required version of Python, installs all required packages and runs our app. This process tends to take several minutes.

- With `gcloud app browse` we can see our application running, and with `gcloud app logs tail -s [INSTANCE]` we can access the our application logs.

- `gcloud app browse` will notify us that no installed browser was found (since this command is normally used in the CLI). However, it will return the url of our application. Click on the link see it live!

- `gcloud app logs tail -s [INSTANCE]`: Replace `[INSTANCE]` with the name of your assigned instance. In our particular case we can use this command to monitor that our daemon is running every minute.

- *App Engine* is also a good way to monitor our app. Click on the hamburguer menu (`≡`). Then go to *App Engine> Dashboard*. It will load our project _dashboard_ and display stats of all the running instances, as well as the billing and error information.

- Click on *Services* to access the services we have configured. Each service can have multiple versions.

- Click on *Versions*. Here we can see each container created with each deploy, each of them versioned. From here we can monitor the status of a container, the number of instances it uses, what language it is using, etc. Most importantly, we can manage our containers. We can start and stop them or remove old versions and keep the most recent deploys.

Hooray! we built a robust Flask app and deployed it!! 🎉