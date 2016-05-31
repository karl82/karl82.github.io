---
title: SLF4J and logging throwables
layout: post
category: java
tags: [java, slf4j, logging]
---

I've been living in the darkness about not documented feature of
[SLF4J](http://www.slf4j.org/) when
[Throwable](https://docs.oracle.com/javase/8/docs/api/java/lang/Throwable.html)
is used as the last parameter.

There are several methods defined for each level:

```java
public interace Logger {
    void debug(String format, Object arg);
    void debug(String format, Object arg1, Object arg2);
    void debug(String format, Object... arguments);
    void debug(String msg, Throwable t);
}
```

When the last argument (`args2` or `arguments[arguments.length - 1]`) is
instance of `Throwable`, then slf4j will log its stack trace like calling
`debug(String,Throwable)`. When was this behavior introduced? Almost **6years**
ago in version 1.6. It was introduced with commit
[3c0ab3466b6fa6e915974c72558d64c570734700](https://github.com/qos-ch/slf4j/commit/3c0ab3466b6fa6e915974c72558d64c570734700).
The documentation isn't clear about it.

And here is the example:

```java
   @Test
   public void lastThrowableLogs() throws Exception {
       LoggerFactory.getLogger(Slf4jTest.class)
               .debug("Hello world {} {} {}", null, null, null, new Exception("It's going to be logged"));
   }

   @Test
   public void sameArgsLogs() throws Exception {
       LoggerFactory.getLogger(Slf4jTest.class)
               .debug("Hello world {} {} {} {}", null, null, null, new Exception("It's going to be logged"));
   }

   @Test
   public void throwableInTheMiddleDoesNotLog() throws Exception {
       LoggerFactory.getLogger(Slf4jTest.class)
               .debug("Hello world {} {} {} {}", null, null, new Exception("It's going to be logged"), null);
   }
```

And output:

```text
1 [main] DEBUG Slf4jTest  - Hello world null null null
java.lang.Exception: It's going to be logged
    at Slf4jTest.lastThrowableLogs(Slf4jTest.java:12)
    ...
    at com.intellij.rt.execution.application.AppMain.main(AppMain.java:144)
4 [main] DEBUG Slf4jTest  - Hello world null null null {}
java.lang.Exception: It's going to be logged
    at Slf4jTest.sameArgsLogs(Slf4jTest.java:18)
    ...
    at com.intellij.rt.execution.application.AppMain.main(AppMain.java:144)
5 [main] DEBUG Slf4jTest  - Hello world null null java.lang.Exception: It's going to be logged null
```

Pretty cool, isn't it?

_Thank you Athanasios!_
