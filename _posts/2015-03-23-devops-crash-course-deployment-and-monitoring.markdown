---
layout: post
title: "DevOps Crash Course: Deployment and Monitoring"
author: freddyrangel
modified:
categories:
comments: true
excerpt: "The basics of DevOps developers need to know"
tags: []
image:
  feature:
date: 2015-03-23T23:24:10-07:00
---


<p>
  DevOps is a huge and complicated topic, so I tried to focus on just the things
  that are most relevant for developers right now. Nevertheless, I decided to
  split up this post into several posts because this topic is just so massive.
</p>

### Continuos Integration and Continuous Deployment

<p>
  Our first priority when getting ready to deploy a new app is setting up the
  necessary infrastructure. Once, it used to be the case where setting up your
  infrastructure, even for a trivial app, required quite a bit of configuration
  and fine tuning. Now, just like you don't have to hire an accountant when all
  you need is Quickbooks, we can take advantage of a Platform as a Service like
  Heroku to manage all those problems while we focus on performance and
  development. At some point we may need to establish our own infrastructure, but
  I'll lay out how we can drastically improve performance so we can stay on Heroku
  as long as possible.
</p>

#### Heroku will handle:

* Horizontal scaling
* Component-level performance tuning
* Infrastructure-level security

#### What we still have to worry about:

* Minimizing load on database
* Application-level performance tuning (caching)
* Application-level security

<p>
  Before SaaS, software releases were a major and infrequent event. In SaaS and
  Agile, deployments are a non-event. Amazon has several deploys per week; GitHub
  deploys dozens of times per day. Typically, deployment is automated using GitHub
  hooks and tools like Capistrano for self-hosted sites.
</p>

<p>
  Continuous Integration is essentially running all your integration tests before
  a release is green lighted to deployment. This is usually done in a staging
  environment. This will include tests not usually done in deployment, such as
  security, cross-browser / cross-version compatibility, and stress testing.
</p>

<p>
  There are a lot of services out there that do this for you. I like CircleCI just
  because I'm familiar with it, but there are others that are just as good in my
  opinion.
</p>

### Performance Monitoring

#### Performance Terms

* Availability or Uptime: What % of time is site up & available?
* Responsiveness: How long after a click does a user get a response?
* Scalability: as # users increases, can you maintain responsiveness without
increasing cost/user?
* Latency, the delay between an input and response, is the most difficult
performance challenge of a SaaS app. The backend database if often the reason an
app has to abandon a PaaS solution. WIth caching and other techniques, we can
stay on PaaS much longer.

<p>
  Speed is a feature. How important is response time? At Amazon, a +100ms increase
  in response time leads to a 1% drop in sales. At Google, a +500ms increase leads
  to 20% fewer searches.
</p>

<p>
  Genarally, responsiveness Within 100ms perceived as instantaneous. Within 1
  second, user perceives cause and effect but will fell the action to be sluggish.
  After 8 seconds, the user's attention drifts to other things. This is the
  abandonment time (8-second-rule). New Relic reports that the average page load
  is 5.3 seconds.
</p>

<p>
  Performance is not normally distributed. Meaning, just because the average
  latency time is at say, 2 seconds, doesn't mean that 95% of users are within 2
  SD. It actually looks more like this:
</p>
<br />

![dataset](/../images/localhost_9292_datasets_histogram.jpg)
Source: http://blog.newrelic.com/2013/09/10/breaking-down-apdex/

<br />

<p>
  The mean here can be misleading. The green line represents the average response
  time (roughly 2.5 seconds). That statistic would seem great if this were
  normally distributed. However,the graph shows over 25% of requests are over 7
  seconds, nearing the abandonment range. 3.6% of requests are simply off the
  chart. The reality is that even though 2.5 second average sounds great, the app
  is actually performing very poorly.
</p>

<p>
  How to set proper performance goals? Rather than focusing on the mean, set goals
  for what % of users get acceptable performance. e.g., 99% of users get a
  response time less than 1 second, over a 5 minute window.
</p>

<p>
  Here are a list of services and tool I recommend we implement for deployment:
</p>

#### Services

* Heroku: makes managing infrastructure much easier
* New Relic: for performance monitoring
* CircleCI: for making deployment and continuous integration easier
* Honeybadger: for error tracking

#### Tools

* Google Analytics: for tracking business analytics
* Bullet: helps find abusive queries
* Unicorn: a pretty good web server. Puma is good too.
* Delayed Job: extracted from Shopify, a queue priority system
* Rack-timeout: aborts requests that are taking longer that 15 seconds
* Rails_indexes: helps find missing database indexes

