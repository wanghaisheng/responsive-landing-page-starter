---
title: "Minimizing Production Headaches with Log Tracing "
description: "How we used a node async wrapper to implement log tracing for the
  requests in our system, and how it substantially improved error
  investigations. "
author: avital-tzubelivonage-com
published: true
published_at: 2021-03-22T13:52:47.403Z
updated_at: 2021-03-22T13:52:47.443Z
category: engineering
tags:
  - node
  - tracing
  - errors
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
Let’s set the scene: 

There’s been a critical incident in production. It’s up to *you* to find and solve it. You investigate the logs of the hundred different services and get totally lost. Which action caused which reaction? Is the error related to this API call, or this other one?
We have all been there. 
And we can all unanimously agree that it is not fun. 

But there *is* a solution. 
A solution that won't solve all of your production problems but it will help tremendously. And the good news is that it can be easily integrated into your services. 

**Log tracing.**

Sounds simple, right? But in reality, it can be a powerful tool for almost any kind of investigation. Because in today’s modern architectures, when a user presses a single button in our site, it fires an API request to the API gateway, to a business service, to a database, to another service, to a centralized messages bus, to another service, and so on and so forth. 

It’s an overwhelming journey that is inundated with hundreds of logs that are produced by every flow that happens hundreds of times per second by hundreds of thousands of users. *Whew.*

### What do you mean, log tracing?

Before we jump into how to implement these concepts, let's quickly understand what we are talking about when we say we want to trace our logs. Tracing usually means adding some identifier so we can aggregate our data by this identifier. As to what each id stands for is a matter of taste. We can have one trace ID that is added to all the data we have on a single request. We can also have another set of span ids which will be used in the context of one service.

![diagram-1](/content/blog/minimizing-production-headaches-with-log-tracing/first-diagram.jpeg "basic log tracing flow ")

**There are two key concepts to understand:**

1. Aggregate all logs from all services for each request via tracing
2. Let the infrastructure deal with the tracing so that it doesn't interfere with the business logic

So, how do we implement it? 

### Aggregate logs for each request

This concept is pretty straightforward. We need to follow two very simple rules: 

* Add the trace ID to every log we write for a request and pass the trace ID whenever we make any IO operations. 
* If two services communicate on the same request via a messages bus, we will add to the message added to the bus the trace ID. 
  If two services communicate via http requests, we will add the trace ID to the headers of the request. This way, we can follow a single request through its entire flow without any noise of other requests or logs occurring in parallel.

### Separating business logic and infrastructure

Making sure we keep track of our tracing through every single pipe we receive or send data can require some good amount of coding. We always attempt to keep infrastructure concepts such as this as far away from our business logic as possible. Luckily, in our case, we can separate the tracing management almost entirely from our business logic. 

In our code, we write logs for various events and errors while adding data to these logs. We want to keep these flows untouched and at the same time add our tracing to all of these logs. Most of the tools we use today allow us to add interceptors to each request they receive. If we make sure to add interceptors to any IO tool we use and to our logging tool, we are in the right way of making sure we will trace all of our logs without touching any of the existing code we have. 

The easiest way I can explain it is by showing an actual example from our own services. This example is from our nodejs services but can be implemented in any language.

```javascript
function tracingMiddleware(req,res,next) {
    ns.run(() => {
        let traceId = req.headers['x-b3-traceid'];
        let spanId = req.headers['x-b3-spanid'];
        if (!traceId || !spanId) {
            traceId = uuid().replace(/-/g, '');
            spanId = uuid().replace(/-/g, '').substring(16);
        }
        ns.set('traceId', traceId);
        ns.set('spanId', spanId);
        next();
    });
}
```

This is code from the service that receives http requests, writes several logs and then makes an http request to the next service. We can see here three places where we need to add interceptors to our tools: inbound request, logging and outbound request.

Notice the `ns.set`. This is using [cls-hooked](https://www.npmjs.com/package/cls-hooked), a continuation local storage package that wraps node async hooks. It allows us to locally store the trace ID for every session. But if you're using Node 14, this functionality is already built in with [async local storage](https://nodejs.org/api/async_hooks.html#async_hooks_class_asynclocalstorage). 

We use Express as our API infrastructure to implement express [middlewares](https://expressjs.com/en/guide/using-middleware.html) that intercept each request. We extract the trace ID from the request or create a new one if we are the first service in the chain. We then set the trace ID in a session storage tool so we can fetch each in every step of the way.

We then add an interceptor to our logging tool to add to every log line the race id from our session storage: 

```javascript
function createBunyanStreamMiddleware(streams) {
    return {
        type: 'raw',
        level: process.env.LOG_LEVEL,
        stream: {
            write: (entry) => {
                if(ns && ns.active) {
                    entry['traceId'] = ns.get('traceId');
                    entry['spanId'] = ns.get('spanId');
                }
                streams.forEach((stream) => {
                        stream.stream.write(entry)
                });
            }
        }
    }
}
```

Finally, we add an interceptor to our http tool to add the trace ID to the headers of the request before it is fired out.

```javascript
function insertTracingHeaders(config = {}){
    if(ns && ns.active) {
        const traceId = ns.get('traceId');
        const spanId = ns.get('spanId');
        if (!config.headers) {
            config.headers = {};
        }
        if (traceId && spanId) {
            config.headers['x-b3-parentspanid'] = spanId;
            config.headers['x-b3-traceid'] = traceId;
        }
    }
    return config;
}
```

The entire flow will look something like this:

![diagram-2](/content/blog/minimizing-production-headaches-with-log-tracing/final-diagram.jpeg "tracing flow with interceptors ")

### In Conclusion

Log tracing can be used in all sorts of contexts, but this particular use prevents a lot of headache when investigating errors. It ensures that the request logs of each flow are easily accessible, so that you don't need to dig through thousands of unrelated logs.

We can personally say that in the services where we applied these concepts, the efficiency and value experienced during log investigation has skyrocketed. Since then, sifting through logs in any other way just doesn’t feel right.

So, next time there's a critical incident in production, and it's up to you to solve it, at least this time the logs will be on your side.