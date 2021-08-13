# Conda 使用指南

为了避免耗尽系统盘空间，请通过以下方式在 `/data` 分区下创建新的环境：

```shell
conda create -p /envs/[env_name]
```

通过以下命令激活新的环境：

```shell
conda activate /envs/[env_name]
```

对于在系统分区（包括但不限于通过 `conda create -n name` 方式创建的环境）下的虚拟环境，请尽快迁移到 `/data` 目录下，迁移方法如下：

```shell
conda create -p /envs/[new_env_name] --clone [old_env]
conda remove -n [old_env] --all
```

!!! tips "`/env` 是指向 `/data/.conda/envs` 的软链接"

