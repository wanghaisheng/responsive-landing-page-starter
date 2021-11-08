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
How can you improve the accessibility of your applications? What difficulties prevent organizations from achieving that goal? How can you solve these problems and make interfaces disability-friendly?

Many organizations dodge these accessibility-related questions during development. We commonly perceive such duties as enriching rather than a necessity, which results in web content accessibility guidelines (WCAG) remaining overlooked until they fade off in the backlog. It might also seem inconsequential to the design pattern we typically practice or even extraneous to users we assume are non-disabled.

To help improve accessibility, organizations can educate teams, recruit in-house experts, and even get 3rd party services to support repair processes. However, the latter won't stick unless carried out forever, as proper compliance to WCAG requires ongoing care and is not a one-time refurbishment.

Failing to apply WCAG and not meeting its success criteria will likely result in broken interfaces for some users. In addition to blocking the target audience from using services and damaging revenue, failing to comply with the WCAG standard is a human rights violation that exposes an organization to lawsuits.

So, we need a strategy to instill the right state of mind, focus on the joint actions teams need to take, set up workflows, and plan gradual steps to make our applications accessible.

## Suggested strategies

Consider [11y accessibility practices](https://www.a11yproject.com/) as an integral part of development. Put it right next to design, code, testing, security vulnerabilities, business logic, and more. Here are some things to think about when creating an accessibility strategy for your application. 

### Separation of concerns

At Vonage, we [maintain a library](https://github.com/Vonage/vivid) (Vivid) where organization-wide engineers can enjoy the benefits of UI-based components built from the ground up to meet WCAG success criteria.

Vonage's Vivid web UI library helps you integrate the library across Vonage products makes it easy to handle violations in a single codebase and meet required standards. 

### Prioritize

Choosing how we handle defects in-house is bound to business needs. Priorities can alter, but here's a few suggestion to start:

1. Issues picked up by crawlers and found using automated tools are often easy to handle and fix to protect the organization from lawsuits and improve sites' ranking.
2. Legal implications pages (e.g., privacy policy, nutritional information, GDRP pages).
3. High traffic pages (e.g., homepage, landing/login/campaign pages).

As for a gradual increase, WCAG defines three levels of conformance (A, AA, AAA) an organization may adopt. Most countries' laws require UX to comply with at least the first level (A). Nonetheless, we can cherry-pick on specific features, go the extra mile, and opt-in to higher levels.

### Level 1

Level 1 tasks are essential tasks with high user impact that generally require a small investment. Completing these tasks addresses many of the top concerns of people with disabilities.

### Level 2

You should complete these tasks in addition to the first level to address the next-most essential issues that may keep certain users from fully using your product.

### Level 3

Completing all three levels achieves compliance with the A and AA criteria of the Web Content Accessibility Guidelines (WCAG).

Product teams should include [accessibility targets](https://www.ibm.com/able/toolkit/plan/release#establishing-the-accessibility-scope-for-the-release) in each release. Each team member takes on specific tasks to ensure they set up the product for success. This process set regularly will outcome in improved, sustainable accessibility.

Remember, every feature you make more accessible improves the experience for some users. You don't have to solve it all at once to improve.

### Tooling

A reasonable starting point is to have automated tools, such as tests, linters, browser addons, and IDE plugins to first aid with reporting identified violations and help get up and ready to brief in on *good-first issues*.

Here are a few tools you may find helpful. 

* [Deque's Axe](https://www.deque.com/axe)
* [Microsoft accessibility insights](http://accessibilityinsights.io/)
* [Wave](https://wave.webaim.org/)
* [Site improve](https://siteimprove.com/) (big pile of services)
* [Pope.tech](https://pope.tech/)
* [Assistive labs](https://assistivlabs.com/) (Like BrowserStack for screen readers)

Keep in mind that automated tools generally pick up less than 40% of errors and are superficial (e.g., color contrast, inputs associated with labels, and more). Furthermore, compliance does not equal a genuinely accessible site. You must manually test and review your code in addition to using tools. 

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

Thank you for reading 🤍

And special thanks to [Yonatan Kra](https://yonatankra.com/) for his helpful, thorough review.