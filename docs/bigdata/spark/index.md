# 介绍

> Apache Spark is a unified analytics engine for large-scale data processing. It provides high-level APIs in Java, Scala, Python and R, and an optimized engine that supports general execution graphs. It also supports a rich set of higher-level tools including Spark SQL for SQL and structured data processing, pandas API on Spark for pandas workloads, MLlib for machine learning, GraphX for graph processing, and Structured Streaming for incremental computation and stream processing.

Apache Spark 是用于大规模数据处理的统一分析引擎。 它提供了 Java、Scala、Python 和 R 的高级 API，以及支持通用执行图的优化引擎。 它还支持一组丰富的高级工具，包括用于 SQL 和结构化数据处理的 Spark SQL、用于 pandas 工作负载的 Spark 上的 pandas API、用于机器学习的 MLlib、用于图形处理的 GraphX 以及用于增量计算和流处理的 Structured Streaming。

# RDD

> A resilient distributed dataset (RDD), which is a collection of elements partitioned across the nodes of the cluster that can be operated on in parallel. RDDs are created by starting with a file in the Hadoop file system (or any other Hadoop-supported file system), or an existing Scala collection in the driver program, and transforming it. Users may also ask Spark to persist an RDD in memory, allowing it to be reused efficiently across parallel operations. Finally, RDDs automatically recover from node failures.

弹性分布式数据集 (RDD)，是一个跨集群节点分区可并行操作的元素集合。 RDD 是通过从 Hadoop 文件系统（或任何其他 Hadoop 支持的文件系统）中的文件或驱动程序中现有的 Scala 集合开始并对其进行转换来创建的。 用户还可以要求 Spark 将 RDD 持久保存在内存中，允许它在并行操作中有效地重用。 最后，RDD 会自动从节点故障中恢复。

## Spark编程思想

Spark的共享变量包含两种：广播变量，用于缓存到所有节点的内存；累加器变量，用于计数、求和等。

代码写入一个惰性求值的驱动程序（driver program）中，通过一个动作（action），驱动代码被分发到集群上，由各个RDD分区上的worker来执行。然后结果会被发送回驱动程序进行聚合或编译。本质上，驱动程序创建一个或多个RDD，调用操作来转换RDD，然后调用动作处理被转换后的RDD。步骤大体如下：

1. 定义一个或多个RDD，可以通过获取存储在磁盘上的数据（HDFS，Cassandra，HBase，Local Disk），并行化内存中的某些集合，转换（transform）一个已存在的RDD，或者，缓存或保存。
2. 通过传递一个闭包（函数）给RDD上的每个元素来调用RDD上的操作。Spark提供了除了Map和Reduce的80多种高级操作。
3. 使用结果RDD的动作（action）（如count、collect、save等）。动作将会启动集群上的计算。

当Spark在一个worker上运行闭包时，闭包中用到的所有变量都会被拷贝到节点上，但是由闭包的局部作用域来维护。Spark提供了两种类型的共享变量，这些变量可以按照限定的方式被所有worker访问。广播变量会被分发给所有worker，但是是只读的。累加器这种变量，worker可以使用关联操作来“加”，通常用作计数器。

## Spark执行

本质上，Spark应用作为独立的进程运行，由驱动程序中的SparkContext协调。这个context将会连接到一些集群管理者（如YARN），这些管理者分配系统资源。集群上的每个worker由执行者（executor）管理，执行者反过来由SparkContext管理。执行者管理计算、存储，还有每台机器上的缓存。

重点要记住的是应用代码由驱动程序发送给执行者，执行者指定context和要运行的任务。执行者与驱动程序通信进行数据分享或者交互。驱动程序是Spark作业的主要参与者，因此需要与集群处于相同的网络。这与Hadoop代码不同，Hadoop中你可以在任意位置提交作业给JobTracker，JobTracker处理集群上的执行。

# 参考资料

* [Apache Spark官网](https://spark.apache.org/)
* [Github - apache/spark](https://github.com/apache/spark)