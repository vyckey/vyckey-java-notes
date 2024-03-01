---
title: Guava Cache
tags: [cache]
sidebar_label: Guava Cache
sidebar_position: 3
---

# 简介

Guava Cache是Google开源的Java重用工具集库Guava里的一款缓存工具。
优点：
1. 本地缓存，读取效率高，不受网络因素影响。
2. 拥有丰富的功能，操作简单。
3. 线程安全。

使用场景：
* 愿意消耗一些内存空间来提升速度
* 预料到某些键会被多次查询
* 缓存中存放的数据总量不会超出内存容量

Guava设计思路来源于 `ConcurrentHashMap` ，使用多个segments方式进行细粒度加锁，保证线程安全，并有高吞吐量的特性。
其主要实现的缓存功能有：
1. 自动将 `entry` 节点加载进缓存结构中；
2. 内部会维护两个队列 `accessQueue`,`writeQueue` 用于记录缓存顺序；
3. 当缓存的数据超过设置的最大值时，使用LRU算法移除；
4. 具备根据 `entry` 节点上次被访问或者写入时间计算它的过期机制；
5. 缓存的 `key` 被封装在 `WeakReference` 引用内；
6. 缓存的 `Value` 被封装在 `WeakReference` 或 `SoftReference` 引用内；
7. 统计缓存使用过程中命中率、异常率、未命中率等统计数据。

# 缓存创建

一般流程：if cached, return; otherwise create/load/compute, cache and return
guava提供了 `CacheLoader` 和 `Callable` 。

# 缓存失效

guava的3中失效策略：size-based eviction, time-based eviction, reference-based eviction, and manually eviction。
guava在执行写操作或者读操作的时候进行缓存失效，没有创建僵尸线程。 `RemovalListener` 可以监听缓存失效事件。
LRU算法。
**size-based eviction**
使用 `CacheBuilder.maxmumSize(long)` 方法，快饱和时进行失效。使用 `CacheLoader` 会在失效后重新加载。

**time-based eviction**
guava提供两种方法：`expireAfterAccess(long, TimeUnit)` 和 `expireAfterWrite(long,TimeUnit)` 。

**reference-based eviction**
guava提供 `entries` 垃圾回收，可以配置 `keys` 的弱引用，配置 `values` 的弱引用和软引用。

**manually eviction**
提供三种自动失效方法：`Cache.invalidate(key)`，`Cache.invalidateAll(keys)`，`Cache.invalidateAll()`

# 缓存统计

`CacheBuilder.recordStats()` 启用缓存的统计功能，`CacheStats` 包含了 `hitCount` 、`missCount`、`loadSuccessCount` 等信息。

# 代码示例

```java
public class GuavaCacheTest {
    @Test
    public void create() throws ExecutionException {
        // use CacheLoader
        LoadingCache<String, Object> loadingCache =
                CacheBuilder.newBuilder()
                        .maximumSize(1000)
                        .expireAfterAccess(1, TimeUnit.HOURS)
                        .expireAfterWrite(1, TimeUnit.DAYS)
                        .removalListener(new RemovalListener<String, Object>() {
                            @Override
                            public void onRemoval(RemovalNotification<String, Object> notification) {
                                // do something
                            }
                        })
                        .build(new CacheLoader<String, Object>() {
                            public Object load(String key) {
                                // load and return
                                return new Object();
                            }
                        });
        Object obj = loadingCache.get("key");
// use Callable
        Cache<String, Object> cache =
                CacheBuilder.newBuilder()
                        .weigher(new Weigher<String, Object>() {
                            @Override
                            public int weigh(String key, Object value) {
                                // define value weight
                                return 0;
                            }
                        })
                        .maximumSize(1000)
                        .weakKeys()
                        .weakValues()
                        .softValues()
                        .recordStats()
                        .build();
        obj = cache.get("key", new Callable<Object>() {
            public Object call() {
                // do something and return
                return "123";
            }
        });
CacheStats stats = cache.stats();
    }
}
```

# 参考资料

* [GitHub - Guava](https://github.com/google/guava)
* [掘金 - Guava Cache](https://juejin.im/post/5c24b7e0e51d451c82526a11)
* [CSDN - 缓存框架Guava Cache部分源码分析](https://blog.csdn.net/zjccsg/article/details/51932252)