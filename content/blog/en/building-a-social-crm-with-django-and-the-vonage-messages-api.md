---
title: Building a Social CRM with Django and the Vonage Messages API
description: Learn how to build a social CRM using Django and the Vonage Messages API
author: cory-althoff
published: true
published_at: 2022-01-04T20:12:13.200Z
updated_at: 2022-01-04T20:12:13.243Z
category: tutorial
tags:
  - messages-api
  - django
  - CRM
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
One of the most used tools in the business world today is Customer Relationship Management (CRM) applications. Every business needs to connect and communicate with its customers in the best way possible. Social media is one of the most effective mediums for businesses to reach their customers conveniently.

In this article, you will learn how to build the core feature of a social CRM using Django and the Vonage Messages API. Our social CRM will help sales agents and the customer support team communicate with potential customers directly on Facebook in real-time. Let's call it Sales Fox.

### Pre-requisites

1. You need a Vonage account. If you do not have one yet, Go to the Vonage Sign-up page \[here](https://dashboard.nexmo.com/sign-up) to create an account. Then, come back here to continue with the next steps.

2. Now, login to your \[Vonage account](https://dashboard.nexmo.com/sign-in). Click on \`Your applications\` in the menu tab and select \`Create a new application\`. (picture-1)

3. Fill in the form provided to create a new application. Under the capabilities section, turn on \*\*Messages\*\* capabilities.

4. You will find two fields in Messages capabilities - inbound URL field and status URL field. These URLs are endpoints to web-hooks that help Vonage Messages API communicate with your application. For now, Fill in a dummy URL or maybe the link to your GitHub account. (picture-2)

5. The next thing is to authorize Vonage to access your Facebook business page. You can create a demo business Facebook page for the sake of this tutorial. Go to \[Connect Facebook Page](https://dashboard.nexmo.com/messages/social-channels/facebook-connect) and follow the steps outlined on the page.

6. Afterwards, Go to your application page and link your already authorized social media page to the application you created in Step 3. (picture-3)

7. Install Redis - If you're using Linux or Mac, follow the instructions \[here](https://redis.io/topics/quickstart#installing-redis). If you're using Windows, follow the instructions \[here](https://redis.com/blog/redis-on-windows-10/).

8. Install Ngrok. Ngrok provides you with a publicly available URL that forward requests to your locally running application. Go to \[Ngrok download page](https://ngrok.com/download) and follow the instructions to set Ngrok up on your computer.

Now that you have the pre-requisites completed. You need to set up your development environment for the tutorial. You need Python >= 3.6, Django 3. Also, I recommend using Visual Studio as your IDE.