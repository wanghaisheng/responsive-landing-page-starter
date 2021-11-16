---
title: How WSGI vs. ASGI is Like Baking a Cake
description: WSGI vs. ASGI
author: tonya-sims-1
published: false
published_at: 2021-11-16T20:13:44.652Z
updated_at: 2021-11-16T20:13:44.703Z
category: tutorial
tags:
  - wsgi
  - asgi
  - python
comments: false
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
If you’re like most of us and want to understand this Python WSGI vs. ASGI business, let’s break it down simply by using an example of baking a cake.



But first, what are WSGI and ASGI?



WSGI stands for Web Server Gateway Interface, and ASGI stands for Asynchronous Server Gateway interface. They both specify the interface and sit in between the web server and a Python web application or framework. 



\[INSERT DIAGRAM LIKE IN WSGI ASGI SIMPLIFIED AT 0:42]



One of their jobs is to handle incoming requests from the client, but they go about it in different ways.



Let’s look at how WSGI does it then we’ll check out ASGI.



WSGI handles requests synchronously. When requests come in, they are processed sequentially or one after the other. They have to wait until the one before it finishes before switching to a new task. 



\[INSERT DIAGRAM OF SYNCHRONOUS REQUESTS]



As you can imagine, it could take a long time for the requests to be processed, especially if there are a lot of them, and hinder performance. 



If you’ve ever used Flask, you may have noticed that it uses WSGI. Flask is a popular micro web framework that has been around for some time. It’s lightweight, battle-tested, and gets served up using WSGI.



WSGI has been the Python standard for many years until ASGI came along.



ASGI is the spiritual successor of WSGI. It processes requests asynchronously, in the opposite way of WSGI.



When requests are processed asynchronously, the beauty is that they don’t have to wait for the others before them to finish doing their tasks. The different requests can do their processing finishing in no particular order. 



\[INSERT DIAGRAM OF ASYNCHRONOUS REQUESTS]



You may have heard of the new Python web framework, FastAPI. By default, it uses ASGI, which makes it lightning fast. FastAPI is also a micro web framework with many advantages, including out-of-the-box support for asynchronous code using the Python async and await keywords, and much more! It’s also gaining traction super fast and has some of the best documentation out there. 



I promised you a cake baking delight, so let’s start our example.



Let’s say you want to bake a cake and make the frosting, both from scratch.



Since we now know that WSGI will process the requests sequentially, it will carry out the instructions step-by-step, one after the other. 



Bake a Cake

1. Prepare the baking pans
2. Preheat the oven
3. Grab the flour, baking powder, and salt
4. Stir together the dry ingredients
5. Grab the butter and sugar
6. Combine the butter and sugar
7. Grab the eggs
8. Add the eggs to the ingredients
9. Stir in the eggs
10. Pour the batter into the baking pans
11. Put the pans in the oven



Make the Frosting

1. Grab a bowl
2. Grab the powdered sugar and butter
3. Mix with a spoon
4. Grab the vanilla extract and milk
5. Stir in the vanilla and milk to the ingredients



Here’s an example of what it would look like in Python pseudocode:



Request 1



def bake_cake(request):

\# task 1

\# task 2

\# task 3



return response



Request 2



def make_frosting(request):

\# task 1

\# task 2

\# task 3



return response



We’d process Request 1 and wait until that request finishes before moving on to Request 2.



For ASGI, remember that we process requests asynchronously. So requests don’t have to wait on the others before it to finish. Our cake baking and frosting making example would look like this:



Bake a Cake

1. Prepare the baking pans
2. Preheat the oven
3. Grab the flour, baking powder, and salt
4. Stir together the dry ingredients

Make the Frosting

1. Grab a bowl
2. Grab the powdered sugar and butter

Bake a Cake

1. Grab the butter and sugar
2. Combine the butter and sugar
3. Grab the eggs
4. Add the eggs to the ingredients
5. Stir in the eggs

Make the Frosting



1. Mix with a spoon
2. Grab the vanilla extract and milk
3. Stir in the vanilla and milk to the ingredients

Bake a Cake

1. Pour the batter into the baking pans
2. Put the pans in the oven



You see here that the requests are not processed sequentially, and we can switch between tasks. Here’s what that would look like in code:



Request 1



async def bake_cake(request):

\# task 1

\# task 2

\# task 3 is taking a long time for process Request 2

\# task 4



return response



Request 2



async def make_frosting(request):

\# task 1

\# task 2

\# task 3 



return response



Hopefully, now you have a better understanding of WSGI vs. ASGI and will be able to choose a Python web framework for your next project based on these interfaces. 



As always, if you have any questions or comments, feel free to reach out to @ vonagedev.



We’d love to hear from you!