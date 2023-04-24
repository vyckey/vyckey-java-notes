# LOGBACK

Logback是Log4j的后续，由log4j的发起者Ceki Gülcü设计。相比现有的日志框架，Logback是一个更快地轻量级的日志框架。重要的是，Logback提供了一些其他框架不具备的新的很有用的功能。

## Logback架构

* logback-core：logback的核心模块，后面两个模块的基础。
* logback-classic：原生地实现了Slf4j，包含log4j 1.x的大部分能力，并进行了一些升级。
* logback-access：整合了Servlet容器，例如：Tomcat、Jetty，并提供了log的能力。

## 参考资料

* [Logback官网](https://logback.qos.ch/index.html)
* [Logback官网 - Manual](https://logback.qos.ch/manual/index.html)