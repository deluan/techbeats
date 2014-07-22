---
layout: post
title: "Profiling web requests in your Grails application"
permalink: /profiling-web-requests-in-grails-application
date: 2010-12-29 01:11:39 -0500
comments: true
categories: [grails]
---

Here’s a simple filter I’ve been using to help me detect points of improvement in my application:

{% gist 744828 %}

To use it, put this class in the `grails-app/conf` folder of your project. To activate the profile, call any URL 
of your application with the `showTime=on` parameter, like this:
<!-- more -->

```
http://localhost:8080/my-grails-app/book/list?showTime=on
```

After calling that URL, all request times will be measured and informed on the application’s log, like this:

```
2010-12-21 12:02:31,698 [http-8080-5] DEBUG filters.UtilFilters  - Request duration for (book/list): 20ms/50ms
```

The first time informed (20ms) is the time spent executing the action (list in this case) and the second (50ms) is 
the time spent rendering the view (`list.gsp`).

To turn off the profiler, call any URL with `showTime=off`:

```
http://localhost:8080/my-grails-app?showTime=on
```

Enjoy :)