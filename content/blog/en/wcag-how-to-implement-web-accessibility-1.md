---
title: "WCAG: How to implement web accessibility "
description: "Learn how to improve your application's accessibility by making it
  WCAG compliant "
author: cory-althoff
published: true
published_at: 2021-11-08T19:14:50.200Z
updated_at: 2021-11-08T19:14:50.242Z
category: inspiration
tags:
  - wcag
  - accessibility
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
How can you improve the accessibility of your applications? Many organizations ignore accessibility during development. Companies often perceive accessibility as a feature rather than a necessity, which results in web content accessibility guidelines (WCAG) remaining overlooked until they fade off in the backlog. Prioritizing accesibility might even feel extraneous to users we assume are non-disabled.

Accessibility, however, is not a feature: it is a social issue. Eveyone has the right to access the internet and companies need to make sure they are creating their applications in a way that allows people with disabilities to use their websites. Failing to do so is a human rights violation that exposes an organization to lawsuits.

To help improve accessibility, organizations can educate teams, recruit in-house experts, and even get 3rd party services to support repair processes. 

Here are some things to think about when creating an accessibility strategy for your application. 

### Prioritize

Choosing how we handle defects in-house is bound to business needs. Priorities can alter, but here's a few suggestion to start:

1. Issues picked up by crawlers and found using automated tools are often easy to handle and fix to protect the organization from lawsuits and improve sites' ranking.
2. Legal implications pages (e.g., privacy policy, nutritional information, GDRP pages).
3. High traffic pages (e.g., homepage, landing/login/campaign pages).

As for a gradual increase, WCAG defines three levels of conformance (A, AA, AAA) an organization may adopt. Most countries' laws require UX to comply with at least the first level (A). Nonetheless, we can cherry-pick on specific features, go the extra mile, and opt-in to higher levels.

Product teams should include [accessibility targets](https://www.ibm.com/able/toolkit/plan/release#establishing-the-accessibility-scope-for-the-release) in each release. Each team member takes on specific tasks to ensure they set up the product for success. This process  will result in improved, sustainable accessibility.

Remember, every feature you make more accessible improves the experience for some users. You don't have to solve it all at once to improve.

### Tooling

You can use automated tools, such as tests, linters, browser addons, and IDE plugins to aid with reporting identified violations.

At Vonage, we [maintain a library](https://github.com/Vonage/vivid) (Vivid) where organization-wide engineers can enjoy the benefits of UI-based components built from the ground up to meet WCAG success criteria.

Vonage's Vivid web UI library helps you integrate the library across Vonage products makes it easy to handle violations in a single codebase and meet required standards. 

Here are a few other tools you may find helpful. 

* [Deque's Axe](https://www.deque.com/axe)
* [Microsoft accessibility insights](http://accessibilityinsights.io/)
* [Wave](https://wave.webaim.org/)
* [Site improve](https://siteimprove.com/) (big pile of services)
* [Pope.tech](https://pope.tech/)
* [Assistive labs](https://assistivlabs.com/) (Like BrowserStack for screen readers)

Keep in mind that automated tools generally pick up less than 40% of errors and are the errors are superficial (e.g., color contrast, inputs associated with labels, and more). 

Furthermore, compliance does not equal a genuinely accessible site. You must manually test and review your code in addition to using tools. 

### Services

If you have the resources, consider using 3rd party services that review applications by actual users, some even with relevant disabilities, which will provide actual "field" data on UX failures. Here are some services you can consider using. 

* [Deque](https://www.deque.com/).
* [Level Access](https://www.levelaccess.com/).
* [Audioeye](https://www.audioeye.com/).
* [Vision Australia Digital Access](https://www.visionaustralia.org/services/digital-access).
* [Digital Accessibility Centre (DAC)](http://digitalaccessibilitycentre.org/).

### Communicate

Your application should also have an accessibility statement to: 

* Show your users that you care about accessibility and them.
* Provide them with information about the accessibility of your content.
* Demonstrate commitment to accessibility and social responsibility.

You can [learn more about developing an accessibility statement here](https://www.w3.org/WAI/planning/statements). 

Here is a [tool for generating your statement ](https://www.accessibilitystatementgenerator.com/)you may find helpful. 

In addition to your accessibility statement, make sure you keep an open channel for user's feedback on your applicatoin as well. 

### Summary

I encourage everyone who is starting to learn about the topic to initiate action in their organizational environment.

Please raise any questions, arguments, concerns in the comments. I would love to hear back.

Thank you for reading.