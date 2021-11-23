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
conda activate [env_name]
```

在集群上使用 srun, sbatch 等命令提交任务前，需要先通过上述命令进入特定环境。

## 非 Bash 环境

如果需要在其他 shell ( zsh 等) 中使用 conda ，使用下面等命令进行初始化：

```shell
/usr/local/miniconda3/bin/conda init
```

然后重新进入 shell ，即可正常使用 conda 。
