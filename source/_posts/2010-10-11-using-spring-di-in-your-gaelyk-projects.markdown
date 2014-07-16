---
layout: post
title: "Using Spring DI in your Gaelyk projects"
permalink: /using-spring-di-in-your-gaelyk-projects
date: 2010-10-11 00:07:55 -0500
comments: true
published: false
categories: [gaelyk, groovy, spring]
---

Since I started using [Gaelyk](http://gaelyk.appspot.com/), one of the features I missed most (coming from a Grails 
background) is Spring's dependency injection. Until recently I didn't even know if it was possible to use Spring in 
[Google App Engine](http://appengine.google.com/), so I decided to do a little investigation on the subject, and 
found out that it's very easy indeed.

Here's a little tutorial on how to configure Spring in your Gaelyk project. I'm assuming you have basic knowledge of 
Spring, Gaelyk and Maven.

First, let's create a Gaelyk project. The easiest way is using the excelent 
[maven-gaelyk archetype](http://code.google.com/p/maven-gaelyk/):
```
mvn archetype:generate -DarchetypeGroupId=org.codeconsole -DarchetypeArtifactId=gaelyk-archetype -DarchetypeVersion=0.5.5 -DarchetypeRepository=http://maven-gaelyk.googlecode.com/svn/repository/ -DartifactId=gaelyk-spring -DgroupId=com.deluan.gaelyk -DgaeApplicationName=gaelyk-spring
```
<!-- more -->
Now open the project in your favorite IDE, so we can edit the configuration files. First, add the Spring dependency to your `pom.xml`:

{% gist 618739 pom.xml %}

Next we need to configure Spring's ContextLoaderListener in `web.xml`:

{% gist 618739 web.xml %}

As you can see from above, we configured Spring to load all context configuration files under the directory `WEB-INF/spring`.

With these configurations in place, your project is already Spring enabled. Now we need a easy way to access the 
Spring's Application Context. One good way to do this is using a singleton that implements the ApplicationContextAware 
interface. To keep this post as small as possible, I borrowed an implementation from 
[this blog post](http://sujitpal.blogspot.com/2007/03/accessing-spring-beans-from-legacy-code.html), where you can learn 
more about the details. Create the directory `src/main/groovy` and put the following SpringApplicationContext singleton 
there (in the correct package):

{% gist 618739 SpringApplicationContext.java %}

Configure the singleton in your spring context:

{% gist 618739 resources.xml %}

As you can see, I also declared some SimpleDateFormat instances as beans to be used in our examples.

Now everything is configured and ready to be used. Let's se how we can obtain a spring bean inside a Groovlet. 
Create the file `WEB-INF/groovy/index.groovy` with the following content:

{% gist 618739 index.groovy %}

Now run your application with the command `mvn gae:run`, point your browser to http://localhost:8080/index.groovy and 
you should see something like this:

{% img /blog/images/image.png %}

Well, that's it! Nothing much different from what you are used to do in a normal Web application, right? But 
remember: Gaelyk is NOT your normal Web framework so let's spice things a little bit.

The solution for looking up beans depicted above is a bit cumbersome. Let's use 
[Gaelyk's plugin system](http://gaelyk.appspot.com/tutorial/plugins) to make things a little more "groovy". 
Using the plugin descriptor bellow, we can provide shortcuts to our SpringApplicationContext's methods, getContext() 
and getBean(). Save it in the file `WEB-INF/plugins/spring.groovy`:

{% gist 618739 spring.groovy %}

Before you can use these shortcuts, you need to tell Gaelyk about your descriptor by "installing" it in your project. 
Save the code bellow in the file WEB-INF/plugins.groovy:

{% gist 618739 plugins.groovy %}

Now you can use the shortcuts in your Groovlets this way:

{% gist 618739 index2.groovy %}


Cool, isn't it? A note on the `autowire` binding: It creates bindings "automagically" for each bean you passed as a 
parameter, as if the beans were declared in your Groovlet.

You can download the sample project used in this post from GitHub: http://github.com/deluan/gaelyk-spring. 
You can follow each commit to see exactly what was changed in each step of this tutorial.

If you have any suggestion or question, please let me know.

**UPDATE**: I've refactored the `autowire`method into a Category, so now it's not necessary to pass `this` as the first 
parameter.  The new version is available at [GitHub](http://github.com/deluan/gaelyk-spring)