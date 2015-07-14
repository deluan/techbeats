---
layout: post
title: "Using Cache-Headers plugin in a non-english server"
date: 2010-12-21 01:03:35 -0500
comments: true
categories: [grails]
---

**UPDATE: This problem has been fixed in version 1.1.3 of the plugin. Thanks Marc and Luke Daley!**

This week I tried the [Marc Palmer](http://www.anyware.co.uk/2005/)’s excelent plugin
[Cache-Headers](http://www.grails.org/plugin/cache-headers), and it really rocks! Using it I can make all my
server-side generated images be cached on the client, reducing significantly the bandwidth and cpu-power necessary by
my application.

But there’s a little gotcha: The plugin (as of version 1.1.2) uses a SimpleDateFormat to generate and check the
Last-Modified header, and the implementation creates this SimpleDateFormat with the system’s default Locale, in my
case Portuguese. This causes errors like this:
<!-- more -->

```
java.lang.IllegalArgumentException: Ter, 21 dez 2010 15:10:33 GMT
    at com.grailsrocks.cacheheaders.CacheHeadersService.withCacheHeaders(CacheHeadersService.groovy:140)
    at com.grailsrocks.cacheheaders.CacheHeadersService$withCacheHeaders.call(Unknown Source)
    at CacheHeadersGrailsPlugin$_addCacheMethods_closure7_closure11.doCall(CacheHeadersGrailsPlugin.groovy:61)
    at org.weceem.controllers.WcmContentController$_closure3.doCall(WcmContentController.groovy:172)
    at org.weceem.controllers.WcmContentController$_closure3.doCall(WcmContentController.groovy)
    at org.apache.shiro.web.servlet.ShiroFilter.executeChain(ShiroFilter.java:687)
    at org.apache.shiro.web.servlet.ShiroFilter.doFilterInternal(ShiroFilter.java:616)
    at org.apache.shiro.web.servlet.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:81)
```

The workaround I found was to override the construction of that SimpleDateFormat. Luckily it’s been created in a small
class, that is easy to extend:

{% gist 749785 %}

All you have to do is put this class in your project (in the `src/groovy` folder) and add the following configuration
override in your `Config.groovy`:

{% codeblock lang:groovy %}
beans {
    cacheHeadersService {
        dateFormatters = new com.deluan.grails.util.EnglishDateFormatterThreadLocal()
    }
}
{% endcodeblock %}

I already filled a bug report here: http://jira.codehaus.org/browse/GRAILSPLUGINS-2707, and probably this post will be
obsolete in a near future :)
