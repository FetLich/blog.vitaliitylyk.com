---
title: 'DevOps with Sitecore JSS: configuration management'
date: 2019-06-17T08:14:35+00:00
summary: I have already touched upon deploying JSS apps in my Guide on migrating your solution to Sitecore JSS. In this post I want to extend a bit on the topic, in particular on configuration management.
description: How to manage Sitecore JSS application configuration per environment. Creating environment agnostic JSS app packages.
url: /devops-with-sitecore-jss-configuration-management/
categories:
  - Sitecore
tags:
  - DevOps
  - JSS
  - Sitecore

---
![Managing configuration in Sitecore JSS deployments](devops.png#center "Managing configuration in Sitecore JSS deployments")

I have already touched upon deploying JSS apps in my [Guide on migrating your solution to Sitecore JSS]({{< ref "2019-03-11-guide-on-refactoring-your-sitecore-solution-to-sitecore-jss" >}} "Guide on migrating your solution to Sitecore JSS"). In this post I want to extend a bit on the topic, in particular on configuration management.

Questions related to this topic popped up occasionally in Sitecore community Slack, so I decided to write up a short blog post 😉

Further I assume that the reader is familiar with general JSS concepts, such as [Layout Service](https://doc.sitecore.com/xp/en/developers/hd/200/sitecore-headless-development/sitecore-layout-service.html "Layout Service") and [API keys](https://doc.sitecore.com/xp/en/developers/hd/200/sitecore-headless-development/create-a-sitecore-api-key.html "API keys").

## Environment-agnostic artifacts

The typical requirement and the best practice in DevOps is to have environment-agnostic deployment artifacts. Any environment specific configuration should not be part of the package, so that you would have one package to be deployed to all environments. All environment specific configuration settings are normally stored and managed in the deployment tool of your choice (Octopus, Azure DevOps, etc).

With JSS apps the story is not different. What makes creation of environment-agnostic package more challenging is that, the [sample JSS apps](https://github.com/Sitecore/jss/tree/dev/samples "sample JSS apps") provided by Sitecore embed the `apiKey` and `layoutServiceHost` configuration settings to the resulting bundles during the build time.

In case you are using the [Integrated Topology](https://doc.sitecore.com/xp/en/developers/hd/200/sitecore-headless-development/jss-topologies.html#integrated-topology "Integrated topology"), the solution is pretty simple: use blank &#8220;&#8221; value for `layoutServiceHost` and setup a fixed `apiKey` for all your environments. For the details on why it works like this &#8211; checkout the [&#8220;Update build and deployment process&#8221; section of my JSS migration guide]({{< ref "2019-03-11-guide-on-refactoring-your-sitecore-solution-to-sitecore-jss#update-build-and-deployment-process" >}} "&#8220;Update build and deployment process&#8221; section of my JSS migration guide").

In this post I would like to focus on the more complex case: [Headless Topology](https://doc.sitecore.com/xp/en/developers/hd/200/sitecore-headless-development/jss-topologies.html#headless-topology "Headless topology"), when you actually have to configure different values for your `layoutServiceHost` and `apiKey` settings per environment.

### Option 1: Token replacement

The idea is to use tokens as values for the configuration settings (e.g `#{LayoutServiceHost}` and `#{ApiKey}`) during build time of your JSS app, which would create an environment agnostic *.js bundles with tokenized configuration. During deployment stage you can replace these tokens with appropriate values per environment using deployment tool of your choice.


_Note:_ if you are using the `jss setup` command to create your `scjssconfig.json` file during the build process, then you would not be able to provide a tokenized value for the `apiKey` setting, since in the current version of JSS CLI it has to be a valid GUID. I have filed an [issue on JSS github](https://github.com/Sitecore/jss/issues/187 "issue on JSS github") related to this. To mitigate this, the simplest solution is to prepare a `scjssconfig.json` file with tokenized values in advance and use it in your builds.

This solution is quite simple and straightforward. However, **it will introduce some challenges if you plan to run your code in a Docker container**. Normally, Docker container configuration is managed via Environment Variables that can be easily provided to a container. In this case, since your configuration settings are embedded into *.js bundles, you would need to either replace values when container boots up, or use some other hacky solution.

### Option 2: Environment variables

Instead of baking the configuration values to *.js bundles, the idea here is to use environment variables:

  * Your Node.JS instance would naturally have access to environment variables
  * To use the same configuration on the client side, your Node.JS server code could propagate the config values to the browser by embedding them to the server response (e.g. put them to the _window object_). For example:

```js
window.app = {    
      sitecoreApiKey: "{2B558AF2-636F-4BD7-BAA0-45633E6E8D5C}",
      sitecoreApiHost: "http://sitecore.local"
};
```

This technique is used in the Umbrella React JSS starter kit, do check it out, it has a lot of other nice features besides that: [https://github.com/macaw-interactive/react-jss-typescript-starter/blob/develop/samples/react-typescript/README.md](https://github.com/macaw-interactive/react-jss-typescript-starter/blob/develop/samples/react-typescript/README.md)

Same approach can be used in Angular and Vue.js, it is just the exact implementation might differ. 

This approach is much more flexible, however you would have to spend some time implementing this in frontend framework of your choice. I really hope Sitecore will update their sample JSS apps to use this approach in future 🙂