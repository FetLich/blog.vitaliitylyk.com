---
title: Microservices architecture for enterprise level Sitecore projects
date: 2018-09-04T18:44:04+00:00
summary: Business logic heavy Sitecore applications are often built as monoliths, which turn out to be a maintenance nightmare over time. If you are building a Sitecore project which is more complex than a set of content pages, then welcome under the cover.
description: Splitting up enterprise level monolithic Sitecore projects into microservices by using Sitecore JSS and ASP.NET Core. Benefits and challenges.
url: /microservices-architecture-for-enterprise-level-sitecore-projects/
cover:
  image: "sitecore-full-on-micro-services-architecture.png"
categories:
  - Sitecore
tags:
  - Architecture
  - JSS
  - Microservices
  - Sitecore

---
Business logic heavy Sitecore applications are often built as monoliths, which turn out to be a maintenance nightmare over time. If you are building a Sitecore project which is more complex than a set of content pages, then this post is for you.

On a high level, there are 2 types of projects being built on top of Sitecore:

  1. Content-heavy websites. These are websites with **little or no business logic**, focused on serving large amount of content to visitors. Examples are: P&G, Heineken, sitecore.net
  2. Business applications, where **Sitecore is used only as a content repository** _(sometimes with all Experience Analytics functionality around it_). These are mainly banks, insurance companies and other enterprises.

This post is dedicated to the latter ones.

## The monolith

As we all know, there is no perfect architecture which suits to all types of projects. However, most of projects using Sitecore that I&#8217;ve seen, follow the same principle — no matter how complex an application is, it will likely be built as a monolithic Sitecore website. By saying this I mean that all business logic Web APIs will be hosted on top of Sitecore instance.

![Monolithic Sitecore project architecture](monolithic-sitecore-project-architecture.png#center "Monolithic Sitecore project architecture")

While this is **perfectly fine for the first type of applications**, the latter ones will eventually suffer from the following:

  * Limited **scalability of an application**. Web APIs cannot be scaled independently, therefore the whole Sitecore instance has to be scaled. And Sitecore requires a lot more hardware resources than a simple ASP.NET Web Api application.
  * Limited **scalability** **of a team**. Developers implementing business logic Web APIs have to know at least basics of Sitecore to be able to work.
  * Limited **choice of technology.** How about implementing business logic APIs on `.NET Core` ? `Node.js`? This kind of options are not available if Web APIs are embedded into a Sitecore solution.
  * **Performance**. Sitecore with all its power adds some overhead to every HTTP request, which is not always needed for Web APIs executing outside of content domain.

Huh, quite a big list&#8230; Solution to these issues would be splitting up the project to independent microservices.

## Split the business domain

Looking into the diagram above, 2 high level sub domains are clearly identifiable: Content and Business. Splitting them to separate microservices will result in something like this:

![Spit out business domain into a separate microservice(s)](spit-out-business-domain-into-a-separate-micro-service.png#center "Spit out business domain into a separate microservice(s)")

Such a split by itself solves all the issues from the list above. Did you notice the `ASP.NET Core` there on the diagram? Hell yeah, now it is possible to play around with latest technology from Microsoft even in Sitecore project. This by itself bring a lot of benefits, e.g. performance improvement and happy developers.

On top of that, the Business Logic application can now be scaled independently and be maintained by developers without Sitecore knowledge.

Okay, now that Business domain is separated, let&#8217;s see what can be improved in the Presentation Layer:

  * Work of front-end and back-end developers is **tightly coupled**: either back-end developers need to integrate work of front-end (copy HTML into `.cshtml` files) or front-end developers need to understand Razor syntax in order to modify it. In the second scenario, to be able to test their work, front-end developers need to know how to setup a Sitecore instance on their machine, which also means that they are bound to Windows. And we all know how front-end guys like Macs.
  * Modern front-end websites are normally built as **Single Page Applications**. Until recently, it was quite a challenge to build a SPA Sitecore application, since out-of-the-box SPAs don&#8217;t integrate well with Experience Editor and Analytics.

To handle these issues, Sitecore Javascript Services come to the rescue.

## Sitecore JSS &#8211; Sitecore as a headless CMS microservice

Recently (finally!) Sitecore made a great move and brought modern front-end development (SPAs on top of `Angular`, `React`, `Vue.js`) into Sitecore world. I will not dig into details here, this topic deserves a separate post, or maybe even a separate dedicated website with documentation, and luckily Sitecore has created one: [https://doc.sitecore.com/xp/en/developers/hd/190/sitecore-headless-development/index-en.html](https://doc.sitecore.com/xp/en/developers/hd/190/sitecore-headless-development/index-en.html).  So why not to make one more step and split up the presentation layer? In the following diagram, front-end application is an orchestrator of application as a whole:

![Sitecore full on micro services architecture](sitecore-full-on-micro-services-architecture.png#center "Sitecore full on micro services architecture")

Now front-end Sitecore developers can finally build Single Page Applications consuming Sitecore content from an API (or from local JSON files, when working in [disconnected mode](https://doc.sitecore.com/xp/en/developers/hd/200/sitecore-headless-development/working-disconnected-in-jss.html)), sitting behind their MacBooks.

Looks amazing, doesn&#8217;t it? It is, but microservices architecture with all its benefits also brings additional challenges and extra complexity.

## Challenges

  * What if there is a need to integrate Business Domain with Sitecore analytics? In case of a banking project, how would, for example, Business API application trigger an analytics goal when a customer creates a new savings account? It is Sitecore independent now.
  * What if Business API needs some localization, for example if error messages coming back from APIs need to be translated into multiple languages? **Getting those from Sitecore database directly is a no-go** because microservices have to be independent and not know about internal structure of each other.
  * Classical challenges of microservices architecture: deployment complexity, communication between microservices, eventual consistency, etc.

All of these issues are solvable of course, however discussion on these topics deserves separate blog post or two.

## Summary

There is no silver bullet architecture, every approach has its benefits and challenges and is applicable for certain type of applications and business domains. In enterprise level Sitecore projects, where content is not the main part, it is definitively beneficial to apply microservices architecture from start of the project.

In simple content-based websites it is perfectly fine to have a monolithic Sitecore solution. Following Helix architecture principles will make sure that code is well structured and maintainable enough. But even then, it is worth considering to apply headless approach and split out the presentation layer by separating out front-end to a Node.JS app. Besides improved scalability, this will give front-end developers ability to use modern frameworks and make them less dependent on Sitecore.