ElasticSearch

[TOC]



# concept

index  和 document概念   用Mysql的数据表和数据row做类比，

index pattern[后面rename成data views]可以类比成数据库 

为什么如此类比因为es提供了sql 查询语言

[]: https://www.elastic.co/guide/en/elasticsearch/reference/current/sql-overview.html
[]: https://www.elastic.co/guide/en/elasticsearch/reference/current/sql-getting-started.html



[倒排索引分词]: https://blog.csdn.net/jiaojiao521765146514/article/details/83750548?spm=1001.2101.3001.6650.5&amp;utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-5-83750548-blog-107901095.pc_relevant_default&amp;depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-5-83750548-blog-107901095.pc_relevant_default&amp;utm_relevant_index=8	"先分词再倒排索引"

![image-20220810150139710](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220810150139710.png)

# Confiugure

![image-20220810151152629](C:\Users\Administrator\Downloads\ElasticSearch.assets\image-20220810151152629.png)

[通过命令行ES_PATH_CONF修改默认配置路径]: https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html

1. 重要配置

   

[数据和日志路径]: https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#path-settings	"默认设置为home/data homr.logs"

![image-20220810151737828](C:\Users\Administrator\Downloads\ElasticSearch.assets\image-20220810151737828.png)



[node的数据闪布在每一个path]: https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#_multiple_data_paths	"shard的概念"

  ![image-20220810152040363](C:\Users\Administrator\Downloads\ElasticSearch.assets\image-20220810152040363.png)

使用前提是每一个路径跑一个Node  有几个路径那就需要几个Node

[高可用情况下需要设置的network host]: https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#network.host	"只有一个server是不需要配置这个的默认配置为127.0.0.1"
[设置jvm heap size]: https://www.elastic.co/guide/en/elasticsearch/reference/current/advanced-configuration.html#set-jvm-heap-size	"大小值保持一致"

![image-20220810152827031](C:\Users\Administrator\Downloads\ElasticSearch.assets\image-20220810152827031.png)

关于Heap size的监控，使用Virtual vm

[节点类型]: https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html

coordinating node每个节点都算是一个协作节点

![image-20220810153729411](C:\Users\Administrator\Downloads\ElasticSearch.assets\image-20220810153729411.png)

node.nodes出现但是指定了空列表代表这只是一个协作节点，但是这是不允许的。这样的节点需要足够的资源去处理  收集阶段。

scatter是拆分阶段。请求分发到对应数据节点，节点本地执行后返回结果集而后进入收集阶段。

[添加节点进集群]: https://www.elastic.co/guide/en/elasticsearch/reference/current/starting-elasticsearch.html#_enroll_nodes_in_an_existing_cluster_4	"配置transport.port，取消注释"

![image-20220810160026826](C:\Users\Administrator\Downloads\ElasticSearch.assets\image-20220810160026826.png)

```
bin\elasticsearch-create-enrollment-token -s node
赋值token 然后后运行时加cmd参数
```

```
bin\elasticsearch --enrollment-token <enrollment-token>
```

# 倒排索引和分词的关系

![image-20220810161018740](C:\Users\Administrator\Downloads\ElasticSearch.assets\image-20220810161018740.png)

[^]: 文本分词是为了添加倒排索引，倒排索引是为了查询文档

[]: https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules-analysis.html

# 数据建模Mapping

![image-20220810161240938](C:\Users\Administrator\Downloads\ElasticSearch.assets\image-20220810161240938.png)

是定义一个文档所包含的字段是如何被存储被索引的处理过程。

显示mapping

[更新Index字段，查询指定字段api]: https://www.elastic.co/guide/en/elasticsearch/reference/current/explicit-mapping.html
[映射参数copy to ,dynamic,enabled字段]: https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-params.html	"enabled是代表展示与否，dynamic代表后续mapping是否可变化"

# Index template

[]: https://www.elastic.co/guide/en/elasticsearch/reference/current/index-templates.html

------

索引模板：告诉es创建index时的配置



## index pattern已弃用，rename为data views

# ES集群管理软件

[]: https://github.com/lmenezes/cerebro/releases

