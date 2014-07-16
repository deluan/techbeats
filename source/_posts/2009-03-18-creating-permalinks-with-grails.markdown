---
layout: post
title: "Creating permalinks with Grails"
permalink: /creating-permalinks-with-grails
date: 2009-03-18 21:51:37 -0500
comments: true
categories: [grails, groovy]
---

I've been working with Grails for some time now and if I had to choose just one good thing to say about it is how it's community is really great!

I think it's time to start giving back some contribution, and here's the first one: A Permalink Codec to generate permalinks based on strings. It strips out all non word chars and convert the resulting string to lowercase:

To use it in your Grails project, save it in `grails-app/utils/com/deluan/grails/codecs` folder as `PermalinkCodec.groovy`.

Please read the (excelent) Grails' manual for more info on how to use codecs.