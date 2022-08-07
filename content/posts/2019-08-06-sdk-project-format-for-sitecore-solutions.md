---
title: SDK project format for Sitecore solutions
author: Vitalii Tylyk
type: post
date: 2019-08-06T08:45:14+00:00
excerpt: Nowadays SDK Visual Studio project format is mainly used by .NET Core projects. It is lean, offers a lot of great features and is generally awesome. But can we use it in traditional ASP.NET Sitecore solutions? What are the benefits and pitfalls?
url: /sdk-project-format-for-sitecore-solutions/
featured_image: /wp-content/uploads/2019/08/sdk_project.jpg
categories:
  - Sitecore
tags:
  - Sitecore

---
Nowadays SDK Visual Studio project format is mainly used by .NET Core projects. It is lean, offers a lot of great features and is generally awesome. But can we use it in traditional ASP.NET Sitecore solutions? What are the benefits and pitfalls?

Luckily I am not alone with this thought: Anders Laub has already setup an example of a Helix solution using the SDK project format: <a rel="noreferrer noopener" aria-label=" (opens in a new tab)" href="https://github.com/LaubPlusCo/helix-msbuild-example" target="_blank">https://github.com/LaubPlusCo/helix-msbuild-example</a>, many thanks for that! Do check it out (make sure you have Visual Studio 2017 _version 15.6_ or later). Not only demonstrates this example a Helix setup with the SDK project format, but also it adds support for:

  * Automatic publishing on build, which is very handy during development
  * XML config transformations
  * With some modifications you can add support for publishing to multiple deployment targets (e.g website/xconnect/etc). More on this here: <a rel="noreferrer noopener" aria-label=" (opens in a new tab)" href="https://github.com/Sitecore/Helix.Docs/issues/22" target="_blank">https://github.com/Sitecore/Helix.Docs/issues/22</a> 
  * The solution is structured in a very clean way: common project file contents (e.g build properties, project includes, common Nuget references, etc) are separated out to a shared location via the _Directory.Build.props_ and _Directory.Build.targets_ files: <a rel="noreferrer noopener" aria-label=" (opens in a new tab)" href="https://docs.microsoft.com/en-us/visualstudio/msbuild/customize-your-build?view=vs-2019#directorybuildprops-and-directorybuildtargets" target="_blank">https://docs.microsoft.com/en-us/visualstudio/msbuild/customize-your-build?view=vs-2019#directorybuildprops-and-directorybuildtargets</a> 
  * All of the above is done using default MSBuild functionality, no custom code whatsoever

In this post I will share my experiences and spread the word. But first of all, let&#8217;s look at the benefits the SDK project format brings.

## Benefits of the SDK project format

So as not to repeat what&#8217;s already there in the internet, I will refer to this post, which well describes the main differences and the benefits: <a rel="noreferrer noopener" aria-label=" (opens in a new tab)" href="https://www.michaeltaylorp3.net/migrating-to-sdk-project-format/" target="_blank">https://www.michaeltaylorp3.net/migrating-to-sdk-project-format/</a> 

I will highlight a couple of points though: 

  * Unlike the &#8220;old&#8221; verbose .csproj format, the SDK project format is **very lean**. You don&#8217;t have to explicitly list files and folders within the project: all relevant files under project root folder are assumed to be included. This drastically decreases the .csproj file size and simplifies source control merges.  
  
    This is an example how tiny a .csproj file for your Helix module could look like:  
  
<img loading="lazy" width="585" height="278" class="wp-image-799" style="width: 585px;" src="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/08/sdk_project.jpg?resize=585%2C278&#038;ssl=1" alt="SDK project format for Helix solutions" srcset="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/08/sdk_project.jpg?w=585&ssl=1 585w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/08/sdk_project.jpg?resize=300%2C143&ssl=1 300w" sizes="(max-width: 585px) 100vw, 585px" data-recalc-dims="1" />  
    
  * The SDK project format **offers new awesome features**. For example the `Microsoft.Build.CentralPackageVersions` MSBuild project SDK allows you to manage your NuGet package versions in one place: <https://github.com/microsoft/MSBuildSdks/tree/master/src/CentralPackageVersions>  
  
    Imagine how easy it becomes to update a Nuget package across your solution, **for example bump Sitecore Nuget versions during Sitecore upgrade &#8211; just update one file. Not to mention, it works blazing fast**!  
  
    For example, this is how your Helix module Nuget references could look like (the actual versions would be specified in a solution-wide file used by all projects):

<div class="wp-block-image">
  <figure class="aligncenter"><img loading="lazy" width="518" height="86" src="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/08/image-1.png?resize=518%2C86&#038;ssl=1" alt="" class="wp-image-801" srcset="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/08/image-1.png?w=518&ssl=1 518w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/08/image-1.png?resize=300%2C50&ssl=1 300w" sizes="(max-width: 518px) 100vw, 518px" data-recalc-dims="1" /><figcaption>No Nuget package versions in specific projects</figcaption></figure>
</div>

  * Only the **PackageReference Nuget packages format** is supported. But it is a good thing, since PackageReference format brings many benefits, you can find more information here <a rel="noreferrer noopener" href="https://community.sitecore.net/technical_blogs/b/technical-marketing/posts/sitecore-9-1-nuget-changes-and-you" target="_blank">https://community.sitecore.net/technical_blogs/b/technical-marketing/posts/sitecore-9-1-nuget-changes-and-you</a> and here <a rel="noreferrer noopener" aria-label=" (opens in a new tab)" href="https://www.konabos.com/blog/sitecore-nuget-package-references" target="_blank">https://www.konabos.com/blog/sitecore-nuget-package-references</a> 
  * It supports **multi-targeting**, check out this post by Anastasiya Flynn for an interesting Sitecore-related use case: <a rel="noreferrer noopener" aria-label=" (opens in a new tab)" href="https://codealamode.blog/multiple-sitecore-versions-in-VS2017" target="_blank">https://codealamode.blog/multiple-sitecore-versions-in-VS2017</a> 
  * And last but not the least, to edit a .csproj file you **don&#8217;t need to unload the project**! 

## Helix and SDK project format

The &#8220;traditional&#8221; Habitat and HabitatHome examples use the Web Application project templates based on the &#8220;old&#8221; .csproj format. Each Helix module is typically a Web Application. Semantically this is wrong, since modules are not independent web apps and cannot be run separately, however, this does bring several benefits:

  1. _Scaffolding_. For example, for Web Application type of projects Visual Studio provides a simple way to create Controllers and Views.  
  
    This is convenient and saves some time, but comparing to the benefits of the SDK format, losing this is not a big sacrifice.  
    
  2. _Publishing_. Web Application projects import `Microsoft.WebApplication.targets`, making it possible to use web publish functionality to be able to deploy your Helix modules. Also you get some tooling support, e.g in Visual Studio you can publish specific Helix module via right click => publish, which can be handy if you don&#8217;t want to republish the whole solution every time.  
  
    If you follow the <a rel="noreferrer noopener" href="https://github.com/LaubPlusCo/helix-msbuild-example" target="_blank">https://github.com/LaubPlusCo/helix-msbuild-example</a> example, then you get automatic publishing on build. This works by importing the same `Microsoft.WebApplication.targets` into the projects (this is done solution-wide via the _Directory.Build.targets_ file). You will lose the right click => publish feature though, but it is compensated by being able to build a specific project, which would auto-publish the changes. 

Besides these two I do not see any other reasons why Web Application project template is used for Helix modules.

<blockquote class="wp-block-quote">
  <p>
    <em>Note:</em> The SDK project format also has a Web SDK variation (<code>Microsoft.NET.Sdk.Web</code>), but it is meant for ASP.NET Core.
  </p>
</blockquote>

## Limitations of SDK project format

While working with a Helix solution setup using SDK project format I have not noticed any considerable downsides which would overweight the benefits. However, there are some things you need to be aware of:

  * To be able to use all the benefits, you need to make sure you understand the basics of MsBuild, e.g what Properties and Targets are, how to conditionally load them, etc.
  * Nested files are not supported. For instance transformation files like these would be shown as a flat list (so this makes only visual difference, not functional)

<div class="wp-block-image">
  <figure class="aligncenter"><img loading="lazy" width="189" height="57" src="https://i2.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/08/image.png?resize=189%2C57&#038;ssl=1" alt="" class="wp-image-795" data-recalc-dims="1" /></figure>
</div>

  * Sometimes tooling is not that mature. For example, if you use `Microsoft.Build.CentralPackageVersions` SDK and install a Nuget package to some project, this will add a version attribute to the PackageReference, which will cause an error. Luckily, the Nuget team is already looking into this: <a rel="noreferrer noopener" aria-label=" (opens in a new tab)" href="https://github.com/microsoft/MSBuildSdks/issues/62" target="_blank">https://github.com/microsoft/MSBuildSdks/issues/62</a>

Obviously, these are not stoppers.