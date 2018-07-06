# influxdb_source_analyse
influxdb源码解析


# influxdb(入门篇)
 ###  特点介绍
 
 ### 专业术语解释
 -  database
  ```
      故名思义，和mysql中的database差不多
  ```
 -  retention policy
  ```
      保留策略，表示数据的保存在服务器上的时间 也可以简称为RP，在集群中还包括包含的副本数目（replication factor）
      以及shard group的时间范围(shard group duration)
  ```
  -  Metric
  
      |字段|类型|举例|备注|
      |:-|:-|:-|:-|
      |Name|string|cpuUsage|名称|
      |Tags|map[string]string|host="ali_1"|索引|
      |Fields|map[string]interface|used=15|不准索引|
      |TimeStamp|time|2135416243314|时间戳主线|
    ```
        其中Name的部分也叫做measurement  类似于mysql中表名   Metric表示整个表
        Tags 是一个map  通过他来索引，fields表示表中的数值字段
        measurement: Metric中的Name 可以理解为表名
        fields：
        field key :  field key是字符串且保存在metadata中
        field value: field value才是真正的数据，可以是字符串，浮点数，整数，布尔型数据。
        field set: 数据点上field key和field value的集合
        tags: tags在InfluxDB的数据中是可选的，存储常用的metadata，tags会被索引
        tag key:tag key是字符串，存在metadata中
        tag value:tag value是字符串，存在metadata中。
        tag set:数据点上tag key和tag value的集合
        timeStamp:数据点关联的日期和时间，在InfluxDB里的所有时间都是UTC的
        series:一个特定的series由measurement，tag set和retention policy组成。 fields set 不属于series
           series1:rp_cpuUsage_host_ali_1
           series2:rp_cpuUsage_host_ali_2
    ```

  -  point
   ```
      InfluxDB数据结构的一部分由series中的的一堆field组成。 每个点由其series和timestamp唯一标识
       point:rp_cpuUsage_host_ali_1_2018_11_22_33_44:[]
   ```
  -  schema
   ```
      描述数据在数据库中是如何组织的，InfluxDB的schema的基础是database，retention policy，series，
      measurement，tag key，tag value以及field keys
   ```
  -  node & server
   ```
      表示influxd进程，一台机器上就一个influxd进程
   ```
  -  shardGroup & shardDuration  &shard
   ```
      
   ```
  -  TSM
   ```
      
   ```
   
