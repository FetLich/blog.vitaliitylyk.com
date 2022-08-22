---
title: Tips on migrating your Sitecore solution to Docker
date: 2020-09-22T08:22:47+00:00
summary: Now that Sitecore officially supports Docker containers, many of us started considering migrating their Sitecore solutions to Docker setup. But how hard is that and what are the important steps/pain points to consider for production-ready setup?
description: Important things to consider when migrating your Sitecore solution to Docker & Kubernetes setup.
url: /tips-on-migrating-your-sitecore-solution-to-docker/
featured_image: /wp-content/uploads/2020/09/Moby-logo.png
showtoc: true
tocopen: true
categories:
  - DevOps
  - Sitecore
tags:
  - DevOps
  - Docker
  - Sitecore

---
![Migrating Sitecore solution to Docker and Kubernetes](Moby-logo.png#center "Migrating Sitecore solution to Docker and Kubernetes")

Now that Sitecore officially supports Docker containers, many of us started considering migrating their Sitecore solutions to Docker setup. But how hard is that and what are the important steps/pain points to consider for production-ready setup?


> **Note:** In this post I assume that the reader has basic understanding of Docker containers and orchestrators. If you are not familiar with Docker, here are some good resources to get started:
> * https://www.sitecore.com/knowledge-center/getting-started/docker-a-quick-overview
> * A great book by Elton Stoneman: _“Docker on Windows: From 101 to production with Docker on Windows“_

First let me start with a little bit of pre-history on why we have decided to migrate to Docker setup in my company.

## Preface

We have started considering migration to Docker far before Sitecore has [officially announced containers support](https://kb.sitecore.net/articles/161310 "officially announced containers support"). Our Sitecore solution was running on AWS EC2 virtual machines, which was not ideal for us from DevOps perspective for obvious reasons:

  * Applying security / windows updates is a hassle
  * Automatic scaling out is tricky
  * We wanted to eliminate the &#8220;it works on my machine&#8221; scenario

So we have decided to take the next step for our Sitecore solution. The options were: Azure PAAS and Docker. 

Azure PAAS was tempting, since it was officially supported by Sitecore (unlike containers at that time) and would solve most of our problems. However, since the rest of our company projects are running in AWS, introducing a new cloud provider was not a great idea. We would have to hire Azure DevOps professionals plus fight with various hybrid-cloud networking issues. And since Sitecore does not support PAAS setup in AWS the most logical step for us was to choose Docker.  
At that time Amazon offered managed Kubernetes service (EKS) with Windows support only in preview mode. With this in mind, combined with absence of containers support from Sitecore we&#8217;ve still decided to take a leap of faith.

Luckily for us, the https://github.com/Sitecore/docker-images repository has been provided by Sitecore community, which was extremely helpful. During our Docker migration process I was able to contribute several Pull Requests to it, and even one to the Microsoft Docker images repository, which I am quite proud of.

Alright, enough pre-face, here are the most important things you need to consider if you want to migrate to Docker.

## 0. When to consider migrating your Sitecore solution to Docker

![When to consider Docker foryour Sitecore solution](docker-when-to-consider.jpg#center "When to consider Docker foryour Sitecore solution")

If you are running on Azure PAAS and you&#8217;re happy with the setup &#8211; it is probably not worth to migrate to Docker. However, you might still benefit from using containers in your local development environment for several reasons:

  * Simplify local environment setup
  * Achieve consistency of environments among your developers and eliminate the &#8220;it works on my machine&#8221; case

If you are running on premises or on IAAS cloud setup, then migrating your Sitecore solution to containers is definitely worth considering.

Another thing to consider: is your organization ready for it? There is a great post by Jason St-Cyr on the topic: https://www.sitecore.com/knowledge-center/getting-started/should-my-team-adopt-docker

## 1. Sitecore version

![Sitecore version to use with Docker containers](sitecore-logo.png#center "Sitecore version to use with Docker containers")

Let&#8217;s assume you are seriously considering migrating your Sitecore solution to Docker.

The first thing to take into account is Sitecore version. Sitecore [officially supports running in containers starting from version 9.3](https://kb.sitecore.net/articles/161310 "officially supports running in containers starting from version 9.3"), but official Docker images are only provided starting from Sitecore 10.

Therefore, there are 3 possible scenarios:

### a) You are running on Sitecore 9.3

You&#8217;re not getting official Docker images from Sitecore, but you can build them yourself from https://github.com/Sitecore/docker-images  
There are quite some blog posts and videos in community on how to do that, for example: 

  * [Sitecore Docker Images Repository](https://www.youtube.com/watch?v=cA1CMdwrNVU "Sitecore Docker Images Repository")
  * [Introduction to using Sitecore Docker Images](https://www.youtube.com/watch?v=SRsCgyPeCTg "Introduction to using Sitecore Docker Images")

### b) You are running Sitecore 10

This is the best-case scenario. Sitecore provides both official support and Docker images for you off the shelf.

In addition to this, there is a great documentation on how to get started: https://doc.sitecore.com/xp/en/developers/101/developer-tools/containers-in-sitecore-development.html accompanied with examples: https://github.com/Sitecore/docker-examples

### c) You&#8217;re running pre-Sitecore 9.3 version

Not only this setup is not officially supported, but also old Sitecore versions are lacking some features which simplify Docker setup. Sitecore 9.3 is a [much better Docker citizen](/sitecore-9-3-loves-docker/ "much better Docker citizen").

On top of that, [community Sitecore docker images repository](https://github.com/Sitecore/docker-images "community Sitecore docker images repository") does not provide images for Sitecore versions lower than 9.0.2, so you will have to build them yourself.

Therefore, I highly recommend you to upgrade, preferably straight to 10. Not that you cannot dockerize pre-9.0.2 versions, it is just that it would require much more effort.

If you decided to go for an upgrade &#8211; do it first before taking any steps related to Docker migration. This will help to avoid Big Bang scenario when you are both working on Sitecore upgrade and Docker migration.

## 2. Windows container base OS version

![Windows container version to use with Sitecore](windows-logo.png#center "Windows container version to use with Sitecore")

At the time of writing, Sitecore officially supports only the ltsc2019 base container OS version on production. This is also the version which most cloud providers currently support in their managed Kubernetes services.  
So this would be the best option for now. We are running on Windows 1909, but that is because our DevOps team likes to be on the edge 😎

Even though for Production setup it is best to use ltsc2019 version, for your local development you can use newer versions. This won&#8217;t give you much benefit, but will allow to use newer Windows versions in combination with [process isolation mode](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/hyperv-container "process isolation mode").

## 3. Local development environment setup

### Performance

For the best performance (which is especially relevant for Sitecore) you need to have your local Windows version matching the version of base OS Docker images. This is needed to be able to run containers in [process isolation mode](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/hyperv-container "process isolation mode"). For example, if you plan to use 1809 (ltsc) base images, then make sure your windows version is Windows 10 1809 as well.

With process isolation I have not noticed any performance degradation in comparison to non-Docker setup. However, based on my experience, **hyper-v isolation has proven to be much, much slower**.

Docker should not eat a lot of extra RAM comparing to your previous local setup, but the more the better. [Sitecore recommends 32GB RAM](https://doc.sitecore.com/xp/en/developers/102/developer-tools/set-up-the-environment.html "Sitecore recommends 32GB RAM") for developer workstations, with 16GB minimum.

#### Extra Tips
 1. Add `C:\ProgramData\Docker\containers` to your antivirus exclusion list, or you might experience some [nasty locking issues](https://docs.docker.com/engine/security/antivirus/ "nasty locking issues").
 2. Make sure your local IIS sites do not collide with your Docker sites (hostnames, ports). Eventually you might disable your local IIS instance completely.

### Development workflow

![Local development workflow with Docker](https://docs.microsoft.com/en-us/dotnet/architecture/containerized-lifecycle/design-develop-containerized-apps/media/docker-apps-inner-loop-workflow/life-cycle-containerized-apps-docker-cli.png#center "Local development workflow with Docker")

With introduction of containers, your local development workflow will change as well:

  * Publishing solution changes will work [slightly different](https://doc.sitecore.com/xp/en/developers/100/developer-tools/deploying-files-into-running-containers.html "slightly different")
  * [Debugging code has it's specifics](https://doc.sitecore.com/xp/en/developers/100/developer-tools/debug-code-running-in-containers.html "Debugging code has it's specifics")
  * Syncing your serialized items will [require some extra configuration](https://doc.sitecore.com/xp/en/developers/100/developer-tools/sync-items-with-a-running-container.html "require some extra configuration")
  * You will need to get used to some tooling to manage containers, such as Docker CLI, VS Code, etc

For a great general overview of containerized development lifecycle check this guide from Microsoft: https://docs.microsoft.com/en-us/dotnet/architecture/containerized-lifecycle/

### Solution setup

For inspiration on how to setup your Git repo and Visual Studio solution you can check the following examples provided by Sitecore:

  * https://github.com/Sitecore/docker-examples
  * https://github.com/Sitecore/MVP-Site &#8211; open-sourced MVP site https://mvp.sitecore.com/
  * https://github.com/Sitecore/Helix.Examples &#8211; the main purpose of this repo is Helix, but it has some examples of Sitecore 10 solution setup with containers.

And here you can find a nice video, which walks you through the MVP site setup: https://www.youtube.com/watch?v=vllu8xKfzQU

## 4. Container orchestrator

![Kubernetes container orchestrator for Sitecore](kubernetes-logo.png#center "Kubernetes container orchestrator for Sitecore")

If you plan to use Docker for your DTAP environments, you will need some sort of container orchestrator.

Sitecore provides [some examples on Kubernetes setup](https://github.com/Sitecore/MVP-Site/tree/master/k8s "some examples on Kubernetes setup"). K8S is also supported as a managed service by most major cloud providers, therefore this is a no-brainer to choose it for your production setup.

You might consider a simpler alternative like Azure Container Instances or AWS Elastic Container Service, but then you are less flexible and more dependent on the specific cloud implementation (which is not bad per se).  
In my case our DevOps team has extensive K8S experience, so it was an easy choice for us to go for AWS EKS.

**Tip:** I do not recommend running Kubernetes locally, since it only adds complexity without not much benefit. Docker Compose is good enough for local development setup.

## 5. Cloud provider to run your containers

![Cloud provider for Sitecore Docker containers](cloud-providers.png#center "Cloud provider for Sitecore Docker containers")

**Note:** If you plan to use Docker only for local development setup, then this is obviously not relevant. However, if you have decided to go all the way to Production then this is important choice to make.

If you are already running in the cloud, then the choice is made. However, if you are running on-premises, this is something you will need to decide.

Comparison between cloud providers is certainly a huge topic and it is impossible to cover it within a single post, but I will outline some tips specific to Sitecore.

Both Azure and AWS are suitable for running Sitecore in containers. They both provide managed Kubernetes services and other container hosting options. Google Cloud also supports running Windows containers, however this would be quite an exotic choice for Sitecore, therefore I did not look into this.

Besides containers you need to consider other system components:

|                  | Azure                                                                                                                                                                              | AWS                                                                              | 
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| **SQL**             | SQL Azure, [officially supported by Sitecore](https://kb.sitecore.net/articles/777703#:~:text=Sitecore%20XP%2FCMS%20supports%20Microsoft,using%20the%20Sitecore%20Azure%20module.) | RDS, [officially supported by Sitecore](https://kb.sitecore.net/articles/013205) |
| **Solr**             | Self-hosted or [SearchStax](https://www.searchstax.com/docs/sitecore-9-solr/)                                                                                                      | Self-hosted or [SearchStax](https://www.searchstax.com/docs/sitecore-9-solr/)    |
| **Redis (optional)** | Azure Cache for Redis                                                                                                                                                              | Amazon ElastiCache for Redis                                                     |


**Note:** I am not including Azure Search as an alternative indexing provider because [in Sitecore 10 it has been officially deprecated](https://dev.sitecore.net/Downloads/Sitecore%20Experience%20Platform/100/Sitecore%20Experience%20Platform%20100/Release%20Notes "in Sitecore 10 it has been officially deprecated").

Depending on your solution, you have to check what other system components you need to deploy and which cloud provider is the most suitable.

If in doubt, I think it is best to go for Azure. The reason is that Sitecore is more focused on Azure and this is what they use themselves for [their demo solutions](https://github.com/Sitecore/MVP-Site "their demo solutions").

## 6. Making Sitecore a good Docker citizen

![Sitecore and Docker](docker-sitecore.png#center "Sitecore and Docker")

I have already blogged about this topic before. So as not to repeat myself: 

  * [Making Sitecore a good Docker citizen](/making-sitecore-a-good-docker-citizen/ "Making Sitecore a good Docker citizen")
  * [Sitecore 9.3 loves Docker](/sitecore-9-3-loves-docker/ "Sitecore 9.3 loves Docker")

In short: Sitecore is almost there to be a good Docker citizen. In my opinion the only thing (but a very important one) it lacks is configuration via environment variables. 

Starting from version 9.3, Sitecore is partially configurable via environment variables (namely AppSettings and ConnectionStrings sections of web.config). However, to be able to modify other parts of Sitecore configuration via environment variables, you have to implement a custom solution: [Sitecore environment variables Config Builder](/sitecore-environment-variables-config-builder/ "Sitecore environment variables Config Builder")

I have registered a feature request to Sitecore via several channels so I hope they will support this out of the box in future versions. Please upvote this idea on https://sitecore-product.ideas.aha.io/ideas/SXP-I-76

## 7. Prepare for CI/CD revamp

![Sitecore DevOps with Docker](devops.png#center "Sitecore DevOps with Docker")

Obviously, your CI/CD process will have to change, together with the mindset.

  * With Docker, you no longer deploy just your application code. Container is now the unit of deployment, which includes the whole environment: OS and application dependencies.
  * Therefore your build process will have to be extended to build Docker containers and push them to container registry. **Important:** make sure to use a private registry, using public registry to store Sitecore images is **not allowed**.
  * Kubernetes is not something you will learn over one day.

There is quite some information on how to build your custom images in Sitecore documentation: https://doc.sitecore.com/xp/en/developers/102/developer-tools/building-custom-sitecore-images.html

On top of that, Microsoft provides a great overview of both inner and outer loops of containerized development life cycles here: https://docs.microsoft.com/en-us/dotnet/architecture/containerized-lifecycle/docker-devops-workflow/docker-application-outer-loop-devops-workflow

## 8. Application Monitoring (Health Checks)

![Sitecore health checks](health-checks.png#center "Sitecore health checks")

Of course, application health monitoring is not something unique to Docker, but there are some specifics. For example, in K8S you can [setup liveness and readiness probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/ "setup liveness and readiness probes") for your pods. Luckily, Sitecore comes with health check endpoints &nbsp;(`/healthz/live&nbsp;`and&nbsp;`/healthz/ready`) out of the box, which you can use for this purpose.

I have blogged before about [health checks in Sitecore 9.3 and on how to add a custom health check](/sitecore-9-3-health-checks/ "health checks in Sitecore 9.3 and on how to add a custom health check") in details, so I will not repeat myself here.

## 9. Centralized logging

If you have a scaled Sitecore setup, you probably already have some centralized logging solution in place (Datadog, Logz.io, etc). If not &#8211; you definitely should look into that.

However, here it is worth mentioning that in Docker world it is considered to be the best practice when your containers output logs to Docker console. This is a very powerful technique, which:

  1. For local development gives you live tail of Sitecore logs. Before Docker times you had to either check .txt files manually or use some additional tooling.
  2. In DTAP environments allows to decouple your application from specifics of the centralized logging solution. Tools like Datadog would listen to the console output of your Docker containers and forward them to their servers.

And Sitecore as a [good Docker citizen](/making-sitecore-a-good-docker-citizen/ "good Docker citizen") streams its logs to a console as well by using Filebeat (relevant for images provided in https://github.com/Sitecore/docker-images) and Sitecore 10 images.

## 10. Containers are immutable

The last but not the least, you have to get used to Docker containers dogma: they are immutable. Which means that any changes to file system within container will be lost as soon as container restarts. So no more hotfixing dlls directly on PROD servers 🙂

Any change you want to persist to your Docker images (e.g. [install a Sitecore module](https://doc.sitecore.com/xp/en/developers/100/developer-tools/add-sitecore-modules.html "install a Sitecore module")) would have to be done as a part of your CI/CD process when you build your Docker images.

If you store your media on File System that would be a problem, because as soon as container is recreated you will lose your data. Therefore, analyze your solution and check that your are not relying on local file system of your servers. And if you do &#8211; you will have to use shared storage instead.

**Note:** In addition to the above, file-based media storage in Sitecore is deprecated since 9.3 and will be removed in future.

## Summary

Looking back, I am very happy that we have decided to migrate our solution to Docker setup. Not to mention all the benefits, it was fun. However make sure to analyze all the pros/cons and specifics of your organization to see if it is a good fit for you. Because as always, it depends 😉