---
layout: post
title: "File encoding auto-detection for ZK Fileupload component"
date: 2015-06-11 22:09:24 -0400
comments: true
published: true
categories: [zk]
---

Proper handling of file encoding can be a royal headache. Recently I spent an
unreasonable amount of time trying to figure out why [ZK's](www.zkoss.com)
Fileupload component was messing with the contents of my CSV file:

<!-- more -->

{% img /images/blog/2015-06-11-file-encoding-auto-detection-for-zk-fileupload-component/0.png %}

The data should be: `código redome;;código hemocentro;nome; da mãe;`

The file was generated with Windows Excel, using the encoding ISO-8859-2
(common encoding for Windows). After some investigation I found
out that Fileupload by default treats all files with content type `text/...`
as UTF-8! Ouch!!

If all your files will be generated using the same file encoding, this can be
fixed with the following configuration:

```xml
<zk>
  <system-config>
    <upload-charset>YOUR ENCODING</upload-charset> <!-- ISO-8859-2 in my case -->
    ...
  </system-config>
  ...
</zk>
```

The [upload-charset](http://books.zkoss.org/wiki/ZK_Configuration_Reference/zk.xml/The_system-config_Element/The_upload-charset_Element) tag did the trick! But what if my user decides to
move to a different (better?) platform in the future, and generates the file
with UTF-8? Or any other encoding?

Then the proper solution is to use the tag [upload-charset-finder-class](http://books.zkoss.org/wiki/ZK_Configuration_Reference/zk.xml/The_system-config_Element/The_upload-charset-finder-class_Element):

```xml
<upload-charset-finder-class>a_class_name</upload-charset-finder-class>
```

The [documentation](http://books.zkoss.org/wiki/ZK_Configuration_Reference/zk.xml/The_system-config_Element/The_upload-charset-finder-class_Element) says that this class has to implement
the interface [CharsetFinder](http://www.zkoss.org/javadoc/latest/zk/org/zkoss/zk/ui/util/CharsetFinder.html)
and its sole method `String getCharset(String contentType, InputStream content)`:

> When a text file is uploaded, the getCharset method is called and it can
> determines the encoding based on the content type and/or the content of the
> uploaded file.

Which leads us to the main reason of this post: *How to detect the file
encoding, if ZK itself does not provide a default implementation for this
interface?*

More research pointed me to the [some solutions](http://stackoverflow.com/questions/499010/java-how-to-determine-the-correct-charset-encoding-of-a-stream), but the one that I ended up implementing was using the
[Apache Any23](https://any23.apache.org). It includes the `TikaEncodingDetector`,
that can be used to auto-detect the file encoding of a stream. The final code
for the CharsetFinder implementation is the following:
```java
package util;

import org.apache.any23.encoding.TikaEncodingDetector;
import org.zkoss.zk.ui.util.CharsetFinder;

import java.nio.charset.Charset;

public class TikaCharsetFinder implements CharsetFinder {
    @Override
    public String getCharset(String contentType, InputStream content) throws IOException {
        return new TikaEncodingDetector().guessEncoding(content);
    }
}
```

Yep, it is that simple. The final ZK configuration to use this class is:
```xml
<zk>
  <system-config>
    <upload-charset-finder-class>util.TikaCharsetFinder</upload-charset-finder-class>
    ...
  </system-config>
  ...
</zk>
```
