---
title: SlowCheetah and NuGet PackageReference format
author: Vitalii Tylyk
type: post
date: 2019-02-09T16:26:40+00:00
excerpt: |
  With all it’s benefits, the NuGet PackageReference format sometimes brings challenges.
  
  Recently I’ve been converting a Visual Studio solution to the NuGet PackageReference format. After conversion, I have encountered an issue that SlowCheetah config transformations stopped working properly when using MsBuild (which is relevant for CI/CD builds). Luckily, I have found a workaround, which I thought is worth sharing.
url: /slowcheetah-and-nuget-packagereference-format/
featured_image: /wp-content/uploads/2019/02/slowcheetah-477x430.jpg
categories:
  - DevOps
tags:
  - DevOps

---
<div class="wp-block-image">
  <figure class="aligncenter"><img loading="lazy" width="500" height="451" src="https://i2.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/02/slowcheetah.jpg?resize=500%2C451&#038;ssl=1" alt="SlowCheetah and NuGet PackageReference format" class="wp-image-517" srcset="https://i2.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/02/slowcheetah.jpg?w=500&ssl=1 500w, https://i2.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/02/slowcheetah.jpg?resize=300%2C271&ssl=1 300w, https://i2.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/02/slowcheetah.jpg?resize=477%2C430&ssl=1 477w" sizes="(max-width: 500px) 100vw, 500px" data-recalc-dims="1" /><figcaption>Some cheetahs are slow</figcaption></figure>
</div>

With all it&#8217;s <a rel="noreferrer noopener" href="https://docs.microsoft.com/en-us/nuget/reference/migrate-packages-config-to-package-reference#benefits-of-using-packagereference" target="_blank">benefits</a>, the NuGet PackageReference format sometimes brings <a rel="noreferrer noopener" href="https://docs.microsoft.com/en-us/nuget/reference/migrate-packages-config-to-package-reference#package-compatibility-issues" target="_blank">challenges</a>.

Recently I&#8217;ve been converting a Visual Studio solution to the NuGet PackageReference format. After conversion, I have encountered an issue that SlowCheetah config transformations stopped working properly when using MsBuild (which is relevant for CI/CD builds). Luckily, I have found a workaround, which I thought is worth sharing.

## The symptoms

  * You are using SlowCheetah for config file transformations (**other than the Web.config**)
  * Your solution is using the **NuGet PackageReference format**
  * Transformations **work perfectly fine from within the VisualStudio** 
  * Transformations **do not work when using MsBuild** (which is relevant for CI/CD builds)

## Workaround

Luckily, I&#8217;ve found this StackOverflow question without answers, but with a workaround in the question itself: <a rel="noreferrer noopener" aria-label=" (opens in a new tab)" href="https://stackoverflow.com/questions/50298798/config-file-transformation-doesnt-work-on-build-server-after-migration-to-packa" target="_blank">https://stackoverflow.com/questions/50298798/config-file-transformation-doesnt-work-on-build-server-after-migration-to-packa</a>. This SO question would benefit if the workaround was posted as an answer, but anyway thanks to Eugene who posted it and saved me a lot of time! 🙂

Normally, the .csproj file for a web project containts the following imports:

<pre class="EnlighterJSRAW" data-enlighter-language="generic" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">&lt;Import Project="$(MSBuildBinPath)\Microsoft.CSharp.targets" />
 &lt;Import Project="$(VSToolsPath)\WebApplications\Microsoft.WebApplication.targets" Condition="'$(VSToolsPath)' != ''" />
 &lt;Import Project="$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v10.0\WebApplications\Microsoft.WebApplication.targets" Condition="false" /></pre>

Change the order of the `Microsoft.CSharp.targets` and `Microsoft.WebApplication.targets` imports in your .csproj file from, so that it becomes:

<pre class="EnlighterJSRAW" data-enlighter-language="generic" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">&lt;Import Project="$(VSToolsPath)\WebApplications\Microsoft.WebApplication.targets" Condition="'$(VSToolsPath)' != ''" />
  &lt;Import Project="$(MSBuildBinPath)\Microsoft.CSharp.targets" />
  &lt;Import Project="$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v10.0\WebApplications\Microsoft.WebApplication.targets" Condition="false" /></pre>

This solves the issue, however I am not sure about any negative consequences of such a switch, since order of import delements matters. I&#8217;ve been using this workaround on my project for a while and **did not notice any issues**.

The issue has also been reported on GitHub: <a rel="noreferrer noopener" href="https://github.com/Microsoft/slow-cheetah/issues/78" target="_blank">https://github.com/Microsoft/slow-cheetah/issues/78.</a> So hopefully, Microsoft will address it in the next SlowCheetah release.