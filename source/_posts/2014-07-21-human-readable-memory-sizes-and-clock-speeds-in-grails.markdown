---
layout: post
title: "Human readable memory sizes and clock speeds in Grails"
date: 2014-07-21 08:33:48 -0400
comments: true
published: false
categories: [grails, codec]
---

So you are working in a application that has a requirement to display memory sizes (MB, GB, ...) or 
clock speeds (hz, Khz, ...) in human readable formatting while storing absolute values in your  

Wouldn't it be great if you could just.... Now you can: 

```
'100'.encodeAsClockSpeed() == '100 hz'
'768606208'.encodeAsClockSpeed() == '733 Mhz'
```

```
'1024'.encodeAsMemorySize() == '1 KB'
'2100000000'.encodeAsMemorySize() == '2 GB'
```

This is what the [Grails Human Readable](http://grails.org/plugin/human-readable) plugin provides. To use it, just add
`compile 'org.grails:human-readable:0.1'` to your `BuildConfig.groovy` and you're done.

Enjoy!