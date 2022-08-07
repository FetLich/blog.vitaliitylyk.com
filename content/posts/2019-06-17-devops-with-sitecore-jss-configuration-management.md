---
title: 'DevOps with Sitecore JSS: configuration management'
author: Vitalii Tylyk
type: post
date: 2019-06-17T08:14:35+00:00
excerpt: I have already touched upon deploying JSS apps in my Guide on migrating your solution to Sitecore JSS. In this post I want to extend a bit on the topic, in particular on configuration management.
url: /devops-with-sitecore-jss-configuration-management/
featured_image: /wp-content/uploads/2019/06/devops-740x419.png
categories:
  - Sitecore
tags:
  - DevOps
  - JSS
  - Sitecore

---
<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img loading="lazy" src="https://i1.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/06/devops.png?fit=300%2C170&ssl=1" alt="Managing configuration in Sitecore JSS deployments" class="wp-image-726" width="450" height="250" srcset="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/06/devops.png?zoom=2&resize=450%2C250&ssl=1 900w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/06/devops.png?zoom=3&resize=450%2C250&ssl=1 1350w" sizes="(max-width: 450px) 100vw, 450px" /></figure>
</div>

I have already touched upon deploying JSS apps in my <a rel="noreferrer noopener" aria-label="Guide on refatoting your solution to Sitecore JSS (opens in a new tab)" href="https://blog.vitaliitylyk.com/guide-on-refactoring-your-sitecore-solution-to-sitecore-jss/" target="_blank">Guide on migrating your solution to Sitecore JSS</a>. In this post I want to extend a bit on the topic, in particular on configuration management.

Questions related to this topic popped up occasionally in Sitecore community Slack, so I decided to write up a short blog post 😉

<blockquote class="wp-block-quote">
  <p>
    Further I assume that the reader is familiar with general JSS concepts, such as <a rel="noreferrer noopener" aria-label="Layout Service (opens in a new tab)" href="https://jss.sitecore.com/docs/fundamentals/services/layout-service" target="_blank">Layout Service</a> and <a rel="noreferrer noopener" aria-label="API keys (opens in a new tab)" href="https://jss.sitecore.com/docs/getting-started/app-deployment#step-2-api-key" target="_blank">API keys</a>.
  </p>
</blockquote>

The typical requirement and the best practice in DevOps is to have environment-agnostic deployment artifacts. Any environment specific configuration should not be part of the package, so that you would have one package to be deployed to all environments. All environment specific configuration settings are normally stored and managed in the deployment tool of your choice (Octopus, Azure DevOps, etc).

With JSS apps the story is not different. What makes creation of environment-agnostic package more challenging is that, the <a rel="noreferrer noopener" aria-label="sample JSS apps (opens in a new tab)" href="https://github.com/Sitecore/jss/tree/dev/samples" target="_blank">sample JSS apps</a> provided by Sitecore embed the **apiKey** and **layoutServiceHost** configuration settings to the resulting bundles during the build time.

In case you are using the <a rel="noreferrer noopener" aria-label="Integrated Topology (opens in a new tab)" href="https://jss.sitecore.com/docs/techniques/devops#integrated-topology" target="_blank">Integrated Topology</a>, the solution is pretty simple: use blank &#8220;&#8221; value for **layoutServiceHost** and setup a fixed **apiKey** for all your environments. For the details on why it works like this &#8211; checkout the <a rel="noreferrer noopener" aria-label="&quot;Update build and deployment process&quot; section of one of my previous posts (opens in a new tab)" href="https://blog.vitaliitylyk.com/guide-on-refactoring-your-sitecore-solution-to-sitecore-jss/#update-build-and-deployment-process" target="_blank">&#8220;Update build and deployment process&#8221; section of my JSS migration guide</a>.

In this post I would like to focus on the more complex case: <a href="https://jss.sitecore.com/docs/techniques/devops#headless-topology" target="_blank" rel="noreferrer noopener" aria-label="Headless Topology (opens in a new tab)">Headless Topology</a>, when you actually have to configure different values for your **layoutServiceHost** and **apiKey** settings per environment.

## Option 1: Token replacement

The idea is to use tokens as values for the configuration settings (e.g _#{LayoutServiceHost}_ and _#{ApiKey}_) during build time of your JSS app, which would create an environment agnostic *.js bundles with tokenized configuration. During deployment stage you can replace these tokens with appropriate values per environment using deployment tool of your choice.

<blockquote class="wp-block-quote">
  <p>
    <strong>Note: </strong>if you are using the <code>jss setup</code> command to create your <em>scjssconfig.json</em> file during the build process, then you would not be able to provide a tokenized value for the <strong>apiKey</strong> setting, since in the current version of JSS CLI it has to be a valid GUID. I have filed an <a rel="noreferrer noopener" aria-label=" issue on JSS github (opens in a new tab)" href="https://github.com/Sitecore/jss/issues/187" target="_blank">issue on JSS github</a> related to this. To mitigate this, the simplest solution is to prepare a <em>scjssconfig.json</em> file with tokenized values in advance and use it in your builds.
  </p>
</blockquote>

This solution is quite simple and straightforward. However, **it will introduce some challenges if you plan to run your code in a Docker container**. Normally, Docker container configuration is managed via Environment Variables that can be easily provided to a container. In this case, since your configuration settings are embedded into *.js bundles, you would need to either replace values when container boots up, or use some other hacky solution.

## Option 2: Environment variables

Instead of baking the configuration values to *.js bundles, the idea here is to use environment variables:

  * Your Node.JS instance would naturally have access to environment variables
  * To use the same configuration on the client side, your Node.JS server code could propagate the config values to the browser by embedding them to the server response (e.g. put them to the _window object_). For example:

<pre class="EnlighterJSRAW" data-enlighter-language="js" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">&lt;script>
	window.app = {    
        sitecoreApiKey: "{2B558AF2-636F-4BD7-BAA0-45633E6E8D5C}",
        sitecoreApiHost: "http://sitecore.local"
	};
&lt;/script></pre>

This technique is used in the Umbrella React JSS starter kit, do check it out, it has a lot of other nice features besides that: <a rel="noreferrer noopener" aria-label=" (opens in a new tab)" href="https://github.com/macaw-interactive/react-jss-typescript-starter/blob/develop/samples/react-typescript/README.md" target="_blank">https://github.com/macaw-interactive/react-jss-typescript-starter/blob/develop/samples/react-typescript/README.md</a>

Same approach can be used in Angular and Vue.js, it is just the exact implementation might differ. 

This approach is much more flexible, however you would have to spend some time implementing this in frontend framework of your choice. I really hope Sitecore will update their sample JSS apps to use this approach in future 🙂