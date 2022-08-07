---
title: Sitecore environment variables config builder
author: Vitalii Tylyk
type: post
date: 2020-02-06T11:57:04+00:00
excerpt: "Inspired by ASP.NET configuration builders I thought: wouldn't it be awesome to be able to modify Sitecore configuration via environment variables in a similar way? This is especially relevant in Docker world."
url: /sitecore-environment-variables-config-builder/
categories:
  - DevOps
  - Sitecore
tags:
  - DevOps
  - Docker
  - Sitecore

---
Inspired by ASP.NET <a rel="noreferrer noopener" aria-label="configuration builders (opens in a new tab)" href="https://docs.microsoft.com/en-us/aspnet/config-builder" target="_blank">configuration builders</a> I thought: wouldn&#8217;t it be awesome to be able to modify Sitecore configuration via environment variables in a similar way? This is especially relevant in Docker world.

As <a rel="noreferrer noopener" aria-label="I wrote before, to be a good Docker citizen (opens in a new tab)" href="https://blog.vitaliitylyk.com/making-sitecore-a-good-docker-citizen/" target="_blank">I&#8217;ve mentioned before, to be a good Docker citizen</a>, a containerized application has to support configuration via environment variables.

Out of the box, Sitecore supports `$(env:ENVIRONMENT_VARIABLE_NAME)` syntax to use environment variables in your configuration. However, this has 2 limitations:

  * This syntax is not supported for Sitecore variables (`sc.variable`)
  * Before being able to actually use environment variables, you first have to modify all places where you want to use them with `$(env:ENVIRONMENT_VARIABLE_NAME)` syntax. 

My goal was to be able to modify Sitecore configuration via environment variables without making any changes to Sitecore config files themselves. In particular:

  * Update settings
  * Update variables
  * Update site settings (e.g set hostName, scheme, etc)

This syntax would look something like this (on the left is an environment variable name):

<pre class="EnlighterJSRAW" data-enlighter-language="generic" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">SITECORE_SETTINGS_&lt;settingname>=&lt;value>
SITECORE_SITES_&lt;sitename>_&lt;attributename>=&lt;value>
SITECORE_VARIABLES_&lt;variablename>=&lt;value></pre>

For example:

<pre class="EnlighterJSRAW" data-enlighter-language="generic" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">SITECORE_SETTINGS_MEDIA.MEDIALINKSERVERURL=https://www.hostname.com
SITECORE_SITES_WEBSITE_HOSTNAME=hostname.com</pre>

At first I considered to follow Microsoft approach and develop a custom configuration builder. However, ASP.NET configuration builders intrude **before** Sitecore does it&#8217;s magic with aggregating all include files. Therefore, this approach will not allow to inject environment variables into include files, which is very limiting.

So I ended up with customizing the Sitecore configuration section handler. By default, Sitecore config section is defined in Web.config in the following way:

<pre class="EnlighterJSRAW" data-enlighter-language="generic" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">&lt;configSections>
    ...
    &lt;section name="sitecore" type="Sitecore.Configuration.RuleBasedConfigReader, Sitecore.Kernel" />
    ...
  &lt;/configSections></pre>

This can be easily customized via pointing the &#8220;sitecore&#8221; section handler to a custom class, which inherits from `RuleBasedConfigReader`. Below you can find an example of a custom handler which loops over environment variables and injects them into Sitecore configuration, if it matches the appropriate `setting`/`sc.variable` or `site setting` (the comparison is case-insensitive). In case of a match the `patch:sourceEnvironmentVariables` attribute is appended to an appropriate XML node to be able to easily track changes in Sitecore&#8217;s `showconfig.aspx`.<figure class="wp-block-embed" style="max-height: 800px; overflow-y: scroll;">

<div class="wp-block-embed__wrapper">
  <div class="gist-oembed" data-gist="ac419bc30993e3a6dfe7ed88767eb2ce.json" data-ts="8">
  </div>
</div></figure> 

And this is an output from `showconfig.aspx` for the `Media.MediaLinkServerUrl` Sitecore setting:

<pre class="EnlighterJSRAW" data-enlighter-language="generic" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">&lt;setting name="Media.MediaLinkServerUrl" value="http://hostname.com" patch:sourceEnvironmentVariables="SITECORE_SETTINGS_MEDIA.MEDIALINKSERVERURL"/></pre>

We are successfully using this approach for a while in our project in combination with Docker. Being able to set any Sitecore setting without patching config files greatly simplifies the configuration management.