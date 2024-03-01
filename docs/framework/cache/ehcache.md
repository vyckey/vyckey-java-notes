---
title: Eh Cache
tags: [cache]
sidebar_label: Eh Cache
sidebar_position: 5
---

# 简介

Ehcache是一个用Java实现的使用简单，高速，实现线程安全的缓存管理类库，ehcache提供了用内存，磁盘文件存储，以及分布式存储方式等多种灵活的cache管理方案。同时ehcache作为开放源代码项目，采用限制比较宽松的Apache License V2.0作为授权方式，被广泛地用于Hibernate, Spring，Cocoon等其他开源系统。Ehcache 从 Hibernate 发展而来，逐渐涵盖了 Cahce 界的全部功能，是目前发展势头最好的一个项目。具有快速,简单,低消耗，依赖性小，扩展性强,支持对象或序列化缓存，支持缓存或元素的失效，提供 LRU、LFU 和 FIFO 缓存策略，支持内存缓存和磁盘缓存，分布式缓存机制等等特点。

# 特性

可参考https://www.iteye.com/blog/raychase-1545906
1. 快速； 
2. 简单；
3. 多种缓存策略； 
4. 缓存数据有两级：内存和磁盘，因此无需担心容量问题；
5. 缓存数据会在虚拟机重启的过程中写入磁盘；
6. 可以通过 RMI、可插入 API 等方式进行分布式缓存；
7. 具有缓存和缓存管理器的侦听接口；
8. 支持多缓存管理器实例，以及一个实例的多个缓存区域；
9. 提供 Hibernate 的缓存实现；

# EhCache架构

# 存储方式

* **堆内存储**：速度快，但是容量有限，并由jvm进行管理。
* **堆外存储**：利用nio的DirectByteBuffers实现，比存储到磁盘上快，而且完全不受GC的影响，可以保证响应时间的稳定性；但是direct buffer的在分配上的开销要比heap buffer大，而且要求必须以字节数组方式存储，因此对象必须在存储过程中进行序列化，读取则进行反序列化操作，它的速度大约比堆内存储慢一个数量级。
* **磁盘存储**：容量大，速度慢。

# 缓存模式

* **cache-aside**：直接操作。先询问cache某条缓存数据是否存在，存在的话直接从cache中返回数据，绕过SOR；如果不存在，从SOR中取得数据，然后再放入cache中。
* **cache-as-sor**：结合了read-through、write-through或write-behind操作，通过给SOR增加了一层代理，对外部应用访问来说，它不用区别数据是从缓存中还是从SOR中取得的。
* **read-through**。
* **write-through**。
* **write-behind（write-back）**：既将写的过程变为异步的，又进一步延迟写入数据的过程。

# 参考资料

* [EhCache官网](https://www.ehcache.org/)