---
title: Tips on migrating your Sitecore solution to Docker
author: Vitalii Tylyk
type: post
date: 2020-09-22T08:22:47+00:00
excerpt: Now that Sitecore officially supports Docker containers, many of us started considering migrating their Sitecore solutions to Docker setup. But how hard is that and what are the important steps/pain points to consider for production-ready setup?
url: /tips-on-migrating-your-sitecore-solution-to-docker/
featured_image: /wp-content/uploads/2020/09/Moby-logo.png
categories:
  - DevOps
  - Sitecore
tags:
  - DevOps
  - Docker
  - Sitecore

---
<div class="wp-block-image">
  <figure class="aligncenter size-medium"><img loading="lazy" width="300" height="215" src="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/Moby-logo.png?resize=300%2C215&#038;ssl=1" alt="Migrating Sitecore solution to Docker and Kubernetes" class="wp-image-1234" srcset="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/Moby-logo.png?resize=300%2C215&ssl=1 300w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/Moby-logo.png?w=601&ssl=1 601w" sizes="(max-width: 300px) 100vw, 300px" data-recalc-dims="1" /></figure>
</div>

Now that Sitecore officially supports Docker containers, many of us started considering migrating their Sitecore solutions to Docker setup. But how hard is that and what are the important steps/pain points to consider for production-ready setup?

<blockquote class="wp-block-quote">
  <p>
    <strong>Note:</strong> In this post I assume that the reader has basic understanding of Docker containers and orchestrators. If you are not familiar with Docker, here are some good resources to get started:<br /><br />* <a rel="noreferrer noopener" href="https://www.sitecore.com/knowledge-center/getting-started/docker-a-quick-overview" target="_blank">https://www.sitecore.com/knowledge-center/getting-started/docker-a-quick-overview</a><br />* A great book by Elton Stoneman:<em> “Docker on Windows: From 101 to production with Docker on Windows“</em>
  </p>
</blockquote>

These are the topics I am gonna cover in this blogpost:

  * <a href="#preface" data-type="internal" data-id="#preface">Preface</a>
  * [0. When to consider migrating your Sitecore solution to Docker][1]
  * <a href="#sitecore-version" data-type="internal" data-id="#sitecore-version">1. Sitecore version</a>
  * [2. Windows container base OS version][2]
  * [3. Local development environment setup][3]
  * [4. Container orchestrator][4]
  * [5. Cloud provider to run your containers][5]
  * <a href="#making-sitecore-a-good-docker-citizen" data-type="internal" data-id="#making-sitecore-a-good-docker-citizen">6. Making Sitecore a good Docker citizen</a>
  * <a href="#prepare-for-cicd-revamp" data-type="internal" data-id="#prepare-for-cicd-revamp">7. Prepare for CI/CD revamp</a>
  * [8. Application Monitoring (Health Checks)][6]
  * [9. Centralized logging][7]
  * [10. Containers are immutable][8]
  * <a href="#summary" data-type="internal" data-id="#summary">Summary</a>

But first a little bit of pre-history on why we have decided to migrate to Docker setup in my company.

## Preface {#preface}

We have started considering migration to Docker far before Sitecore has <a rel="noreferrer noopener" href="https://kb.sitecore.net/articles/161310" target="_blank">officially announced containers support</a>. Our Sitecore solution was running on AWS EC2 virtual machines, which was not ideal for us from DevOps perspective for obvious reasons:

  * Applying security / windows updates is a hassle
  * Automatic scaling out is tricky
  * We wanted to eliminate the &#8220;it works on my machine&#8221; scenario

So we have decided to take the next step for our Sitecore solution. The options were: Azure PAAS and Docker. 

Azure PAAS was tempting, since it was officially supported by Sitecore (unlike containers at that time) and would solve most of our problems. However, since the rest of our company projects are running in AWS, introducing a new cloud provider was not a great idea. We would have to hire Azure DevOps professionals plus fight with various hybrid-cloud networking issues. And since Sitecore does not support PAAS setup in AWS the most logical step for us was to choose Docker.  
At that time Amazon offered managed Kubernetes service (EKS) with Windows support only in preview mode. With this in mind, combined with absence of containers support from Sitecore we&#8217;ve still decided to take a leap of faith.

Luckily for us, the <a rel="noreferrer noopener" href="https://github.com/Sitecore/docker-images" target="_blank">https://github.com/Sitecore/docker-images</a> repository has been provided by Sitecore community, which was extremely helpful. During our Docker migration process I was able to contribute several Pull Requests to it, and even one to the Microsoft Docker images repository, which I am quite proud of.

Alright, enough pre-face, here are the most important things you need to consider if you want to migrate to Docker.

## 0. When to consider migrating your Sitecore solution to Docker {#when-to-consider-docker}

<div class="wp-block-image">
  <figure class="aligncenter size-full is-resized"><img loading="lazy" src="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/docker-when-to-consider.jpg?resize=512%2C393&#038;ssl=1" alt="When to consider Docker foryour Sitecore solution" class="wp-image-1242" width="512" height="393" srcset="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/docker-when-to-consider.jpg?w=1024&ssl=1 1024w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/docker-when-to-consider.jpg?resize=300%2C230&ssl=1 300w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/docker-when-to-consider.jpg?resize=768%2C589&ssl=1 768w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/docker-when-to-consider.jpg?resize=561%2C430&ssl=1 561w" sizes="(max-width: 512px) 100vw, 512px" data-recalc-dims="1" /></figure>
</div>

If you are running on Azure PAAS and you&#8217;re happy with the setup &#8211; it is probably not worth to migrate to Docker. However, you might still benefit from using containers in your local development environment for several reasons:

  * Simplify local environment setup
  * Achieve consistency of environments among your developers and eliminate the &#8220;it works on my machine&#8221; case

If you are running on premises or on IAAS cloud setup, then migrating your Sitecore solution to containers is definitely worth considering.

Another thing to consider: is your organization ready for it? There is a great post by Jason St-Cyr on the topic: <a rel="noreferrer noopener" href="https://www.sitecore.com/knowledge-center/getting-started/should-my-team-adopt-docker" target="_blank">https://www.sitecore.com/knowledge-center/getting-started/should-my-team-adopt-docker</a>

## 1. Sitecore version {#sitecore-version}

<div class="wp-block-image">
  <figure class="aligncenter size-medium"><img loading="lazy" width="300" height="300" src="https://i2.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/sitecore-logo.png?resize=300%2C300&#038;ssl=1" alt="Sitecore version to use with Docker containers" class="wp-image-1246" srcset="https://i2.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/sitecore-logo.png?resize=300%2C300&ssl=1 300w, https://i2.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/sitecore-logo.png?resize=1024%2C1024&ssl=1 1024w, https://i2.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/sitecore-logo.png?resize=150%2C150&ssl=1 150w, https://i2.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/sitecore-logo.png?resize=768%2C768&ssl=1 768w, https://i2.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/sitecore-logo.png?resize=1536%2C1536&ssl=1 1536w, https://i2.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/sitecore-logo.png?resize=430%2C430&ssl=1 430w, https://i2.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/sitecore-logo.png?resize=1100%2C1100&ssl=1 1100w, https://i2.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/sitecore-logo.png?w=1600&ssl=1 1600w" sizes="(max-width: 300px) 100vw, 300px" data-recalc-dims="1" /></figure>
</div>

Let&#8217;s assume you are seriously considering migrating your Sitecore solution to Docker.

The first thing to take into account is Sitecore version. Sitecore <a rel="noreferrer noopener" href="https://kb.sitecore.net/articles/161310" target="_blank">officially supports running in containers starting from version 9.3</a>, but official Docker images are only provided starting from Sitecore 10.

Therefore, there are 3 possible scenarios:

### a) You are running on Sitecore 9.3

You&#8217;re not getting official Docker images from Sitecore, but you can build them yourself from <a rel="noreferrer noopener" href="https://github.com/Sitecore/docker-images" target="_blank">https://github.com/Sitecore/docker-images</a>  
There are quite some blog posts and videos in community on how to do that, for example: 

  * [Sitecore Docker Images Repository][9]
  * <a href="https://www.youtube.com/watch?v=SRsCgyPeCTg" target="_blank" rel="noreferrer noopener">Introduction to using Sitecore Docker Images</a>

### b) You are running Sitecore 10

This is the best-case scenario. Sitecore provides both official support and Docker images for you off the shelf.

In addition to this, there is a great documentation on how to get started: <a rel="noreferrer noopener" href="https://containers.doc.sitecore.com/docs/intro" target="_blank">https://containers.doc.sitecore.com/docs/intro</a> accompanied with examples: <a rel="noreferrer noopener" href="https://github.com/Sitecore/docker-examples" target="_blank">https://github.com/Sitecore/docker-examples</a>

### c) You&#8217;re running pre-Sitecore 9.3 version

Not only this setup is not officially supported, but also old Sitecore versions are lacking some features which simplify Docker setup. Sitecore 9.3 is a <a rel="noreferrer noopener" href="https://blog.vitaliitylyk.com/sitecore-9-3-loves-docker/" target="_blank">much better Docker citizen</a>.

On top of that, <a rel="noreferrer noopener" href="https://github.com/Sitecore/docker-images" target="_blank">community Sitecore docker images repository</a> does not provide images for Sitecore versions lower than 9.0.2, so you will have to build them yourself.

Therefore, I highly recommend you to upgrade, preferably straight to 10. Not that you cannot dockerize pre-9.0.2 versions, it is just that it would require much more effort.

If you decided to go for an upgrade &#8211; do it first before taking any steps related to Docker migration. This will help to avoid Big Bang scenario when you are both working on Sitecore upgrade and Docker migration.

## 2. Windows container base OS version {#windows-container-base-os-version}

<div class="wp-block-image">
  <figure class="aligncenter size-medium"><img loading="lazy" width="300" height="265" src="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/windows-logo.png?resize=300%2C265&#038;ssl=1" alt="Windows container version to use with Sitecore" class="wp-image-1248" srcset="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/windows-logo.png?resize=300%2C265&ssl=1 300w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/windows-logo.png?resize=768%2C678&ssl=1 768w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/windows-logo.png?resize=487%2C430&ssl=1 487w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/windows-logo.png?w=870&ssl=1 870w" sizes="(max-width: 300px) 100vw, 300px" data-recalc-dims="1" /></figure>
</div>

At the time of writing, Sitecore officially supports only the ltsc2019 base container OS version on production. This is also the version which most cloud providers currently support in their managed Kubernetes services.  
So this would be the best option for now. We are running on Windows 1909, but that is because our DevOps team likes to be on the edge 😎

Even though for Production setup it is best to use ltsc2019 version, for your local development you can use newer versions. This won&#8217;t give you much benefit, but will allow to use newer Windows versions in combination with <a rel="noreferrer noopener" href="https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/hyperv-container" target="_blank">process isolation mode</a>.

## 3. Local development environment setup {#local-development-environment-setup}

### Performance

For the best performance (which is especially relevant for Sitecore) you need to have your local Windows version matching the version of base OS Docker images. This is needed to be able to run containers in <a rel="noreferrer noopener" href="https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/hyperv-container" target="_blank">process isolation mode</a>. For example, if you plan to use 1809 (ltsc) base images, then make sure your windows version is Windows 10 1809 as well.

With process isolation I have not noticed any performance degradation in comparison to non-Docker setup. However, based on my experience, **hyper-v isolation has proven to be much, much slower**.

<blockquote class="wp-block-quote">
  <p>
    <strong>Extra Tips:</strong> <br />1. Add <code>C:\ProgramData\Docker\containers</code> to your antivirus exclusion list, or you might experience some <a rel="noreferrer noopener" href="https://docs.docker.com/engine/security/antivirus/" target="_blank">nasty locking issues</a>.<br />2. Make sure your local IIS sites do not collide with your Docker sites (hostnames, ports). Eventually you might disable your local IIS instance completely.
  </p>
</blockquote>

Docker should not eat a lot of extra RAM comparing to your previous local setup, but the more the better. <a href="https://containers.doc.sitecore.com/docs/troubleshooting#container-environment-memory-usage" target="_blank" rel="noreferrer noopener">Sitecore recommends 32GB RAM</a> for developer workstations, with 16GB minimum.

### Development workflow

<div class="wp-block-image">
  <figure class="aligncenter size-large"><img loading="lazy" width="1983" height="912" src="https://i2.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/life-cycle-containerized-apps-docker-cli.png?fit=1024%2C471&ssl=1" alt="Local development workflow with Docker" class="wp-image-1250" srcset="https://i2.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/life-cycle-containerized-apps-docker-cli.png?w=1983&ssl=1 1983w, https://i2.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/life-cycle-containerized-apps-docker-cli.png?resize=300%2C138&ssl=1 300w, https://i2.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/life-cycle-containerized-apps-docker-cli.png?resize=1024%2C471&ssl=1 1024w, https://i2.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/life-cycle-containerized-apps-docker-cli.png?resize=768%2C353&ssl=1 768w, https://i2.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/life-cycle-containerized-apps-docker-cli.png?resize=1536%2C706&ssl=1 1536w, https://i2.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/life-cycle-containerized-apps-docker-cli.png?resize=740%2C340&ssl=1 740w, https://i2.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/life-cycle-containerized-apps-docker-cli.png?resize=1100%2C506&ssl=1 1100w" sizes="(max-width: 1100px) 100vw, 1100px" /><figcaption>Image source: <a href="https://docs.microsoft.com/en-us/dotnet/architecture/containerized-lifecycle/design-develop-containerized-apps/docker-apps-inner-loop-workflow" target="_blank" rel="noreferrer noopener">https://docs.microsoft.com/en-us/dotnet/architecture/containerized-lifecycle/design-develop-containerized-apps/docker-apps-inner-loop-workflow</a></figcaption></figure>
</div>

With introduction of containers, your local development workflow will change as well:

  * Publishing solution changes will work <a rel="noreferrer noopener" href="https://containers.doc.sitecore.com/docs/file-deployment" target="_blank">slightly different</a>
  * <a rel="noreferrer noopener" href="https://containers.doc.sitecore.com/docs/remote-debugging" target="_blank">Debugging code has it&#8217;s specifics</a>
  * Syncing your serialized items will <a rel="noreferrer noopener" href="https://containers.doc.sitecore.com/docs/item-sync-running-container" target="_blank">require some extra configuration</a>
  * You will need to get used to some tooling to manage containers, such as Docker CLI, VS Code, etc

For a great general overview of containerized development lifecycle check this guide from Microsoft: <a rel="noreferrer noopener" href="https://docs.microsoft.com/en-us/dotnet/architecture/containerized-lifecycle/" target="_blank">https://docs.microsoft.com/en-us/dotnet/architecture/containerized-lifecycle/</a>

### Solution setup

For inspiration on how to setup your Git repo and Visual Studio solution you can check the following examples provided by Sitecore:

  * <a rel="noreferrer noopener" href="https://github.com/Sitecore/docker-examples" target="_blank">https://github.com/Sitecore/docker-examples</a>
  * <a rel="noreferrer noopener" href="https://github.com/Sitecore/MVP-Site" target="_blank">https://github.com/Sitecore/MVP-Site</a> &#8211; open-sourced MVP site <a rel="noreferrer noopener" href="https://mvp.sitecore.com/" target="_blank">https://mvp.sitecore.com/</a>
  * <a rel="noreferrer noopener" href="https://github.com/Sitecore/Helix.Examples" target="_blank">https://github.com/Sitecore/Helix.Examples</a> &#8211; the main purpose of this repo is Helix, but it has some examples of Sitecore 10 solution setup with containers.

And here you can find a nice video, which walks you through the MVP site setup: <a rel="noreferrer noopener" href="https://www.youtube.com/watch?v=vllu8xKfzQU" target="_blank">https://www.youtube.com/watch?v=vllu8xKfzQU</a>

## 4. Container orchestrator {#container-orchestrator}

<div class="wp-block-image">
  <figure class="aligncenter size-medium"><img loading="lazy" width="300" height="300" src="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/kubernetes-logo-2.png?resize=300%2C300&#038;ssl=1" alt="Kubernetes container orchestrator for Sitecore" class="wp-image-1254" srcset="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/kubernetes-logo-2.png?resize=300%2C300&ssl=1 300w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/kubernetes-logo-2.png?resize=150%2C150&ssl=1 150w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/kubernetes-logo-2.png?resize=430%2C430&ssl=1 430w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/kubernetes-logo-2.png?w=512&ssl=1 512w" sizes="(max-width: 300px) 100vw, 300px" data-recalc-dims="1" /></figure>
</div>

If you plan to use Docker for your DTAP environments, you will need some sort of container orchestrator.

Sitecore provides <a rel="noreferrer noopener" href="https://github.com/Sitecore/MVP-Site/tree/master/k8s" target="_blank">some examples on Kubernetes setup</a>. K8S is also supported as a managed service by most major cloud providers,  
therefore this is a no-brainer to choose it for your production setup.

You might consider a simpler alternative like Azure Container Instances or AWS Elastic Container Service, but then you are less flexible and more dependent on the specific cloud implementation (which is not bad per se).  
In my case our DevOps team has extensive K8S experience, so it was an easy choice for us to go for AWS EKS.

<blockquote class="wp-block-quote">
  <p>
    <strong>Tip:</strong> I do not recommend running Kubernetes locally, since it only adds complexity without not much benefit. Docker Compose is good enough for local development setup.
  </p>
</blockquote>

## 5. Cloud provider to run your containers {#cloud-provider-to-run-your-containers}

<div class="wp-block-image">
  <figure class="aligncenter size-large"><img loading="lazy" width="640" height="366" src="https://i1.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/cloud-providers.png?resize=640%2C366&#038;ssl=1" alt="Cloud provider for Sitecore Docker containers" class="wp-image-1256" srcset="https://i1.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/cloud-providers.png?w=640&ssl=1 640w, https://i1.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/09/cloud-providers.png?resize=300%2C172&ssl=1 300w" sizes="(max-width: 640px) 100vw, 640px" data-recalc-dims="1" /></figure>
</div>

<blockquote class="wp-block-quote">
  <p>
    <strong>Note:</strong> If you plan to use Docker only for local development setup, then this is obviously not relevant. However, if you have decided to go all the way to Production then this is important choice to make.
  </p>
</blockquote>

If you are already running in the cloud, then the choice is made. However, if you are running on-premises, this is something you will need to decide.

Comparison between cloud providers is certainly a huge topic and it is impossible to cover it within a single post, but I will outline some tips specific to Sitecore.

Both Azure and AWS are suitable for running Sitecore in containers. They both provide managed Kubernetes services and other container hosting options. Google Cloud also supports running Windows containers, however this would be quite an exotic choice for Sitecore, therefore I did not look into this.

Besides containers you need to consider other system components:<figure class="wp-block-table">

<table>
  <tr>
    <td>
    </td>
    
    <td>
      <strong>Azure</strong>
    </td>
    
    <td>
      <strong>AWS</strong>
    </td>
  </tr>
  
  <tr>
    <td>
      <strong>SQL</strong>
    </td>
    
    <td>
      SQL Azure, <a href="https://kb.sitecore.net/articles/777703#:~:text=Sitecore%20XP%2FCMS%20supports%20Microsoft,using%20the%20Sitecore%20Azure%20module." target="_blank" rel="noreferrer noopener">officially supported by Sitecore</a>
    </td>
    
    <td>
      RDS, <a href="https://kb.sitecore.net/articles/013205" target="_blank" rel="noreferrer noopener">officially supported by Sitecore</a>
    </td>
  </tr>
  
  <tr>
    <td>
      <strong>Solr</strong>
    </td>
    
    <td>
      Self-hosted or <a rel="noreferrer noopener" href="https://www.searchstax.com/docs/sitecore-9-solr/" target="_blank">SearchStax</a>
    </td>
    
    <td>
      Self-hosted or <a rel="noreferrer noopener" href="https://www.searchstax.com/docs/sitecore-9-solr/" target="_blank">SearchStax</a>
    </td>
  </tr>
  
  <tr>
    <td>
      <strong>Redis (optional)</strong>
    </td>
    
    <td>
      Azure Cache for Redis
    </td>
    
    <td>
      Amazon ElastiCache for Redis
    </td>
  </tr>
</table></figure> 

<blockquote class="wp-block-quote">
  <p>
    <strong>Note: </strong>I am not including Azure Search as an alternative indexing provider because <a href="https://dev.sitecore.net/Downloads/Sitecore%20Experience%20Platform/100/Sitecore%20Experience%20Platform%20100/Release%20Notes" target="_blank" rel="noreferrer noopener">in Sitecore 10 it has been officially deprecated</a>.
  </p>
</blockquote>

Depending on your solution, you have to check what other system components you need to deploy and which cloud provider is the most suitable.

If in doubt, I think it is best to go for Azure. The reason is that Sitecore is more focused on Azure and this is what they use themselves for <a rel="noreferrer noopener" href="https://github.com/Sitecore/MVP-Site" target="_blank">their demo solutions</a>.

## 6. Making Sitecore a good Docker citizen {#making-sitecore-a-good-docker-citizen}

<div class="wp-block-image">
  <figure class="aligncenter size-medium"><img loading="lazy" width="283" height="300" src="https://i1.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/10/docker-sitecore.png?resize=283%2C300&#038;ssl=1" alt="Sitecore and Docker" class="wp-image-903" srcset="https://i1.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/10/docker-sitecore.png?resize=283%2C300&ssl=1 283w, https://i1.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/10/docker-sitecore.png?resize=405%2C430&ssl=1 405w, https://i1.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/10/docker-sitecore.png?w=600&ssl=1 600w" sizes="(max-width: 283px) 100vw, 283px" data-recalc-dims="1" /></figure>
</div>

I have already blogged about this topic before. So as not to repeat myself: 

  * <a rel="noreferrer noopener" href="https://blog.vitaliitylyk.com/making-sitecore-a-good-docker-citizen/" target="_blank">https://blog.vitaliitylyk.com/making-sitecore-a-good-docker-citizen/</a>
  * <https://blog.vitaliitylyk.com/sitecore-9-3-loves-docker/>

In short: Sitecore is almost there to be a good Docker citizen. In my opinion the only thing (but a very important one) it lacks is configuration via environment variables. 

Starting from version 9.3, Sitecore is partially configurable via environment variables (namely AppSettings and ConnectionStrings sections of web.config). However, to be able to modify other parts of Sitecore configuration via environment variables, you have to implement a custom solution: <a rel="noreferrer noopener" href="https://blog.vitaliitylyk.com/sitecore-environment-variables-config-builder/" target="_blank">https://blog.vitaliitylyk.com/sitecore-environment-variables-config-builder/</a>

I have registered a feature request to Sitecore via several channels so I hope they will support this out of the box in future versions. Please upvote this idea on <a href="https://sitecore-product.ideas.aha.io/ideas/SXP-I-76" target="_blank" rel="noreferrer noopener">https://sitecore-product.ideas.aha.io/ideas/SXP-I-76</a>

## 7. Prepare for CI/CD revamp {#prepare-for-cicd-revamp}

<div class="wp-block-image">
  <figure class="aligncenter size-medium"><img loading="lazy" width="300" height="170" src="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/06/devops.png?resize=300%2C170&#038;ssl=1" alt="Sitecore DevOps with Docker" class="wp-image-726" srcset="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/06/devops.png?resize=300%2C170&ssl=1 300w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/06/devops.png?resize=768%2C435&ssl=1 768w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/06/devops.png?resize=1024%2C580&ssl=1 1024w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/06/devops.png?resize=740%2C419&ssl=1 740w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/06/devops.png?resize=1100%2C623&ssl=1 1100w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2019/06/devops.png?w=1280&ssl=1 1280w" sizes="(max-width: 300px) 100vw, 300px" data-recalc-dims="1" /></figure>
</div>

Obviously, your CI/CD process will have to change, together with the mindset.

  * With Docker, you no longer deploy just your application code. Container is now the unit of deployment, which includes the whole environment: OS and application dependencies.
  * Therefore your build process will have to be extended to build Docker containers and push them to container registry. **Important:** make sure to use a private registry, using public registry to store Sitecore images is **not allowed**.
  * Kubernetes is not something you will learn over one day.

There is quite some information on how to build your custom images in Sitecore documentation: <a rel="noreferrer noopener" href="https://containers.doc.sitecore.com/docs/build-sitecore-images" target="_blank">https://containers.doc.sitecore.com/docs/build-sitecore-images</a>

On top of that, Microsoft provides a great overview of both inner and outer loops of containerized development life cycles here: <a href="https://docs.microsoft.com/en-us/dotnet/architecture/containerized-lifecycle/docker-devops-workflow/docker-application-outer-loop-devops-workflow" target="_blank" rel="noreferrer noopener">https://docs.microsoft.com/en-us/dotnet/architecture/containerized-lifecycle/docker-devops-workflow/docker-application-outer-loop-devops-workflow</a>

## 8. Application Monitoring (Health Checks) {#application-monitoring}

<div class="wp-block-image">
  <figure class="aligncenter size-medium"><img loading="lazy" width="300" height="280" src="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/07/health-checks.png?resize=300%2C280&#038;ssl=1" alt="Sitecore health checks" class="wp-image-1127" srcset="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/07/health-checks.png?resize=300%2C280&ssl=1 300w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/07/health-checks.png?resize=461%2C430&ssl=1 461w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/07/health-checks.png?w=705&ssl=1 705w" sizes="(max-width: 300px) 100vw, 300px" data-recalc-dims="1" /></figure>
</div>

Of course, application health monitoring is not something unique to Docker, but there are some specifics. For example, in K8S you can <a rel="noreferrer noopener" href="https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/" target="_blank">setup liveness and readiness probes</a> for your pods. Luckily, Sitecore comes with health check endpoints &nbsp;(`/healthz/live&nbsp;`and&nbsp;`/healthz/ready`) out of the box, which you can use for this purpose.

I have blogged before about <a rel="noreferrer noopener" href="https://blog.vitaliitylyk.com/sitecore-9-3-health-checks/" target="_blank">health checks in Sitecore 9.3 and on how to add a custom health check</a> in details, so I will not repeat myself here.

## 9. Centralized logging {#centralized-logging}

If you have a scaled Sitecore setup, you probably already have some centralized logging solution in place (Datadog, Logz.io, etc). If not &#8211; you definitely should look into that.

However, here it is worth mentioning that in Docker world it is considered to be the best practice when your containers output logs to Docker console. This is a very powerful technique, which:

  1. For local development gives you live tail of Sitecore logs. Before Docker times you had to either check .txt files manually or use some additional tooling.
  2. In DTAP environments allows to decouple your application from specifics of the centralized logging solution. Tools like Datadog would listen to the console output of your Docker containers and forward them to their servers.

And Sitecore as a <a rel="noreferrer noopener" href="https://blog.vitaliitylyk.com/making-sitecore-a-good-docker-citizen/" target="_blank">good Docker citizen</a> streams its logs to a console as well by using Filebeat (relevant for images provided in <a rel="noreferrer noopener" href="https://github.com/Sitecore/docker-images" target="_blank">https://github.com/Sitecore/docker-images</a>) and Sitecore 10 images.

## 10. Containers are immutable {#containers-are-immutable}

The last but not the least, you have to get used to Docker containers dogma: they are immutable. Which means that any changes to file system within container will be lost as soon as container restarts. So no more hotfixing dlls directly on PROD servers 🙂

Any change you want to persist to your Docker images (e.g. <a href="https://containers.doc.sitecore.com/docs/add-modules" target="_blank" rel="noreferrer noopener">install a Sitecore module</a>) would have to be done as a part of your CI/CD process when you build your Docker images.

If you store your media on File System that would be a problem, because as soon as container is recreated you will lose your data. Therefore, analyze your solution and check that your are not relying on local file system of your servers. And if you do &#8211; you will have to use shared storage instead.

<blockquote class="wp-block-quote">
  <p>
    <strong>Note:</strong> In addition to the above, file-based media storage in Sitecore is deprecated since 9.3 and will be removed in future.
  </p>
</blockquote>

## Summary {#summary}

Looking back, I am very happy that we have decided to migrate our solution to Docker setup. Not to mention all the benefits, it was fun. However make sure to analyze all the pros/cons and specifics of your organization to see if it is a good fit for you. Because as always, it depends 😉

 [1]: #when-to-consider-docker
 [2]: #windows-container-base-os-version
 [3]: #local-development-environment-setuo
 [4]: #container-orchestrator
 [5]: #cloud-provider-to-run-your-containers
 [6]: #application-monitoring
 [7]: #centralized-logging
 [8]: #containers-are-immutable
 [9]: https://www.youtube.com/watch?v=cA1CMdwrNVU