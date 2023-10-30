---
title: Java JVM Tools
tags: [java, jvm, tools]
sidebar_label: Tools
sidebar_position: 3
---

[toc]

# Alibaba Arthas

Arthas 是Alibaba开源的Java诊断工具，深受开发者喜爱。

当你遇到以下类似问题而束手无策时，Arthas可以帮助你解决：

1. 这个类从哪个 jar 包加载的？为什么会报各种类相关的 Exception？
2. 我改的代码为什么没有执行到？难道是我没 commit？分支搞错了？
3. 遇到问题无法在线上 debug，难道只能通过加日志再重新发布吗？
4. 线上遇到某个用户的数据处理有问题，但线上同样无法 debug，线下无法重现！
5. 是否有一个全局视角来查看系统的运行状况？
6. 有什么办法可以监控到JVM的实时运行状态？
7. 怎么快速定位应用的热点，生成火焰图？
8. 怎样直接从JVM内查找某个类的实例？

Arthas支持JDK 6+，支持Linux/Mac/Windows，采用命令行交互模式，同时提供丰富的 Tab 自动补全功能，进一步方便进行问题的定位和诊断。

# 参考资料

* [GitHub arthas](https://github.com/alibaba/arthas)
* [GitHub arthas中文手册](https://github.com/alibaba/arthas/blob/master/README_CN.md)
* [公众号 - JavaGuide/自从学会 Arthas，日常开发效率直接起飞！！](https://mp.weixin.qq.com/s/gi5IovECxwpX3v6OcMbQxA)