Swarm Routing Mesh
=========

>Docker Engine swarm mode makes it easy to publish ports for services to make them available to resources outside the swarm. All nodes participate in an ingress routing mesh. The routing mesh enables each node in the swarm to accept connections on published ports for any service running in the swarm, even if there’s no task running on the node. The routing mesh routes all incoming requests to published ports on available nodes to an active container.

In order to use the ingress network in the swarm, you need to have the following ports open between the swarm nodes before you enable swarm mode:

* Port 7946 TCP/UDP for container network discovery.
* Port 4789 UDP for the container ingress network.

You must also open the published port between the swarm nodes and any external resources, such as an external load balancer, that require access to the port.

# Publish a port for a service
Use the --publish flag to publish a port when you create a service:

```shell
$ docker service create \
  --name <SERVICE-NAME> \
  --publish <PUBLISHED-PORT>:<TARGET-PORT> \
  <IMAGE>
```

The <TARGET-PORT> is the port where the container listens. The <PUBLISHED-PORT> is the port where the swarm makes the service available.

For example, the following command publishes port 80 in the nginx container to port 8080 for any node in the swarm:

```
$ docker service create \
  --name my-web \
  --publish 8080:80 \
  --replicas 2 \
  nginx
```
When you access port 8080 on any node, the swarm load balancer routes your request to an active container.

The routing mesh listens on the published port for any IP address assigned to the node. For externally routable IP addresses, the port is available from outside the host. For all other IP addresses the access is only available from within the host.

![ingress-routing-mesh](/images/2017/07/ingress-routing-mesh.png)

## Publish a port for TCP only or UDP only
By default, when you publish a port, it is a TCP port. You can specifically publish a UDP port instead of or in addition to a TCP port. When you publish both TCP and UDP ports, Docker 1.12.2 and earlier require you to add the suffix /tcp for TCP ports. Otherwise it is optional.

### TCP ONLY

The following two commands are equivalent.
```shell
$ docker service create --name dns-cache -p 53:53 dns-cache

$ docker service create --name dns-cache -p 53:53/tcp dns-cache
```

### TCP AND UDP
```shell
$ docker service create --name dns-cache -p 53:53/tcp -p 53:53/udp dns-cache
```

### UDP ONLY
```shell
$ docker service create --name dns-cache -p 53:53/udp dns-cache
```
