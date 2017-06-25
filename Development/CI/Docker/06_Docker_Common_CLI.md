docker run -d -p 5000:5000 --restart always --name registry registry
docker run -d -p 8081:8081 --name nexus --restart always -v /web/data/sonatype-work/:/nexus/sonatype-work/ nexus:1.0
docker run -d -p 2181:2181 --name zookeeper1 --restart always zookeeper
docker run -d -p 3306:3306 --name mysql1 --restart always -e MYSQL_ROOT_PASSWORD=root -v /web/data/mysql1-data:/var/lib/mysql mysql
docker run -d -p 27017:27017 --name mongo1 --restart always mongo
