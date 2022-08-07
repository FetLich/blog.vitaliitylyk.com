---
title: 'JSS rendering host: what? why? how?'
author: Vitalii Tylyk
type: post
date: 2020-03-05T12:32:14+00:00
excerpt: Sitecore JSS rendering host is an awesome but also an underused feature. So I decided to promote it a bit and describe some use cases from our project.
url: /jss-rendering-host/
featured_image: /wp-content/uploads/2020/02/rendering-host-ngrok.png
categories:
  - Sitecore
tags:
  - JSS

---
<div class="wp-block-image">
  <figure class="aligncenter size-large"><img loading="lazy" width="300" height="160" src="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/03/jss-logo.png?resize=300%2C160&#038;ssl=1" alt="Sitecore JSS rendering host" class="wp-image-610" data-recalc-dims="1" /></figure>
</div>

Sitecore JSS rendering host is an awesome but also an underused feature. So I decided to promote it a bit and describe some use cases from our project.

On the moment of writing, there is not much documentation available online for JSS rendering host. Actually, there is none, except for a mention on the <a rel="noreferrer noopener" aria-label="release notes page for 9.3 (opens in a new tab)" href="https://jss.sitecore.com/release-notes#new-features--improvements" target="_blank">release notes page for 9.3</a>, even though server side support for it has been included into 11.1.0 release of JSS (for Sitecore 9.1.1).

There is, however, the following Github gist by Adam Weber with a lot of technical details: <a rel="noreferrer noopener" aria-label=" (opens in a new tab)" href="https://gist.github.com/aweber1/39d1e9e07fb19aac9bc9b94feb87bb0f" target="_blank">https://gist.github.com/aweber1/39d1e9e07fb19aac9bc9b94feb87bb0f</a> 

Also JSS rendering host <a rel="noreferrer noopener" aria-label="had been showcased on SUGCON Europe 2019 (opens in a new tab)" href="https://youtu.be/5-nu7vIdIww?t=1492" target="_blank">had been showcased on SUGCON Europe 2019</a> by Kam Figy and Adam Weber.

## What is JSS rendering host?

<div class="wp-block-image">
  <figure class="aligncenter size-large"><img src="https://i0.wp.com/media.giphy.com/media/l3q2K5jinAlChoCLS/source.gif?w=1100&#038;ssl=1" alt="" data-recalc-dims="1" /></figure>
</div>

Sitecore JSS has a concept of a _rendering engine_, which is used to render your app server-side in <a rel="noreferrer noopener" aria-label="Integrated Mode (opens in a new tab)" href="https://jss.sitecore.com/docs/fundamentals/application-modes#integrated-mode" target="_blank">Integrated Mode</a> (this mode is always used when editing a site in Experience Editor). First releases of JSS included only one rendering engine: **nodejs**. It uses a Node.JS instance running on the same server as your Sitecore instance to render your app. 

JSS 11.1.0 release (compatible with Sitecore 9.1.1) includes one more rendering engine: **http**. This engine allows to render your app via HTTP call to a remote Node.JS instance (**Rendering Host**). 

So, **Rendering Host** is an process/application responsible for **SSR**-ing of your frontend app.

<div class="wp-block-image">
  <figure class="aligncenter size-large"><img loading="lazy" width="452" height="124" src="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/02/rendering-host.png?resize=452%2C124&#038;ssl=1" alt="" class="wp-image-1037" srcset="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/02/rendering-host.png?w=452&ssl=1 452w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/02/rendering-host.png?resize=300%2C82&ssl=1 300w" sizes="(max-width: 452px) 100vw, 452px" data-recalc-dims="1" /></figure>
</div>

Sitecore provides a sample frontend implementation of a Rendering Host. It is simply a piece of Node.js middlware which listens on POST requests and renders your app with the provided data (JSON payload containing information needed for rendering): <a rel="noreferrer noopener" aria-label=" (opens in a new tab)" href="https://github.com/Sitecore/jss/blob/dev/packages/sitecore-jss-rendering-host/src/ssrMiddleware.ts" target="_blank">https://github.com/Sitecore/jss/blob/dev/packages/sitecore-jss-rendering-host/src/ssrMiddleware.ts</a> 

## What are the use cases?

Obviously, the concept of JSS Rendering Host is only useful for Experience Editor SSR. 

One of the reasons Sitecore JSS is so cool is that it not only helps to independently develop frontend and backend, but also deploy them separately (see <a rel="noreferrer noopener" aria-label="Headless Topology (opens in a new tab)" href="https://jss.sitecore.com/docs/techniques/devops#headless-topology" target="_blank">Headless Topology</a>). However, **if you want to support Experience Editor (and you probably do) then you have to deploy your FE app distribution to your CM instance** (_if you don&#8217;t use Rendering Host_). Normally it is not a problem, however, here are some use cases when you don&#8217;t want that.

### Use case 1: independent deployment pipelines

<div class="wp-block-image">
  <figure class="aligncenter size-large is-resized"><img loading="lazy" src="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/06/devops.png?resize=256%2C145&#038;ssl=1" alt="" class="wp-image-726" width="256" height="145" srcset="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/06/devops.png?resize=1024%2C580&ssl=1 1024w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/06/devops.png?resize=300%2C170&ssl=1 300w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/06/devops.png?resize=768%2C435&ssl=1 768w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/06/devops.png?resize=740%2C419&ssl=1 740w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/06/devops.png?resize=1100%2C623&ssl=1 1100w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/06/devops.png?w=1280&ssl=1 1280w" sizes="(max-width: 256px) 100vw, 256px" data-recalc-dims="1" /></figure>
</div>

If your frontend and backend is developed independently, then you probably also have independent deployment pipelines. Since Sitecore deployment needs FE app distribution to be available, it causes a dependency of Sitecore deployment pipeline on results of frontend pipeline, which is not good and considered as an anti-pattern. What if you need to deploy Sitecore earlier than frontend (which is normally the case)?

This is where JSS Rendering Host shines: it moves responsibility of deploying Experience Editor app distribution to frontend deployment pipeline. Deploying a new version of frontend code does not require any changes to Sitecore, since it is relying on a HTTP endpoint provided by your Rendering Host to SSR your app.

### Use case 2: testing Experience Editor integration

<div class="wp-block-image">
  <figure class="aligncenter size-large"><img loading="lazy" width="110" height="150" src="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/02/experience-editor.png?resize=110%2C150&#038;ssl=1" alt="Sitecore Experience Editor" class="wp-image-1062" data-recalc-dims="1" /></figure>
</div>

You frontend developers (hopefully) want to test how their code works in Experience Editor. How can they do it?

  * Have their own instance of Sitecore running on their laptops? While this is certainly possible, it requires your frontenders to have more Sitecore knowledge. They will also have to occasionally maintain the Sitecore instance: sync item and code changes, etc. Also most of those fancy frontenders use MacBooks, which makes it tricky to run Sitecore.
  * Having a shared Sitecore instance available for all of them. However, how would they deploy their code to this instance? Without overwriting each other&#8217;s changes?

Wouldn&#8217;t it be awesome to have something in the middle? Having a shared Sitecore instance which is able to render an app running on a frontend developer&#8217;s laptop remotely? Here comes the JSS Rendering Host.

In Experience Editor you can specify a rendering host URL via a query string parameter: **sc_httprenderengineurl**. So each frontend developer can point this URL to an app running on their laptops. Great! But how can the Sitecore instance reach out to a developer laptop?

One way to do it is to use a reverse tunneling solution, like <a rel="noreferrer noopener" aria-label="Ngrok (opens in a new tab)" href="https://ngrok.com/" target="_blank">Ngrok</a>, which can make your localhost available to outside world via a public URL, e.g _https://xxx.ngrok.io_ where _xxx_ part is dynamically generated.

In our case you would need to run a tunnel exposing your local JSS Rendering Host and set _sc_httprenderengineurl_= _https://xxx.ngrok.io_ query string parameter in Experience Editor.

<div class="wp-block-image">
  <figure class="aligncenter size-large"><img loading="lazy" width="700" height="358" src="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/02/rendering-host-ngrok.png?resize=700%2C358&#038;ssl=1" alt="Remote JSS Rendering Host exposed via reverse tunnel" class="wp-image-1058" srcset="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/02/rendering-host-ngrok.png?w=700&ssl=1 700w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/02/rendering-host-ngrok.png?resize=300%2C153&ssl=1 300w" sizes="(max-width: 700px) 100vw, 700px" data-recalc-dims="1" /><figcaption>Remote JSS Rendering Host exposed via reverse tunnel</figcaption></figure>
</div>

## How to set up JSS rendering host?

<blockquote class="wp-block-quote">
  <p>
    For the section below I assume that you know how to work with JSS CLI and modify JSS app configuration. If not &#8211; first have a look at <a rel="noreferrer noopener" aria-label=" (opens in a new tab)" href="https://jss.sitecore.com/docs/getting-started/quick-start" target="_blank">https://jss.sitecore.com/docs/getting-started/quick-start</a>
  </p>
</blockquote>

Sitecore provides a <a rel="noreferrer noopener" aria-label="npm package (opens in a new tab)" href="https://www.npmjs.com/package/@sitecore-jss/sitecore-jss-rendering-host" target="_blank">npm package</a> with a rendering host implementation. <a href="https://github.com/Sitecore/jss" target="_blank" rel="noreferrer noopener" aria-label="JSS Github repository (opens in a new tab)">JSS Github repository</a> contains an example of how to use the package (at the time of writing for React implementation only). Let&#8217;s bootstrap a sample React app using JSS CLI:

<pre class="EnlighterJSRAW" data-enlighter-language="shell" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">jss create rendering-host react</pre>

If you inspect the `package.json` file in the resulting output you will notice the following commands: 

  * `build:rendering-host` &#8211; this one does the same as `build` command, but calls `build:client:rendering-host` instead of `build:client`
  * `build:client:rendering-host` &#8211; almost same as `build:client` with the exception that it uses `npm_package_config_tunnelUrl` as a URL for static assets. Since with rendering host setup your static assets do not live on the same server as your Sitecore ContentManagement instance, you have to use URL of your rendering host instead when loading them from Experience Editor.
  * `start:rendering-host` &#8211; this command runs `scripts/http-renderer.js` which starts a rendering host server on _localhost:5000_ and spawns an Ngrok tunnel to make it publicly available.

<blockquote class="wp-block-quote">
  <p>
    <strong>Important:</strong> at the time of writing the React sample provided by Sitecore has some issues which you have to fix before running the rendering host:
  </p>
  
  <p>
    1. Add a missing <code>"@sitecore-jss/sitecore-jss-rendering-host": "^13.0.2"</code> dependency to <em>packages.json</em> and run <code>npm i</code>
  </p>
  
  <p>
    2. Replace path to the build folder in <em>/scripts/http-renderer.js</em> from <em>/build-rendering-host/</em> to <em>/build/</em>
  </p>
  
  <p>
    I have created a <a href="https://github.com/Sitecore/jss/issues/339" target="_blank" rel="noreferrer noopener" aria-label="Github issue (opens in a new tab)">Github issue</a> to address this.
  </p>
</blockquote>

To run the sample, you need to run `build:rendering-host` followed by `start:rendering-host`. You will get a similar message in the console:

<blockquote class="wp-block-quote">
  <p>
    Tunnel started, forwarding &#8216;https://xxxxxx.ngrok.io&#8217; to &#8216;localhost:5000&#8217;<br />Starting rendering host at localhost:5000
  </p>
</blockquote>

Now that rendering host is running you can set it up to be used by Experience Editor. To do that, you have to update configuration of your JSS app (in the _/sitecore/javaScriptServices/apps_ section) by adding the following attributes: 

  * `serverSideRenderingEngine="http"`
  * `serverSideRenderingEngineEndpointUrl="http://localhost:5000/"` &#8211; **Note:** as I&#8217;ve mentioned before, you can also set/override this URL dynamically in Experience Editor by using the **sc_httprenderengineurl** query string parameter.

In context of JSS React sample app, you can make the changes in the _/sitecore/config/JssReactWeb.config_ file and then deploy the configuration to your Sitecore instance (using the `jss deploy config` command).

Now when you open Experience Editor, your app should be rendered by the rendering host. To confirm this you should be able to see messages similar to the following in your console (the one which is running rendering host):

<blockquote class="wp-block-quote">
  <p>
    [SSR] rendering app at C:\Git-Projects\rendering-host\app\build\server.bundle via render function named renderView
  </p>
</blockquote>

<blockquote class="wp-block-quote">
  <p>
    <strong>Note:</strong> you might have problems with displaying images and other static assets. In this case make sure that the <code>tunnelUrl</code> setting in <em>package.json</em> is pointing to your JSS rendering host URL (e.g http://localhost:5000).
  </p>
</blockquote>