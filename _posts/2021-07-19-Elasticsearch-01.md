---
layout:     post
title:      Elasticsearch
subtitle:   实战知识
date:       2021-07-19
author:     chital
header-img: img/post-bg-alibaba.jpg
catalog: true
tags:
    - Elasticsearch
---

> *近三星期早餐：巴比馒头鸡肉包+茶叶蛋*

# Elasticsearch重要概念
* cluster，node，index，document，shards，replica

***
### Cluster

* 集群。由一个或多个节点组成；<br>

***
### Node

* 单个Elasticsearch实例。在大多数环境中，每个节点都在单独的盒子或虚拟机上运行；一个集群由一个或多个node组成；在测试的环境中，可以把多个node运行在一个server上；在实际的部署中，大多数情况还是需要一个server上运行一个node；可以分为以下几类：<br>

***
>**master-eligible**<br>
* 可以作为主node；成为主node后，可以管理整个cluster的设置以及变化：创建、更新、删除cluster；添加或删除node；为node分配shard；<br>

***
>**data**<br>
* 数据node；<br>

***
>**ingest**<br>
* 数据接入，比如pipepline；<br>

***
>**machine learning**<br>
* machine learning (Gold/Platinum License)<br>

***
一般来说，一个node可以有上面一种或者几种功能；可以在命令行或者Elasticsearch的配置文件（Elasticsearch.yml）定义：<br>
|**Node类型**|**配置参数**|**默认值**|
|----|----|----|
|master-eligible | node.master | true|
|data | node.data | true|
|ingest | node.ingest | true|
|machine learning | node.ml | true (除了OSS发布版）|
也可以让一个node做专有的功能及角色；如果node配置参数没有任何配置，那么可以认为这个node是作为一个coordination node：可以接受外部请求，并转发到相应节点处理；针对master node，有时候需要设置cluster.remote.connect:false；<br>

***
### Document
* Elasticsearch是面向文档的，索引或搜索的最小数据单元是文档；文档在Elasticsearch中有以下重要属性：<br>
    * 独立；文档包含字段（名称）及值；<br>
    * 分层；可以被视为文档中的文档；<br>
    * 结构灵活；文档不依赖预定义的架构；<br>
* 文档通常是JSON格式；JSON over HTTP是与ELasticsearch进行通信的最广泛使用的方式；<br>

***
### Type
* 类型是文档的逻辑容器，类似表是行的容器；将具有不同结构（模式）的文档放在不同类型中；<br>
* 每种类型的字段定义称为映射；<br>
* Elasticsearch中的数据库依旧还是需要mapping的；schema-less在Elasticsearch中的正确理解是：不需要事先定义一个类型关系数据库中的table才使用数据库；在Elasticsearch中，可以不开始定义一个mapping而直接写入指定的index中；这个index的mapping是动态生成的（可选择关闭），其中的数据项的每一个数据类型是动态识别的；此外，ta还有一个含义：同一个type，在以后的数据输入中，可能增加新的数据项，从而生产新的mapping；这个也是动态调整的；<br>
* schema-less意味着无需显式指定如何处理文档中可能出现的每个不同字段即可对文档建立索引。启动动态映射后，Elasticsearch自动检测并向索引添加新字段；<br>
* Elasticsearch 6.0之后，一个index只能含有一个type；原因：相同index的不同映射type中具有相同名称的字段是相同；在Elasticsearch索引中，不同映射type中具有相同名称的字段在Lucene中被同一个字段支持。在默认情况下是_doc。在未来8.0的版本中，type将被彻底删除；<br>

***
### Index
* 索引是文档的集合；<br>
* 每个index由一个或多个documents组成，并且这些documents可以分布于不同的shard中；<br>
* index可以说类似于关系数据库中的database，但是并不完全相同；其中一个很重要的原因是：在Elasticsearch中的文档可以有object及nested结构。一个index是一个逻辑空间，ta映射到一个或多个主分片，并且可以具有零个或多个副本分片；当一个文档进来后，根据文档的id自动进行hash计算，并存放于计算出来的shard实例中，这样的结果可以使得所有的shard都有比较均衡的存储；<br>
`shard_num = hash(_routing) % num_primary_shards`
* 在默认情况下，_routing 即是文档的 _id。如果有routing的参与，那么这些文档可能只存放于一个特定的shard，这样的好处是，对于一些情况，可以很快地综合我们所需要的结果而不需要跨node去得到请求。比如针对join的数据类型；<br>
* 从公式中可以看出，shard的数目是不可动态更改的，否则之后也找不到相应的shard号码；但是replica的数目是可以动态修改的；<br>

***
### shard
* Elasticsearch是一个分布式搜索引擎，因此索引通常会拆分为分布在多个节点上的称为分片的元素。Elasticsearch自动管理这些分片的排列。ta还根据需要重新平衡分片，因此用户无需担心细节；<br>
* 一个索引可以存储超出单个结点硬件限制的大量数据；为了解决这个问题，Elasticsearch提供了将索引划分成多份的能力，即分片（shard）；当你创建一个索引的时候，你可以指定你想要的分片（shard）数量；每个分片本身也是一个功能完善并且独立的“索引”，这个“索引”可以被放置到集群中的任何节点上。分片之所以重要，主要有两方面内容：<br>
    * 允许水平分割/扩展你的内容容量；<br>
    * 允许在分片（潜在地，位于多个节点上）之上进行分布式的、并行的操作，今儿提高性能/吞吐量；<br>
* 分片类型：<br>
    * Primary shard：每个文档都存储在一个Primary shard。索引文档时，首先在Primary shard上编制索引，然后在此分片的所有副本上（replica）编制索引。索引可以包含一个或多个主分片。此数字确定索引相对于索引数据大小的可伸缩性。创建索引后，无法更改索引中的主分片数。<br>
    * Replica shard：每个主分片可以具有零个或多个副本。副本是主分片的副本，有两个目的：增加故障转移（如果故障，可以将副本分片提升为主分片）；提高性能（get和search请求可以由主shard副本或副本shard处理）；<br>
* 默认情况下，每个主分片都有一个副本，但可以在现有索引上动态更改副本数。永远不会再与其主分片相同的节点上启动副本分片；<br>

***
### replica
* 默认情况下，Elasticsearch为每个索引创建一个主分片和一个副本；意味着每个索引将包含一个主分片，每个分片将具有一个副本；<br>
* 分配多个分片和副本是分布式搜索功能设计的本质，提供高可用性和快速访问索引中的文档；主副本和副本分片之间的主要区别在于只有主分片可以接受索引请求；副本和主分片都可以提供查询请求；<br>
* number_of_shards值与索引有关，而不是与整个群集有关。此值指定每个索引的分片数（不是群集中的主分片总数）；<br>

***
# 开始使用Elasticsearch

* REST：通过URL定位资源，用HTTP动词（GET，POST，PUT，DELETE，PATCH）描述操作；

***
### 

***
>****<br>