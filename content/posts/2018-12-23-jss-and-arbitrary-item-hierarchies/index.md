---
title: JSS and arbitrary item hierarchies
date: 2018-12-23T20:20:10+00:00
summary: Quite often in Sitecore development you have to work with hierarchical item structures, which resemble your data model. Example could be a multi-level menu or, as in my case, forms. At the time of writing, out-of-the-box Sitecore JSS is not able to serialize item structures with arbitrary number of levels deep into a JSON tree. Luckily, like the most of Sitecore functionality, JSS is easily customizable ;)
url: /jss-and-arbitrary-item-hierarchies/
categories:
  - Sitecore
tags:
  - GraphQL
  - JSS

---
Quite often in Sitecore development you have to work with hierarchical item structures, which resemble your data model. Example could be a multi-level menu or, as in my case, forms. At the time of writing, out-of-the-box Sitecore JSS is not able to serialize item structures with arbitrary number of levels deep into a JSON tree. Luckily, like the most of Sitecore functionality, JSS is easily customizable 😉

## The use case

To be more concrete, let&#8217;s focus on the multi-level menu example, where  
the amount of levels in the hierarchy is arbitrary. Let&#8217;s say we have a datasource&nbsp; item structure like this:

![Multi-level hierarchical item structure](multi-level-items.png#center "Multi-level hierarchical item structure")

Sitecore JSS comes with 5 out-of-the-box rendering contents resolvers. They are well described in here [Customizing Layout Service Rendering Output](https://jss.sitecore.com/docs/techniques/extending-layout-service/layoutservice-rendering-contents "Customizing Layout Service Rendering Output"), together with the information on how to implement a custom one. The **Folder Filter Resolver**&nbsp;can serialize descendants of the datasource item, but it excludes folders and the result is just a &#8220;flat&#8221; array of items. For the above-mentioned example, the Layout Service output for the **Menu** rendering would be (_some fields are omitted for simplicity_): 

```json
{
   "uid":"ff8e58f5-a7c7-484b-b9be-e1c2c63b0fb5",
   "componentName":"Menu",
   "dataSource":"{C3382D8D-3048-41D9-B3FC-80D037476BCD}",
   "fields":{
      "items":[
         {
            "name":"Home",
            "fields":{ ... }
         },
         {
            "name":"Categories",
            "fields":{ ... }
         },
         {
            "name":"Sitecore",
            "fields":{ ... }
         },
         {
            "name":"JSS",
            "fields":{ ... }
         },
         {
            "name":"DevOps",
            "fields":{ ... }
      ]
   }
}
```

As you&nbsp; can see, menu items in the JSON output are flattened into an array, however, what I would like to achieve is (_some fields are omitted for simplicity_):

```json
{
   "uid":"ff8e58f5-a7c7-484b-b9be-e1c2c63b0fb5",
   "componentName":"Menu",
   "dataSource":"{C3382D8D-3048-41D9-B3FC-80D037476BCD}",
   "fields":{
      "items":[
         {
            "name":"Home",
            "fields":{...}
         },
         {
            "name":"Categories",
            "fields":{
               "items":[
                  {
                     "name":"Sitecore",
                     "fields":{
                        "items":[
                           {
                              "name":"JSS",
                              "fields":{...}
                           }
                        ]
                     }
                  },
                  {
                     "name":"DevOps",
                     "fields":{...}
                  }
               ]
            }
         }
      ]
   }
}
```

To accomplish this, I have implemented a custom Rendering Contents Resolver, which is able to serialize arbitrary hierarchies to a tree.

## Implementing the resolver

### **Step 1:**&nbsp;implement the code

Create a custom rendering contents resolver by inheriting from the&nbsp;`Sitecore.LayoutService.ItemRendering.ContentsResolvers.RenderingContentsResolver` class.

Make sure to install the&nbsp;_Sitecore.LayoutService_ Nuget package to your project from Sitecore&#8217;s MyGet feed.

The code below recursively iterates through items, serializing their children to the &#8216;items&#8217; property. Note the use of the base class&nbsp;`ProcessItem` and `ProcessItems` methods, which ensures the **Experience Editor support**.

**<u>Note</u>:**&nbsp;since the below code recursively iterates through item&#8217;s `.Children`&nbsp;properly, this **can lead to performance issues** on large item trees (in my scenario the tree structures are rather small, so it is not an problem). Therefore, depending on your situation, you might want to optimize the code.&nbsp;For example, the recursion can be replaced with a single call of the `.GetDescendants()` method, which would result in better performance. Also **make sure to use HTML caching** of renderings on top of that.

{{< gist vitaliitylyk 8a8216dd0cd5f4c484c7912aa26ee192 >}}

### **Step 2: create a resolver Sitecore item**

To be able to use the resolver in your renderings, you need to create a Rendering Contents Resolver item under&nbsp;_/sitecore/system/Modules/Layout Service/Rendering Contents Resolvers_. For example:

![Custom rendering contents resolver item in Sitecore](content-resolver-item.png#center "Custom rendering contents resolver item in Sitecore")

### Step 3: use the custom resolver in your rendering

Update the _Rendering Contents Resolver_&nbsp;field in the _Layout Service_&nbsp;section of your JSS _Json Rendering_.&nbsp;

![Json rendering configuration](json-rendering-configuration.png#center "Json rendering configuration")

## Alternative solutions

As an alternative to a custom _Rendering Contents Resolver_,&nbsp; you could use the [Sitecore GraphQL API](https://jss.sitecore.com/docs/techniques/graphql/graphql-overview "Sitecore GraphQL API") to shape the JSON output for your rendering and include datasource item children. However, this would not work out of the box with arbitrary item hierarchies, since GraphQL does not support &quot;recursive&quot; queries, which is well described here: [Dealing with arbitrary hierarchies](https://jss.sitecore.com/docs/techniques/graphql/graphql-overview#dealing-with-arbitrary-hierarchies "Dealing with arbitrary hierarchies").

Nevertheless, it is still a valid solution in case you have a fixed amount of levels of item children.&nbsp;

For the menu items example with 3 levels of items, the GraphQL query (for the [Integrated GraphQL](https://doc.sitecore.com/xp/en/developers/hd/200/sitecore-headless-development/integrated-graphql-in-jss-apps.html "Integrated GraphQL")) might look like:

```graphql
{
  dsItem: item(path: $datasource) {
    id
    name
    children {
      ... on MenuItem {
        name
        link {
          value
        }
        children {
          ... on MenuItem {
            name
            link {
              value
            }
            children {
              ... on MenuItem {
                name
                link {
                  value
                }
              }
            }
          }
        }
      }
    }
  }
}
```

## Summary

There are multiple ways to deal with hierarchical item structures in JSS. As usual, the concrete approach depends on the use case. 

  1. If item hierarchy is arbitrary, then you can go for a custom Rendering Contents Resolver. It will work for any rendering and data structure. This also means that the resolver **can be reused in multiple renderings**.
  2. In case amount of item levels in hierarchy is known in advance it might be enough to use GraphQL query to shape the output. 
      * However, if you use Connected Mode take into account that this will result into additional queries to Sitecore&#8217;s GraphQL endpoint.
      * If you use Integrated Mode, there is no additional calls made and data is shaped within the Layout Service output**.**
      * **This approach is not reusable**, meaning you will have to build specific queries for each component.