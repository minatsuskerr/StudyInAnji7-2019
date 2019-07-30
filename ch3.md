# HDInsight

HDInsight就是微软云中的集群，具有以下优点：

+ 无需硬件成本
+ 按需使用付费
+ 部署灵活

## Apache Spark

Apache Spark 是专为大规模数据处理而设计的快速通用的计算引擎，可用来构建大型的、低延迟的数据分析应用程序。

### 基本原理

将数据分为很小的片，用类型批处理的方法处理这部分小数据。

我们提交一个Spark作业之后，会根据使用的部署模式，分配到不同的工作节点，申请运行作业需要使用的资源。

### Spark for Java

一个应用对应一个**SparkContext**。

这里的应用包含一个**Driver Program**和多个**Executor**。

#### 通过RDD处理数据

RDD是Spark底层的分布式存储的数据结构，Spark所有的操作都是基于RDD。

RDD的创建方式有两种：

1. 并行化一个已经存在于程序中的集合，如：

~~~~scala
val arr = Array("cat", "dog", "lion", "monkey", "mouse")
// create RDD by collection
val rdd = sc.parallize(arr)    
~~~~

2. 读取外部存储系统上的数据集。如前面提到的Spark连接到CosmosDB进行数据读取。

RDD的操作函数主要分为两种

https://blog.csdn.net/yzh_1346983557/article/details/80973550

![preview](https://pic4.zhimg.com/v2-dc154708ba16034ed619bb9755fe27bb_r.jpg)

联系：

Transformation操作不会马上提交，只会记录这些操作，当遇到Action时，Spark才会形成一个job，从数据的构建开始，经过Transformation，直到Action操作。这些操作形成一个**有向无环图（DAG）**。

![img](https://pic4.zhimg.com/80/v2-fd1e4b80d8d32c214cc527e1359ae3bb_hd.jpg)

### 生成Spark Java项目

+ 创建Spark项目。

+ 编写自己的Java或者Scala类。

  示例代码：

  ~~~java
  public static void main(String [] arg){
          SparkConf conf=new SparkConf().setAppName("sparkTest");
          JavaSparkContext sc=new JavaSparkContext(conf);
          JavaRDD<String> javaRDD=sc.textFile("wasb:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv");
  
          JavaRDD<String> selectedJavaRDD=filterAccessHvac(javaRDD);
  
          selectedJavaRDD.saveAsTextFile("wasb:///output1");
      }
  
      private static JavaRDD<String> filterAccessHvac(JavaRDD<String> javaRDD){
          return javaRDD.filter(new Function<String, Boolean>() {
              @Override
              public Boolean call(String s) throws Exception {
                  return s.split(",")[6].length()==1;
              }
          });
      }
  ~~~

  

+ 编写好后配置解决方案。

+ 生成.jar可执行文件。

+ 上传到HDInsight存储账户中。

+ 通过curl命令行提交Spark作业运行。

  #### 提交批处理作业

  ```cmd
  curl -k --user "admin:Anji@12345678" -v -H "Content-Type: application/json" -X POST -d "{ \"file\":\"wasb:///wyj_test/HDInsightTest.jar\", \"className\":\"sample.TestToConnect\" }" "https://anji-spark-test.azurehdinsight.cn/livy/batches" -H "X-Requested-By: admin"
  ```

  #### 使用批ID检索特定批的状态

  ```cmd
  curl -k --user "admin:Anji@12345678" -v -X GET "https://anji-spark-test.azurehdinsight.cn/livy/batches/<ID>"
  ```

  