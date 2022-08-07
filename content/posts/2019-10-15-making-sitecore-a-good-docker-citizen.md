---
title: Making Sitecore a good Docker citizen
author: Vitalii Tylyk
type: post
date: 2019-10-15T08:32:22+00:00
excerpt: Lately more and more people get interested in running Sitecore in Docker containers. So how do you make Sitecore a good Docker citizen?
url: /making-sitecore-a-good-docker-citizen/
featured_image: /wp-content/uploads/2019/10/docker-sitecore-405x430.png
categories:
  - DevOps
  - Sitecore
tags:
  - DevOps
  - Docker
  - Sitecore

---
<div class="wp-block-image">
  <figure class="aligncenter is-resized"><img loading="lazy" src="https://i1.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/10/docker-sitecore.png?resize=300%2C319&#038;ssl=1" alt="Sitecore and Docker" class="wp-image-903" width="300" height="319" srcset="https://i1.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/10/docker-sitecore.png?w=600&ssl=1 600w, https://i1.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/10/docker-sitecore.png?resize=283%2C300&ssl=1 283w, https://i1.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/10/docker-sitecore.png?resize=405%2C430&ssl=1 405w" sizes="(max-width: 300px) 100vw, 300px" data-recalc-dims="1" /></figure>
</div>

Lately more and more people get interested in running Sitecore in Docker containers. There have been active developments from community which evolved into this awesome Github repository with Sitecore Docker images <a rel="noreferrer noopener" aria-label=" (opens in a new tab)" href="https://github.com/Sitecore/docker-images" target="_blank">https://github.com/Sitecore/docker-images</a>. Even though these example images are not officially supported by Sitecore, since recently Sitecore started supporting running Sitecore within a containerized environment: <a href="https://sitecore.stackexchange.com/questions/22187/does-sitecore-have-support-for-sitecore-products-in-containers" target="_blank" rel="noreferrer noopener" aria-label=" (opens in a new tab)">https://sitecore.stackexchange.com/questions/22187/does-sitecore-have-support-for-sitecore-products-in-containers</a>.

<blockquote class="wp-block-quote">
  <p>
    <strong>Note: </strong>if you don&#8217;t have any experience with Sitecore on Docker and don&#8217;t know where to start, you can check out this series &#8220;Sitecore Docker for Dummies&#8221; by Mark Cassidy: <a href="https://intothecloud.blog/2019/09/14/Sitecore-Docker-for-Dummies/" target="_blank" rel="noreferrer noopener" aria-label=" (opens in a new tab)">https://intothecloud.blog/2019/09/14/Sitecore-Docker-for-Dummies/</a>
  </p>
  
  <p>
    Regarding Docker itself I can highly recommend this book by Elton Stoneman: &#8220;<em>Docker on Windows: From 101 to production with Docker on Windows</em>&#8220;
  </p>
</blockquote>

These community examples provide a tremendous boost when you start working with Sitecore in containers. You basically get a very solid base which you can take and apply on your projects. It is cool to see how solid Sitecore community has become over time and that there are more and more open-source initiatives in Sitecore world. 

However, as you get more experienced with Docker you will notice that some parts of the puzzle are missing and for production-ready setup you need to do some extra work. Let&#8217;s see what is missing.

## What makes a good Docker citizen?

As defined by Elton Stoneman (<a href="https://blog.sixeyed.com/" target="_blank" rel="noreferrer noopener" aria-label=" (opens in a new tab)">https://blog.sixeyed.com/</a>), an architect at Docker:

<blockquote class="wp-block-quote">
  <p>
    A&nbsp;good citizen&nbsp;Docker container runs a single process, uses environment variables for configuration and writes output to the console.
  </p>
  
  <cite> Elton Stoneman </cite>
</blockquote>

### 1. Single running process

Docker containers should have a single responsibility and run single process. You should not, for example, run website and database in the same container due to maintainability, scalability, security and performance reasons.

In <a href="https://github.com/Sitecore/docker-images" target="_blank" rel="noreferrer noopener" aria-label=" (opens in a new tab)">https://github.com/Sitecore/docker-images</a> all containers are running single processes, so we can cross out this point.

### 2. Using environment variables for configuration

Sooner or later you will want to deploy your Sitecore Docker images to some other environments than your laptop. How would you manage configuration differences (connection strings, etc) across environments?

In traditional, &#8220;pre-Docker&#8221; world you would setup a deployment tool of your choice (Octopus/Azure DevOps/etc) to physically substitute the appropriate configuration settings in config files per environment.

With Docker the story is different, since Docker images are meant to be immutable. Images should not be modified after they are built. This is why it is considered **the best practice to provide container with configuration is via environment variables**. 

Sitecore uses 2 types of configuration: native .NET/ASP.NET configuration (Web.config, AppSettings, ConnectionStrings) and custom Sitecore configuration (`<sitecore>` config section).

In the custom Sitecore configuration, you can use `$(env:VARIABLE_NAME)` syntax to refer to environment variables, for example:

<pre class="EnlighterJSRAW" data-enlighter-language="generic" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">&lt;setting name="MyFeature.MySetting">
    $(env:MyFeature.MySetting)
  &lt;/setting></pre>

<blockquote class="wp-block-quote">
  <p>
    Alternatively, you can use the following solution to manage Sitecore configuration settings, sc.variables and site settings via environment variables: <a href="https://blog.vitaliitylyk.com/sitecore-environment-variables-config-builder/">https://blog.vitaliitylyk.com/sitecore-environment-variables-config-builder/</a>
  </p>
</blockquote>

It is getting more complex when it comes to .NET configuration and you want to be able to retrieve your connection strings from environment variables. With ASP.NET Core this would have been easy: there is a built-in support for reading configuration from environment variables without making any changes to your application code. One just needs to setup an appropriate <a rel="noreferrer noopener" aria-label=" (opens in a new tab)" href="https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-3.0" target="_blank">configuration provider</a>.

Unfortunately, Sitecore (CMS part) is still running on ASP.NET and is using `ConfigurationManager` to read AppSettings and ConnectionStrings, which get configuration from `Web.config`.

<blockquote class="wp-block-quote">
  <p>
    <strong>Note:</strong> having said that, some of the Sitecore microservices (e.g. PublishingService) are built on top of ASP.NET Core and natively support reading configuration from environment variables.
  </p>
</blockquote>

Luckily, there are several ways to make this work for traditional ASP.NET applications without any changes to your application code:

#### 1. Bootstrap script

You can implement a script which would run when container starts up (via `ENTRYPOINT` instruction) and:

  * Reads all environment variables available to container
  * Updates config files with the appropriate variables

This is certainly a valid approach, however let&#8217;s see if there are better alternatives.

#### 2. Environment variables support with .NET 4.7.1 ConfigurationBuilders

Luckily, Microsoft has came up with <a rel="noreferrer noopener" aria-label="ConfigurationBuilders (opens in a new tab)" href="https://docs.microsoft.com/en-us/aspnet/config-builder" target="_blank">ConfigurationBuilders</a>. Starting from .NET 4.7.1 you can use ConfigurationBuilders to &#8220;teach&#8221; your application to read configuration from places other than Web.config file, including support for Environment Variables and Azure KeyVault.

<blockquote class="wp-block-quote">
  <p>
    <strong>Update (31.01.2020):</strong> Sitecore 9.3 is a better Docker citizen as it comes with ConfigurationBuilders enabled out of the box, more information here: <a href="https://blog.vitaliitylyk.com/sitecore-9-3-loves-docker/" target="_blank" rel="noreferrer noopener" aria-label="Sitecore 9.3 loves Docker (opens in a new tab)">Sitecore 9.3 loves Docker</a>
  </p>
</blockquote>

To enable this you only need to install the `Microsoft.Configuration.ConfigurationBuilders.Environment` Nuget package and update your Web.config depending on your needs. Below you can find and example of how to setup configuration builders for `connectionStrings` and `appSettings` Web.config sections:

**Add a config section**

<pre class="EnlighterJSRAW" data-enlighter-language="generic" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">&lt;configSections>
  ...
  &lt;section name="configBuilders" type="System.Configuration.ConfigurationBuildersSection, System.Configuration, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" restartOnExternalChanges="false" requirePermission="false" />
&lt;/configSections></pre>

**Define configuration builders**

Pay attention to the `prefix` attribute. This means that only environment variables starting with _ConnectionString__ and _AppSetting__ will be injected.

<pre class="EnlighterJSRAW" data-enlighter-language="generic" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">&lt;configBuilders>
    &lt;builders>
      &lt;add name="Environment_AppSettings" mode="Greedy" prefix="AppSetting_" stripPrefix="true" type="Microsoft.Configuration.ConfigurationBuilders.EnvironmentConfigBuilder, Microsoft.Configuration.ConfigurationBuilders.Environment, Version=1.0.0.0, Culture=neutral" />
      &lt;add name="Environment_ConnectionStrings" mode="Greedy" prefix="ConnectionString_" stripPrefix="true" type="Microsoft.Configuration.ConfigurationBuilders.EnvironmentConfigBuilder, Microsoft.Configuration.ConfigurationBuilders.Environment, Version=1.0.0.0, Culture=neutral" />
    &lt;/builders>
  &lt;/configBuilders></pre>

**Update `connectionStrings` and `appSettings` sections to enable config builders**

<pre class="EnlighterJSRAW" data-enlighter-language="generic" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">&lt;appSettings configBuilders="Environment_AppSettings">
 ...
&lt;/appSettings>

&lt;connectionStrings configBuilders="Environment_ConnectionStrings">
 ...
&lt;/connectionStrings></pre>

**Use environment variables**

That&#8217;s it! Now you can provide your Docker containers with environment variables and Sitecore will pick them up, e.g:

<pre class="EnlighterJSRAW" data-enlighter-language="generic" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">ConnectionString_solr.search = http://solr:8983/solr
AppSetting_role:define = ContentDelivery</pre>

### 3. Logging to Docker console

Writing your application logs to Docker console has many benefits:

  * Better developer experience. Viewing logs of an application running in container is as simple as opening a Docker console.
  * In scaled environments it is common to use centralized logging solutions. You can take advantage of <a href="https://docs.docker.com/config/containers/logging/configure/" target="_blank" rel="noreferrer noopener" aria-label="logging drivers (opens in a new tab)">logging drivers</a> to be able to write logs to e.g. Splunk. Also some monitoring tools (e.g. Datadog) listen to Docker console output to get application logs.

Unfortunately, ASP.NET applications running under IIS are not able to write logs directly to Stdout (this issue is <a href="https://github.com/Microsoft/IIS.ServiceMonitor/issues/1#issuecomment-401513710" target="_blank" rel="noreferrer noopener" aria-label="mentioned on Github (opens in a new tab)">mentioned on Github</a> and hopefully will be addressed by Microsoft). So you cannot simply use a log4net Console appender to send logs to Docker console 🙁

Probably, there are better ways to solve this, but the simplest one is to relay log entries from your file system to Docker console by running a Powershell script on container startup.

<blockquote class="wp-block-quote">
  <p>
    <strong>Update (16.11.2019):</strong> as of November 2019 a more robust and flexible solution has been implemented in the Sitecore docker-images Github repository. It is based on Elastic <a rel="noreferrer noopener" aria-label="Filebeat (opens in a new tab)" href="https://www.elastic.co/products/beats/filebeat" target="_blank">Filebeat</a>, which allows to aggregate several log sources, including file system logs and UDP. Sitecore is configured to send logs to UDP 127.0.0.1:7777 endpoint, which is set up to be monitored by Filebeat. More details here: <a rel="noreferrer noopener" aria-label=" (opens in a new tab)" href="https://github.com/Sitecore/docker-images/#optional-entrypoint-scripts" target="_blank">https://github.com/Sitecore/docker-images/#optional-entrypoint-scripts</a>
  </p>
</blockquote>

It is not that straightforward though because by default Sitecore rolls log files when they reach a certain size (10MB by default). Also each time Sitecore restarts, it creates a new _log.{date}.{time}.txt_ file. So the script should be able to handle this rolling behavior and be able to detect when new log files are created to start tailing them. 

<blockquote class="wp-block-quote">
  <p>
    <strong>Note:</strong> you might think why not configure Sitecore to <a href="https://www.seanholmesby.com/real-time-auto-reload-logging-with-sitecores-rolling-file-appender/" target="_blank" rel="noreferrer noopener" aria-label="use single file log4net appender (opens in a new tab)">use single file log4net appender</a> instead of the rolling one?
  </p>
  
  <p>
    On the first glance this would work, but unfortunately this brings file locking issues when Sitecore application pool restarts (new application pool is created, while the old one is still locking the file). This behavior is caused by the &#8220;Overlapped Recycle&#8221; IIS feature. More on file locking issues and log4net here: <a rel="noreferrer noopener" aria-label=" (opens in a new tab)" href="https://hectorcorrea.com/blog/log4net-thread-safe-but-not-process-safe/17" target="_blank">https://hectorcorrea.com/blog/log4net-thread-safe-but-not-process-safe/17</a>
  </p>
</blockquote>

I have implemented an example of such a script, which you can find below:<figure class="wp-block-embed">

<div class="wp-block-embed__wrapper">
  <div class="gist-oembed" data-gist="ce10fdce3dcd3aec903a89ee11909599.json" data-ts="8">
  </div>
</div></figure> 

To handle the rolling behavior, the script needs a way to detect when Sitecore has finished writing to a file. This is achieved by searching for the _end file marker_ (`------END------` in this example), which log4net will append before file handle is released. It does not do it by default, so you have to patch Sitecore log4net configuration in the following way:

<pre class="EnlighterJSRAW" data-enlighter-language="generic" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">&lt;?xml version="1.0"?>
&lt;configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
  &lt;sitecore>
    &lt;log4net>
      &lt;appender name="LogFileAppender" type="log4net.Appender.RollingFileAppender, Sitecore.Logging">
        &lt;layout type="log4net.Layout.PatternLayout">
          &lt;footer value="------END------"/>
        &lt;/layout>
      &lt;/appender>
    &lt;/log4net>
  &lt;/sitecore>
&lt;/configuration></pre>

Notice the `<footer />` element, which configures the _end file marker_.

## Summary

It is nice to see Docker being adopted in Sitecore world: there are more and more presentations on the topic in user groups and #docker channel in Sitecore Slack is also getting hot. With this amount of interest I believe that sooner or later Sitecore will release their officially supported Docker images (probably later, I expect this to happen after .NET Core migration which will take a while). But meanwhile we, developers, have to explore this ourselves, which makes our work more exciting, doesn&#8217;t it? 🙂

Happy Dockering!