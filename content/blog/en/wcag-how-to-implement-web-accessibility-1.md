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
What does it take to improve the accessibility of applications? What difficulties prevent organizations from achieving that goal? How can you solve these problems and make interfaces disability-friendly?

Many organizations dodge these accessibility-related questions during development. We commonly perceive such duties as enriching rather than a necessity, which results in **web content accessibility guidelines** (WCAG) remaining overlooked until they fade off in the backlog. It might also seem inconsequential to the design pattern we typically practice or even extraneous to users we assume are non-disabled.

To help improve accessibility, organizations can educate teams, recruit in-house experts, and even get 3rd party services to support repair processes. However, the latter won't stick unless carried out forever, as proper compliance to WCAG requires ongoing care and is not a one-time refurbishment.

Failing to apply WCAG and not meeting its success criteria will likely result in broken interfaces for **some** users. In addition to blocking the target audience from using services and damaging revenue, failing to comply with the WCAG standard is a human rights violation that exposes an organization to lawsuits.

Hence, we need a strategy to instill a state of mind, focus on the joint actions teams need to take, set up workflows, and plan gradual steps to make our applications accessible.

## Suggested strategies

Consider 11y practices as an integral part of development. Put it right next to design, code, testing, security vulnerabilities, business logic, and more.

Let's spread out what we found as quick and most effective workflows.

### Separation of concerns

At Vonage, we strategically maintain a tailor-maid library (Vivid) where organization-wide engineers can enjoy the benefits of UI-based components built from the ground up to meet WCAG success criteria.

Benefits:

##  [Vonage ](https://github.com/Vonage)/ [vivid](https://github.com/Vonage/vivid)

### Vonage's web UI 🎨 toolbelt

1. Battery charged
2. Integrating the library across Vonage products makes it easy to handle violations in a single codebase and continuously improve it to fit required standards.
3. Flexibility
4. Components can be easily adjusted and enhanced to an even more compelling user experience by the consuming authors (devs). Plus, on its own, a11y related API may help authors better understand how to craft a11y practices.

#### In practice

Let say, for example, we default a banner component role to '*status*' with a redundant aria-live attribute set to polite (To maximize compatibility when using this role). This action indicates the screen reader should wait until the user is idle before presenting updates to the user.

However, we can modify the above attributes (role and aria-live) to fit contextually. So if the information is critical, we could still alter the banner's role to '*alert*,' telling assistive technologies to interrupt other processes and provide users with immediate notification.

As a result, Consuming authors can seamlessly concentrate on their products' progress rather than deep dive into a11y practices, as we separate the concern of eligible a11y from the other development concerns by "outsourcing" them.

Some organizations practice an even more isolated codebase dedicated to a11y; see adobe's react-aria.

### Ready-made

If setting up a dedicated team isn't a reasonable choice in your organization, a practical alternative would be to adopt a reliable 3rd party OSS that satisfies the required criteria. The Vivid team also gained a significant head-start by partially subclassing the material web components library under its hood.

### Prioritize

Choosing how we handle defects in-house is bound to business needs. Priorities can alter, but here's a suggestion to start -

1. Issues picked up by crawlers and found using automated tools are often easy to handle and fix to protect the organization from lawsuits and improve sites' ranking.
2. Legal implications pages (e.g., privacy policy, nutritional information, GDRP pages)
3. High traffic pages (e.g., homepage, landing/login/campaign pages)

As for a gradual increase, WCAG defines three levels of conformance (A, AA, AAA) an organization may adopt. Most countries' laws require UX to comply with at least the first level (A). Nonetheless, we can cherry-pick on specific features, go the extra mile, and opt-in to higher levels.

### Level 1

Level 1 tasks are essential tasks with high user impact that generally require a small investment. Completing these tasks addresses many of the top concerns of people with disabilities.

### Level 2

You should complete these tasks in addition to the first level to address the next-most essential issues that may keep certain users from fully using your product.

### Level 3

Completing all three levels achieves compliance with the A and AA criteria of the Web Content Accessibility Guidelines (WCAG).

—IBM, *[pace of completion](https://www.ibm.com/able/toolkit/plan/#pace-of-completion)*

Product teams should include [accessibility targets](https://www.ibm.com/able/toolkit/plan/release#establishing-the-accessibility-scope-for-the-release) in each release. Each team member takes on specific tasks to ensure they set up the product for success. This process set regularly will outcome in improved, sustainable accessibility.

**Remember**, every feature you make more accessible improves the experience for **some** users. You don't have to solve it all at once to improve.

### Tooling

A reasonable starting point is to have automated tools, such as tests, linters, browser addons, and IDE plugins to first aid with reporting identified violations and help get up and ready to brief in on *good-first issues*.

Here are a few of the standard tools out there -

* [Deque's Axe](https://www.deque.com/axe)
* [Microsoft accessibility insights](http://accessibilityinsights.io/)
* [Wave](https://wave.webaim.org/)
* [Site improve](https://siteimprove.com/) (big pile of services)
* [Pope.tech](https://pope.tech/)
* [Assistive labs](https://assistivlabs.com/) (Like BrowserStack for screen readers)

**Note** that "off the shelf" automated tools pick up **< 40%** of the errors and are **superficial** (e.g., color contrast, inputs associated with labels, and more). Furthermore, compliance does not equal a genuinely accessible site. We must manually test and review.

### Services

If feasible and resources apply, get 3rd party services to review applications by actual users, some even with relevant disabilities, which will provide actual "field" data on UX failures.

* [Deque](https://www.deque.com/)
* [Level Access](https://www.levelaccess.com/)
* [Audioeye](https://www.audioeye.com/)
* [Vision Australia Digital Access](https://www.visionaustralia.org/services/digital-access)
* [Digital Accessibility Centre (DAC)](http://digitalaccessibilitycentre.org/)

There are dozen paid services to choose from, but the ones that seem most genuine to me are those with real disabled users.

Disclosure: I ran into DAC reference following [gov. uk's insightful success story](https://accessibility.blog.gov.uk/2018/05/15/what-we-learned-from-getting-our-autocomplete-tested-for-accessibility/) about getting an accessibility audit.

We had our autocomplete tested by the Digital Accessibility Centre (DAC). They are one of several companies that specialize in accessibility testing and auditing. The majority of DAC's testers have access needs. They are expert users, but they depend on services being accessible in their day-to-day lives.

DAC invited us to their offices in Neath to watch them put the autocomplete through its paces...

### Communicate

Provide an accessibility statement to:

* Show your users that you care about accessibility and them
* Provide them with information about the accessibility of your content
* Demonstrate commitment to accessibility and social responsibility

—w3c, *[web accessibility initiative](https://www.w3.org/WAI/planning/statements)*

Start with

[Minimal accessibility statement](https://www.w3.org/WAI/planning/statements/minimal-example)

[Complete accessibility statement](https://www.w3.org/WAI/planning/statements/complete-example)

Or generate with

<https://www.w3.org/WAI/planning/statements/generator/#create>

[https://www.accessibilitystatementgenerator.com](https://www.accessibilitystatementgenerator.com/)

Keep an open channel for users' feedback - a wide range of disability types and assistive tools may fail in play with the applications' UI. The [following article section](https://dev.to/karkranikhil/web-accessibility-by-making-your-site-accessible-you-automatically-increase-the-target-audience-d8d#web-accessibility-statistics) hints at how diverse it gets.

### Summary

To serve all users, we need to operate on a full spectrum of disabilities UX flaws continuously.

Applying and maintaining a11y in your app(s) is a practice much like tests and security. You need to not only apply a11y but also set up the mechanisms that will allow you to maintain it.

You can get a head start with an a11y-ready UI library. If you have the sources, establish a dedicated team solely operating on UI, semantics, and accessible components.

In our experience, you will soon find out there's no "one-fits-all," and you'll need to start handling feedback - either from customers or from the business side. 

Feedback is like feature requests or bugs. You need to maintain and test it to ensure they don't break (regression is bad for business). Here automated tests come into play - either 3rd party tools or tools you build that are dedicated to your UI (using tools like Cypress).

Eventually, when you do the best, you can do it on your own and feel you need another push to get to a new level - get the help of experts.

Just like any other task, you'll need to set the gradual steps to comply with web content accessibility guidelines. It will be a joint effort of teams involved in UI development with set up workflows and a roadmap plan (also known as Epic on Jira).

Remember, every feature you make more accessible improves the experience for **some** users. You don't have to solve it all at once to improve.

I encourage everyone who is starting to learn about the topic to initiate action in their organizational environment.

Please raise any questions, arguments, concerns in the comments. I would love to hear back.

Thank you for reading 🤍

And special thanks to [Yonatan Kra](https://yonatankra.com/) for his helpful, thorough review.