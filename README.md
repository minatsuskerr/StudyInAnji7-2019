# 课题目标

两个要求：

1. 从Azure CosmosDB读取数据做批量处理。
2. 从Azure EventHub读取流分析。

Spark+CosmosDB+EventHub实现的目标

引入数据，处理分析数据，即时查询。

![Azure Cosmos DB IoT åèä½ç³»ç»æ](https://docs.microsoft.com/zh-cn/azure/cosmos-db/media/use-cases/iot.png)

## lambda体系

### 概述

为了解决的问题：

如何实时地在任意大数据集上进行查询。

最简单的方法：Query=Function（ALL DATA）

![Lambaæ¶æ](https://img-blog.csdn.net/20150523220702753)

#### Batch Layer（批处理层）

Batch Layer的功能主要有两点：

- 存储数据集
- 在数据集上预先计算查询函数，构建查询所对应的View

#### Speed Layer（速度层）

Speed Layer用来处理增量的实时数据。

#### Serving Layer（服务层）

Lambda架构的Serving Layer用于响应用户的查询请求，合并Batch View和Realtime View中的结果数据集到最终的数据集。

### 基本原理

1. 所有**数据**同时推送到批处理层和速度层。
2. **批处理层**包含主数据集（不可变、仅限追加的原始数据集），并预先计算批处理视图。
3. **服务层**包含快速查询的批处理视图。
4. **速度层**补偿处理时间（针对服务层），只处理最新的数据。
5. 通过合并批处理视图和实时视图中的结果或者单独 ping 每个结果，可以应答所有查询。

![æ¾ç¤ºä½¿ç¨ Azure Cosmos DBãSpark å Azure Cosmos DB æ´æ¹æº API éå»º lambda ä½ç³»ç»æçç¤ºæå¾](https://docs.microsoft.com/zh-cn/azure/cosmos-db/media/lambda-architecture/lambda-architecture-re-architected.png)