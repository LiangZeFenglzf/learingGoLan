LOGstash

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

