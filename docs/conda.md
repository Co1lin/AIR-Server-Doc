# 使用 Conda

**Conda** 是一个开源、跨平台、语言无关的包管理与环境管理系统。本篇介绍在集群中使用 conda 的方法。

## 换源加速

执行下面的命令，使用国内的镜像源加快包安装速度：

```shell
echo "channels: 
  - defaults
show_channel_urls: true
default_channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
custom_channels:
  conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  msys2: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  bioconda: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  menpo: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  simpleitk: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud" > ~/.condarc
```

## 创建环境

```shell
conda create [env_name]
```

## 激活环境

通过以下命令激活新的环境：

```shell
conda activate /envs/[env_name]
```

