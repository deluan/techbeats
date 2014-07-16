---
layout: post
title: "Apache Shiro tags for Facelets - Securing your JSF pages"
permalink: /apache-shiro-tags-for-jsffacelets
date: 2010-11-01 00:49:27 -0500
comments: true
categories: [java, jsf, shiro] 
---

First, a little introduction. You can skip it and go straight to the 
[source code](http://github.com/deluan/shiro-faces), if you want.

I started working with [Apache Shiro](http://shiro.apache.org/) when it was still called JSecurity, and I have to 
say that it really rocks! I tried to use Spring Security (Acegi) in some projects, but the easiness and lean approach 
of Shiro is unbeatable. For a quick introduction, here's a quote from the project's site:

{% blockquote %}
Apache Shiro is a powerful and easy-to-use Java security framework that performs authentication, authorization, cryptography, and session management. 
With Shiro’s easy-to-understand API, you can quickly and easily secure any application – from the smallest mobile applications to the largest web and enterprise applications.
{% endblockquote %}
<!-- more -->

I've used it in Java and Grails projects, and recently I've even been 
[experimenting it with Google App Engine](https://code.google.com/p/shiro-gae). It fits very well in almost any Java 
platform project that needs security.

Shiro already works great in a JSF/Facelets project. You can use its [filters](http://shiro.apache.org/web.html) to 
grant and deny access to some parts of your application and to force authentication (redirect to login).

The only functionality missing is the power of its JSP taglib, that are used to conditionally render some parts of 
your HTML, based on user's authorization, roles and permissions (you can learn how to use them with this 
[simple example project](http://svn.apache.org/repos/asf/shiro/trunk/samples/web/)). The problem is that this 
conditional rendering [is not totally compatible with JSF's phases](http://www.devx.com/Java/Article/21020/1954). 
Better than trying to fit a cube in a spherical hole, I decided to rewrite Shiro's JSP tags into a 
[Facelets](http://en.wikipedia.org/wiki/Facelets) taglib, totally compatible with JSF.

All original tags are available as their Facelets equivalents, and I have introduced two new ones:

`<shiro:hasAnyPermission>` - Displays body content only if the current user has one of the specified permissions from 
a comma-separated list of permission strings.
`<shiro:remembered>` - Displays body content only if the current user has a known identity that has been obtained from 
'RememberMe' services. Note that this is semantically different from the 'authenticated' tag, which is more restrictive.

I've already submitted a [patch](https://issues.apache.org/jira/browse/SHIRO-206) to Shiro's development team, but 
they're very busy at the moment packaging the new 1.1 version for release. So I decided to 
[share the taglib on GitHub](http://deluan.github.com/shiro-faces), and host the artifacts in my personal maven 
repository, as I need to use the tags in an on-going project.

If you want to try it in your maven project, add my repository to your pom.xml:

{% gist 655983 repositories_pom.xml %}

and add the jar dependency:

{% gist 655983 dependency_pom.xml %}

Now you can declare Shiro's namespace in your xhtml pages and use the tags like this:

{% gist 655983 test.xhtml %}

That's it! Just keep in mind that as soon as this lib gets incorporated officially in Shiro, I'll stop updating it 
in GitHub and all future enhancements will only be available in the official version. If you want to see the tags 
incorporated officially into Shiro sooner than later, you can vote here: https://issues.apache.org/jira/browse/SHIRO-206

And if you have any suggestion, please let me know.

Enjoy ;)