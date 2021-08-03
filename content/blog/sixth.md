+++ 
categories = ["python", "database"]
comments = true
date = "2021-08-02T15:59:13-04:00"
draft = false
showpagemeta = true
showcomments = true
slug = ""
tags = ["python", "database", "engineering", "django"]
title = "Rate Limit With A Recorder"
description = "Have a record of whom has been blocked by your rate limiter"
+++


You all have used rate-limiting in your apps(you know the system to prevent an API from being overwhelmed or being abused!) and if you haven't, I strongly suggest doing so. There is a handy package specifically for Django called django-ratelimit that provides a decorator to rate-limit views. What is not covered by this package, is the ability to see what IPs or users have exceeded the ratelimit. So I have built a proxy and two handlers on top of this package(one for the cache and one for the database backend) which stores the mentioned data and enables you to look it up inside the Django admin panel.

Check it out here:
[django-ratelimit](https://github.com/mohammadrabetian/django-ratelimit)
