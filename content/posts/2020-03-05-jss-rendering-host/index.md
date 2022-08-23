---
title: 'JSS rendering host: what? why? how?'
date: 2020-03-05T12:32:14+00:00
summary: Sitecore JSS rendering host is an awesome but also an underused feature. So I decided to promote it a bit and describe some use cases from our project.
description: Sitecore JSS rendering host use cases, technical details and instructions on how to set it up
url: /jss-rendering-host/
cover:
  image: "rendering-host-ngrok.png"
  relative: true
categories:
  - Sitecore
tags:
  - JSS

---
![Sitecore JSS rendering host](jss-logo.png#center "Sitecore JSS rendering host")

Sitecore JSS rendering host is an awesome but also an underused feature. So I decided to promote it a bit and describe some use cases from our project.

On the moment of writing, there is not much documentation available online for JSS rendering host. Actually, there is none, except for a mention on the [release notes page for 9.3](https://jss.sitecore.com/release-notes#new-features--improvements "release notes page for 9.3"), even though server side support for it has been included into 11.1.0 release of JSS (for Sitecore 9.1.1).

There is, however, the following Github gist by Adam Weber with a lot of technical details: https://gist.github.com/aweber1/39d1e9e07fb19aac9bc9b94feb87bb0f

Also JSS rendering host [had been showcased on SUGCON Europe 2019](https://youtu.be/5-nu7vIdIww?t=1492 "had been showcased on SUGCON Europe 2019") by Kam Figy and Adam Weber.

## What is JSS rendering host?

![](https://i0.wp.com/media.giphy.com/media/l3q2K5jinAlChoCLS/source.gif?w=1100&#038;ssl=1#center)

Sitecore JSS has a concept of a _rendering engine_, which is used to render your app server-side in [Integrated Mode](https://doc.sitecore.com/xp/en/developers/hd/200/sitecore-headless-development/start-a-jss-app-in-integrated-mode.html "Integrated Mode") (this mode is always used when editing a site in Experience Editor). First releases of JSS included only one rendering engine: **nodejs**. It uses a Node.JS instance running on the same server as your Sitecore instance to render your app. 

JSS 11.1.0 release (compatible with Sitecore 9.1.1) includes one more rendering engine: **http**. This engine allows to render your app via HTTP call to a remote Node.JS instance (**Rendering Host**). 

So, **Rendering Host** is an process/application responsible for **SSR**-ing of your frontend app.

![Sitecore rendering host and rendering engine data flow](rendering-host.png#center "Sitecore rendering host and rendering engine data flow")

Sitecore provides a sample frontend implementation of a Rendering Host. It is simply a piece of Node.js middlware which listens on POST requests and renders your app with the provided data (JSON payload containing information needed for rendering): https://github.com/Sitecore/jss/blob/dev/packages/sitecore-jss-rendering-host/src/ssrMiddleware.ts

## What are the use cases?

Obviously, the concept of JSS Rendering Host is only useful for Experience Editor SSR. 

One of the reasons Sitecore JSS is so cool is that it not only helps to independently develop frontend and backend, but also deploy them separately (see [Headless topology](https://doc.sitecore.com/xp/en/developers/hd/200/sitecore-headless-development/jss-topologies.html#headless-topology "Headless topology"). However, **if you want to support Experience Editor (and you probably do) then you have to deploy your FE app distribution to your CM instance** (_if you don&#8217;t use Rendering Host_). Normally it is not a problem, however, here are some use cases when you don&#8217;t want that.

### Use case 1: independent deployment pipelines

![DevOps](devops.png#center)

If your frontend and backend is developed independently, then you probably also have independent deployment pipelines. Since Sitecore deployment needs FE app distribution to be available, it causes a dependency of Sitecore deployment pipeline on results of frontend pipeline, which is not good and considered as an anti-pattern. What if you need to deploy Sitecore earlier than frontend (which is normally the case)?

This is where JSS Rendering Host shines: it moves responsibility of deploying Experience Editor app distribution to frontend deployment pipeline. Deploying a new version of frontend code does not require any changes to Sitecore, since it is relying on a HTTP endpoint provided by your Rendering Host to SSR your app.

### Use case 2: testing Experience Editor integration

![Sitecore Experience Editor](experience-editor.png#center "Sitecore Experience Editor")

You frontend developers (hopefully) want to test how their code works in Experience Editor. How can they do it?

  * Have their own instance of Sitecore running on their laptops? While this is certainly possible, it requires your frontenders to have more Sitecore knowledge. They will also have to occasionally maintain the Sitecore instance: sync item and code changes, etc. Also most of those fancy frontenders use MacBooks, which makes it tricky to run Sitecore.
  * Having a shared Sitecore instance available for all of them. However, how would they deploy their code to this instance? Without overwriting each other&#8217;s changes?

Wouldn&#8217;t it be awesome to have something in the middle? Having a shared Sitecore instance which is able to render an app running on a frontend developer&#8217;s laptop remotely? Here comes the JSS Rendering Host.

In Experience Editor you can specify a rendering host URL via a query string parameter: **sc_httprenderengineurl**. So each frontend developer can point this URL to an app running on their laptops. Great! But how can the Sitecore instance reach out to a developer laptop?

One way to do it is to use a reverse tunneling solution, like [Ngrok](https://ngrok.com/), which can make your localhost available to outside world via a public URL, e.g _https://xxx.ngrok.io_ where _xxx_ part is dynamically generated.

In our case you would need to run a tunnel exposing your local JSS Rendering Host and set _sc_httprenderengineurl_= _https://xxx.ngrok.io_ query string parameter in Experience Editor.

![Remote JSS Rendering Host exposed via reverse tunnel](rendering-host-ngrok.png#center "Remote JSS Rendering Host exposed via reverse tunnel")

## How to set up JSS rendering host?

> For the section below I assume that you know how to work with JSS CLI and modify JSS app configuration. If not &#8211; first have a look at https://jss.sitecore.com/docs/getting-started/quick-start

Sitecore provides a [npm package](https://www.npmjs.com/package/@sitecore-jss/sitecore-jss-rendering-host "npm package") with a rendering host implementation. The [JSS Github repository](https://github.com/Sitecore/jss "JSS Github repository") contains an example of how to use the package (at the time of writing for React implementation only). Let's bootstrap a sample React app using JSS CLI: `jss create rendering-host react`

If you inspect the `package.json` file in the resulting output you will notice the following commands: 

  * `build:rendering-host` &#8211; this one does the same as `build` command, but calls `build:client:rendering-host` instead of `build:client`
  * `build:client:rendering-host` &#8211; almost same as `build:client` with the exception that it uses `npm_package_config_tunnelUrl` as a URL for static assets. Since with rendering host setup your static assets do not live on the same server as your Sitecore ContentManagement instance, you have to use URL of your rendering host instead when loading them from Experience Editor.
  * `start:rendering-host` &#8211; this command runs `scripts/http-renderer.js` which starts a rendering host server on _localhost:5000_ and spawns an Ngrok tunnel to make it publicly available.

> **Important:** at the time of writing the React sample provided by Sitecore has some issues which you have to fix before running the rendering host:
>
> 1. Add a missing <code>"@sitecore-jss/sitecore-jss-rendering-host": "^13.0.2"</code> dependency to <em>packages.json</em> and run <code>npm i</code>
> 
> 2. Replace path to the build folder in <em>/scripts/http-renderer.js</em> from <em>/build-rendering-host/</em> to <em>/build/</em>
> 
> I have created a [Github issue](https://github.com/Sitecore/jss/issues/339 "Github issue") to address this.

To run the sample, you need to run `build:rendering-host` followed by `start:rendering-host`. You will get a similar message in the console:

> Tunnel started, forwarding &#8216;https://xxxxxx.ngrok.io&#8217; to &#8216;localhost:5000&#8217;<br />Starting rendering host at localhost:5000

Now that rendering host is running you can set it up to be used by Experience Editor. To do that, you have to update configuration of your JSS app (in the _/sitecore/javaScriptServices/apps_ section) by adding the following attributes: 

  * `serverSideRenderingEngine="http"`
  * `serverSideRenderingEngineEndpointUrl="http://localhost:5000/"` &#8211; **Note:** as I&#8217;ve mentioned before, you can also set/override this URL dynamically in Experience Editor by using the **sc_httprenderengineurl** query string parameter.

In context of JSS React sample app, you can make the changes in the _/sitecore/config/JssReactWeb.config_ file and then deploy the configuration to your Sitecore instance (using the `jss deploy config` command).

Now when you open Experience Editor, your app should be rendered by the rendering host. To confirm this you should be able to see messages similar to the following in your console (the one which is running rendering host):

> [SSR] rendering app at C:\Git-Projects\rendering-host\app\build\server.bundle via render function named renderView

**Note:** you might have problems with displaying images and other static assets. In this case make sure that the `tunnelUrl` setting in `package.json` is pointing to your JSS rendering host URL (e.g http://localhost:5000).