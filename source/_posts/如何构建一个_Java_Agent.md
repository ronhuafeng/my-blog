---
title: 如何构建一个 Java Agent
id: 488
categories:
  - 技术记录
date: 2017-01-16 18:58:28
tags:
---

First. Implement a static premain (as an analogy to main) method, like this:

```java
import java.lang.instrument.Instrumentation;

class Example {
    public static void premain(String args, Instrumentation inst) {
        ...
    }
}
```

Second. Create a manifest file (say, manifest.txt) marking this class for pre-main execution. Its contents are:

`Premain-Class: Example`

Third. Compile the class and package this class into a JAR archive:

`javac Example.java`
`jar cmf manifest.txt yourAwesomeAgent.jar *.class`

Fourth. Execute your JVM whith -javaagent parameter, like this:

`java -javaagent:yourAwesomeAgent.jar -jar yourApp.jar`