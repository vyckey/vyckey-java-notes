---
title: Redis
tags: []
sidebar_label: Redis
sidebar_position: 5
---

# Redis

Redis是一个开源（BSD许可）的内存中数据结构存储，用作数据库、缓存、消息代理和流媒体引擎。Redis提供数据结构，如 `string` 、 `hash` 、 `list` 、 `set` 、 `sorted set` 、 `bitmap` 、 `hyperloglog` 、 `geospatial index` 和 `stream`。Redis具有内置复制、Lua脚本、LRU驱逐、事务和不同级别的磁盘持久性，并通过Redis Sentinel和Redis Cluster的自动分区提供高可用性。

为了获得最佳性能，Redis使用内存中的数据集。根据您的使用情况，Redis可以通过定期将数据集转储到磁盘或将每个命令附加到基于磁盘的日志中来持久化您的数据。如果您只需要一个功能丰富、联网的内存缓存，也可以禁用持久性。

## Redis命令

Redis命令大全可到 [Redis官网命令](https://redis.io/commands/?group=string) 中进行搜索。

# 参考资料

* [Redis官网](https://redis.io/)
* [Redis官网 - Docs](https://redis.io/docs/get-started/)
* [Redis官网 - Commands](https://redis.io/commands/)
* [GitHub - redis/redis](https://github.com/redis)