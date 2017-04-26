专业术语
=========

**聚集索引**
一种索引，该索引中键值的逻辑顺序决定了表中相应行的物理顺序。 
聚集索引确定表中数据的物理顺序。聚集索引类似于电话簿，后者按姓氏排列数据。由于聚集索引规定数据在表中的物理存储顺序，因此一个表只能包含一个聚集索引。但该索引可以包含多个列（组合索引），就像电话簿按姓氏和名字进行组织一样。
聚集索引对于那些经常要搜索范围值的列特别有效。使用聚集索引找到包含第一个值的行后，便可以确保包含后续索引值的行在物理相邻。例如，如果应用程序执行 的一个查询经常检索某一日期范围内的记录，则使用聚集索引可以迅速找到包含开始日期的行，然后检索表中所有相邻的行，直到到达结束日期。这样有助于提高此 类查询的性能。同样，如果对从表中检索的数据进行排序时经常要用到某一列，则可以将该表在该列上聚集（物理排序），避免每次查询该列时都进行排序，从而节 省成本。
当索引值唯一时，使用聚集索引查找特定的行也很有效率。例如，使用唯一雇员 ID 列 emp_id 查找特定雇员的最快速的方法，是在 emp_id 列上创建聚集索引或 PRIMARY KEY 约束。 

**非聚集索引**
一种索引，该索引中索引的逻辑顺序与磁盘上行的物理存储顺序不同。

**插入因子**
对聚集索引列,行的排列的物理顺序与索引排列的顺序是一致的,这就引起一个问题,如果要向一个表插入一个记录,有可能由于插入在表的中间某个位置,而要使其后的所有列都要向后移动一个行的空间,这种操作是很费时间的。
为此,先考虑在列的物理顺序排列位置上空出一些能插入记录的空间(这样要了解页的概念),然后,要插入的话,就可以不动全表,而只动动页上的内容就行了。
如果填充因子设置的过高，并且表上有频繁的 insert 和 update 操作，便会导致叶级索引页很容易便会被填满，需要进行分页。这样不但会导致 insert 和 update 操作性能降低，还容易产生索引碎片。
如果填充因子设置的过低，这样虽然有利于 insert 和 update 操作，但是由于会导致索引或表占用更多的页面，查询操作便需要读取更多的页面，从而降低查询性能。
一页上,默认占用的存储空间(不含空的地方)占该页总存储空间的百分之几,就是填充因子。

**SARG**
Searchable Arguments操作，因为它通常是指一个特定的匹配，一个值得范围内的匹配或者两个以上条件的AND连接

**数据库集群**
数据库集群，顾名思义，就是利用至少两台或者多台数据库服务器，构成一个虚拟单一数据库逻辑映像，像单数据库系统那样，向客户端提供透明的数据服务。
Mysql集群分为同步集群和异步集群。
1. 同步集群（mysql cluster）
结构：（data + sql + mgm节点）
特点：
1) 内存级别的，对硬件要求较低，但是对内存要求较大。换算比例为：1：1.1；
2) 数据同时放在几台服务器上，冗余较好；
3) 速度一般；
4) 建表需要声明为engine=ndbcluster
5) 扩展性强；
6) 可以实现高可用性和负载均衡，实现对大型应用的支持；
7) 必须是特定的mysql版本，如：已经编译好的max版本；
8) 配置和管理方便，不会丢失数据；
2. 异步集群（mysql replication）
结构：（master + slave）
特点：
1) 主从数据库异步数据；
2) 数据放在几台服务器上，冗余一般；
3) 速度较快；
4) 扩展性差；
5) 无法实现高可用性和负载均衡（只能在程序级别实现读写分离，减轻对主数据库的压力）；
6) 配置和管理较差，可能会丢失数据；

**分布式数据库**
分布式数据库系统通常使用较小的计算机系统，每台计算机可单独放在一个地方，每台计算机中都可能有DBMS的一份完整拷贝副本，或者部分拷贝副本，并具有自己局部的数据库，位于不同地点的许多计算机通过网络互相连接，共同组成一个完整的、全局的逻辑上集中、物理上分布的大型数据库。
目前主要分布存储的方式都是按照一定的方式进行切分，主要是垂直切分（纵向）和水平切分（横向）两种方式，当然，也有两种结合的方式，达到更贴切的切分粒度。
1. 垂直切分（纵向）数据是数据库切分按照网站业务、产品进行切分，比如用户数据、博客文章数据、照片数据、标签数据、群组数据等等每个业务一个独立的数据库或者数据库服务器。
2. 水平切分（横向）数据是把所有数据当作一个大产品，但是把所有的平面数据按照某些Key（比如用户名）分散在不同数据库或者数据库服务器上，分散对数据访问的压力，这种方式也是本文主要要探讨的。

主要产品框架有：Amoeba和Cobar（正式对外开源的数据库中间件Cobar，前身是早已经开源的Amoeba，不过其作者陈思儒离职去盛大之后，阿里巴巴内部考虑到Amoeba的稳定性、性能和功能支持，以及其他因素，重新设立了一个项目组并且更换名称为Cobar）
https://github.com/alibaba/cobar
https://github.com/alibaba/cobar/wiki

**Catalog**
按照SQL标准的解释，在SQL环境下Catalog和Schema都属于抽象概念，可以把它们理解为一个容器或者数据库对象命名空间中的一个层次，主要用来解决命名冲突问题。从概念上说，一个数据库系统包含多个Catalog，每个Catalog又包含多个Schema，而每个Schema又包含多个数据库对象（表、视图、字段等），反过来讲一个数据库对象必然属于一个Schema，而该Schema又必然属于一个Catalog，这样我们就可以得到该数据库对象的完全限定名称从而解决命名冲突的问题了；例如数据库对象表的完全限定名称就可以表示为：Catalog名 称.Schema名称.表名称。这里还有一点需要注意的是，ＳＱＬ标准并不要求每个数据库对象的完全限定名称是唯一的，就象域名一样，如果喜欢的话，每个ＩＰ地址都可以拥有多个域名。  从实现的角度来看，各种数据库系统对Catalog和Schema的支持和实现方式千差万别，针对具体问题需要参考具体的产品说明书，比较简单而常用的实现方式是使用数据库名作为Catalog名，使用用户名作为Schema名，具体可参见下表

供应商	Catalog支持	Schema支持
Oracle 	不支持 	Oracle User ID
MySQL	不支持	数据库名
MS SQL Server	数据库名	对象属主名，2005版开始有变
DB2	指定数据库对象时，Catalog部分省略	Catalog属主名
Sybase 	数据库名	数据库属主名