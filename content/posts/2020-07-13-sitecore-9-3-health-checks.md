---
title: Sitecore 9.3 health checks
date: 2020-07-13T14:08:54+00:00
summary: Starting from version 9.3, Sitecore includes health check HTTP endpoints which you can use to check health of your Sitecore instance. In this post I will show how to add custom health checks.
description: Sitecore health monitoring. How to implement a custom health check for Sitecore?
url: /sitecore-9-3-health-checks/
featured_image: /wp-content/uploads/2020/07/health-checks-461x430.png
categories:
  - DevOps
  - Sitecore
tags:
  - Sitecore

---
<div class="wp-block-image">
  <figure class="aligncenter size-medium"><img loading="lazy" width="300" height="280" src="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/07/health-checks.png?resize=300%2C280&#038;ssl=1" alt="How to create Sitecore 9.3 custom health checks" class="wp-image-1127" srcset="https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/07/health-checks.png?resize=300%2C280&ssl=1 300w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/07/health-checks.png?resize=461%2C430&ssl=1 461w, https://i0.wp.com/blog.vitaliitylyk.com/wp-content/uploads/2020/07/health-checks.png?w=705&ssl=1 705w" sizes="(max-width: 300px) 100vw, 300px" data-recalc-dims="1" /></figure>
</div>

Starting from version 9.3, Sitecore includes health check HTTP endpoints which you can use to check health of your Sitecore instance. In this post I will show how to add custom health checks.

I have already described [built-in health checks][1] in this post before: <a rel="noreferrer noopener" href="https://blog.vitaliitylyk.com/sitecore-9-3-loves-docker/" target="_blank">Sitecore 9.3 loves Docker</a>. As a short recap, Sitecore 9.3 health checks are based on <a href="https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks?view=aspnetcore-3.1" target="_blank" rel="noreferrer noopener">Microsoft.Extensions.Diagnostics.HealthChecks</a> with some sauce on top:

  * Since Sitecore is not based on ASP.NET Core, it cannot use health checks middleware to expose liveness and readiness endpoints (`/healthz/live` and `/healthz/ready`). Instead, it is using a regular Web API controller for this: `Sitecore.Diagnostics.HealthCheckController`
  * Sitecore is exposing custom extension methods for `Microsoft.Extensions.DependencyInjection.IServiceCollection` to simplify dependency registration.

## Monitoring health

I think it is obvious that you need to monitor health of your application if you want to minimize downtime and react on failures quickly. If you have mature DevOps culture in your organization, you will likely have some infrastructure/tooling for monitoring application health, logs and alerting your DevOps team if something is wrong. <a rel="noreferrer noopener" href="https://www.datadoghq.com/" target="_blank">Datadog</a> is one example of such tooling.

In case you have this infrastructure already in place, all you have to do is to point it to your Sitecore health check endpoints (`/healthz/live` and `/healthz/ready`) and set up alerts. In my case I use these endpoints for <a rel="noreferrer noopener" href="https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/" target="_blank">Kubernetes liveness and readiness probes</a>.

<blockquote class="wp-block-quote">
  <p>
    <strong>Note: </strong>XConnect, Identity and Publishing Service also expose liveness and readiness HTTP endpoints (<code>/healthz/live </code>and <code>/healthz/ready</code>).
  </p>
</blockquote>

If you don&#8217;t have such a setup, have a look at the Advanced Sitecore Healthcheck module by Mihály Árvai (@mitya_1988): <a rel="noreferrer noopener" href="https://github.com/Mitya88/SitecoreHealthcheck" target="_blank">https://github.com/Mitya88/SitecoreHealthcheck</a>. It includes nice SPEAK 3 dashboards, which would allow you to monitor health of your Sitecore instances (including XConnect) from a central location. Not to mention tons of pre-built health checks.

<blockquote class="wp-block-quote">
  <p>
    <strong>Note:</strong> <em>This module is not based on top of Sitecore Health checks (and <a rel="noreferrer noopener" href="https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks?view=aspnetcore-3.1" target="_blank">Microsoft.Extensions.Diagnostics.HealthChecks</a>) under the hood and does not expose any HTTP endpoints (as far as I know). </em>It was built with a different purpose in mind: monitoring Sitecore health from within Sitecore UI.
  </p>
  
  <p>
    <strong>Update (17.07.2020</strong>): it is now possible to <a rel="noreferrer noopener" href="https://medium.com/@mitya_1988/exposing-advanced-sitecore-health-check-result-b537b6089f0a" target="_blank">expose the module&#8217;s health check result in Sitecore health HTTP endpoint</a>. Good job Mihály!
  </p>
</blockquote>

## Adding custom health checks

By default Sitecore health checks include SQL and SOLR, which is normally enough for most of the cases. However, when your application is more complex you might consider adding custom health checks. 

For example, in my project I use readiness check to warmup some of the APIs and make sure they return 200OK. This becomes very useful during our Kubernetes deployment: in case one of the APIs is unhealthy, the readiness probe fails which will prevent pods from receiving the traffic. And if it is healthy &#8211; the instance would be warmed up before receiving any traffic, which results in smooth deployment experience.

Below are the steps needed to implement such a simple health check.

### 1. Implement custom health check

Implement a class inheriting from `Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck`. This interface resides in `Microsoft.Extensions.Diagnostics.HealthChecks.Abstractions` Nuget package (version `2.2.0.0` for Sitecore 9.3).<figure class="wp-block-embed">

<div class="wp-block-embed__wrapper">
  <div class="gist-oembed" data-gist="910c89b13c37fa8052a822a35ecfe05b.json" data-ts="8">
  </div>
</div></figure> 

### 2. Register a custom health check

In your DI registration import the `Sitecore.HealthCheck.DependencyInjection` namespace and use the `AddHealthChecks` extension method from Sitecore to register your health check class:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp" data-enlighter-theme="" data-enlighter-highlight="" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-title="" data-enlighter-group="">services.AddHealthChecks().AddCheck&lt;ApiHealthCheck>(
	"API Smoke Test",
	HealthStatus.Unhealthy,
	new[] { "ready" });</pre>

<blockquote class="wp-block-quote">
  <p>
    <strong>Note:</strong> the last argument is an array of tags. You can tag your health checks with <code>ready</code> and/or <code>live</code> tags, which would enlist your custom healthcheck to be used in <code>/healthz/ready</code> and/or <code>/healthz/live</code> HTTP endpoints respectively.
  </p>
</blockquote>

## Summary

All Sitecore roles, including Identity, XConnect and Publishing Service expose liveness and readiness HTTP endpoints which you can use to monitor health of your Sitecore deployment. If needed, you can implement custom health checks on top of the default ones.

That&#8217;s it, stay healthy out there!

 [1]: https://doc.sitecore.com/developers/93/platform-administration-and-architecture/en/monitoring-the-health-of-web-roles.html