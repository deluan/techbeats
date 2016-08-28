---
layout: post
title: "Processing large templates with Gradle and FreeMarker"
date: 2015-07-10 20:44:24 -0400
comments: true
published: true
categories: [gradle, groovy]
---

__TL;DR__ This article shows a solution for overcoming the issue [GRADLE-3122](https://issues.gradle.org/browse/GRADLE-3122).
You can [jump straight to the implementation](/blog/2015/07/10/processing-large-templates-with-gradle-and-freemarker/#final-solution)
at the bottom of this post.

In my current project we have a need to generate a set of files for each environment, using templates. As this is a
[Gradle](http://gradle.org) project, this requirement is easily accomplished with a `CopyTask`:

<!-- more -->

```groovy build.gradle
task processTemplates(type: Copy) {
     from 'src/templates', {
         include '**/*.*'
     }

     def env = loadEnvironment()
     eachFile { FileCopyDetails file ->
         if (file.name.endsWith('.template')) {
             expand(env)
             file.name = file.sourceName - '.template'
         }
     }

     inputs.file "env/${envName}.properties"
     into "$buildDir/output/$envName"
}

def getEnvName() {
    hasProperty('env') ? env : 'dev'
}

def loadEnvironment() {
    def properties = new Properties()
    new File("env/${envName}.properties").withInputStream { properties.load(it) }
    properties
}
```

The `processTemplates` task above will copy all files from `src/templates` to `build/output/<envName>`. All files whose
name ends with `.template` will be processed and tokens (ex: `${variable}`) will be replaced by their values from the
`env/<envName>.properties` file. Adding the properties file as an input for the task is important (`inputs.file` method
call), so when you change it, the task will be re-executed on the next build.

Simple, right?

### Not so fast, Johnny...

This worked fine until we had one template that was really big (120KB) and we found out about issue
[GRADLE-3122](https://issues.gradle.org/browse/GRADLE-3122). Gradle uses Groovy's
[SimpleTemplateEngine](http://docs.groovy-lang.org/latest/html/documentation/template-engines.html#_simpletemplateengine),
that can only process files up to 64KB!

Newest versions of Groovy (2.4+) include
[StreamingTemplateEngine](http://docs.groovy-lang.org/latest/html/documentation/template-engines.html#_streamingtemplateengine)
that does not have this limit, but the most recent version of Gradle (2.5 as of this post)
still uses Groovy 2.3...

One way to overcome this would be to use Ant's `ReplaceTokens` filter, simply by changing our templates to use Ant's
token syntax (ex: `@variable@`) and changing the line `expand(env)` to `filter(ReplaceTokens, tokens: env)`

But because we need to use logic in our templates (if's and loops), we had to come up with a different approach.
The solutions available were too simple for our needs or too complicated to implement in a clear way, making them
unsuitable for our project. So we decided to roll...

### <a name="final-solution"></a>Our own solution

Finally we decided to implement a simple template processor using [FreeMarker](http://freemarker.org). We select
this awesome template engine for its feature set, IDE support and excelent OSS reputation, although the solution bellow
could be adapted to be used with any other template engine ([Velocity](http://velocity.apache.org),
[JMustache](https://github.com/samskivert/jmustache), etc...)

```groovy buildSrc/src/main/groovy/your/package/TemplateProcessor.groovy
package your.package

import freemarker.template.Configuration
import org.apache.commons.io.FileUtils
import org.apache.tools.ant.DirectoryScanner

import static freemarker.template.TemplateExceptionHandler.RETHROW_HANDLER

class TemplateProcessor {
    private static final String TEMPLATE_EXTENSION = '.ftl'
    private String templatesDir
    private String outputDir
    private Configuration cfg

    TemplateProcessor(String templatesDir, String outputDir) {
        this.templatesDir = templatesDir
        this.outputDir = outputDir

        cfg = new Configuration()
        cfg.setDirectoryForTemplateLoading(new File(templatesDir))
        cfg.setDefaultEncoding("UTF-8")
        cfg.setTemplateExceptionHandler(RETHROW_HANDLER)
    }

    void execute(Map properties) {
        DirectoryScanner scanner = createScanner()
        scanner.scan()
        scanner.includedFiles.each { String fileName ->
            if (fileName.endsWith(TEMPLATE_EXTENSION)) {
                process(fileName, properties)
            } else {
                copy(fileName)
            }
        }
    }

    private process(String fileName, Map properties) {
        def outputFile = new File(outputDir, fileName - TEMPLATE_EXTENSION)
        def template = cfg.getTemplate(fileName)

        outputFile.withWriter { out ->
            template.process(properties, out)
        }
    }

    private copy(String fileName) {
        def inputFile = new File(templatesDir, fileName)
        def outputFile = new File(outputDir, fileName)
        FileUtils.copyFile(inputFile, outputFile)
    }

    private DirectoryScanner createScanner() {
        def scanner = new DirectoryScanner()
        scanner.includes = ['**/*']
        scanner.basedir = new File(templatesDir)
        scanner
    }
}
```
Note that we now use the `.ftl` file extension for our templates, to enable support in our
[IDE of choice](http://jetbrains.com/idea). To use the processor, you have to put it under the
[buildSrc project](https://docs.gradle.org/current/userguide/organizing_build_logic.html#sec:build_sources).
This is a special "module" in your project that is a simple way to organize build logic in your build scripts. It is
all automatically handled by Gradle. You'll also need a small `build.gradle` just for  declaring the dependencies for
FreeMarker and Apache Commons IO (used for the `copyFile` method):

```groovy buildSrc/build.gradle
repositories { mavenCentral() }

dependencies {
    compile 'org.freemarker:freemarker:2.3.23'
    compile 'commons-io:commons-io:2.4'
}
```

To have this code available to your main build script, add this two files in your project, in the following paths:

* _buildSrc/build.gradle_
* _buildSrc/src/main/groovy/your/package/TemplateProcessor.groovy_

The last step is to actually use it in our `processTemplates` task:

```groovy build.gradle
import your.package.TemplateProcessor

task processTemplates() {
    def fromDir = 'src/templates'
    def intoDir = "$buildDir/output/$envName"
    doLast {
      def env = loadEnvironment()
      new TemplateProcessor(fromDir, intoDir).execute(env)
    }

    inputs.dir fromDir
    inputs.file "env/${envName}.properties"
    outputs.dir intoDir
}
```

Not hard, eh? Note that this task is not a `CopyTask` anymore, so we now need to specify its inputs and outputs.

My plan is to convert this code into a proper Gradle plugin. But for now: That's all, folks!
