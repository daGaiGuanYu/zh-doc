# Dockerfile 参考

Docker 可以通过读取一个 Dockerfile 文件里的指令，自动构建镜像。Dockerfile 是一个文本文档，包含用来组装一个镜像的所有命令。使用 ```docker build```，用户可以创建一个自动化的构建过程，若干命令行指令被连续地执行。

本页描述你可以在 dockerfile 中使用的命令。读完本页，可以参考 dockerfile 的[最佳实践](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/)（一个带提示的教程）。

## 用法
```docker build`` 命令从一个 Dockerfile 和一个上下文构建一个镜像。构建过程的上下文是一组指定位置的文件（用 PATH 或者 URL 指定）。PATH 是你本地文件系统的一个文件夹。URL 是一个 git 仓库的位置。

上下文被递归地处理。所以，一个 PATH 包含所有子文件，并且 URL 包含整个仓库和它的子模块。下面的例子展示了一个构建命令，它使用当前目录作为上下文。
```
$ docker build .
Sending build context to Docker daemon  6.51 MB
...
```
构建是 Docker 的守护进程运行的，而不是 CLI。构建过程第一步，就是把整个上下文（递归地）发送给守护进程。在大多数情况下，使用一个空目录作为上下文并且把你的 Dockerfile 放在那个目录，并且只把需要的文件添加到里面。

> 警告：不要使用根目录作为 PATH，因为这会使构建过程把硬盘上的所有内容传送给 Docker 守护进程。

要使用构建上下文里的文件，Dockerfile 可以引用通过指令声明的文件，比如，COPY 指令。为了提高构建性能，使用一个 .dockerignore 文件排除无关文件和目录（译者注：就想 git 里的 .gitignore 那样）。本页下面会有更多关于如何创建一个 .dockerignore 文件的信息。

一般来说，Dockerfile 被命名为 Dockerfile，被放在上下文的根目录下。你也可以使用 -f 标记（跟随在 docker build 之后）指定一个你文件系统中任意位置的 Dockerfile。
```
$ docker build -f /path/to/a/Dockerfile .
```
你可以指定一个仓库和标签，来存放新镜像，如果构建成功的话。
```
$ docker build -t shykes/myapp .
```
想把镜像标记到多个仓库（在构建之后），可以在运行 build 命令的时候添加 -t 参数：
```
$ docker build -t shykes/myapp:1.0.2 -t shykes/myapp:latest .
```
在 docker 守护进程运行 Dockerfile 里的指令之前，它将对 Dockerfile 执行一个预先的校验，并且返回一个错误，如果语法错误的话：
```
$ docker build -t test/myapp .
Sending build context to Docker daemon 2.048 kB
Error response from daemon: Unknown instruction: RUNCMD
```
Docker 守护进程一个接着一个地运行 Dockerfile 里的指令。在输出新镜像的 id 之前，如果有必要的话，提交每个指令的结果到新的镜像。Docker 守护进程会自动清理你发送的上下文。

注意，Dockerfile 里每一个指令单独地运行，并且生成新的镜像。所以 ```RUN cd /tmp``` 对之后的指令没有任何影响。

如果可能的话，Docker 会复用中间的镜像（缓存），以明显加速```docker build```这个过程。控制台输出中的 Using cache 就是这个意思。（更多信息，请参考最佳实践的[构建缓存部分](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#build-cache)。

```
$ docker build -t svendowideit/ambassador .
Sending build context to Docker daemon 15.36 kB
Step 1/4 : FROM alpine:3.2
 ---> 31f630c65071
Step 2/4 : MAINTAINER SvenDowideit@home.org.au
 ---> Using cache
 ---> 2a1c91448f5f
Step 3/4 : RUN apk update &&      apk add socat &&        rm -r /var/cache/
 ---> Using cache
 ---> 21ed6e7fbb73
Step 4/4 : CMD env | grep _TCP= | (sed 's/.*_PORT_\([0-9]*\)_TCP=tcp:\/\/\(.*\):\(.*\)/socat -t 100000000 TCP4-LISTEN:\1,fork,reuseaddr TCP4:\2:\3 \&/' && echo wait) | sh
 ---> Using cache
 ---> 7ea8aef582cc
Successfully built 7ea8aef582cc
```
构建缓存（build cache）仅当一个本地的父级链存在时才会被使用。也就是说这些镜像被之前的构建创建，或者整个镜像链通过 ```docker load```被加载。如果你想使用一个指定的镜像的构建缓存，你可以使用```--cache-from``` 指令来声明它。被```cache-from```声明的镜像不需要有父级链，并且可能从其他的 registries 拉取。

当你完成你的构建，可以研究一下[推送一个仓库到registry](https://docs.docker.com/engine/tutorials/dockerrepos/#/contributing-to-docker-hub)。
