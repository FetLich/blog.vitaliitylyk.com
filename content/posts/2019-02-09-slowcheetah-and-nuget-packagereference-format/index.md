---
title: SlowCheetah and NuGet PackageReference format
date: 2019-02-09T16:26:40+00:00
summary: |
  With all it’s benefits, the NuGet PackageReference format sometimes brings challenges.
  
  Recently I’ve been converting a Visual Studio solution to the NuGet PackageReference format. After conversion, I have encountered an issue that SlowCheetah config transformations stopped working properly when using MsBuild (which is relevant for CI/CD builds). Luckily, I have found a workaround, which I thought is worth sharing.
description: Workaround for SlowCheetah not transforming config files when using MsBuild and NuGet PackageReference format.
url: /slowcheetah-and-nuget-packagereference-format/
categories:
  - DevOps
tags:
  - DevOps

---

![SlowCheetah and NuGet PackageReference format](slowcheetah.jpg#center "SlowCheetah and NuGet PackageReference format")

With all it&#8217;s [benefits](https://docs.microsoft.com/en-us/nuget/reference/migrate-packages-config-to-package-reference#benefits-of-using-packagereference "benefits"), the NuGet PackageReference format sometimes brings [challenges](https://docs.microsoft.com/en-us/nuget/reference/migrate-packages-config-to-package-reference#package-compatibility-issues "challenges").

Recently I&#8217;ve been converting a Visual Studio solution to the NuGet PackageReference format. After conversion, I have encountered an issue that SlowCheetah config transformations stopped working properly when using MsBuild (which is relevant for CI/CD builds). Luckily, I have found a workaround, which I thought is worth sharing.

## The symptoms

  * You are using SlowCheetah for config file transformations (**other than the Web.config**)
  * Your solution is using the **NuGet PackageReference format**
  * Transformations **work perfectly fine from within the VisualStudio** 
  * Transformations **do not work when using MsBuild** (which is relevant for CI/CD builds)

## Workaround

Luckily, I&#8217;ve found this StackOverflow question without answers, but with a workaround in the question itself: [https://stackoverflow.com/questions/50298798/config-file-transformation-doesnt-work-on-build-server-after-migration-to-packa](https://stackoverflow.com/questions/50298798/config-file-transformation-doesnt-work-on-build-server-after-migration-to-packa). This SO question would benefit if the workaround was posted as an answer, but anyway thanks to Eugene who posted it and saved me a lot of time! 🙂

Normally, the .csproj file for a web project containts the following imports:

```xml
<Import Project="$(MSBuildBinPath)\Microsoft.CSharp.targets" />
<Import Project="$(VSToolsPath)\WebApplications\Microsoft.WebApplication.targets" Condition="'$(VSToolsPath)' != ''" />
<Import Project="$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v10.0\WebApplications\Microsoft.WebApplication.targets" Condition="false" />
```

Change the order of the `Microsoft.CSharp.targets` and `Microsoft.WebApplication.targets` imports in your .csproj file from, so that it becomes:

```xml
<Import Project="$(VSToolsPath)\WebApplications\Microsoft.WebApplication.targets" Condition="'$(VSToolsPath)' != ''" />
<Import Project="$(MSBuildBinPath)\Microsoft.CSharp.targets" />
<Import Project="$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v10.0\WebApplications\Microsoft.WebApplication.targets" Condition="false" />
```

This solves the issue, however I am not sure about any negative consequences of such a switch, since order of import delements matters. I&#8217;ve been using this workaround on my project for a while and **did not notice any issues**.

The issue has also been reported on GitHub: [https://github.com/Microsoft/slow-cheetah/issues/78](https://github.com/Microsoft/slow-cheetah/issues/78). So hopefully, Microsoft will address it in the next SlowCheetah release.