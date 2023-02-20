JanusGraph 批量导入

# 配置选项优化

## 优化Bulk Loading

```properties
storage.batch-loading = true # 要求用户自己保证导入数据是一致的，且与存在的数据也是一致的
schema.default = none # 关闭自动schema类型创建
```
对大多数的应用而言，开启`storage.batch-loading`将对批量导入有比较大的正向影响。开启之后，将关闭JanusGraph内部的一致性检查。更重要的是，关闭了锁，也就是JanusGraph假定导入的数据是一致的，因此关闭自检将会带来性能提升。

## 优化ID分配

```
ids.block-size = 20000000 # ID获取块的大小，
ids.authority.wait-time = # 存储后端向ID池管理器申请的等待时间ms，需要设置一个合理的值，否则太小了容易失败。建议值设置为存储后端第95%时读写时间之和。
ids.renew-timeout = 3600000 # ID池管理器在失败申请新的ID块的总等待时间间隔ms。
```

每个新增的顶点和边都需要分配唯一ID，ID池管理器按block获取新的ID。ID块的获取处理是非常昂贵的，因为它需要保证块分配的全局唯一。增加`ids.block-size`的值将减少获取ID的次数，也会潜在地有一些未分配的ID没有使用。

## 优化读写

### 缓存大小

JanusGraph的缓存块可以减少小批次执行对后端存储的请求，提高性能。这个参数由`storage.buffer-size`控制，增加这个值将降低写请求的数量，提高吞吐量，但是也会更容易造成失败。因此对于批量导入应该小心尝试调整该值。
```
storage.buffer-size = 20240 #请求缓冲区大小配置
```

### 读写健壮性

批量导入将增加读写失败的可能性，特别是缓存块大小的增大。
```
storage.read-attempts=100#该配置项配置JanusGraph在放弃之前尝试对存储后端执行读取或写入操作的次数。
storage.write-attempts=100
storage.attempt-wait=1000 #该配置项指定JanusGraph在重新尝试失败的后端操作之前将等待的毫秒数。
```

# 存储和索引优化

## 存储后端HBase

1. HBase Client会在数据累积到设置的阈值后才提交Region Server。这样做的好处在于可以减少RPC连接次数。

```
hbase.client.write.buffer = xxx
```

2. Region Server通过RPC Handler接收外部请求并加以处理。所以提升RPC Handler的数量可以一定程度上提高HBase接收请求的能力。

```
hbase.regionserver.handler.count = xxx
```

3. HBase存储数据压缩算法的配置，支持 lzo、gz、snappy、lz4、bzip2、zstd五种压缩算法和不压缩配置：none。

```
storage.hbase.compression-algorithm = none
```

## 索引后端ES

1. 降低索引记录刷新频率
```
index.[X].elasticsearch.bulk-refresh
```

2. 设置最大连接超时（毫秒）
```
index.[X].elasticsearch.connect-timeout
```

3. ElasticSearch在滚动上下文中保持活动的时间（秒）
```
index.[X].elasticsearch.setup-max-open-scroll-contexts
```

# 方案列举

## 基于java API

此方案可以用于数据量较小的情况下使用，不适合大批量的数据导入。

节点导入流程：
1. 获取图实例
2. 获取图实例事务对象
3. 插入节点
4. 提交事务

边导入流程：
1. 获取图实例
2. 获取图实例事务对象
3. 查询源节点 + 目标节点（这个地方可能是性能瓶颈）
4. 在两个节点中插入边
5. 提交事务

优化点：
此处的事务提交，我们可以通过一个常用的优化手段： 处理多个vertex或者edge后再提交事务！
可以减少janus与底层存储的交互，减少网络消耗和连接数，提升导入的性能！

## 基于Gremlin Server的批量导入

这里需要我们搭建一个Gremlin server服务器，通过在服务器执行gremlin-server.sh即可，暴露出一个tcp接口；则可以将对应的gremlin语句提交到对应的gremlin服务器执行；

其中插入边比较慢，最主要的原因是每插入一条边都需要检索两个顶点。社区里面建议是维持 name 索引到顶点id的一个 map 存放到内存中，但是注意几十亿节点的需要不少内存。

优化点：
* gremlin server池参数调整
* threadPoolWorke：最大2*core个数，用于处理非阻塞读写的Gremlin服务器可用的线程数；
* gremlinPool：用于在ScriptEngine中执行实际脚本的“Gremlin”线程的数量。此池表示Gremlin服务器中可用于处理阻塞操作的工作者；

和线程池调优一样，要找出最合适的一个值，太小不好，太大也不好；

> 该方案本质上和第一个方案类似，只不过是一个是通过给定的java api提交插入请求，一个直接通过gremlin语句提交插入请求到gremlin server。

## IBM的janusgraph-utils

这个主要也是通过多线程对数据进行导入；自己手动组装对应的schema文件，将schema导入到数据库；然后将组装为特定格式的csv文件中的数据，导入到图库中；

github地址： https://github.com/IBM/janusgraph-utils

优点：
1. 使用难度不高，让我们不用再去手写多线程的导入了；减少工作量
2. 直连hbase和es，相对于前两种减少了对应的gremlin server和janus server的网络交互
3. 支持通过配置文件自动创建Janusgraph schema和index
4. 可配置化的线程池大小和每次批量提交的数量

问题：
1. schema和csv文件也是要用户组装出对应格式
2. 相对于前两种方式性能提升有限，主要是少了一层网络交互。多线程和批量提交，前两种都可以手动去实现；还需要引入一个新的组件
3. 支持janus版本较低，可以手动升级，不难
4. 相对于下面方案，性能还是较低

## 官网提供的Bulk Loader

官方提供的批量导入方式；需要hadoop集群和spark集群的支持；

hadoop和spark集群配置，可以看官网：https://docs.janusgraph.org/advanced-topics/hadoop/

该方案对导入的数据有着严格的要求，支持多种数据格式：json、csv、xml、kryo；数据要求：节点、节点对应的属性、节点对应的边需要在一行中（一个json中、一个xml项中）

数据案例： 下面给一下官网的案例，在data目录下：

* json格式数据
```json
{"id":2,"label":"song","inE":{"followedBy":[{"id":0,"outV":1,"properties":{"weight":1}},{"id":323,"outV":34,"properties":{"weight":1}}]},"outE":{"followedBy":[{"id":6190,"inV":123,"properties":{"weight":1}},{"id":6191,"inV":50,"properties":{"weight":1}}],"sungBy":[{"id":7666,"inV":525}],"writtenBy":[{"id":7665,"inV":525}]},"properties":{"name":[{"id":3,"value":"IM A MAN"}],"songType":[{"id":5,"value":"cover"}],"performances":[{"id":4,"value":1}]}}
```

* xml格式数据
```xml
<node id="4"><data key="labelV">song</data><data key="name">BERTHA</data><data key="songType">original</data><data key="performances">394</data></node><node id="5"><data key="labelV">song</data><data key="name">GOING DOWN THE ROAD FEELING BAD</data><data key="songType">cover</data><data key="performances">293</data></node><node id="6"><data key="labelV">song</data><data key="name">MONA</data><data key="songType">cover</data><data key="performances">1</data></node><node id="7"><data key="labelV">song</data><data key="name">WHERE HAVE THE HEROES GONE</data><data key="songType"></data><data key="performances">0</data></node>
```

* csv格式数据
```csv
2,song,IM A MAN,cover,1 followedBy,50,1|followedBy,123,1|sungBy,525|writtenBy,525       followedBy,1,1|followedBy,34,1
```

数据整理方案： spark的cogroup的作用就是将多个 RDD将相同的key jion成一行，从而使用csv格式进行导入，操作实示例如下：

```java
val rdd1 = sc.parallelize(Array(("aa",1),("bb",2),("cc",6)))
val rdd2 = sc.parallelize(Array(("aa",3),("dd",4),("aa",5)))
rdd1.cogroup(rdd2).collect()

output:
(aa,(CompactBuffer(1),CompactBuffer(3, 5)))
(dd,(CompactBuffer(),CompactBuffer(4)))
(bb,(CompactBuffer(2),CompactBuffer()))
(cc,(CompactBuffer(6),CompactBuffer()))
```

> 此处的原始数据的准备需要细致，一致性保证完全依赖于原始数据的一致性保证；

这里大家可以参考360对这方面的处理，转化代码github地址：https://github.com/360jinrong/janusgraph-data-importer

# 应用场景

## 图库已存在数据

对于方案四：bulk loader 可能就无法使用了；我们可以采取两种方式：

1. 使用第一种方案和第二种方案进行导入（注意数据一致性）
2. 整体迁移图库，将图库中现有数据和将要导入的数据整体迁移到另外一个新图库，就可以使用方案四进行导入

## 图库初始化或迁移

1. 数据量小，建议使用方案一：基于JanusGraph Api的数据导入 和 方案二：基于Gremlin Server的批量导入 和方案三：IBM的janusgraph-utils；
2. 数据量大，建议使用方案四：bulk loader

基于业务数据量给一些建议：

* 小数据量（亿级以下）： 直接janusgraph api 或者 gremlin server导入即可，几小时就ok了； 如果想要更快可以使用另外的方式，只是会增加人力成本；
* 中等数据量（十亿级以下）：数据充分探查，开启storage.batch-loading完全可以支持，使用api，2天左右可以完成全量的数据导入
* 大数据量（百亿级数据）：推荐采用bulk load方式，配置hadoop集群，使用spark cluster导入

# 参考文献

* [JanusGraph官网 - Batch-Loading](https://docs.janusgraph.org/operations/bulk-loading/#batch-loading)
* [JanusGraph导入优化方案汇总](http://how2up.com/archives/janusgraph-dao-ru-you-hua-fang-an-hui-zong)
* [简书 - JanusGraph TinkerPop’s Hadoop-Gremlin](https://www.jianshu.com/p/68117c2082a9)
* [简书 - 百亿级图数据JanusGraph迁移之旅](https://www.jianshu.com/p/f372f0ef6c42)
* [GitHub - 360digitech/janusgraph-data-importer](https://github.com/360digitech/janusgraph-data-importer)