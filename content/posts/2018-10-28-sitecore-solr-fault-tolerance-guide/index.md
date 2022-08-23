---
title: Sitecore + Solr – fault tolerance guide
date: 2018-10-28T19:34:07+00:00
summary: A search engine is a crucial component of Sitecore scaled setup, as almost all of the Sitecore server roles depend on it. However, like any other software, it can fail. Therefore, it is very important to make sure that such failures are handled gracefully. Sitecore eventually did a good job in handling Solr connectivity issues, however there are still some pitfalls you should know about.
description: Guide on how to prepare for Solr down times, and pitfalls related to graceful error handling, fault tolerance and high availability Solr setup for Sitecore.
url: /sitecore-solr-fault-tolerance-guide/
cover:
  image: "Sitecore-Solr.jpg"
  relative: true
categories:
  - Sitecore
tags:
  - Sitecore
  - Solr

---
A search engine is a crucial component of Sitecore scaled setup, as almost all of the Sitecore server roles depend on it. However, like any other software, it can fail.&nbsp;Therefore, it is very important to make sure that such failures are handled gracefully. Sitecore eventually did a good job in handling Solr connectivity issues, however there are still some pitfalls you should know about.

![Solr fault tolerance guilde](Sitecore-Solr.jpg#center "Sitecore and Solr connection")

*Disclaimer:* this post is not an ultimate guide on how to handle all possible Solr errors, however it provides an idea on what to take into account when using Solr as a Sitecore search engine.

## Sitecore versions earlier than 8.2 Update-1

As discussed here [Gracefully handle Solr search connectivity issues](https://sitecore.stackexchange.com/questions/2993/gracefully-handle-solr-search-connectivity-issues "Gracefully handle Solr search connectivity issues") and here [Solr failure brings down live Sitecore site](https://blogs.perficientdigital.com/2017/02/07/solr-failure-brings-down-live-sitecore-site/ "Solr failure brings down live Sitecore site") prior to **Sitecore 8.2 Update-1** Sitecore had a hard time when Solr was not available, bringing the whole site down.&nbsp;

The fix Sitecore came up with introduces the `IsSolrAliveAgent`, which is a Sitecore job running periodically to check Solr status. In case Solr is not available, Sitecore gracefully handles failures (queries will return empty results). In addition to that, the new&nbsp;`IndexingStateSwitcher`&nbsp;job makes sure that indexing operations are suspended, while Solr is not available.  


As soon as Solr becomes available, everything gets back to normal.&nbsp;For the record, the issue reference numbers are 391039 and 94024.

Eventually, in newer Sitecore releases, Solr fault handling became better and better. However, there are&nbsp;still some specific cases where issues might pop up. For example, there is an issue with initialization of the&nbsp;`SwitchOnRebuildSolrCloudSearchIndex`&nbsp;in case Solr is not available at the moment when Sitecore starts up. If this is your case, you should contact Sitecore Product Support Services to double check that this fix is applicable for you: [https://github.com/SitecoreSupport/Sitecore.Support.163850.171950/releases/tag/8.2.6.0](https://github.com/SitecoreSupport/Sitecore.Support.163850.171950/releases/tag/8.2.6.0)

There is also a nice blog post about the whole story of the IsSolrAliveAgent by Grant Killian: [The tale of the IsSolrAliveAgent for Sitecore](https://grantkillian.wordpress.com/2018/09/28/the-tale-of-the-issolraliveagent-for-sitecore/ "The tale of the IsSolrAliveAgent for Sitecore")

## Sitecore 9 and rendering datasource validation

Sitecore performs rendering datasource validation in Experience Editor mode to make sure that the appropriate datasource item exists.

This logic is baked into the&nbsp;`Sitecore.Mvc.ExperienceEditor.Pipelines.Response.RenderRendering.AddWrapper` processor, which is a part of the&nbsp;`mvc.renderRendering` pipeline. Sitecore is patching this pipeline in the&nbsp;`Sitecore.MVC.ExperienceEditor.config`&nbsp;file.

Starting from **Sitecore 9.0 Update-1**, Sitecore validates datasource using a search provider, performing an index lookup instead of checking the database, as before. This makes sense, so far so good.

However **Sitecore 9.0 Update-1** comes with an unfortunate bug: the above mentioned processor does not have a check on whether it is executed in the Experience Editor context. This means that the logic of checking the rendering datasource for validity will also be executed as a part of normal page rendering. And in case Solr is not available &#8211; boom, **all website renderings will not be rendered and you will see empty pages on your live website.**

*Note:* this is also relevant if you are using Azure Search provider.

The symptoms are the following errors in Sitecore logs:

> WARN '{XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX}' is not valid datasource for master or user does not have permissions to access.
> WARN Failed to execute datasource query System.NullReferenceException: Object reference not set to an instance of an object.

This bug has been fixed in **Sitecore 9.0 Update-2.**&nbsp;If you are running **Update-1**&nbsp;you will have several options:

  * Upgrade to&nbsp;**Sitecore 9.0 Update-2**. This is the preferable approach since besides this fix it also comes with a bunch of other bug fixes and improvements. However, as we all know, upgrades can be painful, so&#8230;
  * Disable the&nbsp;_App_Config\Sitecore\MVC.ExperienceEditor\Sitecore.MVC.ExperienceEditor.config_&nbsp;on your **Content Delivery** servers. This will make sure, that the above-mentioned processor is not patched and Sitecore does not validate each rendering against your Solr index. As a side note, I don&#8217;t think that the&nbsp;_Sitecore.MVC.ExperienceEditor.config_ patch should be part of Content Delivery WDP packages in Sitecore 9 distribution at all, since Experience Editor functionality is not relevant for CD nodes.

## Use SolrCloud

Starting from 9.0 Update-2, Sitecore supports SolrCloud (cluster of Solr nodes, which enables fault tolerance and high availability). Such a&nbsp;setup is more complex and introduces additional servers to your infrastructure, so you should decide whether additional costs and work involved are relevant for your project.

First of all, there are 3 types of users, which can be affected by Solr down times: _website visitors_, _content authors_ and _marketeers_. Let&#8217;s see how they can be affected in the 2 following scenarios:

  1. You are using Sitecore Content Search on your website. If Solr is not available in this scenario, parts of functionality which depend on it (e.g search forms) will not be working. In addition to that, this will affect Sitecore UI (e.g Content Editor search box, Experience Profile) and Analytics data processing components which rely on indexes. Therefore, this scenario affects all 3 above-mentioned groups of users. In this scenario, it is very important that Solr is always up and running and failures are handled gracefully.  
     **In this case SolrCloud setup is highly recommended.**
  2. You use Solr **only** for Sitecore internals (Analytics indexes, Sitecore UI search, etc). In this case, it is acceptable that Solr is occasionally down, since website visitors are not affected, so only _content authors_ and _marketers_ will experience issues. In the worst case marketers would not be able to see up-to-date analytics, or content authors won&#8217;t be able to use some Sitecore UI features, which rely on search engine.  
    There is **no big value** in setting up a highly available SolrCloud cluster. But still, if you have available resources, than this would be a more future-proof approach.

## Gracefully handle connectivity errors in code

Make sure that you don&#8217;t put any logic, relying on the search engine into the page processing pipeline. Isolate the code using the Content Search API within the appropriate components so that in the edge case, only these parts would not be available.

## Summary

To wrap up, here are some takeaways for you:

  * **Always test** how your website behaves in case certain parts of your application are not available. This is relevant not only for Solr, but for any component in your system.
  * Plan your Solr infrastructure setup for high availability, especially if your website functionality, which is visible to website visitors, depends on it.
  * **Keep Sitecore up-to-date** when possible, as its error handling logic keeps improving. Not to mention, this also brings a lot of other benefits, such as latest features and improvements.
  * Handle search-related errors gracefully in your custom code.