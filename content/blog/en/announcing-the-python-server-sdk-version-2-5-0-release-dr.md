---
title: Announcing the Python Server SDK Version 2.5.X Release
description: The 2.5.0 release of the Python Server SDK includes changes to make
  the SDK more modular, including separate classes for the SMS, Voice, and
  Verify APIs.
thumbnail: /content/blog/announcing-the-python-server-sdk-version-2-5-0-release-dr/Blog_SDK-Updates_1200x600.png
author: diana-rodriguez
published: true
published_at: 2020-08-24T15:19:39.000Z
updated_at: 2021-05-11T17:21:37.470Z
category: release
tags:
  - python
comments: true
redirect: ""
canonical: ""
---
I'm super excited to announce the release of our new Python v2.5.X SDK. This is my first release since joining the Platform & Developer Experience team at Vonage this year, and I'm thrilled to share all the new things we have in store.

In the past couple of days, the excitement of releasing a new version led me to introduce some breaking changes that are suitable for a major release. These changes have been patched with our last version 2.5.2.

Up to version 2.4.0 the Voice, SMS and Verify methods were contained in the client. From now on and for future versions these methods are now separate independent classes, outside of the client. For this version and until our next major release we will still support these methods in the client, but I added a `@deprecated` tag to indicate that for future releases, these methods will not be in the client anymore.

Please take a look at our updated readme to get familiar with the new classes. Many amazing things are coming up for our next major release, and we are looking forward to it!

## New Features

As a way to make our SDK more modular, we have added separate classes for SMS, Voice and Verify APIs. All implementation examples are contained within our repository documentation.

We have also added getters and setters to extract or rewrite custom attributes, and we are improving our error handling methodology with a best-effort description of what went wrong if they encounter an error.

### Unit Tests

We've added a whole new suite of unit tests. Everything that is not tested is either legacy or a third party file we've incorporated into the SDK.

## Breaking Changes

We've done our best to ensure that the upgrade path to 2.5.2 will be as seamless as possible.

The new structures should not affect current users of the SDK.

### Dropping support for Python 2.7

After sunsetting of Python 2.7 in January 1st 2020, upcoming releases will not include support for this version. We encourage users to upgrade to Python 3. You can read more about it directly from the Python Software Foundation in [here](https://www.python.org/doc/sunset-python-2/#:~:text=We%20are%20volunteers%20who%20make,a%20security%20problem%20in%20it.)

## What's coming next

There are many more changes coming up soon. We are aiming for a more modular and strongly typed architecture for our SDK!

Until then, if you have any questions, feel free to find us on our community slack.

## Resources

- You can find our Pypi package [here](https://pypi.org/project/nexmo/)
- Our SDK is maintained in [GitHub](https://github.com/nexmo/nexmo-python), please feel free to come and explore, open an issue if you encounter one, or maybe even contribute to the effort!
