---
本文档不关注各个组件的安全验证
---

LOGstash

[TOC]

字段引用和filter插件区别

[translate filter注意一下]: https://www.elastic.co/guide/en/logstash/current/lookup-enrichment.html#translate-def
[grok和delimiter的区别数据结构因行而异]: https://www.elastic.co/guide/en/logstash/current/field-extraction.html#field-extraction
[具体接口使用在filter pludin有介绍]: https://www.elastic.co/guide/en/logstash/current/plugins-filters-dissect.html
[字段引用视为特殊的string值]: https://www.elastic.co/guide/en/logstash/8.3/configuration-file-structure.html
[固有字段@metadata字段]: https://www.elastic.co/guide/en/logstash/8.3/event-dependent-configuration.html	"输出时不用于展示，只适用于做条件筛查，通过设置true可展示该部分信息"
[event的固有字段]: https://www.elastic.co/guide/en/logstash/8.3/processing.html



![image-20220808141939360](C:\Users\Administrator\Downloads\LOGstash808.assets\image-20220808141939360.png)

![image-20220808142235181](C:\Users\Administrator\Downloads\LOGstash808.assets\image-20220808142235181.png)

![image-20220808142328605](C:\Users\Administrator\Downloads\LOGstash808.assets\image-20220808142328605.png)

[推荐不使用额外layer去做数据缓冲，PQ可]: https://www.elastic.co/guide/en/logstash/current/deploying-and-scaling.html#integrating-with-messaging-queues
[VirtualVM可视化heap空间]: https://www.elastic.co/guide/en/logstash/current/tuning-logstash.html#profiling-the-heap
[命令行参数]: https://www.elastic.co/guide/en/logstash/8.3/running-logstash-command-line.html#



运行端口9600

![image-20220811182714885](C:\Users\Administrator\Downloads\LOGstash808.assets\image-20220811182714885.png)

# intro

Logstash can dynamically unify data from disparate sources and normalize the data into destinations of your choice. Cleanse and democratize all your data for diverse advanced downstream analytics and visualization use cases.

联合收集多个不同数据源的数据，操作数据发送给所选定的目的地。清理和细化数据方便不同的进一步的分析和可视化。

![image-20220812171815928](C:\Users\Administrator\Downloads\LOGstash808.assets\image-20220812171815928.png)

A Logstash pipeline has two required elements, `input` and `output`, and one optional element, `filter`. The input plugins consume data from a source, the filter plugins modify the data as you specify, and the output plugins write the data to a destination.

# Configure

[不同配置文件的配置目的]: https://www.elastic.co/guide/en/logstash/current/config-setting-files.html



[beats input]: https://www.elastic.co/guide/en/logstash/current/advanced-pipeline.html#_configuring_logstash_for_filebeat_input	"input pludin"

| 检查配置，报错         | 自动加载新配置            |
| ---------------------- | ------------------------- |
| --config.test_and_exit | --config.reload.automatic |

```
Because you’ve enabled automatic config reloading, you don’t have to restart Logstash to pick up your changes.
```

pipelines.yml是配置Multiple pipes。

```
run Logstash and specify the configuration file with the `-f` flag.
```

[配置pipeline值的类型语法]: https://www.elastic.co/guide/en/logstash/current/configuration-file-structure.html#plugin-value-types
[export环境变量，类似keystore]: https://www.elastic.co/guide/en/logstash/current/environment-variables.html	"keystore键值对，参数引用 https://www.elastic.co/guide/en/logstash/current/keystore.html"



## re-configure

https://www.elastic.co/guide/en/logstash/current/advanced-pipeline.html#configuring-filebeat

![image-20220812172657548](C:\Users\Administrator\Downloads\LOGstash808.assets\image-20220812172657548.png)

```
sudo ./filebeat -e -c filebeat.yml -d "publish"
```

logstash不管是否是自动重启，只要重启,beats就必须重启；

codecs可以额外添加在Input/out https://www.elastic.co/guide/en/logstash/current/pipeline.html#_codecs 

# WorkTheory

[^process pipeline ]: https://www.elastic.co/guide/en/logstash/current/pipeline.html#_codecs
[^执行模型]: https://www.elastic.co/guide/en/logstash/current/execution-model.html

[默认使用in-mem队列，无法预防数据丢失]: https://www.elastic.co/guide/en/logstash/current/execution-model.html	"每个input独占thread执行"
[event的概念]: https://www.elastic.co/guide/en/logstash/current/event-dependent-configuration.html	"Inputs generate events, filters modify them, and outputs ship them elsewhere."

```
All events have properties. For example, an Apache access log has properties like status code (200, 404), request path ("/", "index.html"), HTTP verb (GET, POST), client IP address, and so forth. Logstash calls these properties "fields".
```

[pipeline-filter插件可使用field reference和conditionals expr]: https://www.elastic.co/guide/en/logstash/current/event-dependent-configuration.html#metadata	"meta字段"
[meta字段概念]: https://www.elastic.co/guide/en/logstash/current/event-dependent-configuration.html#metadata	"The contents of @metadata are not part of any of your events at output time, 只充当查询条件."
[]: 



# Setup and Run

filebeat配置setting.template.pattern之类的。
