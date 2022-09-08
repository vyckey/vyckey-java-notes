# Java日志框架

## 前言

在应用程序中添加日志记录一般来说有以下几种目的：
* 监视变量的变化情况，周期性地记录到文件中供后续进行统计分析和审计工作。
* 跟踪代码运行时轨迹，分析代码的运行逻辑。
* 程序员的调试器，向文件或控制台打印代码的调试信息。

## 日志框架有哪些？

### 门面型日志框架

* JCL：Apache基金会所属的项目，是一套Java日志接口，之前叫Jakarta Commons Logging，后更名为Commons Logging。
* SLF4J：**Simple Logging Facade for Java**，缩写Slf4j，是一套简易Java日志门面，本身并无日志的实现。

### 记录型日志框架

* Jul (Java Util Logging)：JDK中的日志记录工具，也常称为JDKLog、jdk-logging，自Java1.4以来的官方日志实现。
* Log4j：Apache Log4j是一个基于Java的日志记录工具。它是由Ceki Gülcü首创的，现在则是Apache软件基金会的一个项目。 Log4j是几种Java日志框架之一。
* Log4j2：一个具体的日志实现框架，是Log4j 1的下一个版本，与Log4j 1发生了很大的变化，**Log4j 2不兼容Log4j 1**。
* Logback：一个具体的日志实现框架，和Slf4j是同一个作者，但其性能更好，推荐使用。


## 日志框架发展史

* 1996年早期，欧洲安全电子市场项目组决定编写它自己的程序跟踪API(Tracing API)。经过不断的完善，这个API终于成为一个十分受欢迎的Java日志软件包，即Log4j（由Ceki创建）。
* 后来Log4j成为Apache基金会项目中的一员，Ceki也加入Apache组织。后来Log4j近乎成了Java社区的日志标准。据说Apache基金会还曾经建议Sun引入Log4j到Java的标准库中，但Sun拒绝了。
* 2002年Java1.4发布，Sun推出了自己的日志库JUL(Java Util Logging),其实现基本模仿了Log4j的实现。在JUL出来以前，Log4j就已经成为一项成熟的技术，使得Log4j在选择上占据了一定的优势。
* 接着，Apache推出了Jakarta Commons Logging，JCL只是定义了一套日志接口(其内部也提供一个Simple Log的简单实现)，支持运行时动态加载日志组件的实现，也就是说，在你应用代码里，只需调用Commons Logging的接口，底层实现可以是Log4j，也可以是Java Util Logging。
* 后来(2006年)，Ceki不适应Apache的工作方式，离开了Apache。然后先后创建了Slf4j(日志门面接口，类似于Commons Logging)和Logback(Slf4j的实现)两个项目，并回瑞典创建了QOS公司，QOS官网上是这样描述Logback的：The Generic，Reliable Fast&Flexible Logging Framework(一个通用，可靠，快速且灵活的日志框架)。
* Java日志领域被划分为两大阵营：Commons Logging阵营和Slf4j阵营。
Commons Logging在Apache大树的笼罩下，有很大的用户基数。但有证据表明，形式正在发生变化。2013年底有人分析了GitHub上30000个项目，统计出了最流行的100个Libraries，可以看出Slf4j的发展趋势更好。
* Apache眼看有被Logback反超的势头，于2012-07重写了Log4j 1.x，成立了新的项目Log4j 2, Log4j 2具有Logback的所有特性。

log4j的日志使用写法：
```java
import org.apache.log4j.Logger;
//省略...
Logger logger = Logger.getLogger(Test.class);
logger.trace("trace");
//省略...
```

jul的日志使用写法：
```java
import java.util.logging.Logger；
//省略...
Logger loggger = Logger.getLogger(Test.class.getName()); 
logger.finest("finest");
//省略...
```

jcl不同于前面两种，JCL 只提供 log 接口，具体的实现则在运行时动态寻找。这样一来组件开发者只需要针对 JCL 接口开发，而调用组件的应用程序则可以在运行时搭配自己喜好的日志实践工具。

jcl默认的配置：如果能找到Log4j 则默认使用log4j 实现，如果没有则使用jul(jdk自带的) 实现，再没有则使用jcl内部提供的SimpleLog 实现。

于是，日志记录的代码里变成了：
```java
import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;
//省略...
Log log = LogFactory.getLog(Test.class);
log.trace('trace');
//省略...
```
JCL会在ClassLoader中进行查找Log具体的实现类，但是三个缺点:
1. 效率较低。
2. 容易引发混乱。
3. 在使用了自定义ClassLoader的程序中，使用JCL会引发内存泄露。

log4j的作者（Ceki）觉得jcl不好用，自己又写了一个新的接口api，slf4j。在代码中，并不会出现具体日志框架的api。程序根据classpath中的桥接器类型，和日志框架类型，判断出*logger.info*应该以什么框架输出。

slf4j的日志使用写法如下：
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
//省略...
Logger logger = LoggerFactory.getLogger(Test.class);
//省略...
logger.info("info");
```

## Spring中使用的日志框架

## 日志框架的选择

如果是在一个新的项目中建议使用Slf4j与Logback组合，有如下几个优点:
* Slf4j实现机制决定Slf4j限制较少，使用范围更广。由于Slf4j在编译期间，静态绑定本地的LOG库使得通用性要比Commons Logging要好。
* Logback拥有更好的性能。Logback声称：某些关键操作，比如判定是否记录一条日志语句的操作，其性能得到了显著的提高。这个操作在Logback中需要3纳秒，而在Log4J中则需要30纳秒。LogBack创建记录器（logger）的速度也更快：13毫秒，而在Log4J中需要23毫秒。更重要的是，它获取已存在的记录器只需94纳秒，而Log4J需要2234纳秒，时间减少到了1/23。跟JUL相比的性能提高也是显著的。
* Logback文档免费，不像Log4J那样只提供部分免费文档而需要用户去购买付费文档。
* 

## 参考资料

* [Slf4j官网](http://www.slf4j.org/)
* [Logback官网](https://logback.qos.ch/)
* [Apache commons logging](https://commons.apache.org/proper/commons-logging/)
* [掘金 - 深入掌握Java日志体系，再也不迷路了](https://juejin.cn/post/6905026199722917902) (作者：一角钱技术)