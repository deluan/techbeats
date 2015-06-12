---
layout: post
title: "Calling the correct Grails version automatically from command line"
permalink: /calling-the-right-grails-version-from-command
date: 2010-10-04 23:48:53 -0500
comments: true
categories: [bash, grails]
---

**UPDATE**: For latest version and instructions, see http://github.com/deluan/grails.sh

Now that I decided to organize and publish some of my code/hacks here, I thought
it would be a good thing to republish here my Grails caller script.

I work on and maintain various Grails projects at the same time, and some of
them uses versions of Grails as old as 1.0.3! So the question is: How to call
the right version of grails command for a given project, the version that the
project was created with?
<!-- more -->

First I tried changing the `GRAILS_HOME` environment variable every time I was
going to work with a project that uses a different version than the default.
But it’s just too much work for a thing that should be transparent. So I decided
to create a shell script to solve this problem. The script should detect which
Grails version to call when it’s executed. Here’s the script I came up with:

{% gist 601039 grails %}

## How it Works?

The script first checks if you specified a version in the command line,
like: `grails 1.3.5-SNAPSHOT create-app`. If not, it looks for an
`application.properties` file in the current folder. This file keeps some
metadata for Grails projects, including the Grails version the project was
created with. If it does not find any `application.properties` in the current
folder, it then just calls the
[default Grails installed](http://grails.org/Installation) in your system,
the one that `GRAILS_HOME` points to.

## Prerequisites

* All your Grails versions must be installed under the same base directory. Ex:

```
  /opt/grails-1.0.3
  /opt/grails-1.1.1
  /opt/grails-1.2-M2
  /opt/grails-1.3.3
  /opt/grails-1.3.5-SNAPSHOT
```

* `GRAILS_HOME` environment variable must be set and point to your “default”
  Grails installation

* This script was tested on Mac OS X (Snow Leopard), Linux (Ubuntu) and
  Windows (with cygwin)

## Installation

* Download the script: http://github.com/deluan/grails.sh/raw/master/grails
* Include the folder where it is installed in your `PATH`.
* Exclude `$GRAILS_HOME/bin` from your `PATH`

## Usage

Using the script is as transparent as possible:

* If you invoke it from a project folder, it will detect the version used by
  the project and call the correct grails (if it is installed in your system)
* If you invoke it from any other folder that does not contain a Grails project,
  it will call the “default” Grails installation
* If you want to call a specific Grails version (i.e. when doing an upgrade) you
  can specify the version you want in the first parameter. Ex:
```
$ grails 1.3.3 upgrade
```
