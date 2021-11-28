# 在集群中使用容器应用

由于 Docker 容器存在的一些不足，因此我们采用 NVIDIA 提供的工具 [enroot](https://github.com/NVIDIA/enroot) 来在集群上运行 Docker 应用。如果需要使用容器应用，请参照下面的步骤进行操作。

## 将 Docker 镜像转换为 enroot 格式镜像

```sh
# From Docker hub
enroot import docker://[USER@][REGISTRY#]IMAGE[:TAG]
# Example: 
# docker pull pytorch/pytorch -> enroot import docker://pytorch/pytorch

# From Docker daemon
enroot import dockerd://IMAGE[:TAG]
# Examples
docker images
# REPOSITORY		TAG		IMAGE ID				CREATED					SIZE
# openjdk				8     448f9e615795    11 days ago     526MB
# -> enroot import dockerd://openjdk:8
```

运行命令后，会在当前目录下生成`.sqsh` 格式的本地镜像。

## 通过 `srun` 命令提交在容器中运行的命令

只能通过 `srun` 命令提交在容器中运行的命令（不支持 `sbatch` ）。

阅读以下部分前请先阅读[ `srun` 命令](https://co1lin.github.io/AIR-Server-Doc/gpu/#21-srun)的使用方法。如果需要在容器中运行命令，仅需要在调用 `srun` 命令时额外添加下面部分参数即可：

* `--container-image`: 容器镜像。可以为上一步生成 `.sqsh` 文件路径，也可以是镜像 URL 如 `pytorch/pytorch`. 推荐使用前者作为输入，避免重复 `import` 浪费时间
* `--container-mounts=SRC:DST[:FLAGS][,SRC:DST...]`: （可选）路径映射。将当前系统中 `SRC` 映射至容器内 `DST`，如为可读写映射，需将 `FLAGS` 置为 `rw`
* `--container-save=SAVE_PATH`: （可选）容器保存目录。如果指定这一参数，运行结束后容器会保存自身状态至 `SAVE_PATH` 下的 `.sqsh` 格式文件
* `--container-mount-home`: （可选）挂载 `/home` 目录
* `--no-container-mount-home`: （可选）不挂载 `/home` 目录
* `--container-remap-root`:  （可选）在容器内身份为 `root`
* `--container-writable`: （可选）容器内文件系统挂载为可读写

* `--container-readonly`: （可选）容器内文件系统挂载为只读