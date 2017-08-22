docker run -d -p 5000:5000 --restart always --name registry registry
docker run -d -p 8081:8081 --name nexus --restart always -v /web/data/sonatype-work/:/nexus/sonatype-work/ nexus:1.0
docker run -d -p 2181:2181 --name zookeeper1 --restart always zookeeper
docker run -d -p 3306:3306 --name mysql1 --restart always -e MYSQL_ROOT_PASSWORD=root -v /web/data/mysql1-data:/var/lib/mysql mysql
docker run -d -p 27017:27017 --name mongo1 --restart always mongo

docker run -d -p 5000:5000 --restart always -v /web/data/registry:/tmp/registry --name registry registry
docker run -d -p 3306:3306 --name mysql1 --restart always -e MYSQL_ROOT_PASSWORD=root -v /web/data/mysql1-data:/var/lib/mysql mysql
docker run -d -p 8081:8081 --name nexus --restart always -v /web/data/sonatype-work/:/nexus/sonatype-work/ nexus:1.0
docker run -d -p 2181:2181 --name zookeeper1 --restart always zookeeper
eocker run -d -p 6379:6379 --name redis1 --restart always -d -v /web/data/redis/data:/data redis
docker run -d -p 27017:27017 --restart always -v /web/data/mongo1:/data/db --name mongo1 mongo

mkdir /web/data/sonatype-work && chown -R 200 /web/data/sonatype-work
docker run -d -p 8081:8081 --restart always --name nexus -v /web/data/sonatype-work:/sonatype-work sonatype/nexus

docker run -ti -d -p 2375:2375 --restart=always --name shipyard-proxy -v /var/run/docker.sock:/var/run/docker.sock -e PORT=2375 shipyard/docker-proxy:latest
docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock --name portainer -v /web/data/portainer:/data portainer/portainer
