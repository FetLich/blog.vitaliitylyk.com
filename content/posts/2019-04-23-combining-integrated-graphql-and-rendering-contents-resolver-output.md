---
title: Combining integrated GraphQL and Rendering Contents Resolver output
author: Vitalii Tylyk
type: post
date: 2019-04-23T07:49:25+00:00
excerpt: 'When developing with Sitecore JSS, you can use 2 approaches for shaping JSON data for a rendering  in LayoutService output: using a Rendering Contents Resolver or integrated GraphQL. They are mutually exclusive by design, however, there are situations when you might want to use both of them for the same rendering.'
url: /combining-integrated-graphql-and-rendering-contents-resolver-output/
categories:
  - Sitecore
tags:
  - GraphQL
  - JSS

---
When developing with Sitecore JSS, you can use 2 approaches for shaping JSON data for a rendering in LayoutService output: using a Rendering Contents Resolver or integrated GraphQL. They are mutually exclusive by design, however, there are situations when you might want to use both of them for the same rendering.<figure style="text-align:center"></figure> 

For example, you have implemented a <a rel="noreferrer noopener" aria-label="custom Rendering Contents Resolver to serialize arbitrary item hierarchies (opens in a new tab)" href="https://blog.vitaliitylyk.com/jss-and-arbitrary-item-hierarchies/" target="_blank">custom Rendering Contents Resolver to serialize arbitrary item hierarchies</a>, but at the same time you want to provide some extra data (items located in other parts of Sitecore tree). If you try to use integrated GraphQL, **your custom Rendering Contents Resolver will not be executed anymore** and Sitecore will only serve the result of GraphQL query. So you would have several choices:

  * Implement a Rendering Contents Resolver, which would have all the needed logic specifically for this rendering. This solution is good enough, however it is not reusable, since you would need to build a custom resolver for each rendering which requires extra data.
  * Use <a rel="noreferrer noopener" aria-label="Connected GraphQL (opens in a new tab)" href="https://jss.sitecore.com/docs/techniques/graphql/connected-graphql" target="_blank">Connected GraphQL</a> to retrieve required items on frontend side. This is quite a flexible approach, however the downside is that you application would have to make an extra HTTP call to retrieve the data, which makes it slower. Moreover, your frontend code would need to know which items to retrieve, what sort of breaks encapsulation.
  * Implement a solution to make both integrated GraphQL and custom revolvers work simultaneously. This is the approach I am going to describe further on.

## Combining the best of both worlds

The code below provides an example on how you can implement a custom Rendering Contents Resolver which is able to both execute the resolver logic **and** execute a GraphQL query defined for a rendering. Make sure to reference _Sitecore.JavaScriptServices.GraphQL_ and _Microsoft.Extensions.DependencyInjection.Abstractions_ Nuget packages.<figure class="wp-block-embed">

<div class="wp-block-embed__wrapper">
  <div class="gist-oembed" data-gist="c011bf4e03237301959d8d29f7303f44.json" data-ts="8">
  </div>
</div></figure> 

This approach is quite powerful in combination with  
custom Rendering Contents Resolver, which is able to <a rel="noreferrer noopener" href="https://blog.vitaliitylyk.com/jss-and-arbitrary-item-hierarchies/" target="_blank">serialize datasource item descendants</a>. To give an example, here is how we are using this approach on my project:

  * We have enhanced the abovementioned custom resolver to be able to both serialize datasource item descendants and execute a GraphQL query (by inheriting the `DescendantsTreeRenderingContentsResolver` from the `GraphQLCombinedResolver`).
  * All the component data is stored within the datasource item, or it&#8217;s descendants.
  * In case a rendering needs some extra data (which is not contained outside the datasource item tree), we define a GraphQL query on the rendering item.

Having the 2 approaches combined, there is no coding required to provide Sitecore items data for any rendering. I have widely used this technique while <a href="https://blog.vitaliitylyk.com/guide-on-refactoring-your-sitecore-solution-to-sitecore-jss/" target="_blank" rel="noreferrer noopener" aria-label="migrating existing Sitecore MVC solution to JSS (opens in a new tab)">migrating existing Sitecore MVC solution to JSS</a>.