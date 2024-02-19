---
title: Jackson Framework
tags: [java, jackson]
sidebar_label: Framework Introduction
sidebar_position: 6
---

# Jackson的组成部分

Jackson的核心模块由三部分组成（从Jackson 2.x开始）：

1. **jackson-annotations**：注解包，提供标准的Jackson注解功能。
2. **jackson-core**：核心包，定义了低级流Streaming API，提供基于**流模式**解析。Jackson内部实现正是通过高性能的流模式API的JsonGenerator和JsonParser来生成和解析json。
3. **jackson-databind**：数据绑定包，实现了数据绑定（和对象序列化）支持，它依赖于Streaming和Annotations包。提供基于“**对象绑定**”解析的API（ObjectMapper）和"树模型"解析的API（JsonNode）；基于“对象绑定”解析的API和”树模型“解析的API依赖基于“流模式”解析的API。

# 参考资料

* [GitHub - FasterXML/jackson](https://github.com/FasterXML/jackson)
* [GitHub - FasterXML/jackson wiki](https://github.com/FasterXML/jackson/wiki)
* [GitHub - FasterXML/jackson-doc](https://github.com/FasterXML/jackson-docs)