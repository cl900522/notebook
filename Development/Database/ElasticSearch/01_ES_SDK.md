ElasticSearch Sdk
=================

# Http Restful Api

其他所有程序语言都可以使用RESTful API，通过9200端口的与Elasticsearch进行通信，你可以使用你喜欢的WEB客户端，事实上，如你所见，你甚至可以通过curl命令与Elasticsearch通信。

Elasticsearch官方提供了多种程序语言的客户端——Groovy，Javascript， .NET，PHP，Perl，Python，以及 Ruby——还有很多由社区提供的客户端和插件，所有这些可以在文档中找到。
向Elasticsearch发出的请求的组成部分与其它普通的HTTP请求是一样的：

```sh
curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
VERB HTTP方法：GET, POST, PUT, HEAD, DELETE
PROTOCOL http或者https协议（只有在Elasticsearch前面有https代理的时候可用）
```

* HOST Elasticsearch集群中的任何一个节点的主机名，如果是在本地的节点，那么就叫localhost
* PORT Elasticsearch HTTP服务所在的端口，默认为9200
* PATH API路径（例如_count将返回集群中文档的数量），PATH可以包含多个组件，例如_cluster/stats或者_nodes/stats/jvm
* QUERY_STRING 一些可选的查询请求参数，例如?pretty参数将使请求返回更加美观易读的JSON数据
* BODY 一个JSON格式的请求主体（如果请求需要的话）

## 服务管理
* 关闭

```sh
curl -XPOST 'http://localhost:9200/_shutdown'
```

## 服务状态查看

* 集群健康
  ```sh
  GET /_cluster/health
  ```
status字段提供一个综合的指标来表示集群的的服务状况。三种颜色各自的含义：

颜色|	意义|
:--:|:---
green|	所有主要分片和复制分片都可用|
yellow|	所有主要分片可用，但不是所有复制分片都可用|
red|	不是所有的主要分片都可用|

## 索引管理
* 创建

创建一个叫做blogs的索引。默认情况下，一个索引被分配5个主分片，但是为了演示的目的，我们只分配3个主分片和一个复制分片（每个主分片都有一个复制分片）：
```sh
PUT /blogs
{
   "settings" : {
      "number_of_shards" : 3,
      "number_of_replicas" : 1
   }
}
```

## 数据CRUD
* 新增

自增_id
```sh
POST /website/blog/
{
  "title": "My second blog entry",
  "text":  "Still trying this out...",
  "date":  "2014/01/01"
}
```

指定_id
```sh
PUT /megacorp/employee/2
{
    "first_name" :  "Jane",
    "last_name" :   "Smith",
    "age" :         32,
    "about" :       "I like to collect rock albums",
    "interests":  [ "music" ]
}
```

想使用自定义的_id，告诉Elasticsearch应该在_index、_type、_id三者都不存在时才接受请求，否则创建失败
```sh
PUT /website/blog/123?op_type=create
{ ... }

PUT /website/blog/123/_create
{ ... }
```

* 更新

乐观锁修改
```sh
PUT /website/blog/1?version=1 <1>
{
  "title": "My first blog entry",
  "text":  "Starting to get the hang of this..."
}
```

外部版本号

Elasticsearch的查询字符串后面添加version_type=external来使用这些版本号。版本号必须是整数，大于零小于9.2e+18——Java中的正的long。

外部版本号与之前说的内部版本号在处理的时候有些不同。它不再检查_version是否与请求中指定的一致，而是检查是否小于指定的版本。如果请求成功，外部版本号就会被存储到_version中。
```sh
PUT /website/blog/2?version=5&version_type=external
{
  "title": "My first external blog entry",
  "text":  "Starting to get the hang of this..."
}
```

局部更新

文档是不可变的——它们不能被更改，只能被替换。update API必须遵循相同的规则。表面看来，我们似乎是局部更新了文档的位置，**内部却是像我们之前说的一样简单的使用update API处理相同的检索-修改-重建索引流程**，我们也减少了其他进程可能导致冲突的修改。

最简单的update请求表单接受一个局部文档参数doc，它会合并到现有文档中——对象合并在一起，存在的标量字段被覆盖，新字段被添加。举个例子，我们可以使用以下请求为博客添加一个tags字段和一个views字段：
```
POST /website/blog/1/_update
{
   "doc" : {
      "tags" : [ "testing" ],
      "views": 0
   }
}
```

* 检索
```sh
GET /megacorp/employee/1
```

指定检索返回字段
```sh
GET /website/blog/123/_source

GET /website/blog/123/_source=name,age
```

检查存在

如果你想做的只是检查文档是否存在——你对内容完全不感兴趣——使用HEAD方法来代替GET。HEAD请求不会返回响应体，只有HTTP头
```sh
curl -i -XHEAD http://localhost:9200/website/blog/123
```

* 删除
```sh
DELETE /megacorp/employee/1
```

## DSL搜索
* 条件搜索
```sh
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        },
        "filtered" : {
            "filter" : {
                "range" : {
                    "age" : { "gt" : 30 }
                }
            },
            "query" : {
                "match" : {
                    "last_name" : "smith"
                }
            }
        }
    }
}
```

* 全文搜索
```sh
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "about" : "rock climbing"
        }
    }
}
```

* 短语搜索

确切的匹配若干个单词或者短语(phrases)
```sh
GET /megacorp/employee/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    }
}
```

## 聚合分析
* 条件过滤
```sh
GET /megacorp/employee/_search
{
  "query": {
    "match": {
      "last_name": "smith"
    }
  },
  "aggs": {
    "all_interests": {
      "terms": {
        "field": "interests"
      }
    }
  }
}
```