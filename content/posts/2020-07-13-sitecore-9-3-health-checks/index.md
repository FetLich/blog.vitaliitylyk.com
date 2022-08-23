---
title: Sitecore 9.3 health checks
date: 2020-07-13T14:08:54+00:00
summary: Starting from version 9.3, Sitecore includes health check HTTP endpoints which you can use to check health of your Sitecore instance. In this post I will show how to add custom health checks.
description: Sitecore health monitoring. How to implement a custom health check for Sitecore?
url: /sitecore-9-3-health-checks/
cover:
  image: "health-checks.png"
categories:
  - DevOps
  - Sitecore
tags:
  - Sitecore

---
![How to create Sitecore 9.3 custom health checks](health-checks.png#center "How to create Sitecore 9.3 custom health checks")

Starting from version 9.3, Sitecore includes health check HTTP endpoints which you can use to check health of your Sitecore instance. In this post I will show how to add custom health checks.

I have already described [built-in health checks](https://doc.sitecore.com/developers/93/platform-administration-and-architecture/en/monitoring-the-health-of-web-roles.html) in this post before: [Sitecore 9.3 loves Docker](/sitecore-9-3-loves-docker/ "Sitecore 9.3 loves Docker"). As a short recap, Sitecore 9.3 health checks are based on [Microsoft.Extensions.Diagnostics.HealthChecks](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks?view=aspnetcore-3.1 "Microsoft.Extensions.Diagnostics.HealthChecks") with some sauce on top:

  * Since Sitecore is not based on ASP.NET Core, it cannot use health checks middleware to expose liveness and readiness endpoints (`/healthz/live` and `/healthz/ready`). Instead, it is using a regular Web API controller for this: `Sitecore.Diagnostics.HealthCheckController`
  * Sitecore is exposing custom extension methods for `Microsoft.Extensions.DependencyInjection.IServiceCollection` to simplify dependency registration.

## Monitoring health

I think it is obvious that you need to monitor health of your application if you want to minimize downtime and react on failures quickly. If you have mature DevOps culture in your organization, you will likely have some infrastructure/tooling for monitoring application health, logs and alerting your DevOps team if something is wrong. [Datadog](https://www.datadoghq.com/ "Datadog") is one example of such tooling.

In case you have this infrastructure already in place, all you have to do is to point it to your Sitecore health check endpoints (`/healthz/live` and `/healthz/ready`) and set up alerts. In my case I use these endpoints for [Kubernetes liveness and readiness probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/ "Kubernetes liveness and readiness probes").


**Note:** XConnect, Identity and Publishing Service also expose liveness and readiness HTTP endpoints (`/healthz/live` and `/healthz/ready`).

If you don&#8217;t have such a setup, have a look at the Advanced Sitecore Healthcheck module by Mihály Árvai (@mitya_1988): https://github.com/Mitya88/SitecoreHealthcheck. It includes nice SPEAK 3 dashboards, which would allow you to monitor health of your Sitecore instances (including XConnect) from a central location. Not to mention tons of pre-built health checks.


> **Note:** <em>This module is not based on top of Sitecore Health checks (and [Microsoft.Extensions.Diagnostics.HealthChecks](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks?view=aspnetcore-3.1 "Microsoft.Extensions.Diagnostics.HealthChecks")) under the hood and does not expose any HTTP endpoints (as far as I know). </em>It was built with a different purpose in mind: monitoring Sitecore health from within Sitecore UI.
> 
> **Update (17.07.2020**): it is now possible to [expose the module's health check result in Sitecore health HTTP endpoint](https://medium.com/@mitya_1988/exposing-advanced-sitecore-health-check-result-b537b6089f0a "expose the module's health check result in Sitecore health HTTP endpoint"). Good job Mihály!

## Adding custom health checks

By default Sitecore health checks include SQL and SOLR, which is normally enough for most of the cases. However, when your application is more complex you might consider adding custom health checks. 

For example, in my project I use readiness check to warmup some of the APIs and make sure they return 200OK. This becomes very useful during our Kubernetes deployment: in case one of the APIs is unhealthy, the readiness probe fails which will prevent pods from receiving the traffic. And if it is healthy &#8211; the instance would be warmed up before receiving any traffic, which results in smooth deployment experience.

Below are the steps needed to implement such a simple health check.

### 1. Implement custom health check

Implement a class inheriting from `Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck`. This interface resides in `Microsoft.Extensions.Diagnostics.HealthChecks.Abstractions` Nuget package (version `2.2.0.0` for Sitecore 9.3).

{{< gist vitaliitylyk 910c89b13c37fa8052a822a35ecfe05b >}}

### 2. Register a custom health check

In your DI registration import the `Sitecore.HealthCheck.DependencyInjection` namespace and use the `AddHealthChecks` extension method from Sitecore to register your health check class:

```csharp
services.AddHealthChecks().AddCheck<ApiHealthCheck>(
	"API Smoke Test",
	HealthStatus.Unhealthy,
	new[] { "ready" });
```

**Note:** the last argument is an array of tags. You can tag your health checks with `ready` and/or `live` tags, which would enlist your custom healthcheck to be used in `/healthz/ready` and/or `/healthz/live` HTTP endpoints respectively.


## Summary

All Sitecore roles, including Identity, XConnect and Publishing Service expose liveness and readiness HTTP endpoints which you can use to monitor health of your Sitecore deployment. If needed, you can implement custom health checks on top of the default ones.

That&#8217;s it, stay healthy out there!