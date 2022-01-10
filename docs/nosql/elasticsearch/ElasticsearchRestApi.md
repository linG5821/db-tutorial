# ElasticSearch Rest API

> **[Elasticsearch](https://github.com/elastic/elasticsearch) 是一个分布式、RESTful 风格的搜索和数据分析引擎**，能够解决不断涌现出的各种用例。 作为 Elastic Stack 的核心，它集中存储您的数据，帮助您发现意料之中以及意料之外的情况。
>
> [Elasticsearch](https://github.com/elastic/elasticsearch) 基于搜索库 [Lucene](https://github.com/apache/lucene-solr) 开发。ElasticSearch 隐藏了 Lucene 的复杂性，提供了简单易用的 REST API / Java API 接口（另外还有其他语言的 API 接口）。
>
> _以下简称 ES_。

> REST API 最详尽的文档应该参考：[ES 官方 REST API](https://www.elastic.co/guide/en/elasticsearch/reference/current/rest-apis.html)

## 索引 API

> 参考资料：[Elasticsearch 官方之 cat 索引 API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cat-indices.html)

### 创建索引

新建 Index，可以直接向 ES 服务器发出 `PUT` 请求。

（1）直接创建索引

```bash
curl -X POST 'localhost:9200/user'
```

服务器返回一个 JSON 对象，里面的 `acknowledged` 字段表示操作成功。

```javascript
{"acknowledged":true,"shards_acknowledged":true,"index":"user"}
```

（2）创建索引时指定配置

语法格式：

```bash
$ curl -X PUT /my_index
{
    "settings": { ... any settings ... },
    "mappings": {
        "type_one": { ... any mappings ... },
        "type_two": { ... any mappings ... },
        ...
    }
}
```

示例：

```bash
$ curl -X PUT -H 'Content-Type: application/json' 'localhost:9200/user'  -d '
{
    "settings" : {
        "index" : {
            "number_of_shards" : 3,
            "number_of_replicas" : 2
        }
    }
}'
```

如果你想禁止自动创建索引，可以通过在 `config/elasticsearch.yml` 的每个节点下添加下面的配置：

```js
action.auto_create_index: false
```

### 删除索引

然后，我们可以通过发送 `DELETE` 请求，删除这个 Index。

```bash
curl -X DELETE 'localhost:9200/user'
```

删除多个索引

```js
DELETE /index_one,index_two
DELETE /index_*
```

### 查看索引

可以通过 GET 请求查看索引信息

```bash
# 查看索引相关信息
GET kibana_sample_data_ecommerce

# 查看索引的文档总数
GET kibana_sample_data_ecommerce/_count

# 查看前10条文档，了解文档格式
GET kibana_sample_data_ecommerce/_search

# _cat indices API
# 查看indices
GET /_cat/indices/kibana*?v&s=index

# 查看状态为绿的索引
GET /_cat/indices?v&health=green

# 按照文档个数排序
GET /_cat/indices?v&s=docs.count:desc

# 查看具体的字段
GET /_cat/indices/kibana*?pri&v&h=health,index,pri,rep,docs.count,mt

# 查看索引占用的内存
GET /_cat/indices?v&h=i,tm&s=tm:desc
```

### 打开/关闭索引

通过在 `POST` 中添加 `_close` 或 `_open` 可以打开、关闭索引。

打开索引

```bash
# 打开索引
POST kibana_sample_data_ecommerce/_open
# 关闭索引
POST kibana_sample_data_ecommerce/_close
```

## 文档

```
############Create Document############
#create document. 自动生成 _id
POST users/_doc
{
	"user" : "Mike",
    "post_date" : "2019-04-15T14:12:12",
    "message" : "trying out Kibana"
}

#create document. 指定Id。如果id已经存在，报错
PUT users/_doc/1?op_type=create
{
    "user" : "Jack",
    "post_date" : "2019-05-15T14:12:12",
    "message" : "trying out Elasticsearch"
}

#create document. 指定 ID 如果已经存在，就报错
PUT users/_create/1
{
     "user" : "Jack",
    "post_date" : "2019-05-15T14:12:12",
    "message" : "trying out Elasticsearch"
}

### Get Document by ID
#Get the document by ID
GET users/_doc/1


###  Index & Update
#Update 指定 ID  (先删除，在写入)
GET users/_doc/1

PUT users/_doc/1
{
	"user" : "Mike"

}


#GET users/_doc/1
#在原文档上增加字段
POST users/_update/1/
{
    "doc":{
        "post_date" : "2019-05-15T14:12:12",
        "message" : "trying out Elasticsearch"
    }
}



### Delete by Id
# 删除文档
DELETE users/_doc/1


### Bulk 操作
#执行两次，查看每次的结果

#执行第1次
POST _bulk
{ "index" : { "_index" : "test", "_id" : "1" } }
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_id" : "2" } }
{ "create" : { "_index" : "test2", "_id" : "3" } }
{ "field1" : "value3" }
{ "update" : {"_id" : "1", "_index" : "test"} }
{ "doc" : {"field2" : "value2"} }


#执行第2次
POST _bulk
{ "index" : { "_index" : "test", "_id" : "1" } }
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_id" : "2" } }
{ "create" : { "_index" : "test2", "_id" : "3" } }
{ "field1" : "value3" }
{ "update" : {"_id" : "1", "_index" : "test"} }
{ "doc" : {"field2" : "value2"} }

### mget 操作
GET /_mget
{
    "docs" : [
        {
            "_index" : "test",
            "_id" : "1"
        },
        {
            "_index" : "test",
            "_id" : "2"
        }
    ]
}


#URI中指定index
GET /test/_mget
{
    "docs" : [
        {

            "_id" : "1"
        },
        {

            "_id" : "2"
        }
    ]
}


GET /_mget
{
    "docs" : [
        {
            "_index" : "test",
            "_id" : "1",
            "_source" : false
        },
        {
            "_index" : "test",
            "_id" : "2",
            "_source" : ["field3", "field4"]
        },
        {
            "_index" : "test",
            "_id" : "3",
            "_source" : {
                "include": ["user"],
                "exclude": ["user.location"]
            }
        }
    ]
}

### msearch 操作
POST kibana_sample_data_ecommerce/_msearch
{}
{"query" : {"match_all" : {}},"size":1}
{"index" : "kibana_sample_data_flights"}
{"query" : {"match_all" : {}},"size":2}


### 清除测试数据
#清除数据
DELETE users
DELETE test
DELETE test2
```

### 创建文档

#### 指定 ID

语法格式：

```bash
PUT /_index/_type/_create/_id
```

示例：

```bash
PUT /user/_doc/_create/1
{
  "user": "张三",
  "title": "工程师",
  "desc": "数据库管理"
}
```

> 注意：指定 Id，如果 id 已经存在，则报错

#### 自动生成 ID

新增记录的时候，也可以不指定 Id，这时要改成 POST 请求。

语法格式：

```bash
POST /_index/_type
```

示例：

```bash
POST /user/_doc
{
  "user": "张三",
  "title": "工程师",
  "desc": "超级管理员"
}
```

### 删除文档

语法格式：

```bash
DELETE /_index/_doc/_id
```

示例：

```bash
DELETE /user/_doc/1
```

### 更新文档

#### 先删除，再写入

语法格式：

```bash
PUT /_index/_type/_id
```

示例：

```bash
PUT /user/_doc/1
{
  "user": "李四",
  "title": "工程师",
  "desc": "超级管理员"
}
```

#### 在原文档上增加字段

语法格式：

```bash
POST /_index/_update/_id
```

示例：

```bash
POST /user/_update/1
{
    "doc":{
        "age" : "30"
    }
}
```

### 查询文档

#### 指定 ID 查询

语法格式：

```
GET /_index/_type/_id
```

示例：

```bash
GET /user/_doc/1
```

结果：

```json
{
  "_index": "user",
  "_type": "_doc",
  "_id": "1",
  "_version": 1,
  "_seq_no": 536248,
  "_primary_term": 2,
  "found": true,
  "_source": {
    "user": "张三",
    "title": "工程师",
    "desc": "数据库管理"
  }
}
```

返回的数据中，`found` 字段表示查询成功，`_source` 字段返回原始记录。

如果 id 不正确，就查不到数据，`found` 字段就是 `false`

#### 查询所有记录

使用 `GET` 方法，直接请求 `/index/type/_search`，就会返回所有记录。

```bash
$ curl 'localhost:9200/user/admin/_search?pretty'
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 3,
    "successful" : 3,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 2,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "user",
        "_type" : "admin",
        "_id" : "WWuoDG8BHwECs7SiYn93",
        "_score" : 1.0,
        "_source" : {
          "user" : "李四",
          "title" : "工程师",
          "desc" : "系统管理"
        }
      },
      {
        "_index" : "user",
        "_type" : "admin",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "user" : "张三",
          "title" : "工程师",
          "desc" : "超级管理员"
        }
      }
    ]
  }
}
```

上面代码中，返回结果的 `took`字段表示该操作的耗时（单位为毫秒），`timed_out`字段表示是否超时，`hits`字段表示命中的记录，里面子字段的含义如下。

- `total`：返回记录数，本例是 2 条。
- `max_score`：最高的匹配程度，本例是`1.0`。
- `hits`：返回的记录组成的数组。

返回的记录中，每条记录都有一个`_score`字段，表示匹配的程序，默认是按照这个字段降序排列。

### 全文搜索

ES 的查询非常特别，使用自己的[查询语法](https://www.elastic.co/guide/en/elasticsearch/reference/5.5/query-dsl.html)，要求 GET 请求带有数据体。

```bash
$ curl -H 'Content-Type: application/json' 'localhost:9200/user/admin/_search?pretty'  -d '
{
"query" : { "match" : { "desc" : "管理" }}
}'
```

上面代码使用 [Match 查询](https://www.elastic.co/guide/en/elasticsearch/reference/5.5/query-dsl-match-query.html)，指定的匹配条件是`desc`字段里面包含"软件"这个词。返回结果如下。

```javascript
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 3,
    "successful" : 3,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 2,
    "max_score" : 0.38200712,
    "hits" : [
      {
        "_index" : "user",
        "_type" : "admin",
        "_id" : "WWuoDG8BHwECs7SiYn93",
        "_score" : 0.38200712,
        "_source" : {
          "user" : "李四",
          "title" : "工程师",
          "desc" : "系统管理"
        }
      },
      {
        "_index" : "user",
        "_type" : "admin",
        "_id" : "1",
        "_score" : 0.3487891,
        "_source" : {
          "user" : "张三",
          "title" : "工程师",
          "desc" : "超级管理员"
        }
      }
    ]
  }
}
```

Elastic 默认一次返回 10 条结果，可以通过`size`字段改变这个设置，还可以通过`from`字段，指定位移。

```bash
$ curl 'localhost:9200/user/admin/_search'  -d '
{
  "query" : { "match" : { "desc" : "管理" }},
  "from": 1,
  "size": 1
}'
```

上面代码指定，从位置 1 开始（默认是从位置 0 开始），只返回一条结果。

### 逻辑运算

如果有多个搜索关键字， Elastic 认为它们是`or`关系。

```bash
$ curl 'localhost:9200/user/admin/_search'  -d '
{
"query" : { "match" : { "desc" : "软件 系统" }}
}'
```

上面代码搜索的是`软件 or 系统`。

如果要执行多个关键词的`and`搜索，必须使用[布尔查询](https://www.elastic.co/guide/en/elasticsearch/reference/5.5/query-dsl-bool-query.html)。

```bash
$ curl -H 'Content-Type: application/json' 'localhost:9200/user/admin/_search?pretty'  -d '
{
 "query": {
  "bool": {
   "must": [
    { "match": { "desc": "管理" } },
    { "match": { "desc": "超级" } }
   ]
  }
 }
}'
```

### 批量执行

支持在一次 API 调用中，对不同的索引进行操作

支持四种类型操作

- index
- create
- update
- delete

操作中单条操作失败，并不会影响其他操作。

返回结果包括了每一条操作执行的结果。

```bash
POST _bulk
{ "index" : { "_index" : "test", "_id" : "1" } }
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_id" : "2" } }
{ "create" : { "_index" : "test2", "_id" : "3" } }
{ "field1" : "value3" }
{ "update" : {"_id" : "1", "_index" : "test"} }
{ "doc" : {"field2" : "value2"} }
```

> 说明：上面的示例如果执行多次，执行结果都不一样。

### 批量读取

读多个索引

```bash
GET /_mget
{
    "docs" : [
        {
            "_index" : "test",
            "_id" : "1"
        },
        {
            "_index" : "test",
            "_id" : "2"
        }
    ]
}
```

读一个索引

```bash
GET /test/_mget
{
    "docs" : [
        {

            "_id" : "1"
        },
        {

            "_id" : "2"
        }
    ]
}

GET /_mget
{
    "docs" : [
        {
            "_index" : "test",
            "_id" : "1",
            "_source" : false
        },
        {
            "_index" : "test",
            "_id" : "2",
            "_source" : ["field3", "field4"]
        },
        {
            "_index" : "test",
            "_id" : "3",
            "_source" : {
                "include": ["user"],
                "exclude": ["user.location"]
            }
        }
    ]
}
```

### 批量查询

```bash
POST kibana_sample_data_ecommerce/_msearch
{}
{"query" : {"match_all" : {}},"size":1}
{"index" : "kibana_sample_data_flights"}
{"query" : {"match_all" : {}},"size":2}
```

## 集群 API

> [Elasticsearch 官方之 Cluster API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster.html)

一些集群级别的 API 可能会在节点的子集上运行，这些节点可以用节点过滤器指定。例如，任务管理、节点统计和节点信息 API 都可以报告来自一组过滤节点而不是所有节点的结果。

节点过滤器以逗号分隔的单个过滤器列表的形式编写，每个过滤器从所选子集中添加或删除节点。每个过滤器可以是以下之一：

- `_all`：将所有节点添加到子集
- `_local`：将本地节点添加到子集
- `_master`：将当前主节点添加到子集
- 根据节点 ID 或节点名将匹配节点添加到子集
- 根据 IP 地址或主机名将匹配节点添加到子集
- 使用通配符，将节点名、地址名或主机名匹配的节点添加到子集
- `master:true`, `data:true`, `ingest:true`, `voting_only:true`, `ml:true` 或 `coordinating_only:true`, 分别意味着将所有主节点、所有数据节点、所有摄取节点、所有仅投票节点、所有机器学习节点和所有协调节点添加到子集中。
- `master:false`, `data:false`, `ingest:false`, `voting_only:true`, `ml:false` 或 `coordinating_only:false`, 分别意味着将所有主节点、所有数据节点、所有摄取节点、所有仅投票节点、所有机器学习节点和所有协调节点排除在子集外。
- 配对模式，使用 `*` 通配符，格式为 `attrname:attrvalue`，将所有具有自定义节点属性的节点添加到子集中，其名称和值与相应的模式匹配。自定义节点属性是通过 `node.attr.attrname: attrvalue` 形式在配置文件中设置的。

```bash
# 如果没有给出过滤器，默认是查询所有节点
GET /_nodes
# 查询所有节点
GET /_nodes/_all
# 查询本地节点
GET /_nodes/_local
# 查询主节点
GET /_nodes/_master
# 根据名称查询节点（支持通配符）
GET /_nodes/node_name_goes_here
GET /_nodes/node_name_goes_*
# 根据地址查询节点（支持通配符）
GET /_nodes/10.0.0.3,10.0.0.4
GET /_nodes/10.0.0.*
# 根据规则查询节点
GET /_nodes/_all,master:false
GET /_nodes/data:true,ingest:true
GET /_nodes/coordinating_only:true
GET /_nodes/master:true,voting_only:false
# 根据自定义属性查询节点（如：查询配置文件中含 node.attr.rack:2 属性的节点）
GET /_nodes/rack:2
GET /_nodes/ra*:2
GET /_nodes/ra*:2*
```

### 集群健康 API

```bash
GET /_cluster/health
GET /_cluster/health?level=shards
GET /_cluster/health/kibana_sample_data_ecommerce,kibana_sample_data_flights
GET /_cluster/health/kibana_sample_data_flights?level=shards
```

### 集群状态 API

集群状态 API 返回表示整个集群状态的元数据。

```bash
GET /_cluster/state
```

## 节点 API

> [Elasticsearch 官方之 cat Nodes API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cat-nodes.html)——返回有关集群节点的信息。

```bash
# 查看默认的字段
GET /_cat/nodes?v=true
# 查看指定的字段
GET /_cat/nodes?v=true&h=id,ip,port,v,m
```

## 分片 API

> [Elasticsearch 官方之 cat Shards API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cat-shards.html)——shards 命令是哪些节点包含哪些分片的详细视图。它会告诉你它是主还是副本、文档数量、它在磁盘上占用的字节数以及它所在的节点。

```bash
# 查看默认的字段
GET /_cat/shards
# 根据名称查询分片（支持通配符）
GET /_cat/shards/my-index-*
# 查看指定的字段
GET /_cat/shards?h=index,shard,prirep,state,unassigned.reason
```

## 参考资料

- **官方**
  - [Elasticsearch 官网](https://www.elastic.co/cn/products/elasticsearch)
  - [Elasticsearch Github](https://github.com/elastic/elasticsearch)
  - [Elasticsearch 官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)