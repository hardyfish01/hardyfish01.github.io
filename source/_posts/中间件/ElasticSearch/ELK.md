---
title: ELK
categories: 
- 中间件
- Elasticsearch
---

ELK 指的是一套解决方案，是 Elasticsearch、Logstash 和 Kibana 三种软件产品的首字母缩写，Beats 是 ELK 协议栈的新成员。

- E：代表 Elasticsearch，负责日志的存储和检索；
- L：代表 Logstash，负责日志的收集、过滤和格式化；
- K：代表 Kibana，负责日志数据的可视化；
- Beats：是一类轻量级数据采集器；

<img src="https://img-blog.csdnimg.cn/90d4902ab8434879b42b28affc35ba1f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5pyI5Ly06aOe6bG8,size_14,color_FFFFFF,t_70,g_se,x_16" style="zoom:50%;" />

<img src="https://img-blog.csdnimg.cn/3f4edce9653443f08461eae8f831b34c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5pyI5Ly06aOe6bG8,size_20,color_FFFFFF,t_70,g_se,x_16" style="zoom:50%;" />

Beats 的系统性能开销更小，所以应用服务器性能开销可以忽略不计；另一方面，Beats 可以作为数据采集插件形式工作，可以按需启用 Beats 下不同功能的插件，更灵活，扩展性更强。

例如，应用服务器只启用 Filebeat，则只收集日志文件数据，如果某天需要收集系统性能数据时，再启用 Metricbeat 即可，并不需要太多的修改和配置。

如果日志量巨大，可能还会引入Kafka用以均衡网络传输，从而降低了网络闭塞，尤其是丢失数据的可能性；另一方面，这样可以系统解耦，具有更好的灵活性和扩展性。

<img src="https://img-blog.csdnimg.cn/b0488c15e50942499cfa32ced5271643.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5pyI5Ly06aOe6bG8,size_20,color_FFFFFF,t_70,g_se,x_16" style="zoom:50%;" />

**ELk版本**

官网下载地址：https://www.elastic.co/downloads

- elasticsearch：https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.6.2.tar.gz
- kibana：https://artifacts.elastic.co/downloads/kibana/kibana-5.6.2-linux-x86_64.tar.gz
- logstash：https://artifacts.elastic.co/downloads/logstash/logstash-5.6.2.tar.gz
- filebeat：https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-5.6.2-linux-x86_64.tar.gz

历史版本汇集页：https://www.elastic.co/downloads/past-releases