---
layout: post
title: "Grape on Rails: Part II"
author: freddyrangel
modified:
categories:
comments: true
excerpt: "Authentication for a Grape JSON API"
tags: []
comments: true
image:
  feature:
date: 2015-04-10T21:57:03-07:00
---

<p>
  In the first part of this tutorial, we talked about how to initially set up a
  Grape application inside of a Rails app. In this post, we're going to go over
  how to get a simple authentication solution setup in a Grape app.
</p>


<p>
  If you're not used to working with JSON APIs, then authentication for an API is
  going to be a little foreign to you. An API should be truly stateless, so we're
  not doing anything involving cookies or sessions. Probably the easiest way to
  handle authorization here is to generate an authorization token every time a
  login request is made and respond with that token. Then, the client will always
  put that token in the HTTP header, which we will then use for authorization.
  There are quite a few other ways, but today we're just going to focus on this
  approach.
</p>
