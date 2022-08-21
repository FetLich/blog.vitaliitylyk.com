---
title: Sitecore 9.3 loves Docker
date: 2020-01-17T13:58:21+00:00
summary: Sitecore 9.3 release includes some features which improve containers support, but at the time of writing they are not mentioned neither in release notes nor in documentation. In this post I highlight these features, since I consider them an important step in direction of containers support.
url: /sitecore-9-3-loves-docker/
featured_image: /wp-content/uploads/2020/01/sitecore-93-loves-docker-740x226.png
categories:
  - DevOps
  - Sitecore
tags:
  - DevOps
  - Docker
  - Sitecore

---
![Sitecore 9.3 loves Docker containers](sitecore-93-loves-docker.png#center "Sitecore 9.3 loves Docker containers")

Sitecore 9.3 release includes some features which improve containers support, but at the time of writing they are not mentioned neither in [release notes](https://dev.sitecore.net/Downloads/Sitecore%20Experience%20Platform/93/Sitecore%20Experience%20Platform%2093%20Initial%20Release/Release%20Notes "release notes") nor in documentation. In this post I highlight these features, since I consider them an important step in direction of containers support.

## Configuration via environment variables

Some time ago I wrote about [making Sitecore a good Docker citizen](/making-sitecore-a-good-docker-citizen/ "making Sitecore a good Docker citizen"). As one of the points, I&#8217;ve mentioned that a good Docker citizen application can be configured via environment variables.

Sitecore 9.3 comes in with pre-configured [configuration builders](https://docs.microsoft.com/en-us/aspnet/config-builder "configuration builders"), making it possible to set `appSettings` and `connectionStrings` via environment variables out of the box. Here is how configuration looks like in 9.3 web.config:

```
<configBuilders>
    <builders>
      <add name="SitecoreAppSettingsBuilder" mode="Strict" prefix="SITECORE_APPSETTINGS_" stripPrefix="true"
        type="Microsoft.Configuration.ConfigurationBuilders.EnvironmentConfigBuilder, Microsoft.Configuration.ConfigurationBuilders.Environment, Version=1.0.0.0, Culture=neutral"/>
      <add name="SitecoreConnectionStringsBuilder" mode="Strict" prefix="SITECORE_CONNECTIONSTRINGS_" stripPrefix="true"
        type="Microsoft.Configuration.ConfigurationBuilders.EnvironmentConfigBuilder, Microsoft.Configuration.ConfigurationBuilders.Environment, Version=1.0.0.0, Culture=neutral"/>
    </builders>
  </configBuilders>
```

So for example when you set `SITECORE_CONNECTIONSTRINGS_WEB` environment variable for your Docker container, Sitecore uses it as a connection string for your web database. To set an application setting, you use `SITECORE_APPSETTINGS_` prefix, for example: `SITECORE_APPSETTINGS_ROLE:DEFINE`. You can find more examples of possible values in docker-images repository: https://github.com/Sitecore/docker-images/blob/0f8039719d80fc95a9c81b846707b73fb1055aaf/build/windows/tests/9.3.x/docker-compose.xm.yml

### License as environment variable

In addition to the above, starting from Sitecore 9.3 you can set your license via an environment variable `SITECORE_LICENSE`. The value us expected to be gzipped base64 encoded version of your license file. Here you can find an example PowerShell script, which converts a physical license file to it&#8217;s environment variable version: https://github.com/Sitecore/docker-images/blob/0f8039719d80fc95a9c81b846707b73fb1055aaf/build/Set-LicenseEnvironmentVariable.ps1

## Health checks

Health checks are of course important in non-Docker context too, however with Docker and Kubernetes proper health check setup is a must-have. You are expected to provide a way to Docker to determine if your application is healthy and if not &#8211; it will be restarted.

Kubernetes provides a more advanced mechanism for health checks: [probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/ "probes").

**Liveness** probe is used by K8S to know when to restart a container. If application is unresponsive (e.g it is out of memory, deadlocked, or for some reason unable to handle new connections), it will be restarted, which gives a chance to fix an issue.

**Readiness** probe is used to decide when container is ready to accept traffic. If readiness probe fails, Kubernetes will not make a pod available to handle requests and will remove it from load balancers. The readiness probe is typically used to check if application is able to connect to all its dependencies (to eliminate configuration errors, e.g wrong connection strings) or to warm up caches.

While this is a powerful tool, you have to [be careful not to shoot yourself in the foot with K8S probes](https://blog.colinbreck.com/kubernetes-liveness-and-readiness-probes-how-to-avoid-shooting-yourself-in-the-foot/ "be careful not to shoot yourself in the foot with K8S probes").

Coming back to Sitecore, 9.3 includes health checks internally powered by [Microsoft.AspNetCore.Diagnostics.HealthChecks](https://www.nuget.org/packages/Microsoft.AspNetCore.Diagnostics.HealthChecks "Microsoft.AspNetCore.Diagnostics.HealthChecks"). It is more of an [ASP.NET CORE concept](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks?view=aspnetcore-3.1 "ASP.NET CORE concept"), however since the `Microsoft.AspNetCore.Diagnostics.HealthChecks` package is based on .NET Standard, it can also be used in a fully compliant way with .NET 4.7.2+.

Sitecore comes with 2 endpoints: 

  * `/healthz/live` &#8211; this endpoint simply returns 200 OK status without checking any dependencies which is good enough to be used as a liveness probe.
  * `/healthz/ready` &#8211; this one is more complex: by default it checks if SOLR and SQL databases (master, core, web and security) are reachable. This one is a perfect candidate for K8S readiness probe.

## Summary

With these new features Sitecore 9.3 is a [much better Docker citizen](/making-sitecore-a-good-docker-citizen/ "much better Docker citizen") than previous versions. If you are considering to run Sitecore in Docker I advice to upgrade to 9.3, it will simplify a lot of things. Let&#8217;s hope to see even more Docker love in next versions of Sitecore!