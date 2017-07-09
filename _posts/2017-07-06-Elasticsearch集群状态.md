---
layout: post
title: Elasticsearch集群状态
category: blog
description: Elasticsearch集群状态
---

## 集群的一些概念
### 集群与节点

节点(node)是你运行的Elasticsearch实例。一个集群(cluster)是一组具有相同cluster.name的节点集合，他们协同工作，共享数据并提供故障转移和扩展功能，当有新的节点加入或者删除节点，集群就会感知到并平衡数据。集群中一个节点会被选举为主节点(master),它用来管理集群中的一些变更，例如新建或删除索引、增加或移除节点等;当然一个节点也可以组成一个集群。

### 节点通信

我们能够与集群中的任何节点通信，包括主节点。任何一个节点互相知道文档存在于哪个节点上，它们可以转发请求到我们需要数据所在的节点上。我们通信的节点负责收集各节点返回的数据，最后一起返回给客户端。这一切都由Elasticsearch透明的管理。

### 集群生态

1.同集群中节点之间可以扩容缩容, 
2.主分片的数量会在其索引创建完成后修正，但是副本分片的数量会随时变化。 
3.相同的分片不会放在同一个节点上.

## 集群状态查看
![](/images/posts/2017-07-06-Elasticsearch集群状态/cat_health.png)

或者

![](/images/posts/2017-07-06-Elasticsearch集群状态/cluster_health.png)

这两个REST API都可以用来查询集群的状态，其中[cat API](https://www.elastic.co/guide/en/elasticsearch/guide/current/_cat_api.html), [Cluster Health](https://www.elastic.co/guide/en/elasticsearch/guide/current/_cluster_health.html)

我们最关心的是status字段，不同颜色代表的意义如下:

* 绿色，最健康的状态，代表所有的分片包括备份都可用
* 黄色，基本的分片可用，但是备份不可用（也可能是没有备份）
* 红色，部分的分片可用，表明分片有一部分损坏。此时执行查询部分数据仍然可以查到，遇到这种情况，还是赶快解决比较好。

### 集群重启
* 停止所有节点服务

```
curl -XPOST 'http://xxx.xxx.xxx.xxx:9200/_cluster/nodes/_shutdown'
```

* 分别启动每一个节点

```
./bin/elasticsearch -d
```

* 查看健康度

```
[user_00@localhost ~]$ curl 'http://xxx.xxx.xxx.xxx:9200/_cluster/health?pretty'
{
  "cluster_name" : "imageIndex",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 4,
  "number_of_data_nodes" : 4,
  "active_primary_shards" : 5,
  "active_shards" : 5,
  "relocating_shards" : 0,
  "initializing_shards" : 5,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0
}
```

### 参考
[elasticsearch status red问题解决](https://www.liudon.org/1319.html)
[elasticsearch如何安全重启节点](http://zhaoyanblog.com/archives/831.html)
[how-to-fix-your-elasticsearch-cluster-stuck-in-initializing-shards-mode](https://t37.net/how-to-fix-your-elasticsearch-cluster-stuck-in-initializing-shards-mode.html)