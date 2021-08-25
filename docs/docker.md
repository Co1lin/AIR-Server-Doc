# Docker

## 简介

Docker 在 AIR 服务器上的常见作用：

- 无需主机上的 `sudo` 权限执行命令；
- 使用他人打包好的实验环境进行实验，免去配环境的麻烦，以及修改 CUDA 版本等在主机上会影响他人的操作。

!!! warning "区分 image 和 container"
    
    初学者常常混淆 Docker 中的 image 和 container 两个概念。
    - image （镜像）是 `docker pull` 下来的，可以通过 `docker images` 查看系统上所有的 docker image 。
    - container （容器）是执行 `docker run` 之后产生的。可以通过 `docker ps -a` 查看系统上所有 container 的运行情况。
    - image 相当于系统安装光盘， container 相当于用 image 安装之后运行起来的主机。所以在 container 做修改并不会修改 image 。同样的 image ，无论从它跑起来的 container 怎么改，每次执行 `docker run` 之后生成的新容器内部文件都是一样的。

## 请及时清理不用的 image 和 container

前面提到：

- 用 `docker images` 查看所有的镜像。
- 用 `docker ps -a` 查看所有的容器。

为节约磁盘空间，

- 对于不用的镜像，请及时用 `docker rmi <image_id>` 删除。
- 对于不用的容器，请及时用 `docker stop <container_id>` 和  `docker rm <container_id>` 停止并删除。

## 如何保存 image/container ？

前面提到，修改 container 并不会修改 image 。但是可以通过

```shell
docker commit <container_id> <img_name>:<img_tag>
```

将一个 container commit 成为一个被标记为 `img_name:img_tag` 的 image 。这可以用于：在某个 image 的 container 中做了修改，保存修改之后状态为 image ，这样可以从新 image 启动，不必从旧 image 启动之后再重复自己的修改。

在不同机器间传输 image ，除了将 image 发布到类似于 [Docker Hub](https://hub.docker.com/) 的仓库然后在其它机器上拉取，还可以通过以下方式以文件形式保存、加载 image 。

```shell
docker save container_id > container.tar
docker load < container.tar
```

显然，结合上述的 `commit`, `save`, `load` 三者，就可以实现 container 的保存与加载。

!!! danger "防止 Docker 的 layer （层）信息丢失！"
    docker 分层的文件系统使得不同 image 可以共享共有的层。而上述 `commit`, `save`, `load` 操作都能够保留层信息，可以实现层共享或者回退。
    
    而另一组操作 `export`, `import` 虽然也能实现 image 的保存和加载，但会丢失层信息！先 `export` 再 `import` 之后的 image 只会有（通常很大的）一层！这使得你的 image 如果传到 docker 仓库上再被 pull 时无法共享机器上现有的层，每次更新时不能仅仅更新最后做了修改的层，而是要更新整个 image ，增加了网络传输量和磁盘空间占用。同时也无法根据历史实现回退。因此不推荐这种操作。

