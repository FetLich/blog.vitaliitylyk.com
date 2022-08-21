---
title: Sitecore environment variables config builder
date: 2020-02-06T11:57:04+00:00
summary: "Inspired by ASP.NET configuration builders I thought: wouldn't it be awesome to be able to modify Sitecore configuration via environment variables in a similar way? This is especially relevant in Docker world."
url: /sitecore-environment-variables-config-builder/
description: "Using Environment variables for managing Sitecore configuration: settings, sc.variables and site settings in Docker"
categories:
  - DevOps
  - Sitecore
tags:
  - DevOps
  - Docker
  - Sitecore

---
Inspired by ASP.NET [configuration builders](https://docs.microsoft.com/en-us/aspnet/config-builder "configuration builders") I thought: wouldn&#8217;t it be awesome to be able to modify Sitecore configuration via environment variables in a similar way? This is especially relevant in Docker world.

As [I&#8217;ve mentioned before, to be a good Docker citizen](/making-sitecore-a-good-docker-citizen/ "I've mentioned before, to be a good Docker citizen"), a containerized application has to support configuration via environment variables.

Out of the box, Sitecore supports `$(env:ENVIRONMENT_VARIABLE_NAME)` syntax to use environment variables in your configuration. However, this has 2 limitations:

  * This syntax is not supported for Sitecore variables (`sc.variable`)
  * Before being able to actually use environment variables, you first have to modify all places where you want to use them with `$(env:ENVIRONMENT_VARIABLE_NAME)` syntax. 

My goal was to be able to modify Sitecore configuration via environment variables without making any changes to Sitecore config files themselves. In particular:

  * Update settings
  * Update variables
  * Update site settings (e.g set hostName, scheme, etc)

This syntax would look something like this (on the left is an environment variable name):

```
SITECORE_SETTINGS_<settingname>=<value>
SITECORE_SITES_<sitename>_<attributename>=<value>
SITECORE_VARIABLES_<variablename>=<value>
```

For example:

```
SITECORE_SETTINGS_MEDIA.MEDIALINKSERVERURL=https://www.hostname.com
SITECORE_SITES_WEBSITE_HOSTNAME=hostname.com
```

At first I considered to follow Microsoft approach and develop a custom configuration builder. However, ASP.NET configuration builders intrude **before** Sitecore does it&#8217;s magic with aggregating all include files. Therefore, this approach will not allow to inject environment variables into include files, which is very limiting.

So I ended up with customizing the Sitecore configuration section handler. By default, Sitecore config section is defined in Web.config in the following way:

```xml
<configSections>
    ...
    <section name="sitecore" type="Sitecore.Configuration.RuleBasedConfigReader, Sitecore.Kernel" />
    ...
</configSections>
```

This can be easily customized via pointing the &#8220;sitecore&#8221; section handler to a custom class, which inherits from `RuleBasedConfigReader`. Below you can find an example of a custom handler which loops over environment variables and injects them into Sitecore configuration, if it matches the appropriate `setting`/`sc.variable` or `site setting` (the comparison is case-insensitive). In case of a match the `patch:sourceEnvironmentVariables` attribute is appended to an appropriate XML node to be able to easily track changes in Sitecore&#8217;s `showconfig.aspx`.

{{< gist vitaliitylyk ac419bc30993e3a6dfe7ed88767eb2ce >}}

And this is an output from `showconfig.aspx` for the `Media.MediaLinkServerUrl` Sitecore setting:

```xml
<setting name="Media.MediaLinkServerUrl" value="http://hostname.com" patch:sourceEnvironmentVariables="SITECORE_SETTINGS_MEDIA.MEDIALINKSERVERURL"/>
```

We are successfully using this approach for a while in our project in combination with Docker. Being able to set any Sitecore setting without patching config files greatly simplifies the configuration management.