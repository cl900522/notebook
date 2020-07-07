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
POST _shutdown
```

## 服务状态查看

* 集群健康
```sh

GET _cluster/health

# 检查集群状态
GET _cat/health

#检查节点状态
GET _cat/nodes?v

# 查询所有索引
GET /_cat/indices

```

status字段提供一个综合的指标来表示集群的的服务状况。三种颜色各自的含义：

|  颜色  | 意义                                       |
| :----: | :----------------------------------------- |
| green  | 所有主要分片和复制分片都可用               |
| yellow | 所有主要分片可用，但不是所有复制分片都可用 |
|  red   | 不是所有的主要分片都可用                   |

## 分词测试

指定分词器测试
```sh
POST _analyze
{
  "text": "xi fei jian",
  "analyzer": "english"
}
```

自定义分词器测试
```sh
GET _analyze
{
  "tokenizer": "standard",
  "filter":["lowercase"],
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}
```

指定基于某个索引的字段相同的分词器测试
```sh
POST goods/_analyze
{
  "field": "name",
  "text": "xi fei jian"
}
```



## 索引管理

* 查看

```sh
GET /goods
```

* 删除

```sh
DELETE /goods
```

* 创建

创建一个叫做goods的索引。默认情况下，一个索引被分配5个主分片
Es (<= 5) 支持每个索引创建多个mapping;但是从Es6开始，每个索引只能有一个mapping，创建索引是的入参，也不支持多个mapping。

* number_of_shards: 是设置的分片数，设置之后无法更改！
* refresh_interval: 是设置es缓存的刷新时间，如果写入较为频繁，但是查询对实时性要求不那么高的话，可以设置高一些来提升性能。-1表示关闭刷新，一个绝对值 1 表示的是 1毫秒
* number_of_replicas : 是设置该索引库的副本数，建议设置为1以上。

字段的设置参数参数:

* store: true/false 表示该字段是否存储，默认存储。
* doc_values: true/false 表示该字段是否参与聚合和排序。
* index: true/false 表示该字段是否建立索引，默认建立。

```sh
# 语法Es<=5
PUT /goods?pretty
{
  "settings": {
    "number_of_replicas": 1,
    "number_of_shards": 10,
    "refresh_interval": "1s"
  },
  "mappings": {
    "map1"{
      "properties": {
        "name": {
          "type": "text"
        },
        "adwords": {
          "type": "text"
        }
      }
    }
    "map2" :{
        "yn": {
          "type": "boolean"
        },
        "created": {
          "type": "date",
          "format": "yyyy-MM-dd HH:mm:ss"
        }
    }
  }
}

# 语法Es>=6
PUT /goods
{
  "settings": {
    "number_of_replicas": 1,
    "number_of_shards": 10,
    "refresh_interval": "2m"
  },
  "mappings": {
      "properties": {
        "name": {
          "type": "text",
          "fields": {
            "wholeName": {
              "type": "keyword",
              "ignore_above": 100
            }
          }
        },
        "adwords": {
          "type": "text",
          "doc_values": true,
          "index": true,
          "store": true
        },
        "category": {
          "type": "keyword"
        },
        "unit": {
          "type": "keyword"
        },
        "quantity": {
          "type": "double"
        },
        "price": {
          "type": "double"
        },
        "yn": {
          "type": "boolean"
        },
        "created": {
          "type": "date",
          "format": "yyyy-MM-dd HH:mm:ss"
        }
      }
  }
}

#
```

## mapping管理

* 查看
```sh
GET /goods/_mapping
```

* 新建或者修改
```sh
PUT /goods/_mapping
{
  "properties": {
    "upc": {
      "type": "keyword"
    },
    "modified": {
      "type": "date",
      "format": "yyyy-MM-dd HH:mm:ss"
    }
  }
}


```

## 数据CRUD
* 新增

自增_id
```sh
POST /goods/_doc/
{
  "title": "My second blog entry",
  "text":  "Still trying this out...",
  "date":  "2014/01/01"
}
```

指定_id创建（其实也可用来修改）

```sh
PUT /goods/_doc/2
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
PUT /goods/_doc/2?op_type=create
{ ... }

PUT /goods/_doc/2/_create
{ ... }
```

* 更新

乐观锁修改
1. _version表示当前数据新的版本号；
2. _seq_no其实和version同一个道理，一旦数据发生更改，数据也一直是累计的；
3. _primary_term表示是由谁分配的，意思说如果文档在一个集群里面，文档肯定会被分配一个位置，_primary_term表示的就是一个位置；

_seq_no和_primary_term是对_version的优化，7.X版本的ES默认使用这种方式控制版本，所以当在高并发环境下使用乐观锁机制修改文档时，要带上当前文档的_seq_no和_primary_term进行更新：
```sh
PUT /goods/_doc/2?if_seq_no=5&if_primary_term=1
{
  "title": "My first blog entry",
  "text":  "Starting to get the hang of this..."
}
```

外部版本号

Elasticsearch的查询字符串后面添加version_type=external来使用这些版本号。版本号必须是整数，大于零小于9.2e+18——Java中的正的long。

外部版本号与之前说的内部版本号在处理的时候有些不同。它不再检查_version是否与请求中指定的一致，而是检查是否小于指定的版本。如果请求成功，外部版本号就会被存储到_version中。
```sh
PUT /goods/_doc/2?version=5&version_type=external
{
  "title": "My first external blog entry",
  "text":  "Starting to get the hang of this..."
}
```

局部更新

文档是不可变的——它们不能被更改，只能被替换。update API必须遵循相同的规则。表面看来，我们似乎是局部更新了文档的位置，**内部却是像我们之前说的一样简单的使用update API处理相同的检索-修改-重建索引流程**，我们也减少了其他进程可能导致冲突的修改。

最简单的update请求表单接受一个局部文档参数doc，它会合并到现有文档中——对象合并在一起，存在的标量字段被覆盖，新字段被添加。举个例子，我们可以使用以下请求为博客添加一个tags字段和一个views字段：

```sh

# 也可以使用，但是不推荐
POST /goods/_doc/2/_update
{
  "doc": {
    "subGoods":["1","3","4"]
  }
}

# 新版语法推荐
POST /goods/_update/2
{
  "doc": {
    "subGoods":["1","3","4"]
  }
}
```

* 检索

```sh
GET /goods/_doc/2
```

指定检索返回字段

```sh
GET /goods/_doc/2?_source

GET /goods/_doc/2?_source=name,subGoods
```

检查存在
>如果你想做的只是检查文档是否存在——你对内容完全不感兴趣——使用HEAD方法来代替GET。HEAD请求不会返回响应体，只有HTTP头

```sh
HEAD /goods/_doc/2
```

* 删除

```sh
DELETE /goods/_doc/2
```

## 搜索

```sh
GET /goods/_search?q=name:娃哈哈&sort=created:desc
```

## DSL搜索

* 条件搜索

```sh
GET /goods/_search
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
GET /goods/_search
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
GET /goods/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    }
}
```
