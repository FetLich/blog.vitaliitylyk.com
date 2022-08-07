---
title: Microservices architecture for enterprise level Sitecore projects
author: Vitalii Tylyk
date: 2018-09-04T18:44:04+00:00
summary: Business logic heavy Sitecore applications are often built as monoliths, which turn out to be a maintenance nightmare over time. If you are building a Sitecore project which is more complex than a set of content pages, then welcome under the cover.
url: /microservices-architecture-for-enterprise-level-sitecore-projects/
cover:
  image: "images/sitecore-full-on-micro-services-architecture.png"
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

<div class="wp-block-image">
  <figure class="aligncenter"><img loading="lazy" width="568" height="446" src="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/monolithic-sitecore-project-architecture-white.png?resize=568%2C446&#038;ssl=1" alt="Monolithic Sitecore project architecture" class="wp-image-292" srcset="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/monolithic-sitecore-project-architecture-white.png?w=568&ssl=1 568w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/monolithic-sitecore-project-architecture-white.png?resize=300%2C236&ssl=1 300w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/monolithic-sitecore-project-architecture-white.png?resize=548%2C430&ssl=1 548w" sizes="(max-width: 568px) 100vw, 568px" data-recalc-dims="1" /><figcaption>Monolithic Sitecore project architecture</figcaption></figure>
</div>

While this is **perfectly fine for the first type of applications**, the latter ones will eventually suffer from the following:

  * Limited **scalability of an application**. Web APIs cannot be scaled independently, therefore the whole Sitecore instance has to be scaled. And Sitecore requires a lot more hardware resources than a simple ASP.NET Web Api application.
  * Limited **scalability** **of a team**. Developers implementing business logic Web APIs have to know at least basics of Sitecore to be able to work.
  * Limited **choice of technology.** How about implementing business logic APIs on `.NET Core` ? `Node.js`? This kind of options are not available if Web APIs are embedded into a Sitecore solution.
  * **Performance**. Sitecore with all its power adds some overhead to every HTTP request, which is not always needed for Web APIs executing outside of content domain.

Huh, quite a big list&#8230; Solution to these issues would be splitting up the project to independent microservices.

## Split the business domain

Looking into the diagram above, 2 high level sub domains are clearly identifiable: Content and Business. Splitting them to separate microservices will result in something like this:

<div class="wp-block-image">
  <figure class="aligncenter"><img loading="lazy" width="701" height="430" src="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/spit-out-business-domain-into-a-separate-micro-service-white.png?resize=701%2C430&#038;ssl=1" alt="Spit out business domain into a separate microservice(s)" class="wp-image-293" srcset="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/spit-out-business-domain-into-a-separate-micro-service-white.png?w=701&ssl=1 701w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/spit-out-business-domain-into-a-separate-micro-service-white.png?resize=300%2C184&ssl=1 300w" sizes="(max-width: 701px) 100vw, 701px" data-recalc-dims="1" /><figcaption>Spit out business domain into a separate microservice(s)</figcaption></figure>
</div>

Such a split by itself solves all the issues from the list above. Did you notice the `ASP.NET Core` there on the diagram? Hell yeah, now it is possible to play around with latest technology from Microsoft even in Sitecore project. This by itself bring a lot of benefits, e.g. performance improvement and happy developers.

On top of that, the Business Logic application can now be scaled independently and be maintained by developers without Sitecore knowledge.

Okay, now that Business domain is separated, let&#8217;s see what can be improved in the Presentation Layer:

  * Work of front-end and back-end developers is **tightly coupled**: either back-end developers need to integrate work of front-end (copy HTML into `.cshtml` files) or front-end developers need to understand Razor syntax in order to modify it. In the second scenario, to be able to test their work, front-end developers need to know how to setup a Sitecore instance on their machine, which also means that they are bound to Windows. And we all know how front-end guys like Macs.
  * Modern front-end websites are normally built as **Single Page Applications**. Until recently, it was quite a challenge to build a SPA Sitecore application, since out-of-the-box SPAs don&#8217;t integrate well with Experience Editor and Analytics.

To handle these issues, Sitecore Javascript Services come to the rescue.

## Sitecore JSS &#8211; Sitecore as a headless CMS microservice

Recently (finally!) Sitecore made a great move and brought modern front-end development (SPAs on top of `Angular`, `React`, `Vue.js`) into Sitecore world. I will not dig into details here, this topic deserves a separate post, or maybe even a separate dedicated website with documentation, and luckily Sitecore has created one: <a href="https://jss.sitecore.net/" target="_blank" rel="nofollow noopener">https://jss.sitecore.net/</a>.  So why not to make one more step and split up the presentation layer? In the following diagram, front-end application is an orchestrator of application as a whole:

<div class="wp-block-image">
  <figure class="aligncenter"><img loading="lazy" width="724" height="580" src="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/sitecore-full-on-micro-services-architecture-white.png?resize=724%2C580&#038;ssl=1" alt="Sitecore full on micro services architecture" class="wp-image-294" srcset="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/sitecore-full-on-micro-services-architecture-white.png?w=724&ssl=1 724w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/sitecore-full-on-micro-services-architecture-white.png?resize=300%2C240&ssl=1 300w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/sitecore-full-on-micro-services-architecture-white.png?resize=537%2C430&ssl=1 537w" sizes="(max-width: 724px) 100vw, 724px" data-recalc-dims="1" /><figcaption>Sitecore full on micro services architecture </figcaption></figure>
</div>

Now front-end Sitecore developers can finally build Single Page Applications consuming Sitecore content from an API (or from local JSON files, when working in <a href="https://jss.sitecore.net/docs/fundamentals/application-modes#disconnected-developer-mode" target="_blank" rel="nofollow noopener">disconnected mode</a>), sitting behind their MacBooks.

Looks amazing, doesn&#8217;t it? It is, but microservices architecture with all its benefits also brings additional challenges and extra complexity.

## Challenges

  * What if there is a need to integrate Business Domain with Sitecore analytics? In case of a banking project, how would, for example, Business API application trigger an analytics goal when a customer creates a new savings account? It is Sitecore independent now.
  * What if Business API needs some localization, for example if error messages coming back from APIs need to be translated into multiple languages? **Getting those from Sitecore database directly is a no-go** because microservices have to be independent and not know about internal structure of each other.
  * Classical challenges of microservices architecture: deployment complexity, communication between microservices, eventual consistency, etc.

All of these issues are solvable of course, however discussion on these topics deserves separate blog post or two.

## Summary

There is no silver bullet architecture, every approach has its benefits and challenges and is applicable for certain type of applications and business domains. In enterprise level Sitecore projects, where content is not the main part, it is definitively beneficial to apply microservices architecture from start of the project.

In simple content-based websites it is perfectly fine to have a monolithic Sitecore solution. Following Helix architecture principles will make sure that code is well structured and maintainable enough. But even then, it is worth considering to apply headless approach and split out the presentation layer by separating out front-end to a Node.JS app. Besides improved scalability, this will give front-end developers ability to use modern frameworks and make them less dependent on Sitecore.