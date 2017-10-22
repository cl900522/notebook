# registry
docker run -d -p 5000:5000 --restart always --name registry registry
docker run -d -p 5000:5000 --restart always -v /web/data/registry:/tmp/registry --name registry registry

# zookeeper
docker run -d -p 2181:2181 --name zookeeper1 --restart always zookeeper
<<<<<<< HEAD
docker run -d -p 2181:2181 --name zookeeper1 --restart always zookeeper

# mysql
docker run -d -p 3306:3306 --name mysql1 --restart always -e MYSQL_ROOT_PASSWORD=root -v /web/data/mysql1-data:/var/lib/mysql mysql
docker run -d -p 3306:3306 --name mysql1 --restart always -e MYSQL_ROOT_PASSWORD=root -v /web/data/mysql1-data:/var/lib/mysql mysql

# mongo
=======
docker run -d -p 3306:3306 --name mysql1 --restart always -e MYSQL_ROOT_PASSWORD=root -v /web/data/mysql1-data:/var/lib/mysql mysql --character-set-server=utf8 --collation-server=utf8_bin
>>>>>>> 374b5517aa199202f1fec4b207af9d3d30a13325
docker run -d -p 27017:27017 --name mongo1 --restart always mongo
docker run -d -p 27017:27017 --restart always -v /web/data/mongo1:/data/db --name mongo1 mongo

# redis
eocker run -d -p 6379:6379 --name redis1 --restart always -d -v /web/data/redis/data:/data redis

# sonatype-work
mkdir /web/data/sonatype-work && chown -R 200 /web/data/sonatype-work
docker run -d -p 8081:8081 --restart always --name nexus -v /web/data/sonatype-work:/sonatype-work sonatype/nexus

# portainer
docker run -ti -d -p 2375:2375 --restart=always --name shipyard-proxy -v /var/run/docker.sock:/var/run/docker.sock -e PORT=2375 shipyard/docker-proxy:latest
docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock --name portainer -v /web/data/portainer:/data portainer/portainer

# active mq
docker pull webcenter/activemq
docker run --name activemq -p 61616:61616 -e ACTIVEMQ_ADMIN_LOGIN=admin -e ACTIVEMQ_ADMIN_PASSWORD=admin123 --restart=always -d webcenter/activemq:latest

# rabbit mq
docker run -d --restart=always --hostname my-rabbit --name main-rabbit -p 5672:5672 rabbitmq:3.6.11-alpine
docker run -d --restart=always --hostname my-rabbit --name portal-rabbit -p 15672:15672 rabbitmq:3.6.11-management-alpine

# hadoop hdfs
docker network create --subnet=172.19.0.0/16 hadoopNet

docker rm -f hdfs-namenode hdfs-datanode1 hdfs-datanode2

docker run -d --name hdfs-namenode -h master --net hadoopNet --ip 172.19.0.10 -p 50070:50070 -p 8020:8020 -e "HDFS_CONF_DFS_REPLICATION=2"  -e "CLUSTER_NAME=cluster0" uhopper/hadoop-namenode

docker run -d --name hdfs-datanode1 --net hadoopNet --ip 172.19.0.11 -e "CORE_CONF_fs_defaultFS=hdfs://192.168.100.200:8020" -e "HDFS_CONF_DFS_REPLICATION=2" -e "CLUSTER_NAME=cluster0" uhopper/hadoop-datanode:latest

docker run -d --name hdfs-datanode2 --net hadoopNet --ip 172.19.0.12 -e "CORE_CONF_fs_defaultFS=hdfs://172.19.0.10:8020" -e "HDFS_CONF_DFS_REPLICATION=2" -e "CLUSTER_NAME=cluster0" uhopper/hadoop-datanode:latest
