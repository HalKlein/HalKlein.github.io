# Elasticsearch 入门


<!--more-->

## Elasticsearch简介

![Elasticsearch](https://i.loli.net/2020/02/05/AOeK6Jn94HfgrqQ.png)

Elasticsearch，分布式搜索引擎，简称ES，是目前最流行性能最好的搜索引擎之一，像Facebook、Github、Wikipedia、Baidu等很多公司都使用ES进行搜索。

一个分布式的、Restful风格的搜索引擎。

支持对各种类型的数据的检索。

搜索速度快，可以提供 **实时** 的搜索服务。

便于水平扩展，每秒可以处理PB级海量数据。

</br>

## Elasticsearch相关术语

**索引：**类似于MySQL中的数据库（Database）。

**类型：**类似于MySQL中的表（Table）。

**文档：**类似于MySQL中的一条记录，常用Json格式。

**字段：**Json中的每一个属性，我们叫做字段。

注：在Elasticsearch 6.0之后，“类型” 的概念逐渐被废弃，取而代之，“索引” 对应了一张表。

**集群：**分布式部署，多台Elasticsearch服务器组合使用就是一个集群。

**节点：**集群中的每一台服务器我们就叫做一个节点。

**分片：**将一个索引拆分为多个分片来存，提高并发能力。

**副本：**副本是分片的备份。

</br>

## Elasticsearch 的安装

https://www.elastic.co

这里我使用我当前Springboot版本中父pom推荐的6.8.5版本，注意7.0开始的版本和之前的版本区别较大，所以请根据自己的实际情况慎重选择。

**配置config/elasticsearch.yml文件**

```properties
cluster.name: community
path.data: e:\Test\data\elasticsearch-6.8.5\data
path.logs: e:\Test\data\elasticsearch-6.8.5\logs
```

**配置环境变量**

将bin目录添加到系统的Path环境变量中。

</br>

## Elasticsearch-analysis-ik 中文分词插件

英文的分词，ES默认是支持的（只有空格就能分隔），但是中文的分词较为复杂，需要安装插件。

Github地址：[elasticsearch-analysis-ik](https://github.com/medcl/elasticsearch-analysis-ik)，请安装与你的ES适配的版本。

解压的时候，请注意！你**必须**在ES安装目录的plugins文件夹下新建一个名为“ik”的文件夹，将其解压到该目录下。

![目录示例](https://i.loli.net/2020/02/05/2CfLWpzrRxtvB7b.png)

其中，config目录下有一个 `extra_main.dic` 文件，它是常用的中文分词字典库。

如果有新的词出现，或者你希望添加一些自定义的分词，需要在 IKAnalyzer.cfg.xml 中配置，其中有中文说明 ，这里不再赘述。

</br>

## Postman工具的安装

https://www.getpostman.com

Postman能够模拟客户端发送HTTP请求，我们如果使用命令来往ES中存数据，这个命令会比较复杂，而ES支持HTTP访问，所以我们使用Postman工具会比使用命令更方便的操作ES。

</br>

## 命令测试Elasticsearch

**启动Elasticsearch**

双击或命令行启动，bin目录下的 elasticsearch.bat 批处理文件以启动ES，启动成功后，能看到绑定的地址和端口，默认是9200端口。

**查看ES集群的健康状况**

```properties
E:\ProgramFile\kafka_2.12-2.4.0
$ curl -X GET "localhost:9200/_cat/health?v"
epoch      timestamp cluster   status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1580812558 10:35:58  community green           1         1      0   0    0    0        0             0
 -                100.0%
```

第一行表示标题，第二行是具体数据。 cluster对应集群；status对应健康状态，green表示健康，red表示有问题；后面是一些节点和分片信息。

**查看集群中的节点**

```properties
$ curl -X GET "localhost:9200/_cat/nodes?v"
ip        heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
127.0.0.1           11          49   4                          mdi       *      sGjUs63
```

**创建索引**

```properties
$ curl -X PUT "localhost:9200/test"
{"acknowledged":true,"shards_acknowledged":true,"index":"test"}
```

可以看到创建的索引的格式是JSON格式的

**查看集群中的索引**

```properties
$ curl -X GET "localhost:9200/_cat/indices?v"
health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   test  ca0IN3o6QLSLjQuDNJqBqg   5   1          0            0      1.1kb          1.1kb
```

**删除索引**

```properties
$ curl -X DELETE "localhost:9200/test"
{"acknowledged":true}$ curl -X DELETE "localhost:9200/test"
```

</br>

## 使用Postman访问Elasticsearch

**创建索引**

![创建索引](https://i.loli.net/2020/02/05/F4XaHSMkLr2lfTg.png)

**查看集群中的索引**

![查看集群中的索引](https://i.loli.net/2020/02/05/qeS42Jxb8dZMNzi.png)

**删除索引**

![删除索引](https://i.loli.net/2020/02/05/TLhZPCBVS1OHfAD.png)

**存入JSON格式的数据**

选择PUT请求，填入索引名称和id，其中_doc是占位用的。在Body中，选择raw，再选择数据格式为JSON，填写完毕之后点击发送。

![存入JSON格式的数据](https://i.loli.net/2020/02/05/wDjQym8R5HsSzUc.png)

**查询存入的数据**

![查询存入的数据](https://i.loli.net/2020/02/05/T3oDpzCXyGiqEQP.png)

**删除存入的数据**

![删除存入的数据](https://i.loli.net/2020/02/05/9vbh4Vu8AjS6WQt.png)

**搜索数据测试**

PUT请求，存如多条数据

```json
# localhost:9200/test/_doc/1
{
	"title":"校园招聘求职",
	"content":"求一份Java后端开发岗位"
}
# localhost:9200/test/_doc/2
{
	"title":"招聘信息",
	"content":"我公司急需招聘一名资深程序员"
}
# localhost:9200/test/_doc/3
{
	"title":"公司内推",
	"content":"本人在某为任职，可推荐Java开发实习生。"
}
```

简单GET查询

![GET查询](https://i.loli.net/2020/02/05/2wEmJVtP9ho8i4p.png)

![GET查询](https://i.loli.net/2020/02/05/fw5A89LnYPDb7dm.png)

可以看到在查询content的时候ES对关键词进行了拆分，再查询，得到了两条数据，匹配度高的在前面。

**复合的多条件查询**

```json
{
	"query":{
		"multi_match":{
			"query":"公司",
			"fields":["title","content"]
		}
	}
}
```

![复合的多条件查询](https://i.loli.net/2020/02/05/JHIUBVFkOnAbv7w.png)



</br>

<center> ·End </center>
<center> 转载请注明出处: <a href="https://halklein.github.io/">https://halklein.github.io/</a> </center>
