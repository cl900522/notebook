
# Neo4j

## 简介

Neo4j是由Java和Scala写成的一个NoSql数据库，专门用于网络图的存储。作为一个图形数据库，Neo4j有以下优点：

* 更快的数据库操作。当然，有一个前提条件，那就是数据量较大，在MySql中存储的话需要许多表，并且表之间联系较多（即有不少的操作需要join表）。
* 数据更直观，相应的SQL语句也更好写（Neo4j使用Cypher语言，与传统SQL有很大不同）。
* 更灵活。不管有什么新的数据需要存储，都是一律的节点和边，只需要考虑节点属性和边属性。而MySql中即意味着新的表，还要考虑和其他表的关系。
* 数据库操作的速度并不会随着数据库的增大有明显的降低。这得益于Neo4j特殊的数据存储结构和专门优化的图算法。

### 数据读写

* 数据存储
    Neo4j对于图的存储自然是经过特别优化的。不像传统数据库的一条记录一条数据的存储方式，Neo4j的存储方式是：节点的类别，属性，边的类别，属性等都是分开存储的，这将大大有助于提高图形数据库的性能。如下图：neo4j-1

* 数据读写
    在Neo4j中，存储节点时使用了"index-free adjacency"，即每个节点都有指向其邻居节点的指针，可以让我们在O(1)的时间内找到邻居节点。另外，按照官方的说法，在Neo4j中边是最重要的,是"first-class entities"，所以单独存储，这有利于在图遍历的时候提高速度，也可以很方便地以任何方向进行遍历。

### 图形数据库

官方的定义：a database that uses graph structures for semantic queries with nodes, edges and properties to represent and store data – independent of the way the data is stored internally. It’s really the model and the implemented algorithms that matter.注意，这里只是说数据模型是图结构的，没有说数据的存储也一定要是图结构的。其数据模型如下图

## Cypher语言

### 结构维护


### 数据导入


### 增删改查


### 聚合搜索

## 对比

### 商业版对比

![neo4j-compare](/images/2019/01/neo4j-community.jpg)

### 同款对比

作为较早的一批图形数据库之一，文档和各种技术博客较多。最开始曾尝试过flockdb（据说操作简单+轻量级），但是败于安装过程...，依赖太多。网上经常有人将orientdb，arangodb与neo4j做对比，我当然也考虑过orientdb和arangodb。从易用性来说都差不多。速度上的话看过一些评测，arangodb应该是相对最快的，因为其使用了混合索引。但是从稳定性来说，neo4j是最好的。