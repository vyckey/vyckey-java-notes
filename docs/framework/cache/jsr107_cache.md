---
title: JSR-107 Cache
tags: [cache]
sidebar_label: JSR-107 Cache
sidebar_position: 2
---

# JSR-107简介

JSR是`Java Specification Requests`的缩写，意思是Java 规范提案。2012年10月26日JSR规范委员会发布了JSR 107（JCache API）的首个早期草案。
JCache规范定义了一种对Java对象临时在内存中进行缓存的方法，包括对象的创建、共享访问、假脱机（spooling）、失效、各JVM的一致性等，可被用于缓存JSP内最经常读取的数据。有以下特点：
* 定义出通用的抽象类和工具
* 对业务的侵略性低
* 提供进程内和分布式的缓存实现
* 支持按值或者引用来缓存数据
* 支持注解来实现缓存功能

# Jcache

JavaCache(简称JCache)定义了Java标准的api。

```groovy
compile group: 'javax.cache', name: 'cache-api', version: '1.1.1'
compile group: 'org.jsr107.ri', name: 'cache-ri-impl', version: '1.1.1'
```

| 接口  |	功能    |
| --- | --- |
|   `CachingProvider` |	管理多个 `CacheManager` ，制定建立，配置，请求机制  |
|   `CacheManager`    |	管理多个 `Cache` ，制定建立，配置，请求机制，只有一个对应的 `CacheProvider`    |
|   `Cache`   |	对缓存操作，只有一个对应的 `CacheManager`  |
|   `Cache.Entry` |	`Cache` 接口的内部接口，真正的存储实体 |
|   `ExporyPolicy`    |	控制缓存的过期时间  |

## CacheProvider

这个接口的实现类提供创建和管理 `CacheManager` 生命周期的方法。
`Caching` 类时javax提供的一个工具类，为了方便开发者去获取合适的 `CachingProvider` 实例的（该接口的实现类是管理 `CacheManager` 的生命周期）。
`Caching` 类大致提供了3种获取 `CachingProvider` 实例的方式
1. 获取默认的 `CachingProvider` 
2. 根据 `ClassLoader` 获取 `CachingProvider` 
3. 根据全类名创建/获取开发者实现的实例

## CacheManager

代码示例
```java
public class JCacheTest {
    @Test
    public void simpleCache() {
        //创建一个配置管理器
        Configuration<Integer, String> configuration =
                new MutableConfiguration<Integer, String>().setTypes(Integer.class, String.class);
        //创建一个缓存管理器
        CacheManager manager = Caching.getCachingProvider().getCacheManager();
        //生成一个缓存对象
        Cache<Integer, String> simpleCache = manager.getCache("simpleCache22");
        //缓存数据
        simpleCache = manager.createCache("simpleCache22", configuration);
        simpleCache.put(2, "value");
        //获取数据
        String value = simpleCache.get(2);
        System.out.println("Value: " + value);
    }
}
```

# 参考资料

* [简书 - 《剖析缓存系列》—— 熟悉JSR-107 JAVA缓存规范](https://www.jianshu.com/p/f6a1eae03efb)
