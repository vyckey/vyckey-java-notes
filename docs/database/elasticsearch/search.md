---
title: EQL
tags: []
sidebar_label: EQL
sidebar_position: 1
---

[toc]

# ES查询语法

## 1.基本查询结构

```sql
GET /{索引名}/_search
{
  "_source" :[ ...需要返回的字段... ],
	"query" : { ...query子句... },
	"aggs" : { ..aggs子句..  },
	"sort" : { ..sort子句..  },
  "from" : 0,// 分页偏移量
  "size" : 10 // 分页大小
}
```

检索索引支持一到多个，对于单个索引采用 `GET /{索引名}/_search` 格式，对于多个索引可采用逗号或通配符匹配，例如 `GET /order1,order2/_search` 或 `GET /order*/_search` 。

执行查询语句，返回的样例JSON数据格式如下：

```sql
{
  "took" : 12, // 查询消耗时间，单位毫秒 
  "timed_out" : false, // 查询是否超时
  "_shards" : { // 本次查询参与的ES分片信息，查询中参与分片的总数，以及这些分片成功了多少个失败了多少个
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : { // hits字段包含我们搜索匹配的结果
    "total" : { // 匹配到的文档总数
      "value" : 15, // 找到15个文档
      "relation" : "eq"
    },
    "max_score" : 1.0, // 匹配到的最大分值
    "hits" : [ 
        {...}, // 这里就是我们具体的搜索结果，是一个JSON文档数组
        {...}
    ]
  },
  "aggregations": {
    ...
  }
}
```

## 2.查询子句

query子句主要用来编写类似SQL的Where语句，支持布尔查询（and/or）、IN、全文搜索、模糊匹配、范围查询（大于小于）。

`query` 和 `filter` 的区别点在于 `filter` 仅做条件过滤，而 `query` 会额外计算文档的相关性，即打分 `_score` 。

### 2.1 全匹配查询

#### match_all 查询

官方文档：[Match all query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-all-query.html)

匹配所有文档。

```sql
GET /_search
{
    "query": {
        "match_all": {}
    }
}

GET /_search
{
  "query": {
    "match_all": { "boost" : 1.2 }
  }
}
```

#### match_none 查询

与 `match_all` 查询相关，不匹配任何文档。

```sql
GET /_search
{
  "query": {
    "match_none": {}
  }
}
```

### 2.2 组合查询

官网文档：[Compound queries](https://www.elastic.co/guide/en/elasticsearch/reference/current/compound-queries.html)

复合查询包装其他复合查询或叶查询，以组合它们的结果和分数，改变它们的行为，或者从查询切换到过滤上下文。

#### `bool` 查询

`bool` 查询用于组合多个叶或复合查询子句（`must` 、`should` 、`must_not` 、`filter`）进行过滤。需要注意的是 `must` 、`should` 、`must_not` 等同于逻辑操作 `and` 、`or` 、`not` 。其中 `must` 、`should` 会对打分贡献，而 `must_not` 、`filter` 仅用于过滤，不对打分有影响。

```sql
GET /_search
{
  "query": {
    "bool" : {
      "must" : {
        "term" : { "user.id" : "kimchy" }
      },
      "filter": {
        "term" : { "tags" : "production" }
      },
      "must_not" : {
        "range" : {
          "age" : { "gte" : 10, "lte" : 20 }
        }
      },
      "should" : [
        { "term" : { "tags" : "env1" } },
        { "term" : { "tags" : "deployed" } }
      ],
      "minimum_should_match" : 1,
      "boost" : 1.0
    }
  }
}
```

#### `boosting` 查询

如果有匹配内容对结果有正向（positive）的打分贡献，否则会有负向（negtive）的贡献。

```sql
GET /_search
{
  "query": {
    "boosting": {
      "positive": {
        "term": {
          "text": "apple"
        }
      },
      "negative": {
        "term": {
          "text": "pie tart fruit crumble tree"
        }
      },
      "negative_boost": 0.5
    }
  }
}
```

#### `constant_score` 查询

对另一个查询的进行包装，但仅在过滤器上下文中执行。所有匹配的文档都被赋予相同的“常量” `_score` 。

```sql
GET /_search
{
  "query": {
    "constant_score": {
      "filter": {
        "term": { "user.id": "kimchy" }
      },
      "boost": 1.2
    }
  }
}
```

#### `dis_max` 查询

接受多个查询并返回与任何查询子句匹配的任何文档的查询。虽然 `bool` 查询组合了所有匹配查询的分数，但 `dis_max` 查询使用单个最佳匹配查询子句的分数。

```sql
GET /_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "term": { "title": "Quick pets" } },
        { "term": { "body": "Quick pets" } }
      ],
      "tie_breaker": 0.7
    }
  }
}
```

#### `function_score` 询问

使用函数自定义主查询返回的分数，例如考虑流行度、最近程度、距离或通过脚本实现的自定义算法等因素。

```sql
GET /_search
{
  "query": {
    "function_score": {
      "query": { "match_all": {} },
      "boost": "5", 
      "functions": [
        {
          "filter": { "match": { "test": "bar" } },
          "random_score": {}, 
          "weight": 23
        },
        {
          "filter": { "match": { "test": "cat" } },
          "weight": 42
        }
      ],
      "max_boost": 42,
      "score_mode": "max",
      "boost_mode": "multiply",
      "min_score": 42
    }
  }
}
```

### 2.3 Term 精确匹配

官方文档：[Term-level queries](https://www.elastic.co/guide/en/elasticsearch/reference/current/term-level-queries.html)

#### `exists` 查询

返回包含字段的任何索引值的文档。

```sql
GET /_search
{
  "query": {
    "bool": {
      "must_not": {
        "exists": {
          "field": "user.id"
        }
      }
    }
  }
}
```

#### `fuzzy` 查询

返回包含与搜索词类似的词的文档。Elasticsearch 使用Levenshtein 编辑距离来测量相似性或模糊性。

```sql
GET /_search
{
  "query": {
    "fuzzy": {
      "user.id": {
        "value": "ki", // 必填，你希望找的值
        "fuzziness": "AUTO", // 允许匹配的最大编辑距离
        "max_expansions": 50,// 允许创建的最大变体个数，默认50
        "prefix_length": 0,// 字符前缀长度，默认0
        "transpositions": true,// 是否允许两个临近字符变换位置（ab→ba）
        "rewrite": "constant_score_blended"// 重写query的方法
      }
    }
  }
}
```

#### `ids` 查询

根据文档 ID 返回文档。

```sql
GET /_search
{
  "query": {
    "ids" : {
      "values" : ["1", "4", "100"]
    }
  }
}
```

#### `prefix` 查询

返回在提供的字段中包含特定前缀的文档。

```sql
GET /_search
{
  "query": {
    "prefix": {
      "user.id": {
        "value": "ki"
      }
    }
  }
}
```

#### `range` 查询

返回包含给定范围内的术语的文档。

```sql
GET /_search
{
  "query": {
    "range": {
      "age": {
        "gte": 10,
        "lte": 20,
        "boost": 2.0
      }
    }
  }
}
```

#### `regexp` 查询

返回包含与 正则表达式匹配的术语的文档。

```sql
GET /_search
{
  "query": {
    "regexp": {
      "user.id": {
        "value": "k.*y",
        "flags": "ALL",
        "case_insensitive": true,
        "max_determinized_states": 10000,
        "rewrite": "constant_score_blended"
      }
    }
  }
}
```

#### `term` 查询

返回在提供的字段中包含确切术语的文档。

```sql
GET /_search
{
  "query": {
    "term": {
      "user.id": {
        "value": "kimchy",
        "boost": 1.0
      }
    }
  }
}
```

#### `terms` 查询

返回在所提供的字段中包含一个或多个精确术语的文档。

```sql
GET /_search
{
  "query": {
    "terms": {
      "user.id": [ "kimchy", "elkbee" ],
      "boost": 1.0
    }
  }
}
```

#### `terms_set` 查询

返回在提供的字段中包含最少数量的精确术语的文档。您可以使用字段或脚本定义匹配术语的最小数量。

```sql
GET /job-candidates/_search
{
  "query": {
    "terms_set": {
      "programming_languages": {
        "terms": [ "c++", "java", "php" ],
        "minimum_should_match_field": "required_matches"
      }
    }
  }
}
```

#### `wildcard` 查询

返回包含与通配符模式匹配的术语的文档。

```sql
GET /_search
{
  "query": {
    "wildcard": {
      "user.id": {
        "value": "ki*y",
        "boost": 1.0,
        "rewrite": "constant_score_blended"
      }
    }
  }
}
```

### 2.4 全文匹配查询

官方文档：[Full text queries](https://www.elastic.co/guide/en/elasticsearch/reference/current/full-text-queries.html)

全文查询使您能够搜索分析的文本字段，例如电子邮件正文。使用在索引期间应用于字段的同一分析器来处理查询字符串。

#### match 查询

用于执行全文查询的标准查询，包括模糊匹配和短语或邻近查询。

```sql
GET /_search
{
  "query": {
    "match": {
      "message": {
        "query": "this is a test"
      }
    }
  }
}
```

#### match_bool_prefix 查询

创建一个bool查询，将每个术语作为term查询进行匹配，最后一个术语除外，它作为prefix查询 进行匹配。

```sql
GET /_search
{
  "query": {
    "match_bool_prefix" : {
      "message" : "quick brown f"
    }
  }
}
```

#### match_phrase 查询

类似于match查询，但用于匹配精确的短语或单词邻近匹配。

```sql
GET /_search
{
  "query": {
    "match_phrase": {
      "message": "this is a test"
    }
  }
}
```

#### match_phrase_prefix 查询

与查询类似match_phrase，但对最终单词进行通配符搜索。

```sql
GET /_search
{
  "query": {
    "match_phrase_prefix": {
      "message": {
        "query": "quick brown f"
      }
    }
  }
}
```

#### multi_match 查询

查询的多字段版本match。

```sql
GET /_search
{
  "query": {
    "multi_match" : {
      "query":    "this is a test", 
      "fields": [ "subject", "message" ] 
    }
  }
}
```

#### combined_fields 查询

匹配多个字段，就像它们已被索引到一个组合字段中一样。

```sql
GET /_search
{
  "query": {
    "combined_fields" : {
      "query":      "database systems",
      "fields":     [ "title", "abstract", "body"],
      "operator":   "and"
    }
  }
}
```

#### intervals 查询

全文查询，允许对匹配术语的顺序和接近度进行细粒度控制。

下面的 `intervals` 检索返回文档 `my_text` 字段中包含 `my favorite food` 没有任何的gap, 接着是 `hot water` or `cold porridge` 。这个检索将匹配 `my_text` 字段值 `my favorite food is cold porridge` 但是不会匹配 `when it's cold my favorite food is porridge` 。

```sql
GET /_search
{
  "query": {
    "intervals" : {
      "my_text" : {
        "all_of" : {
          "ordered" : true,
          "intervals" : [
            {
              "match" : {
                "query" : "my favorite food",
                "max_gaps" : 0,
                "ordered" : true
              }
            },
            {
              "any_of" : {
                "intervals" : [
                  { "match" : { "query" : "hot water" } },
                  { "match" : { "query" : "cold porridge" } }
                ]
              }
            }
          ]
        }
      }
    }
  }
}
```

#### query_string 查询

支持紧凑的 Lucene查询字符串语法，允许您在单个查询字符串中指定 `AND|OR|NOT` 条件和多字段搜索。仅供专家用户使用。

```sql
GET /_search
{
  "query": {
    "query_string": {
      "query": "(new york city) OR (big apple)",
      "default_field": "content"
    }
  }
}
```

#### simple_query_string 查询

更简单、更健壮的语法版本query_string，适合直接向用户公开。

```sql
GET /_search
{
  "query": {
    "simple_query_string" : {
        "query": "\"fried eggs\" +(eggplant | potato) -frittata",
        "fields": ["title^5", "body"],
        "default_operator": "and"
    }
  }
}
```

### 2.5 地址位置查询

参考文档：[Geo queries](https://www.elastic.co/guide/en/elasticsearch/reference/current/geo-queries.html)

### 2.6 形状查询

参考文档：[Shape queries](https://www.elastic.co/guide/en/elasticsearch/reference/current/shape-queries.html)

### 2.7 Join 查询

参考文档：[Joining queries](https://www.elastic.co/guide/en/elasticsearch/reference/current/joining-queries.html)

### 2.8 Span 查询

参考文档：[Span queries](https://www.elastic.co/guide/en/elasticsearch/reference/current/span-queries.html)

### 2.9 其他查询

* 正则表达式：[Regular expression syntax](https://www.elastic.co/guide/en/elasticsearch/reference/current/regexp-syntax.html)
* 特殊查询：[Specialized queries](https://www.elastic.co/guide/en/elasticsearch/reference/current/specialized-queries.html)
* [minimum_should_match 参数](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-minimum-should-match.html)
* [rewrite 参数](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-term-rewrite.html)

## 3.聚合子句

官方文档：[Aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html)

Elasticsearch 将聚合分为三类：

1. Metric：根据字段值计算指标（例如总和或平均值）的指标聚合。
2. Bucket：存储桶聚合，根据字段值、范围或其他条件将文档分组到存储桶（也称为存储桶）中。
3. Pipeline：管道聚合从其他聚合而不是文档或字段获取输入。

### 3.1 Metrics 聚合

#### 均值、求和、最大、最小、计数、加权平均

* [avg](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-avg-aggregation.html)
* [sum](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-sum-aggregation.html)
* [min](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-min-aggregation.html)
* [max](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-max-aggregation.html)
* [value_count](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-valuecount-aggregation.html)
* [Weighted avg](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-weight-avg-aggregation.html)

样例输入：

```sql
POST /exams/_search?size=0
{
  "aggs": {
    "avg_grade": { "avg": { "field": "grade" } },
    "max_price": { "max": { "field": "price" } },
    "weighted_grade": {
      "weighted_avg": {
        "value": {
          "field": "grade"
        },
        "weight": {
          "field": "weight"
        }
      }
    }
  }
}
```

输出结果：

```sql
{
  ...
  "aggregations": {
    "avg_grade": {
        "value": 75.0
    },
    "max_price": {
        "value": 200.0
    },
    "weighted_grade": {
      "value": 70.0
    }
  }
}
```

#### 多项统计

* [stats](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-stats-aggregation.html)：会返回 `min`, `max`, `sum`, `count`, `avg` 多项聚合指标。
* [extended_stats](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-extendedstats-aggregation.html)： `stats` 的扩展，会额外地包含 `sum_of_squares`, `variance`, `std_deviation` and `std_deviation_bounds` 。
* [matrix_stats](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-matrix-stats-aggregation.html)：计算指标包含： `count`, `mean`, `variance`, `skewness`, `kurtosits`, `covariance`, `correlation` 多项聚合指标。
* [boxplot](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-boxplot-aggregation.html)：返回一些重要的信息： `min`, `max`, `median`, `25th percentile`, `75th percentile` 。
* [type_count](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-cardinality-aggregation.html)：等同于SQL中的 `count(distinct field)` 操作。
* [median_absolute_deviation](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-median-absolute-deviation-aggregation.html)：中位绝对偏差聚合。
* [percentiles](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-percentile-aggregation.html)：百分数聚合。
* [percentile_ranks](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-percentile-rank-aggregation.html)：百分位数排名聚合。
* [string_stats](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-string-stats-aggregation.html)：`multi-value` 用于计算从聚合文档中提取的字符串值的统计信息： `count`, `min_length`, `max_length`, `avg_length`, `entropy` 。

```sql
POST /exams/_search?size=0
{
  "aggs": {
    "grades_stats": { "stats": { "field": "grade" } }
  }
}
```

```sql
{
  ...

  "aggregations": {
    "grades_stats": {
      "count": 2,
      "min": 50.0,
      "max": 100.0,
      "avg": 75.0,
      "sum": 150.0
    }
  }
}
```

#### 地理位置聚合

* [geo_bounds](https://www.elastic.co/guide/en/elasticsearch/reference/current/*search-aggregations-metrics-geobounds-aggregation.html)：地理位置边界。
* [geo_centroid](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-geocentroid-aggregation.html)：地址位置中间。
* [geo_line](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-geo-line.html)：地址位置连线。

#### 点和形状聚合

* [cartesian_bounds](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-cartesian-bounds-aggregation.html)：笛卡尔边界。
* [cartesian_centroid](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-cartesian-centroid-aggregation.html)：笛卡尔中心。

#### Top聚合

* [top_hits](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-top-hits-aggregation.html)：top_hits 度量聚合器跟踪正在聚合的最相关的文档。该聚合器旨在用作子聚合器，以便可以聚合每个存储桶的顶部匹配文档。
* [top_metrics](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-top-metrics.html)：top_metrics 聚合从具有最大或最小“排序”值的文档中选择指标。

#### 其他聚合

* [rate](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-rate-aggregation.html)：指标聚合只能在 `date_histogram` 或 `composite` 聚合内部使用。
* [scripted_metric](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-scripted-metric-aggregation.html)：使用脚本进行聚合运算。
* [t_test](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-ttest-aggregation.html)：执行统计假设检验的 t_test 指标聚合，其中检验统计量遵循从聚合文档中提取的数值的零假设下的学生 t 分布。


### 3.2 Bucket 聚合

官方文档：[Bucket aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket.html)

存储桶聚合不会像指标聚合那样计算字段的指标，而是创建文档存储桶。每个存储桶都与一个标准（取决于聚合类型）相关联，该标准确定当前上下文中的文档是否“落入”其中。换句话说，桶有效地定义了文档集。除了存储桶本身之外，bucket聚合还计算并返回“落入”每个存储桶的文档数量。

与聚合相反metrics，桶聚合可以保存子聚合。这些子聚合将针对由其“父”存储桶聚合创建的存储桶进行聚合。

有不同的存储桶聚合器，每个存储桶聚合器都有不同的“存储桶”策略。有些定义单个存储桶，有些定义固定数量的多个存储桶，还有一些在聚合过程中动态创建存储桶。

* [histogram](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-histogram-aggregation.html)：基于多桶值源的聚合，可应用于从文档中提取的数值或数值范围值。
* [range](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-range-aggregation.html)：按范围进行分桶。
* 更多地查看官方文档。

```sql
POST /sales/_search?size=0
{
  "aggs": {
    "prices": {
      "histogram": {
        "field": "price",
        "interval": 50
      }
    },
    "price_ranges": {
      "range": {
        "field": "price",
        "ranges": [
          { "to": 100.0 },
          { "from": 100.0, "to": 200.0 },
          { "from": 200.0 }
        ]
      }
    }
  }
}
```

```sql
{
  ...
  "aggregations": {
    "prices": {
      "buckets": [
        {
          "key": 0.0,
          "doc_count": 1
        },
        {
          "key": 50.0,
          "doc_count": 1
        },
        {
          "key": 100.0,
          "doc_count": 0
        },
        {
          "key": 150.0,
          "doc_count": 2
        },
        {
          "key": 200.0,
          "doc_count": 3
        }
      ]
    },
    "price_ranges": {
      "buckets": [
        {
          "key": "*-100.0",
          "to": 100.0,
          "doc_count": 2
        },
        {
          "key": "100.0-200.0",
          "from": 100.0,
          "to": 200.0,
          "doc_count": 2
        },
        {
          "key": "200.0-*",
          "from": 200.0,
          "doc_count": 3
        }
      ]
    }
  }
}
```

### 3.3 Pipeline 聚合

官方文档：[Pipeline aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-pipeline.html)

管道聚合作用于其他聚合（而不是文档集）生成的输出，从而将信息添加到输出树中。有许多不同类型的管道聚合，每种类型都计算来自其他聚合的不同信息，但这些类型可以分为两类：

1. Parent：一系列管道聚合，提供其父聚合的输出，并且能够计算新存储桶或新聚合以添加到现有存储桶。
2. Sibling：管道聚合提供同级聚合的输出，并且能够计算与同级聚合处于同一级别的新聚合。

`buckets_path` 管道聚合可以通过使用参数来指示所需指标的路径来引用执行计算所需的聚合。定义这些路径的语法可以在 下面的 `buckets_path` 语法部分找到。

管道聚合不能有子聚合，但根据类型，它可以在 `buckets_path` 允许链接管道聚合时引用另一个管道。例如，您可以将两个导数链接在一起来计算二阶导数（即导数的导数）。

#### `buckets_path` 语法

大多数的管道聚合需要另一个管道聚合作为他们的输入。输入聚合通过 `buckets_path` 参数定义，需要遵循下面的格式：

```
AGG_SEPARATOR       =  `>` ;
METRIC_SEPARATOR    =  `.` ;
AGG_NAME            =  <the name of the aggregation> ;
METRIC              =  <the name of the metric (in case of multi-value metrics aggregation)> ;
MULTIBUCKET_KEY     =  `[<KEY_NAME>]`
PATH                =  <AGG_NAME><MULTIBUCKET_KEY>? (<AGG_SEPARATOR>, <AGG_NAME> )* ( <METRIC_SEPARATOR>, <METRIC> ) ;
```

例如，该路径 `"my_bucket>my_stats.avg"` 将指向度量 `avg` 中的值 `"my_stats"` ，该度量包含在 `"my_bucket"` 存储桶聚合中。

这里还有一些例子：

* `multi_bucket["foo"]>single_bucket>multi_metric.avg` 会取多桶聚合 `"multi_bucket"` 中的  `"foo"` ，再取 `"single_bucket""foo"` 中的，`"multi_metric"` 的聚合 `avg` 的指标。
* `agg1["foo"]._count` 将获取多桶聚合 `multi_bucket` 中桶 `"foo"` 的 `_count` 指标。

路径是相对于管道聚合的位置而言的；它们不是绝对路径，并且该路径不能返回聚合树“向上”。

#### 聚合指令

聚合指令见[官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-pipeline.html)。

```sql
POST _search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "sales": {
          "sum": {
            "field": "price"
          }
        }
      }
    },
    "avg_monthly_sales": {
// tag::avg-bucket-agg-syntax[]               
      "avg_bucket": {
        "buckets_path": "sales_per_month>sales",
        "gap_policy": "skip",
        "format": "#,##0.00;(#,##0.00)"
      }
// end::avg-bucket-agg-syntax[]               
    }
  }
}
```

```sql
{
  "took": 11,
  "timed_out": false,
  "_shards": ...,
  "hits": ...,
  "aggregations": {
    "sales_per_month": {
      "buckets": [
        {
          "key_as_string": "2015/01/01 00:00:00",
          "key": 1420070400000,
          "doc_count": 3,
          "sales": {
            "value": 550.0
          }
        },
        {
          "key_as_string": "2015/02/01 00:00:00",
          "key": 1422748800000,
          "doc_count": 2,
          "sales": {
            "value": 60.0
          }
        },
        {
          "key_as_string": "2015/03/01 00:00:00",
          "key": 1425168000000,
          "doc_count": 2,
          "sales": {
            "value": 375.0
          }
        }
      ]
    },
    "avg_monthly_sales": {
      "value": 328.33333333333333,
      "value_as_string": "328.33"
    }
  }
}
```

## 4.检索API

### 4.1 排序

官方文档：[Sort search results](https://www.elastic.co/guide/en/elasticsearch/reference/current/sort-search-results.html)

允许您在特定字段上添加一种或多种排序。每种排序也可以颠倒。排序是在每个字段级别定义的， `_score` 的特殊字段名称按分数排序， `_doc` 按索引顺序排序。

```sql
GET /my-index-000001/_search
{
  "sort" : [
    { "post_date" : {"order" : "asc", "format": "strict_date_optional_time_nanos"}},
    "user",
    { "name" : "desc" },
    { "age" : "desc" },
    "_score"
  ],
  "query" : {
    "term" : { "user" : "kimchy" }
  }
}
```

### 4.2 分页

官方文档：[Paginate search results](https://www.elastic.co/guide/en/elasticsearch/reference/current/paginate-search-results.html)

默认情况下，搜索会返回前 `10` 个匹配结果。要翻阅更大的结果集，您可以使用搜索API 的 `from` 和 `size` 参数。`from` 参数定义要跳过的命中数，默认为 `0`。 `size` 参数是要返回的最大命中数。这两个参数一起定义结果页面。

```sql
GET twitter/_search
{
    "query": {
        "match": {
            "title": "elasticsearch"
        }
    },
    "search_after": [1463538857, "654323"],
    "sort": [
        {"date": "asc"},
        {"tie_breaker_id": "asc"}
    ]
}
```

### 4.3 按字段召回

默认情况下，搜索响应中的每个命中都包含文档 `_source` ，它是索引文档时提供的整个 JSON 对象。有两种推荐的方法可以从搜索查询中检索选定的字段：

1. 使用 `fields` 选项提取索引映射中存在的字段值。
2. 如果需要访问在索引时传递的原始数据，请使用 `_source` 选项。

您可以使用这两种方法，但首选 fields 选项，因为它会参考文档数据和索引映射。

**`fields` 选项**

```sql
POST my-index-000001/_search
{
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  },
  "fields": [
    "user.id",
    "http.response.*",         
    {
      "field": "@timestamp",
      "format": "epoch_millis" 
    }
  ],
  "_source": false
}
```
```sql
{
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "my-index-000001",
        "_id" : "0",
        "_score" : 1.0,
        "fields" : {
          "user.id" : [
            "kimchy"
          ],
          "@timestamp" : [
            "4098435132000"
          ],
          "http.response.bytes": [
            1070000
          ],
          "http.response.status_code": [
            200
          ]
        }
      }
    ]
  }
}
```

**`_source` 选项**

```sql
GET /_search
{
  "_source": false,
  // "_source": "obj.*",
  // "_source": [ "obj1.*", "obj2.*" ],
  // "_source": {
  //   "includes": [ "obj1.*", "obj2.*" ],
  //   "excludes": [ "*.description" ]
  // },
  // "stored_fields" : ["user", "postDate"],
  "query": {
    "match": {
      "user.id": "kimchy"
    }
  }
}
```

### 4.4 多个数据流或索引选择

检索索引支持一到多个，对于单个索引采用 `GET /{索引名}/_search` 格式，对于多个索引可采用逗号或通配符匹配，例如 `GET /order1,order2/_search` 或 `GET /order*/_search` 。

### 4.5 高亮

官方文档：[Highlighting](https://www.elastic.co/guide/en/elasticsearch/reference/current/highlighting.html#highlighting)

高亮使您能够从搜索结果的一个或多个字段中获取突出显示的片段，以便您可以向用户显示查询匹配的位置。当您请求高亮显示时，响应会为每个搜索命中包含一个附加突出显示元素，其中包括突出显示的字段和突出显示的片段。

### 4.6 搜索模板

搜索模板是一个存储的搜索，您可以使用不同的变量运行。如果您使用 Elasticsearch 作为搜索后端，则可以将搜索栏中的用户输入作为搜索模板的参数传递。这使您可以运行搜索，而无需向用户公开 Elasticsearch 的查询语法。

```sql
PUT _scripts/my-search-template
{
  "script": {
    "lang": "mustache",
    "source": {
      "query": {
        "match": {
          "message": "{{query_string}}"
        }
      },
      "from": "{{from}}",
      "size": "{{size}}"
    },
    "params": {
      "query_string": "My query string"
    }
  }
}
```

# 参考资料

* [Elasticsearch Guide](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started.html)