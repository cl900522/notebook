# 镜像pull
docker pull registry.docker-cn.com/library/zookeeper
docker pull registry.docker-cn.com/canal/canal-server

# registry
docker run -d -p 5000:5000 --restart always -v /web/data/registry:/tmp/registry --name registry registry

# zookeeper
docker run -d -p 2181:2181 --name zookeeper1 --restart always zookeeper

# mysql
docker run -d -p 3306:3306 --name mysql1 --restart always -e MYSQL_ROOT_PASSWORD=root -v /web/data/mysql1-data:/var/lib/mysql mysql --character-set-server=utf8 --collation-server=utf8_bin  --default-authentication-plugin=mysql_native_password

docker run -d -p 3306:3306 --name mysql1 --restart always -e MYSQL_ROOT_PASSWORD=root -v /share/volume3/Temp/Docker/mysql:/var/lib/mysql mysql --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci  --default-authentication-plugin=mysql_native_password


>从MySQL8.0 开始，默认的加密规则使用的是 caching_sha2_password,需要修改为默认的mysql_native_password，否则会导致登陆连接

# mongo
docker run -d -p 27017:27017 --name mongo1 --restart always mongo
docker run -d -p 27017:27017 --restart always -v /web/data/mongo1:/data/db --name mongo1 mongo


docker run -d -p 27017:27017 --restart always -v /web/data/mongo1:/data/db --name mongo1 mongo --auth
>开启权限认证，默认是关闭的

# redis
docker run -d -p 6379:6379 --name redis1 --restart always -d -v /web/data/redis/data:/data redis

# sonatype-work
mkdir /web/data/sonatype-work && chown -R 200 /web/data/sonatype-work
docker run -d -p 8081:8081 --restart always --name nexus -v /share/volume3/Temp/Docker/sonatype-work:/sonatype-work sonatype/nexus

docker run -d -p 8081:8081 --restart always --name nexus -v /share/volume3/Temp/Docker/sonatype-work:/nexus-data sonatype/nexus3


# portainer
docker run -ti -d -p 2375:2375 --restart=always --name shipyard-proxy -v /var/run/docker.sock:/var/run/docker.sock -e PORT=2375 shipyard/docker-proxy:latest
docker run -d --restart=always -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock --name portainer -v /web/data/portainer:/data portainer/portainer

# active mq
docker pull webcenter/activemq
docker run --name activemq -p 61616:61616 -e ACTIVEMQ_ADMIN_LOGIN=admin -e ACTIVEMQ_ADMIN_PASSWORD=admin123 --restart=always -d webcenter/activemq:latest

# rabbit mq
docker run -d --restart=always -v /web/data/rabbitmq:/var/lib/rabbitmq -e RABBITMQ_DEFAULT_USER=guest -e RABBITMQ_DEFAULT_PASS=guest --hostname my-rabbit --name rabbitmq -p 15672:15672 -p 5672:5672 rabbitmq:3-management

# hadoop hdfs
docker network create --subnet=172.19.0.0/16 hadoopNet

docker rm -f hdfs-namenode hdfs-datanode1 hdfs-datanode2

docker run -d --name hdfs-namenode -h master --net hadoopNet --ip 172.19.0.10 -p 50070:50070 -p 8020:8020 -e "HDFS_CONF_DFS_REPLICATION=2"  -e "CLUSTER_NAME=cluster0" uhopper/hadoop-namenode

docker run -d --name hdfs-datanode1 --net hadoopNet --ip 172.19.0.11 -e "CORE_CONF_fs_defaultFS=hdfs://192.168.100.200:8020" -e "HDFS_CONF_DFS_REPLICATION=2" -e "CLUSTER_NAME=cluster0" uhopper/hadoop-datanode:latest

docker run -d --name hdfs-datanode2 --net hadoopNet --ip 172.19.0.12 -e "CORE_CONF_fs_defaultFS=hdfs://172.19.0.10:8020" -e "HDFS_CONF_DFS_REPLICATION=2" -e "CLUSTER_NAME=cluster0" uhopper/hadoop-datanode:latest

docker run -itd --name=hadoopserver -p 8030:8030 -p 8040:8040 -p 8042:8042 -p 8088:8088 -p 19888:19888 -p 49707:49707 -p 50010:50010 -p 50020:50020 -p 50070:50070 -p 50075:50075 -p 50090:50090 -p 8020:9000 sequenceiq/hadoop-docker:2.7.0

# gitlab
docker run -itd --env 'GITLAB_PORT=10080' --env 'GITLAB_SSH_PORT=10022' -e 'GITLAB_HOST=192.168.2.208' --publish 10080:80 --publish 1022:22 --name gitlab --restart always  --volume /home/data/gitlab/config:/etc/gitlab  --volume /home/data/gitlab/logs:/var/log/gitlab --volume /home/data/gitlab/data:/var/opt/gitlab     gitlab/gitlab-ce:latest

# elastic search
mkdir -f -v /web/data/elasticsearch

elsasticsearch.yml

```yml
http.host: 0.0.0.0
network.host: 0.0.0.0
cluster.name: elasticsearch

# head插件跨域
http.cors.enabled: true
http.cors.allow-origin: "*"

# Uncomment the following lines for a production cluster deployment
#transport.host: 0.0.0.0
#discovery.zen.minimum_master_nodes: 1

```

docker run -d --restart always --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node"  -v /web/data/elasticsearch-head/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /web/data/elasticsearch:/usr/share/elasticsearch/data elasticsearch

docker run -d --restart always --name elasticsearch-head -p 9100:9100 mobz/elasticsearch-head:5


## 修复elasticsearch 9300端口问题

1. 在宿主机执行：sudo sysctl -w vm.max_map_count=262144

2. 修改docker中jre的java.policy

通过docker命令   docker exec -it myelasticsearch(这个是我的elasticsearch容器的名字) /bin/bash进去后找到java.policy文件并修改                 

经过研究这是java的权限设置问题，ES5.x之前的版本可以在elasticsearch.yml里设置security.manager.enabled为false。在ES5.x删除了该设置参数，暂时我们可以修改jdk中配置。以jdk8为例，在${JAVA_HOME}/jre/lib/security，修改java.policy，在grant{ }中添加下列内容：

```conf
permission java.security.AllPermission；
```

## 修复elasticsearch-head请求错误

1. 拷贝出app.js
docker cp elasticsearch-head:/usr/src/app/_site/app.js ./

2. 修改app.js
在1305行插入
contentType: 'application/json',

3. 将app.js拷贝回去覆盖
docker cp  ./app.js elasticsearch-head:/usr/src/app/_site/app.js

4. 刷新页面即可


# jenkins
docker run -u root -d --name jenkins -p 10000:8080  -v /web/soft/jenkins/home:/var/jenkins_home -v /home/:/web/ -v /var/run/docker.sock:/var/run/docker.sock jenkinsci/blueocean

# canal
docker run --name canal-server -e canal.instance.master.address=192.168.100.200:3306 -e canal.instance.dbUsername=canal -e canal.instance.dbPassword=canal -e canal.instance.connectionCharset=UTF-8 -p 11111:11111 -d canal/canal-server

# neo4j
docker run --name neo4j -d -p 7474:7474 -p 7687:7687  -v /web/data/neo4j:/data  neo4j