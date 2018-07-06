# influxdb_source_analyse
influxdb源码解析
# 时序数据库背景介绍
   ### 时序数据库遇到的挑战
   ```
    1.时序数据的写入 ：如何支持每秒钟上千万上亿数据点的写入。
    2.时序数据的读取 ：又如何支持在秒级对上亿数据的分组聚合运算。
    3.成本敏感 ：由海量数据存储带来的是成本问题。如何更低成本的存储这些数据，将成为时序数据库需要解决的重中之重。
   ```
   ### 时序数据库特点
   ```
    1.读少写多；数据几乎不被更新，或者删除（只有删除过期数据时），新增数据是按时间来说最近的数据
    2.数据可以以时间为主线排序
    3.单条数据并不重要；同样的数据出现多次，则认为是同一条数据
   ```
  ### 时序数据分类
  ```
  1. 按照特性
  高频率低保留期（数据采集，实时展示）---->influxdb
  低频率高保留期（数据展现、分析） ----->ES
  2.按照频度
  规则间隔（数据采集）  ----->定期采样数据
  不规则间隔（事件驱动）----->事件型数据
  ```
  ### 时序数据库种类和比较 
  
  
  ### 时序数据库存储角度(https://www.ctolib.com/topics-117377.html)
  
 
 
 
# influxdb(入门篇)
 ###  特点介绍
  - 多写少读基本不删除不更新
  - 实时聚合计算能力要强
 ### 专业术语解释
 -  database
  ```
      故名思义，和mysql中的database差不多;在 InfluxDB 中可以创建多个数据库，
      不同数据库中的数据文件是隔离存放的，存放在磁盘上的不同目录
  ```
 -  retention policy
  ```
      保留策略，表示数据的保存在服务器上的时间 也可以简称为RP，在集群中还包括包含的副本数目（replication factor）
      以及shard group的时间范围(shard group duration)
      每个数据库刚开始会自动创建一个默认的存储策略 autogen，数据保留时间为永久，之后用户可以自己设置，
      例如保留最近2小时的数据。插入和查询数据时如果不指定存储策略，则使用默认存储策略，且默认存储策略可以修改。
      InfluxDB 会定期清除过期的数据。
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
        measurement: Metric中的Name 可以理解为表名,测量指标名，例如 cpu_usage 表示 cpu 的使用率。
        fields：
        field key :  field key是字符串且保存在metadata中
        field value: field value才是真正的数据，可以是字符串，浮点数，整数，布尔型数据。
        field set: 数据点上field key和field value的集合
        tags: tags在InfluxDB的数据中是可选的，存储常用的metadata，tags会被索引
        tag key:tag key是字符串，存在metadata中
        tag value:tag value是字符串，存在metadata中。
        tag set:数据点上tag key和tag value的集合,tags 在 InfluxDB 中会按照字典序排序，
                不管是 tagk 还是 tagv，只要不一致就分别属于两个 key，例如 host=server01,region=us-west 
                和 host=server02,region=us-west 就是两个不同的 tag set
        timeStamp:数据点关联的日期和时间，在InfluxDB里的所有时间都是UTC的
        series:一个特定的series由measurement，tag set和retention policy组成。 fields set 不属于series
           series1:rp_cpuUsage_host_ali_1
           series2:rp_cpuUsage_host_ali_2
    ```
  - series
   ```
      在同一个 database 中，retention policy、measurement、tag sets 完全相同的数据同属于一个 series，
      同一个 series 的数据在物理上会按照时间顺序排列存储在一起;
      series 的 key 为 measurement + 所有 tags 的序列化字符串，这个 key 在之后会经常用到
      type Series struct {
         mu          sync.RWMutex
         Key         string              // series key
         Tags        map[string]string   // tags
         id          uint64              // id
         measurement *Measurement        // measurement
       }
   ```
  -  point
   ```
      InfluxDB数据结构的一部分由series中的的一堆field组成。 每个点由其series和timestamp唯一标识
      InfluxDB 中单条插入语句的数据结构，series + timestamp 可以用于区别一个 point，也就是说一个 point 
      可以有多个 field name 和 field value
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
  -  Line protocal
   ```
   在 InfluxDB 中，我们可以粗略的将要存入的一条数据看作一个虚拟的 key 和其对应的 value(field value)，格式如下：
   cpu_usage,host=server01,region=us-west value=0.64 1434055562000000000
   虚拟的 key 包括以下几个部分： database, retention policy, measurement, tag sets, field name, timestamp。
   database 和 retention policy 在上面的数据中并没有体现，通常在插入数据时在 http 请求的相应字段中指定。
      
   ```
  
  
  
  
# influxdb(单机版架构设计篇v0.12版本)
   针对这种用例进行优化需要进行一些权衡，主要是以牺牲功能为代价来提高性能。以下列出了一些权衡过的设计见解
  ### influxdb 设计权衡
   - 增加写性能
   ```
    对于时间序列用例，我们假设如果相同的数据被多次发送，那么认为客户端几次都是同一笔数据。
     方法：通过简化的冲突解决增加了写入性能
     妥协：不能存储重复数据;可能会在极少数情况下覆盖数据
   ```
   - 增加查询和写入性能
   ```
    删除是罕见的事情。当它们发生时，肯定会阻止大面积的旧数据的写入。
    方法：限制对删除，从而增加查询和写入性能
    妥协：删除功能受到很大限制
    同样更新也是一样
   ```
   - 在高负载操作下增加写和查的性能
   ```
   能够写入和查询数据比具有强一致性更重要。
   方法：写入和查询数据库可以由多个客户端和高负载完成
   妥协：如果数据库负载较重，查询返回可能不包括最近的点
   ```
  ### schema 相关设计权衡
  
  
  
  ### 
  
  
  文章版权归本人所有，欢迎转载，但未经作者同意必须保留此段声明，且在文章页面明显位置给出原文链接，否则保留追究法律责任的权利。
  
  
