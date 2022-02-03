---
title: Serverless Contact Centre with C# and Azure
description: Creating a low cost contact centre with Voice API and Azure Functions
thumbnail: /content/blog/serverless-contact-centre-with-c-and-azure/serverless-contact-center.png
author: matt-hunt
published: true
published_at: 2022-02-11T09:00:43.238Z
updated_at: 2022-02-11T09:00:43.252Z
category: tutorial
tags:
  - voice-api
  - "#dotnet"
  - "#azure"
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
I would be surprised if there isn’t a company out there, from the smallest startup to the largest mega-corporation, that doesn’t want to provide its customers with brilliant customer service. Part of that service could be offering a dynamic and affordable contact centre that can provide self-service options or direct calls to the correct agent or department. Vonage's Voice API and it’s NCCOs are an easy way to build high-quality voice applications that can control the flow of inbound and outbound calls, create conference calls, record and store calls, playback pre-recorded messages and send text-to-speech messages in 40 different languages.

These days most software one way or the other is hosted wholly or partially in the cloud and it’s no secret that without regulation cloud hosting costs can grow quickly over time. Having worked with Azure for many years I love learning about the different services it has to offer, my favourite for a while now has been Azure Functions, Microsoft’s serverless offering. They offer all of the security, reliability and scalability that you’d expect from any cloud provider at a cost that is very reasonable. In fact, using the Consumption plan the first 1,000,000 executions are free.

Armed with these two great bits of technology I thought it would be a good idea to see what it would take to create a low-cost serverless contact centre that could be expanded on or customised to suit many different requirements.

# Prerequisites

* Visual Studio 2022 Preview or Visual Studio Code
* [Vonage CLI](https://developer.vonage.com/application/vonage-cli)
* Azure Functions [Core Tools](https://github.com/Azure/azure-functions-core-tools) (V4 is used in this demo)
* An [Azure Account](https://azure.microsoft.com/en-gb/free/) (This can be set up for Free and Azure Functions are free for the first 1M calls)
* [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)

<sign-up number></sign-up>

# Overview

For our Contact Centre we will need to set up a couple of things. First a Vonage Application and Number, the Number will be linked to the Application and it's the Application that will call into our Azure Function. The Function will then return Nexmo Call Control Objects (NCCO) as JSON and it is this that will describe the flow of the incoming call.

When a call is received to our number we will use text-to-speech to play a message and give the caller a couple of options. 



# Azure Resources

Before we can set up the Vonage Application we will need to know the Answer and Event URLs. To obtain these we will first need to create an Azure Function App. 

```shell
#!/bin/bash

# You must be logged into your Azure Account
az login

# Function app and storage account names must be unique.
storageName=contactcentrestorage$RANDOM
functionAppName=contactcentre$RANDOM
region=westeurope

# Create a resource group.
az group create --name ContactCentreResourceGroup --location $region

# Create an Azure storage account in the resource group.
az storage account create \
  --name $storageName \
  --location $region \
  --resource-group ContactCentreResourceGroup \
  --sku Standard_LRS

# Create a serverless function app in the resource group.
az functionapp create \
  --name $functionAppName \
  --storage-account $storageName \
  --consumption-plan-location $region \
  --resource-group ContactCentreResourceGroup \
  --functions-version 4
```

Once your function app has been created we will need to retrieve the hostname. We will use the hostname when creating the Vonage Application. We can easily retrieve the hostname with the Azure CLI command below, changing the "webapp-name" for the name of the function we just created.

```
 az functionapp config hostname list --resource-group ContactCentreResourceGroup --webapp-name contactcentre123 -o tsv --query [].name
```



# Creating the Project

```shell
func init ContactCentre —dotnet
```

This will create a Function App project in the folder ContactCentre. Change into that folder and we will create two functions that both have HTTP Triggers, one that is the Answer endpoint and the second that will be the Event endpoint.

```shell
func new --name Answer --template "HTTP trigger" --authlevel "anonymous"
func new --name Event --template "HTTP trigger" --authlevel "anonymous"
```