---
layout: post
title: "How to use an external log4j.properties in you Grails project"
permalink: /how-to-use-an-external-log4jproperties-in-you-0
date: 2010-09-30 22:33:52 -0500
comments: true
categories: [grails, spring]
---

In a recent Grails project, I had to follow some corporate guidelines regarding application deployment, and one of 
those were that the log4j configuration for an application must be externalized in a properties file.

I searched for a [Grails plugin](http://grails.org/plugin/home) that could help me with this, with no luck. 
Then I remembered that a Grails application is just a 
[Spring application in disguise](http://blog.springsource.com/2010/06/08/spring-the-foundation-for-grails/), 
so I looked for a Spring way to do this.

There are at least two ways to do this using Spring: 
[Log4jConfigListener](http://static.springsource.org/spring/docs/3.0.x/javadoc-api/org/springframework/web/util/Log4jConfigListener.html) 
and 
[Log4jConfigurer](http://static.springsource.org/spring/docs/3.0.x/javadoc-api/org/springframework/util/Log4jConfigurer.html). 
I chose the later because the former assumes an expanded WAR file, which was not my case.

Here’s the recipe I came up with:
<!-- more -->

* Configure a Log4jConfigurer Spring bean in your `grails-app/conf/resources.groovy`: 
(click [here for a xml version](https://gist.github.com/deluan/605359#file-resources-xml))

{% gist 605359 resources.groovy %}

* Install the templates in your project with grails install-templates, so you can change some files used for Grails' 
code generation. The one we are interested in is the web.xml template.

* Comment out the Grails' Log4jConfigListener from the `src/templates/war/web.xml` template:

{% gist 605359 web.xml %}

* You can (and should) remove the log4j configuration block from your Config.groovy

* That’s it!

This was tested with Grails 1.3.3, deploying to an Oracle WebLogic 10.3.0 container.