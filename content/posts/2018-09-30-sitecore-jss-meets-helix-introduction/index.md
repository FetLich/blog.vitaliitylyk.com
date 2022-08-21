---
title: 'Sitecore JSS meets Helix: introduction'
date: 2018-09-30T17:07:22+00:00
summary: 'A while ago, the Technical Preview 4 of Sitecore Javascript Services has been released, and GA release is coming soon, together with Sitecore 9.1. I am really excited about this and in my company we have already established JSS as a standard for new projects. But... How does it fit with Helix architecture? Do we still need an ORM tool like GlassMapper with JSS? If these questions are also bugging your mind - read on.'
description: "Considerations related to implementing a project on Sitecore Javascript Services (JSS): Helix architecture, ORM tools and solution setup."
url: /sitecore-jss-meets-helix-introduction/
cover:
  image: "sitecore-jss-helix.jpg"
categories:
  - Sitecore
tags:
  - JSS
  - Sitecore

---
A while ago, the Technical Preview 4 of Sitecore Javascript Services has been released, and GA release is coming soon, together with Sitecore 9.1. I am really excited about this and in my company we have already established JSS as a standard for new projects. But&#8230; How does it fit with Helix architecture? Do we still need an ORM tool like GlassMapper with JSS? If these questions are also bugging your mind &#8211; read on.

![Sitecore JSS and Helix architecture](sitecore-jss-helix.jpg#center "Sitecore JSS and Helix architecture")

JSS introduces the major difference to the way Sitecore projects are implemented: front-end developers take **full ownership of front-end code**. In comparison with the classic Sitecore project development approach, where your front-end code is mixed up and tight together with Razor views, in the JSS approach all your HTML, JS and CSS files are now isolated into a `React`, `Angular`, `Vue.js`, etc. application.

How does this influence your Helix approach?

This blog post is an introduction and food for thought on the topic. In the nearest future I plan to implement a concrete sample on GitHub and write some in-depth blog posts about it.

## Helix architecture and Sitecore JSS

To understand the impact, you should analyze the contents of each Helix layer and see how it can be mapped to the JSS way of doing things.

![Layers in Helix architecture](helix-layer-dependencies.png#center "Layers in Helix architecture")

### Feature layer

Normally in a Feature layer module you can find:

  * MVC Controllers

    **Classic approach:** Sitecore Controller Renderings use MVC controllers to implement extra logic, for example, retrieval of Sitecore items, mapping\converting them to a view model to be used in a view.  
  
    **JSS approach**: front-end application communicates with Sitecore via the [Layout Service API](https://doc.sitecore.com/xp/en/developers/hd/200/sitecore-headless-development/sitecore-layout-service.html) with no Razor rendering engine involved. Consequently, MVC controller is no longer required in-between to render a view. If you need to extend\customize the Layout Service output, you can implement custom [Rendering Contents Resolvers](https://doc.sitecore.com/xp/en/developers/hd/200/sitecore-headless-development/customizing-the-layout-service-rendering-output.html) or [Layout Service context extensions](https://doc.sitecore.com/xp/en/developers/hd/200/sitecore-headless-development/extending-context-data-returned-by-the-layout-service.html).  

    **Summary:** you don&#8217;t need MVC controllers in your JSS solutions.  
  
    &nbsp;
    
  * API Controllers
  
    Most often you use API controllers to provide front-end with non-Sitecore related data, for example, data coming from your Business Domain. I see no changes here, however, I strongly recommend you considering to split out all Business APIs from Sitecore. You can find more details on this in [my previous blog post]({{< ref "2018-09-04-microservices-architecture-for-enterprise-level-sitecore-projects" >}} "my previous blog post").  

    &nbsp;
    
  * Views
  
    **Classic approach:** `.cshtml` Razor views are responsible for producing HTML for your Sitecore components.  
  
    **JSS approach:** front-end code is separated out and Razor views are not used anymore.  
  
    **Summary:** I see no reason in trying to squeeze in JSS app files into a Helix module. I strongly believe that front-end source code should live separately from Sitecore back-end related code.  
  
    However, it definitely makes sense to **keep in sync the way front-end and back-end are split** into features by maintaining the same naming conventions for the appropriate feature folders and components. This will make it easier for you to navigate through the source code and make sure front-end and back-end developers speak the same language.  

    &nbsp;
    
  * Models/View Models
  
    **Classic approach:** Models in Sitecore projects are normally classes representing Sitecore items (typically mapped by an ORM). View models are used to pass data to views, typically combining Models with some extra data.  
  
    **JSS approach:** since there are no more controllers and views, you will likely not use models/view models very often, unless some complex Sitecore item structures need to be implemented in Layout Service API output.  
  
    **Summary:** Models will still remain, but will not be used as often as before.  

    &nbsp;
    
  * Configuration
  
    Feature layer modules contain feature-specific configuration, for example: Sitecore settings, pipeline extensions, serialization configuration. I foresee no changes here, this is still relevant for JSS projects.

    &nbsp;

  * Services, Repositories, Infrastructure Code

    **Classic approach:** typically, you can find examples of services and repositories abstracting Sitecore item API access and rarely performing some business logic.  
  
    **JSS approach:** as I&#8217;ve already mentioned, there will be less need in manual Sitecore items manipulation in your code, since most of the work is handled by the Layout Service API.   
  
    **Summary:** services and repositories will still remain, but mostly for business logic.  
    

### Project layer

Typical Project layer modules contain:

  * Layouts
  
    **Classic approach:**  `.cshtml` Razor views representing project-specific layouts.  
  
    **JSS approach:** front-end code is separated out and Razor views are not used anymore.  

    &nbsp;    

  * <span style="text-decoration: underline;">Configuration</span>  
  
    Normally the Project layer contains configuration for Site definitions and serialization configuration (e.g Unicorn config files). Same will remain in JSS projects.  
  
    &nbsp;

  * Front-end assets</span> (fonts, js, styling, images)
  
    **Classic approach:** In Project layer you normally store some project-specific markup overrides, for example, in the multi-site setup you might want the same Helix feature to have different look & feel, and Project layer is the right place for this. 

      **JSS approach:** you will store all front-end assets separately from Sitecore solution, consequently the Project layer won&#8217;t contain any front-end code.

### Foundation layer

When you think about a Foundation layer module in Helix architecture, you imagine:

  * Back-end wise
  
    Framework-type functionality, extensions, helpers which are shared across the solution. This will remain the same in JSS projects.
  * Front-end wise
  
    **Classic approach:** Shared front-end framework functionality, typically some extensions over existing frameworks, common CSS stylesheets, which can be re-used in features.  
  
    **JSS approach:** front-end app still needs to have some shared functionality to be reused across features, however, this code will not be stored within a Sitecore solution anymore.

### Alright, so what&#8217;s the verdict?

These are the key takeaways:

  * Helix modules will contain less code, due to the fact that Sitecore JSS does a lot of item manipulation work for us. This decreases the value of keeping Helix modules in separate Visual Studio projects. And as you know, having a lot of projects in your VS solution increases build time significantly.  
  
    You should remember that in Helix architecture, it is **the logical boundary** and the proper **dependency direction** which matters, **not the physical boundary** (which is created by separate VS projects). Having many projects in a Visual Studio solution is often considered as an [anti-pattern](https://lostechies.com/chadmyers/2008/07/16/project-anti-pattern-many-projects-in-a-visual-studio-solution-file/ "anti-pattern").  
  
    Therefore, you should consider combing your modules, especially in Feature and Project layers into a single Visual Studio project and enforcing Helix dependency principles in [different ways](https://www.hhog.com/blog/sitecore-helix-fxcop-rules "different ways"). But, as usual, it depends.  
    
  * Front-end developers will store their code separately from Sitecore back-end code, in some cases maybe even in separate source code repositories. However, even then it is important to keep in sync Helix feature names with an appropriate folder structure in the JSS application to make sure that code base is readable and easily maintainable.

Concrete setup will also depend on the fact if you have a multi-site solution, where you might want to share some functionality between sites.  
  
Visually, this is how I imagine Helix architecture evolving in multi-site Sitecore JSS projects:

![Multi-site JSS setup on Helix architecture](sitecore-jss-helix-architecture.png#center "Multi-site JSS setup on Helix architecture")

There are a couple of points to note:

  * In case websites do not share any functionality, there is no benefit in the shared Feature layer. 
  * Front-end application code lives separately from back-end solution, either in a separate folder of the same source control repository or even in a different repository.
  * Front-end apps internally should follow same naming conventions for Sitecore component\feature names, however, there is no benefit in forcing any Helix terminology\structure there. Give freedom to front-end developers, let them apply **their** best practices, do not enforce Helix.  
    
![Give freedom to front-end developers](freedom.jpg#center "Front-end developers demanding freedom from Sitecore/Helix restrictions")

## What about Sitecore ORM frameworks?

It is quite common nowadays to use an ORM framework (GlassMapper, Synthesis, etc) in Sitecore projects instead of using Sitecore Item APIs directly. This brings a lot of benefits, such as abstraction and improved unit testability. After mapping Sitecore items to their strongly typed POCO representations, you typically pass them to the views, which transform the data into HTML markup.

However, JSS applications communicate with Sitecore via the Layout Service API. This API transforms page structure, including placeholders, components, and their datasources into JSON format. Front-end application then uses this JSON to build a page dynamically, effectively replacing MVC Razor rendering engine.

![Wait a minute...](wait-a-minute.jpg#center "Wait a minute...")

If Sitecore items are already transformed into their JSON representation, do we still need Sitecore ORM frameworks?

We do. But, referring to the discussion above about contents of a Helix module, **much less then before:**

  * Sometimes, Sitecore component datasources are more complex than just one or more items with a bunch of fields. In these situations JSS Layout Service API won&#8217;t always be able to generate proper JSON output for front-end, hence custom code is required, which will process these complex item structures. And if this is the case, it is definitely beneficial to use an ORM tool.
  * Besides Sitecore components, it is quite common to store some site configuration in the Sitecore item tree. These are typically some settings, which might need to be changed and &#8220;owned&#8221; by content authors or website administrators. Code using those configuration settings items can still benefit from using an ORM.

## Summary

Sitecore Javascript Services SDK is definitely a game changer in Sitecore development. JSS brings a lot of innovation for front-end and back-end Sitecore developers and opens tons of possibilities. However, before starting a project on top of JSS you should do a revision of your existing solutions and plan how certain things should be done in the JSS way: solution\repository structure, deployment, development workflow (especially communication between back-end and front-end teams), etc.

Not to mention, since JSS shifts focus on front-end, you should make sure that you have the proper capacity of front-end developers in your team.