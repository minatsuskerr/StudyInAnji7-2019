# CosmosDB

## 特点

+ NoSQL
+ 分布式
+ 高吞吐、低延迟

有了这些特点，我们可以将CosmosDB用于IoT的远程信息处理的数据加载。

## CosmosDB实现Lambda体系

https://docs.microsoft.com/zh-cn/azure/cosmos-db/lambda-architecture

### 需要使用的工具

+ CosmosDB
+ HDInsight集群
+ Spark连接器
+ Event Hub

### 速度层——流式数据分析

![çªåºæ¾ç¤º lambda ä½ç³»ç»æçæ°æ°æ®ãéåº¦å±åä¸»æ°æ®éé¨åçç¤ºæå¾](https://docs.microsoft.com/zh-cn/azure/cosmos-db/media/lambda-architecture/lambda-architecture-change-feed.png)

#### CosmosDB更改源

它的作用是当容器中发生任何变化，包括插入和更新时，会自动执行指定的操作。

##### 如何使用更改源

可以创建小型的反应式 Azure Functions，每当 Azure Cosmos 容器的更改源中出现新事件时，就会将其触发。

##### 创建一个Function

https://docs.microsoft.com/zh-cn/azure/azure-functions/functions-bindings-cosmosdb-v2#trigger---java-example

+ 在function.json里绑定以及使用绑定的Java函数。
+ 在本地编写函数，并部署到Azure。

https://docs.microsoft.com/zh-cn/azure/azure-functions/functions-create-first-java-maven

### 批处理层和服务层

![çªåºæ¾ç¤º lambda ä½ç³»ç»æçæ¹å¤çå±åæå¡å±çç¤ºæå¾](https://docs.microsoft.com/zh-cn/azure/cosmos-db/media/lambda-architecture/lambda-architecture-batch-serve.png)

从载入数据后开始，可以使用HDInsight执行从批处理层到服务层的预先计算功能。

也就是，我们可以使用HDInsight预先计算存储的主数据集，并将处理好的数据存储到CosmosDB集合中，也就是提供给**服务层**中的**批处理视图**。

#### 使用Spark连接器

~~~~scala
// Connect to Cosmos DB Database
val readConfig2 = Config(Map("Endpoint" -> "https://doctorwho.documents.azure.com:443/",
"Masterkey" -> "le1n99i1w5l7uvokJs3RT5ZAH8dc3ql7lx2CG0h0kK4lVWPkQnwpRLyAN0nwS1z4Cyd1lJgvGUfMWR3v8vkXKA==",
"Database" -> "DepartureDelays",
"preferredRegions" -> "Central US;East US 2;",
"Collection" -> "flights_pcoll", 
"SamplingRatio" -> "1.0"))

// Create collection connection 
val coll = spark.sqlContext.read.cosmosDB(readConfig2)
coll.createOrReplaceTempView("c")

// Run, get row count, and time query
val originSEA = spark.sql("SELECT c.date, c.delay, c.distance, c.origin, c.destination FROM c WHERE c.origin = 'SEA'")
originSEA.createOrReplaceTempView("originSEA")
~~~~

## 如何连接到CosmosDB

核心对象：**DocumentClient**

创建DocumentClient直接连接CosmosDB，只需要输入url和密钥。

![1564384066944](C:\Users\kssn5\AppData\Roaming\Typora\typora-user-images\1564384066944.png)

可以查询指定容器的集合：**DocumentClient.queryDocuments**

可以修改指定集合的内容：**DocumentClient.replaceDocuments**