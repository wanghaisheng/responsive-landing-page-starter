---
title: Creating a Complex IVR System with Ease with XState
description: In this post we’ll see how to create very complex and elaborate IVR
  systems while keeping the code simple and easy to maintain
thumbnail: /content/blog/creating-a-complex-ivr-system-with-ease-with-xstate-dr/unnamed.jpg
author: yonatan-mevorach
published: true
published_at: 2019-06-20T18:33:17.000Z
updated_at: 2021-05-13T21:18:42.196Z
category: tutorial
tags:
  - developer-spotlight
comments: true
redirect: ""
canonical: ""
---

<a href="http://developer.nexmo.com/spotlight?utm_campaign=dev_spotlight&amp;utm_content=Stripe_SMS_Notifications_Laravel_Nexmo"><img src="https://www.nexmo.com/wp-content/uploads/2019/03/blog-top-banner.png" alt="Nexmo Developer Spotlight" width="810" height="150" class="aligncenter size-full wp-image-28699" /></a>

Even if you didn't know that's what they're called, you probably use IVR systems all the time. An <a href="https://www.nexmo.com/use-cases/interactive-voice-response" target="_blank" rel="noopener noreferrer">IVR system</a> lets you call a phone number, listen the audio-cues, and navigate your way through the call to get the info you need. Nexmo makes creating a full-fledged IVR system as simple as spinning up a Web server. In this post we'll see how to create very complex and elaborate IVR systems while keeping the code simple and easy to maintain. In order to accomplish this, we'll use <a href="https://xstate.js.org/" target="_blank" rel="noopener noreferrer">XState</a> which is a popular State Machine library for Javascript.

<h2>An IVR System with Less Than 35 Lines of Code</h2>
The key to implementing an IVR System with Nexmo is to create a Web server that will instruct Nexmo how to handle each step of the call. Typically, this means that as soon as a user calls your virtual incoming number, Nexmo will send an HTTP request to your <span class="lang:default decode:true crayon-inline ">/answer </span> endpoint and expect you to respond with a JSON payload composed of <a href="https://developer.nexmo.com/voice/voice-api/ncco-reference" target="_blank" rel="noopener noreferrer">NCCO</a> objects that specify what the user should hear. Similarly, when the user uses their keypad to choose what they want to listen to next, then Nexmo makes a request to a different endpoint typically called <span class="lang:default decode:true crayon-inline ">/dtmf</span>. The <span class="lang:default decode:true crayon-inline ">/dtmf</span>  endpoint will be called with a request payload that includes the number that the user has chosen, which your server should use to figure out what set of NCCO objects to respond with.

Let's see this what this looks like in code when using <a href="https://expressjs.com/" target="_blank" rel="noopener noreferrer">express</a> to power our Web server.
<pre class="lang:default decode:true ">const express = require('express');
const bodyParser = require('body-parser');

const port = process.env.PORT || 3000;
const app = express();
app.use(bodyParser.json());

app.post('/answer', (req, res) =&gt; {
  const ncco = [
    { action: 'talk', text: "Hi. You've reached Joe's Restaurant! Springfield's top restaurant chain!" },
    { action: 'talk', text: 'Please select one of our locations.' },
    { action: 'talk', text: 'Press 1 for our Main Street location.' },
    { action: 'talk', text: 'Press 2 for our Broadway location.' },
    { action: 'input', eventUrl: [ 'https://example.com/dtmf' ], maxDigits: 1 },
  ];
  res.json(ncco);
});

app.post('/dtmf', (req, res) =&gt; {
  const { dtmf } = req.body;
  let ncco;
  switch (dtmf) {
    case '1':
      ncco = [ { action: 'talk', text: "Joe's Main Street is located at Main Street number 11, Springfield." } ];
      break;
    case '2':
      ncco = [ { action: 'talk', text: "Joe's Broadway is located at Broadway number 46, Springfield." } ];
      break;
  }
  res.json(ncco);
});

app.listen(port, () =&gt; console.log(`Example app listening on port ${port}!`));
</pre>
<h2>Trying It For Yourself</h2>
You can start writing your app code right away. But in order to be able to call in and test for yourself that everything is working, you'll need to complete the following:
<ol>
 	<li>Make sure your Web server is accessible on the Web. You can do this by exposing your local development machine <a href="https://www.nexmo.com/blog/2017/07/04/local-development-nexmo-ngrok-tunnel-dr/?utm_campaign=dev_spotlight&utm_content=IVR_XState_Mevorach" target="_blank" rel="noopener noreferrer">using Ngrok</a> or by developing using <a href="https://glitch.com/" target="_blank" rel="noopener noreferrer">Glitch</a>.</li>
 	<li><a href="https://dashboard.nexmo.com/sign-up?utm_campaign=dev_spotlight&utm_content=IVR_XState_Mevorach" target="_blank" rel="noopener noreferrer">Sign up</a> for Nexmo if you haven't already done so.</li>
 	<li>Create a Voice application. You can do this via the <a href="https://dashboard.nexmo.com/voice/create-application" target="_blank" rel="noopener noreferrer">Nexmo Website</a>, or using the <a href="https://github.com/Nexmo/nexmo-cli#create-a-new-application" target="_blank" rel="noopener noreferrer">Nexmo CLI</a>. You'll need to enter the public url of your /answer endpoint when you set up your application.</li>
 	<li>Obtain a virtual incoming number and connect it to your app using the <a href="https://dashboard.nexmo.com/buy-numbers?utm_campaign=dev_spotlight&utm_content=IVR_XState_Mevorach" target="_blank" rel="noopener noreferrer">Website</a> or <a href="https://github.com/Nexmo/nexmo-cli#create-a-new-application" target="_blank" rel="noopener noreferrer">CLI</a>.</li>
</ol>
When all this is in place you'll be able to call your number and you'll hear the audio response that's based on the data you return from your Web server.
<h2>Going Beyond the "Hello World" of IVR Systems</h2>
The example shown above works as expected, but a real-world IVR System will yield to the user for input many times, and will interpret the user's numeric input based on the state of the user in the call. To illustrate this, let's assume that in our example the user will be asked to choose the restaurant location that they're interested in, and then to choose whether they want to listen to the open hours or make a reservation. In both of these cases, the user may press 1 on their keypad, but how we interpret this depends on the previous audio-cue and the state of the user in the call.

To support this use-case we'll need to change the code we just wrote. Ideally, we'll change it in a way so that as we add functionality and make our IVR System more complex over time, our code will stay simple and we won't have to rethink how to structure it. To achieve this, we'll model our call structure as a <a href="https://en.wikipedia.org/wiki/Finite-state_machine" target="_blank" rel="noopener noreferrer">Finite-State Machine</a> using <a href="https://xstate.js.org/" target="_blank" rel="noopener noreferrer">XState</a>, a State Machine library for Javascript.
<h2>A Primer on State Machines</h2>
A State Machine is simply a model for a "machine" that can be in only one state at any given time, and can only transition from one state to another given specific inputs. XState and other State Machine libraries let you model and instantiate a machine in code, in a way that the "rules" of the State Machine are guaranteed to be enforced.
<h2>Modeling our Call Structure as a State Machine</h2>
To model our call structure as a State Machine, we'll use the <span class="lang:default decode:true crayon-inline ">Machine</span> function that XState exposes:
<pre class="lang:default decode:true ">// machine.js
const { Machine } = require('xstate');

module.exports = Machine({
  id: 'call',
  initial: 'intro',
  states: {
    intro: {
      on: {
        'DTMF-1': 'mainStLocation',
        'DTMF-2': 'broadwayLocation'
      }
    },
    mainStLocation: {
    },
    broadwayLocation: {
    }
  }
});
</pre>
As you can see in the code above, our call can only be in one of three states:
<ul>
 	<li>The <span class="lang:default decode:true crayon-inline ">intro</span>  state where the user is listening to the introduction and is instructed to choose the location they're interested in.</li>
 	<li>The <span class="lang:default decode:true crayon-inline ">mainStLocation</span>  state where they're listening to information about the Main St. location of our hypothetical restaurant chain</li>
 	<li>The <span class="lang:default decode:true crayon-inline ">broadwayLocation</span>  state when they're listening to information about the Broadway location.</li>
</ul>
You can also see that:
<ul>
 	<li>The only way to transition to the <span class="lang:default decode:true crayon-inline ">mainStLocation</span> state is be in the <span class="lang:default decode:true crayon-inline ">intro</span> state and send the <span class="lang:default decode:true crayon-inline ">DTMF-1</span>  <a href="https://xstate.js.org/docs/guides/events.html" target="_blank" rel="noopener noreferrer">event</a>.</li>
 	<li>The only way to transition to the <span class="lang:default decode:true crayon-inline ">broadwayLocation</span> state is to be in the intro state and send the <span class="lang:default decode:true crayon-inline ">DTMF-2</span>  event.</li>
</ul>
We can choose to colocate the NCCO objects related to each state inside the event definition using XState's <a href="https://xstate.js.org/docs/guides/states.html#state-meta-data" target="_blank" rel="noopener noreferrer">metaproperty</a>
<pre class="lang:default decode:true ">// machine.js
const { Machine } = require('xstate');

module.exports = Machine({
  id: 'call',
  initial: 'intro',
  states: {
    intro: {
      on: {
        'DTMF-1': 'mainStLocation',
        'DTMF-2': 'broadwayLocation'
      },
      meta: {
        ncco: [
          { action: 'talk', text: "Hi. You've reached Joe's Restaurant! Springfield's top restaurant chain!" },
          { action: 'talk', text: 'Please select one of our locations.' },
          { action: 'talk', text: 'Press 1 for our Main Street location.' },
          { action: 'talk', text: 'Press 2 for our Broadway location.' },
          { action: 'input', eventUrl: [ 'https://example.com/dtmf' ], maxDigits: 1 }
        ]
      }
    },
    mainStLocation: {
      meta: {
        ncco: [
          { action: 'talk', text: "Joe's Main Street is located at Main Street number 11, Springfield." }
        ]
      }
    },
    broadwayLocation: {
      meta: {
        ncco: [
          { action: 'talk', text: "Joe's Broadway is located at Broadway number 46, Springfield." }
        ]
      }
    }
  }
});
</pre>
<h2>Utilizing our Machine</h2>
The object that the <span class="lang:default decode:true crayon-inline ">Machine</span> function returns should be treated as an immutable stateless object that defines the structure of our machine. To actually create an instance of our machine that we can use as a source-of-truth for the state of a call, we'll use XState <span class="lang:default decode:true crayon-inline ">interpret</span> function. The <span class="lang:default decode:true crayon-inline ">interpret</span>  function returns an object which is referred to as a "<a href="https://xstate.js.org/docs/guides/interpretation.html" target="_blank" rel="noopener noreferrer">Service</a>". You can access the current state of each machine instance using the <span class="lang:default decode:true crayon-inline ">state</span> property of the service. And you can send an event to change the state of the machine instance using the service's <span class="lang:default decode:true crayon-inline ">send()</span> method. We'll create a <span class="lang:default decode:true crayon-inline">callManager </span>module to be in charge of creating machine instances for every incoming call, sending the appropriate events as the call progresses, and removing each machine instance when the call ends.
<pre class="lang:default decode:true ">// callManager.js
const { interpret } = require('xstate');
const machine = require('./machine');

class CallManager {
  constructor() {
    this.calls = {};
  }

  createCall(uuid) {
    const service = interpret(machine).start();
    this.calls[uuid] = service;
  }

  updateCall(uuid, event) {
    const call = this.calls[uuid];
    if(call) {
      call.send(event);
    }
  }

  getNcco(uuid) {
    const call = this.calls[uuid];
    if(!call) {
      return [];
    }
    return call.state.meta[`${call.id}.${call.state.value}`].ncco;
  }

  endCall(uuid) {
    delete this.calls[uuid];
  }
}

exports.callManager = new CallManager();
</pre>
As you can see, each call is identified by its <span class="lang:default decode:true crayon-inline ">uuid</span>  which Nexmo takes care of assigning to each call.
<h2>Putting It All Together</h2>
Now we can modify our Web server code to defer to the <span class="lang:default decode:true crayon-inline ">callManager</span> whenever the Nexmo backend calls our endpoints.
<pre class="lang:default decode:true ">/// server.js
const express = require('express');
const bodyParser = require('body-parser');
const { callManager} = require('./callManager');

const port = process.env.PORT || 3000;
const app = express();
app.use(bodyParser.json());

app.post('/answer', (req, res) =&gt; {
  callManager.createCall(req.body.uuid);
  const ncco = callManager.getNcco(req.body.uuid);
  res.json(ncco);
});

app.post('/dtmf', (req, res) =&gt; {
  callManager.updateCall(req.body.uuid, `DTMF-${req.body.dtmf}`);
  const ncco = callManager.getNcco(req.body.uuid);
  res.json(ncco);
});

app.post('/event', (req, res) =&gt; {
  if(req.body.status == 'completed') {
    callManager.endCall(req.body.uuid);
  }
  res.json({ status: 'OK' });
});

app.listen(port, () =&gt; console.log(`Example app listening on port ${port}!`));
</pre>
As you can see, in order to know when the call has ended we added an /event endpoint. If you associate it with your Nexmo Application as the "Event URL" webhook then Nexmo will make a request to it asynchronously when the overall call state changes (e.g. the user hangs up). Unlike the <span class="lang:default decode:true crayon-inline ">/answer</span> or <span class="lang:default decode:true crayon-inline ">/dtmf</span> <span class="crayon-v"> endpoint</span>, you cannot respond with NCCO objects to this request and influence what the user hears.
<h2>Changing the Call Structure with Ease</h2>
We just completed a refactor of our app code, but it behaves exactly the same as before. But in contrast to before, now modifying the call structure becomes as simple as changing the JSON object that we pass to the the <span class="lang:default decode:true crayon-inline ">Machine</span> function.

So if, as mentioned earlier, we want to let the user decide if they want to listen to the location's opening hours or make a reservation, we just have to add a few more states, transitions, and NCCO arrays to our Machine's definition.
<pre class="lang:default decode:true ">// machine.js
const { Machine } = require('xstate');

module.exports = Machine({
  id: 'call',
  initial: 'intro',
  states: {
    intro: {
      on: {
        'DTMF-1': 'mainStLocation',
        'DTMF-2': 'broadwayLocation'
      },
      meta: {
        ncco: [
          { action: 'talk', text: "Hi. You've reached Joe's Restaurant! Springfield's top restaurant chain!" },
          { action: 'talk', text: 'Please select one of our locations.' },
          { action: 'talk', text: 'Press 1 for our Main Street location.' },
          { action: 'talk', text: 'Press 2 for our Broadway location.' },
          { action: 'input', eventUrl: [ 'https://example.com/dtmf' ], maxDigits: 1 }
        ]
      }
    },
    mainStLocation: {
      on: {
        'DTMF-1': 'mainStReservation',
        'DTMF-2': 'mainStHours',
      },
      meta: {
        ncco: [
          { action: 'talk', text: "Joe's Main Street is located at Main Street number 11, Springfield." },
          { action: 'talk', text: 'Press 1 to make a reservation.' },
          { action: 'talk', text: 'Press 2 to hear our operating hours.' },
          { action: 'input', eventUrl: [ 'https://example.com/dtmf' ], maxDigits: 1 },
        ]
      }
    },
    broadwayLocation: {
      on: {
        'DTMF-1': 'broadwayReservation',
        'DTMF-2': 'broadwayHours',
      },
      meta: {
        ncco: [
          { action: 'talk', text: "Joe's Broadway is located at Broadway number 46, Springfield." },
          { action: 'talk', text: 'Press 1 to make a reservation.' },
          { action: 'talk', text: 'Press 2 to hear our operating hours.' },
          { action: 'input', eventUrl: [ 'https://example.com/dtmf' ], maxDigits: 1 },
        ]
      }
    },
    mainStReservation: { /* ... */ },
    mainStHours: { /* ... */ },
    broadwayReservation: { /* ... */ },
    broadwayHours: { /* ... */ }
  }
});
</pre>
<h2>More XState Goodness</h2>
XState has more useful features that can help us out as our call model becomes more intricate.
<h2>XState Visualizer</h2>
The XState Visualizer is an online tool to generate Statechart diagrams based on your existing XState Machine definitions. All we have to do to generate a Statechart is to paste your call to the <span class="lang:default decode:true crayon-inline ">Machine</span> function. This is particularly handy to share with non-developer stakeholders to have discussions about the call structure.

<a href="https://www.nexmo.com/wp-content/uploads/2019/06/image1.png"><img class="alignnone size-full wp-image-29448" src="https://www.nexmo.com/wp-content/uploads/2019/06/image1.png" alt="XState Visualizer" width="800" height="259" /></a>
<h2>Self-Referencing Transitions</h2>
A state can transition into <a href="https://xstate.js.org/docs/guides/transitions.html#self-transitions" target="_blank" rel="noopener noreferrer">itself</a>. This can be useful for cases where you want to allow the user to playback the latest piece of information given.
<pre class="lang:default decode:true ">mainStHours: {
  on: {
    'DTMF-1': 'mainStHours',
    'DTMF-2': 'intro'  },
  meta: {
    ncco: [
      { action: 'talk', text: "Joe's Main Street is open Monday through Friday, 8am to 8pm." },
      { action: 'talk', text: 'Saturday and Sunday 9am to 7pm.' },
      { action: 'talk', text: 'Press 1 to hear this information again.' },
      { action: 'talk', text: 'Press 2 to go back to the opening menu.' },
      { action: 'input', eventUrl: [ 'https://example.com/dtmf' ], maxDigits: 1 }
    ]
  }
}
</pre>
<h2>Persistence</h2>
You can register a function to be called whenever the machine transitions from one state to another using the service's <a href="https://xstate.js.org/docs/guides/interpretation.html#transitions" target="_blank" rel="noopener noreferrer"><span class="lang:default decode:true crayon-inline">onTransition</span></a> method. This can be useful to log the steps the user is taking and sending them to a remote database for future reference\analysis.

In general, XState supports <a href="https://xstate.js.org/docs/guides/states.html#persisting-state" target="_blank" rel="noopener noreferrer">serializing</a> a machine instance's data so you can persist it.
<h2>Strict Mode</h2>
When prompting the user for keypad input at any point in the call it's possible for the user to enter an input value you don't expect. For example, the user may be in a state in the call where you expect them to choose 1 if they would like to make a reservation or press 2 to listen to the opening hours. But if the user presses 9 the event sent will be <span class="lang:default decode:true crayon-inline ">DTMF-9</span>  and that's not a possible transition given the current state. Ideally we'd like to find a generic way of detecting when the user has entered an invalid input and instruct them to make the selection again.

By defining our machine with <a href="https://xstate.js.org/docs/guides/machines.html#configuration" target="_blank" rel="noopener noreferrer"><span class="lang:default decode:true crayon-inline">strict: true</span></a> we can cause the <span class="lang:default decode:true crayon-inline ">send()</span> method to throw an exception if it's passed an event that's not possible giving the current state. We can then catch that error further on up and reply with an appropriate NCCO response that will tell the user to make the selection again.
<h2>Wrapping Up</h2>
In this post we introduced the XState library and how it can be used to control the progress of a call in an IVR System powered by Nexmo, in a way that scales well for a real-world use-case. The complete code covered in this post can be found <a href="https://github.com/cowchimp/ivr-xstate-demo" target="_blank" rel="noopener noreferrer">here</a>. If you're looking for more info, both <a href="https://developer.nexmo.com/voice/voice-api/overview?utm_campaign=dev_spotlight&utm_content=IVR_XState_Mevorach" target="_blank" rel="noopener noreferrer">Nexmo</a> and <a href="https://xstate.js.org/docs" target="_blank" rel="noopener noreferrer">XState</a> have excellent documentation.