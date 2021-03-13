---
title: Introducing Vonage for Visual Studio Code
description: Announcing the new Vonage for Visual Studio Code extension,
  allowing developers to manage their Vonage applications, numbers and more from
  within their favorite IDE.
author: michaeljolley
published: true
published_at: 2021-03-13T20:52:57.077Z
updated_at: 2021-03-13T20:52:57.085Z
category: announcement
tags:
  - opensource
  - vscode
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
We're always working to make building with our APIs easier for all developers.
With our new Vonage for Visual Studio Code extension, we're bringing our APIs
closer to your development experience. Vonage for VS Code gives you full
control of your Vonage applications &amp; numbers and provides code snippets to
make building your app faster than ever.

Let's review what you can do.

## Installing the Extension 

You can find the Vonage for VS Code extension in the
[Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=Vonage.vscode).
Click **install** to add the extension to your editor.

Alternatively, you can manually install the plugin:

1. See the latest [Releases](https://github.com/Vonage/vscode/releases) for
the Vonage/vscode GitHub project.
1. Download the release.vsix file from the latest release. The link is listed
under the Assets section of the release.
1. Run `code --install-extension <path to release.vsix>` from your terminal.

## Authenticate Your Account

With the extension installed, click the Vonage logo in the activity bar. Then
you'll be prompted to authenticate the extension with your account. Click the
`Provide API key & secret` button to authenticate your account. Once you've
logged in, your account information will begin to populate in the window.

![Example of logging into the extension](/content/blog/introducing-vonage-for-visual-studio-code/login.gif)

## Numbers

The Unassigned Numbers view contains, as you might expect, a listing of
your numbers that aren't assigned to applications. You can right-click any
numbers in the view to see additional options for that number, including
copying the number to the clipboard or assigning it to an application.

You can also purchase a new number by pressing the button in the title bar
of the view. This will allow you to select the country, type, and pattern
(i.e. starts with 12) for your new number.

![Quick view of some of the features for managing numbers](/content/blog/introducing-vonage-for-visual-studio-code/numbers.gif)