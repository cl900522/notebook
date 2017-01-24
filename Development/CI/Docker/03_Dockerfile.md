Docker 可以通过 Dockerfile 的内容来自动构建镜像。Dockerfile 是一个包含创建镜像所有命令的文本文件，通过docker build命令可以根据 Dockerfile 的内容构建镜像，在介绍如何构建之前先介绍下 Dockerfile 的基本语法结构。

Dockerfile 有以下指令选项:
1. FROM
2. MAINTAINER
3. RUN
4. CMD
5. EXPOSE
6. ENV
7. ADD
8. COPY
9. ENTRYPOINT
10. VOLUME
11. USER
12. WORKDIR
13. ONBUILD
