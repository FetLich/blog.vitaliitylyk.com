---
title: 'Sitecore JSS meets Helix: introduction'
author: Vitalii Tylyk
type: post
date: 2018-09-30T17:07:22+00:00
excerpt: 'A while ago, the Technical Preview 4 of Sitecore Javascript Services has been released, and GA release is coming soon, together with Sitecore 9.1. I am really excited about this and in my company we have already established JSS as a standard for new projects. But... How does it fit with Helix architecture? Do we still need an ORM tool like GlassMapper with JSS? If these questions are also bugging your mind - read on.'
url: /sitecore-jss-meets-helix-introduction/
featured_image: /wp-content/uploads/2018/09/sitecore-jss-helix-architecture-657x430.png
categories:
  - Sitecore
tags:
  - JSS
  - Sitecore

---
A while ago, the Technical Preview 4 of Sitecore Javascript Services has been released, and GA release is coming soon, together with Sitecore 9.1. I am really excited about this and in my company we have already established JSS as a standard for new projects. But&#8230; How does it fit with Helix architecture? Do we still need an ORM tool like GlassMapper with JSS? If these questions are also bugging your mind &#8211; read on.

<div class="wp-block-image">
  <figure class="aligncenter"><img loading="lazy" width="512" height="220" src="https://i2.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/sitecore-jss-helix.jpg?resize=512%2C220&#038;ssl=1" alt="Sitecore JSS and Helix architecture" class="wp-image-322" srcset="https://i2.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/sitecore-jss-helix.jpg?w=512&ssl=1 512w, https://i2.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/sitecore-jss-helix.jpg?resize=300%2C129&ssl=1 300w" sizes="(max-width: 512px) 100vw, 512px" data-recalc-dims="1" /></figure>
</div>

JSS introduces the major difference to the way Sitecore projects are implemented: front-end developers take **full ownership of front-end code**. In comparison with the classic Sitecore project development approach, where your front-end code is mixed up and tight together with Razor views, in the JSS approach all your HTML, JS and CSS files are now isolated into a `React`\`Angular`\`Vue.js`\ etc. application.

How does this influence your Helix approach?

This blog post is an introduction and food for thought on the topic. In the nearest future I plan to implement a concrete sample on GitHub and write some in-depth blog posts about it.

## Helix architecture and Sitecore JSS

To understand the impact, you should analyze the contents of each Helix layer and see how it can be mapped to the JSS way of doing things.<figure class="wp-block-image">

<img loading="lazy" width="1100" height="446" src="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/helix-layer-dependencies.png?resize=1100%2C446&#038;ssl=1" alt="Layers in Helix architecture" class="wp-image-332" srcset="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/helix-layer-dependencies.png?w=1404&ssl=1 1404w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/helix-layer-dependencies.png?resize=300%2C122&ssl=1 300w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/helix-layer-dependencies.png?resize=768%2C311&ssl=1 768w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/helix-layer-dependencies.png?resize=1024%2C415&ssl=1 1024w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/helix-layer-dependencies.png?resize=740%2C300&ssl=1 740w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/helix-layer-dependencies.png?resize=1100%2C446&ssl=1 1100w" sizes="(max-width: 1100px) 100vw, 1100px" data-recalc-dims="1" /> <figcaption>Layers in Helix architecture</figcaption></figure> 

### Feature layer

Normally in a Feature layer module you can find:

  * <span style="text-decoration: underline;">MVC Controllers</span>  
  
    **Classic approach:** Sitecore Controller Renderings use MVC controllers to implement extra logic, for example, retrieval of Sitecore items, mapping\converting them to a view model to be used in a view.  
  
    **JSS approach**: front-end application communicates with Sitecore via the <a href="https://jss.sitecore.net/docs/fundamentals/services/layout-service" target="_blank" rel="noopener">Layout Service API</a> with no Razor rendering engine involved. Consequently, MVC controller is no longer required in-between to render a view. If you need to extend\customize the Layout Service output, you can implement custom <a href="https://jss.sitecore.net/docs/techniques/extending-layout-service/layoutservice-rendering-contents" target="_blank" rel="noopener">Rendering Contents Resolvers</a> or <a href="https://jss.sitecore.net/docs/techniques/extending-layout-service/layoutservice-extending-context" target="_blank" rel="noopener">Layout Service context extensions</a>.  
  
    **Summary: **you don&#8217;t need MVC controllers in your JSS solutions.  
  
    
  * <span style="text-decoration: underline;">API Controllers</span>  
  
    Most often you use API controllers to provide front-end with non-Sitecore related data, for example, data coming from your Business Domain. I see no changes here, however, I strongly recommend you considering to split out all Business APIs from Sitecore. You can find more details on this in [my previous blog post][1].  
  
    
  * <span style="text-decoration: underline;">Views</span>  
  
    **Classic approach:** `.cshtml` Razor views are responsible for producing HTML for your Sitecore components.  
  
    **JSS approach:** front-end code is separated out and Razor views are not used anymore.  
  
    **Summary:** I see no reason in trying to squeeze in JSS app files into a Helix module. I strongly believe that front-end source code should live separately from Sitecore back-end related code.  
  
    However, it definitely makes sense to **keep in sync the way front-end and back-end are split** into features by maintaining the same naming conventions for the appropriate feature folders and components. This will make it easier for you to navigate through the source code and make sure front-end and back-end developers speak the same language.  
    
  * <span style="text-decoration: underline;">Models/View Models<br /><br /></span>**Classic approach:** Models in Sitecore projects are normally classes representing Sitecore items (typically mapped by an ORM). View models are used to pass data to views, typically combining Models with some extra data.  
  
    **JSS approach:** since there are no more controllers and views, you will likely not use models/view models very often, unless some complex Sitecore item structures need to be implemented in Layout Service API output.  
  
    **Summary:** Models will still remain, but will not be used as often as before.  
    
  * <span style="text-decoration: underline;">Configuration<br /><br /></span>Feature layer modules contain feature-specific configuration, for example: Sitecore settings, pipeline extensions, serialization configuration. I foresee no changes here, this is still relevant for JSS projects.<span style="text-decoration: underline;"><br /><br /></span>
  * <span style="text-decoration: underline;">Services, Repositories, Infrastructure Code<br /></span>  
    **Classic approach:** typically, you can find examples of services and repositories abstracting Sitecore item API access and rarely performing some business logic.  
  
    **JSS approach:** as I&#8217;ve already mentioned, there will be less need in manual Sitecore items manipulation in your code, since most of the work is handled by the Layout Service API.   
  
    **Summary:** services and repositories will still remain, but mostly for business logic.  
    

### Project layer

Typical Project layer modules contain:

  * <span style="text-decoration: underline;">Layouts</span>  
  
    **Classic approach:**  `.cshtml` Razor views representing project-specific layouts.  
  
    **JSS approach:** front-end code is separated out and Razor views are not used anymore.  
    
  * <span style="text-decoration: underline;">Configuration</span>  
  
    Normally the P<g class="gr_ gr\_35 gr-alert gr\_spell gr\_inline\_cards gr\_run\_anim ContextualSpelling ins-del multiReplace" id="35" data-gr-id="35">roject</g> layer contains configuration for Site definitions and serialization configuration (e.g Unicorn config files). Same will remain in JSS projects.  
  
    
  * <span style="text-decoration: underline;">Front-end assets</span> (fonts, js, styling, images)<span style="text-decoration: underline;"><br /><br /></span>**Classic approach:** In Project layer you normally store some project-specific markup overrides, for example, in the multi-site setup you might want the same Helix feature to have different look & feel, and Project layer is the right place for this.<span style="text-decoration: underline;"><br /></span>  
    **JSS approach:** you will store all front-end assets separately from Sitecore solution, consequently the Project layer won&#8217;t contain any front-end code.

### Foundation layer

When you think about a Foundation layer module in Helix architecture, you imagine:

  * <span style="text-decoration: underline;">Back-end wise<br /><br /></span>Framework-type functionality, extensions, helpers which are shared across the solution. This will remain the same in JSS projects.<span style="text-decoration: underline;"><br /><br /></span>
  * <span style="text-decoration: underline;">Front-end wise<br /><br /></span>**Classic approach:** Shared front-end framework functionality, typically some extensions over existing frameworks, common CSS stylesheets, which can be re-used in features.  
  
    **JSS approach:** front-end app still needs to have some shared functionality to be reused across features, however, this code will not be stored within a Sitecore solution anymore.

### Alright, so what&#8217;s the verdict?

These are the key takeaways:

  * Helix modules will contain less code, due to the fact that Sitecore JSS does a lot of item manipulation work for us. This decreases the value of keeping Helix modules in separate Visual Studio projects. And as you know, having a lot of projects in your VS solution increases build time significantly.  
  
    You should remember that in Helix architecture, it is **the logical boundary** and the proper **dependency direction** which matters, **not the physical boundary** (which is created by separate VS projects). Having many projects in a Visual Studio solution is often considered as an <a href="https://lostechies.com/chadmyers/2008/07/16/project-anti-pattern-many-projects-in-a-visual-studio-solution-file/" target="_blank" rel="noopener">anti-pattern</a>.  
  
    Therefore, you should consider combing your modules, especially in Feature and Project layers into a single Visual Studio project and enforcing Helix dependency principles in <a href="https://www.hhog.com/blog/sitecore-helix-fxcop-rules" rel="nofollow noopener">different ways</a>. But, as usual, it depends.  
    
  * Front-end developers will store their code separately from Sitecore back-end code, in some cases maybe even in separate source code repositories. However, even then it is important to keep in sync Helix feature names with an appropriate folder structure in the JSS application to make sure that code base is readable and easily maintainable.

Concrete setup will also depend on the fact if you have a multi-site solution, where you might want to share some functionality between sites.  
  
Visually, this is how I imagine Helix architecture evolving in multi-site Sitecore JSS projects:

<div class="wp-block-image">
  <figure class="aligncenter"><img loading="lazy" width="1047" height="685" src="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/sitecore-jss-helix-architecture.png?resize=1047%2C685&#038;ssl=1" alt="Multi-site JSS setup on Helix architecture" class="wp-image-338" srcset="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/sitecore-jss-helix-architecture.png?w=1047&ssl=1 1047w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/sitecore-jss-helix-architecture.png?resize=300%2C196&ssl=1 300w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/sitecore-jss-helix-architecture.png?resize=768%2C502&ssl=1 768w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/sitecore-jss-helix-architecture.png?resize=1024%2C670&ssl=1 1024w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/sitecore-jss-helix-architecture.png?resize=657%2C430&ssl=1 657w" sizes="(max-width: 1047px) 100vw, 1047px" data-recalc-dims="1" /><figcaption>Multi-site JSS setup on Helix architecture<br /><br /><span style="font-size: 12px; font-style: italic;">*  Visual Studio signs on the figure represent a single Visual Studio project.</span></figcaption></figure>
</div>

There are a couple of points to note:

  * In case websites do not share any functionality, there is no benefit in the shared Feature layer. 
  * Front-end application code lives separately from back-end solution, either in a separate folder of the same source control repository or even in a different repository.
  * Front-end apps internally should follow same naming conventions for Sitecore component\feature names, however, there is no benefit in forcing any Helix terminology\structure there. Give freedom to front-end developers, let them apply **their** best practices, do not enforce Helix.  
    

<div class="wp-block-image">
  <figure class="aligncenter"><img loading="lazy" width="768" height="480" src="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/freedom.jpg?resize=768%2C480&#038;ssl=1" alt="Give freedom to front-end developers" class="wp-image-340" srcset="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/freedom.jpg?w=768&ssl=1 768w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/freedom.jpg?resize=300%2C188&ssl=1 300w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/freedom.jpg?resize=688%2C430&ssl=1 688w" sizes="(max-width: 768px) 100vw, 768px" data-recalc-dims="1" /><figcaption>Front-end developers demanding freedom from Sitecore/Helix restrictions</figcaption></figure>
</div>

## What about Sitecore ORM frameworks?

It is quite common nowadays to use an ORM framework (GlassMapper, Synthesis, etc) in Sitecore projects instead of using Sitecore Item APIs directly. This brings a lot of benefits, such as abstraction and improved unit testability. After mapping Sitecore items to their strongly typed POCO representations, you typically pass them to the views, which transform the data into HTML markup.

However, JSS applications communicate with Sitecore via the Layout Service API. This API transforms page structure, including placeholders, components, and their datasources into JSON format. Front-end application then uses this JSON to build a page dynamically, effectively replacing MVC Razor rendering engine.

<div class="wp-block-image">
  <figure class="aligncenter"><img loading="lazy" width="300" height="300" src="https://i2.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/wait-a-minute.jpg?resize=300%2C300&#038;ssl=1" alt="Wait a minute..." class="wp-image-325" srcset="https://i2.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/wait-a-minute.jpg?w=300&ssl=1 300w, https://i2.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2018/09/wait-a-minute.jpg?resize=150%2C150&ssl=1 150w" sizes="(max-width: 300px) 100vw, 300px" data-recalc-dims="1" /><figcaption>Wait a minute&#8230;</figcaption></figure>
</div>

If Sitecore items are already transformed into their JSON representation, do we still need Sitecore ORM frameworks?

We do. But, referring to the discussion above about contents of a Helix module, **much less then before:**

  * Sometimes, Sitecore component datasources are more complex than just one or more items with a bunch of fields. In these situations JSS Layout Service API won&#8217;t always be able to generate proper JSON output for front-end, hence custom code is required, which will process these complex item structures. And if this is the case, it is definitely beneficial to use an ORM tool.
  * Besides Sitecore components, it is quite common to store some site configuration in the Sitecore item tree. These are typically some settings, which might need to be changed and &#8220;owned&#8221; by content authors or website administrators. Code using those configuration settings items can still benefit from using an ORM.

## Summary

Sitecore Javascript Services SDK is definitely a game changer in Sitecore development. JSS brings a lot of innovation for front-end and back-end Sitecore developers and opens tons of possibilities. However, before starting a project on top of JSS you should do a revision of your existing solutions and plan how certain things should be done in the JSS way: solution\repository structure, deployment, development workflow (especially communication between back-end and front-end teams), etc.

Not to mention, since JSS shifts focus on front-end, you should make sure that you have the proper capacity of front-end developers in your team.

 [1]: https://blog.vitaliitylyk.com/microservices-architecture-for-enterprise-level-sitecore-projects/