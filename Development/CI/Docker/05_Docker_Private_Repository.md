Docker私有仓库
=========
# 关于
官方的Docker hub是一个用于管理公共镜像的好地方，我们可以在上面找到我们想要的镜像，也可以把我们自己的镜像推送上去。但是，有时候，我们的使用场景需要我们拥有一个私有的镜像仓库用于管理我们自己的镜像。这个可以通过开源软件Registry来达成目的。

Registry在github上有两份代码：老代码库和新代码库。老代码是采用python编写的，存在pull和push的性能问题，出到0.9.1版本之后就标志为deprecated，不再继续开发。从2.0版本开始就到在新代码库进行开发，新代码库是采用go语言编写，修改了镜像id的生成算法、registry上镜像的保存结构，大大优化了pull和push镜像的效率。

官方在Docker hub上提供了registry的镜像（详情），我们可以直接使用该registry镜像来构建一个容器，搭建我们自己的私有仓库服务。

# 部署
```shell
    $ sudo docker pull registry
    $ sudo docker run -d -v /web/data/registry:/var/lib/registry -p 5000:5000 --restart=always --name registry registry
```
Registry服务默认会将上传的镜像保存在容器的/var/lib/registry，我们将主机的/web/data/registry目录挂载到该目录，即可实现将镜像保存到主机的/web/data/registry目录了。

打开浏览器输入http://127.0.0.1:5000/v2，返回{}说明registry运行正常。

# 验证
```shell
    # 上传到本地镜像
    $ sudo docker tag hello-world 127.0.0.1:5000/hello-world
    $ sudo docker images
    $ sudo docker push 127.0.0.1:5000/hello-world /** push成功 **/
    The push refers to a repository [127.0.0.1:5000/hello-world] (len: 1)
    975b84d108f1: Image successfully pushed
    3f12c794407e: Image successfully pushed
    latest: digest: sha256:1c7adb1ac65df0bebb40cd4a84533f787148b102684b74cb27a1982967008e4b size: 2744

    # 删除本地镜像
    $ sudo docker rmi hello-world
    $ sudo docker rmi 127.0.0.1:5000/hello-world
    $ sudo docker images

    # 重新获取镜像
    $ sudo docker pull 127.0.0.1:5000/hello-world
    Using default tag: latest
    latest: Pulling from hello-world
    b901d36b6f2f: Pull complete
    0a6ba66e537a: Pull complete
    Digest: sha256:1c7adb1ac65df0bebb40cd4a84533f787148b102684b74cb27a1982967008e4b

    $ sudo docker tag 127.0.0.1:5000/hello-world hello-world
    $ sudo docker images
```

# Docker配置
可能会出现无法push镜像到私有仓库的问题。这是因为我们启动的registry服务不是安全可信赖的。这是我们需要修改docker的配置文件/etc/default/docker，添加下面的内容，
```shell
    DOCKER_OPTS="--insecure-registry xxx.xxx.xxx.xxx:5000"
```
