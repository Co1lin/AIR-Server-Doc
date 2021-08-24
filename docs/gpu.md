# GPU资源使用

正在引入 Slurm 调度系统，在不久的将来在所有机器上都可以使用 Slurm 提交任务。

## AIR1 & AIR2

AIR1 服务器配备 8 张 RTX 3090 24GB 显卡；AIR2 服务器配备 10 张 RTX 3090 24GB 显卡。

在运行实验前，请通过以下指令指定需要使用的 GPU 卡的编号（编号从 0 开始）

```shell
export CUDA_VISIBLE_DEVICES='0,1,2' # 指定程序在 GPU#0，GPU#1，GPU#2 上运行
```

!!! warning "避免程序异常占用显存"
    请定时检查程序的运行状态。如果你的程序异常退出，可能会有子进程仍然存活，此时其它程序无法利用该子进程占用的显存。请通过 `nvidia-smi` 命令检查是否有异常进程占用显存。

## AIR3

AIR3 服务器配备 3 张 A100 显卡，其中一张显卡按照运算单元 3:2:2，显存 20GB:10GB:10GB 的方式分割成为三张虚拟显卡。完整、20GB、10GB 显卡在下文中用 `7c40gb`、`3c20gb`、`2c10gb` 指代。

AIR3服务器使用Slurm系统分配GPU资源。

!!! tip "Slurm系统"
    AIR3 服务器部署了 Slurm 系统。**Slurm 任务调度工具**，是一个用于 Linux 和 Unix 内核系统的免费、开源的任务调度工具，被世界范围内的超级计算机和计算集群广泛采用。它提供了三个关键功能：

    1. 为用户分配一定时间的专享或非专享的资源(计算机节点)，以供用户执行工作
    2. 它提供了一个框架，用于启动、执行、监测在节点上运行着的任务(通常是并行的任务，例如 MPI)
    3. 为任务队列合理地分配资源

常用命令如下：

### 查看队列的任务

  ```shell
  squeue  # 包括各任务的 JOBID
  ```

### 进入 shell

  ```shell
  sterm --gres=gpu:7c40gb:1  # 申请使用一张完整的A100卡
  sterm --gres=gpu:3c20gb:1  # 申请使用一张占3/7流处理器，20GB显存的虚拟显卡
  sterm --gres=gpu:2c10gb:1  # 申请使用一张占2/7流处理器，10GB显存的虚拟显卡
  ```

### 前台执行命令

  ```shell
  srun --gres=gpu:3c20gb:1 python run.py  # 申请方式同上，在前台执行命令
  ```

	脚本运行结果会显示在命令行中显示。

### 后台执行脚本 （推荐）

  ```shell
  sbatch [batchfile]
  ```

  运行脚本样例如下：

  ```shell
  #!/bin/bash
  #SBATCH -J example                # 作业名为 test
  #SBATCH -o example.out            # 屏幕上的输出文件重定向到 example.out
  #SBATCH --gres=gpu:2c10gb:1       # 单个节点使用 1 块 2c10gb 类型 GPU 卡
  #SBATCH -p a100                   # 作业提交的分区为 a100 分区
  #SBATCH -N 1                      # 作业申请 1 个节点
  #SBATCH --ntasks-per-node=1       # 单节点启动的进程数为 1
  #SBATCH --cpus-per-task=4         # 单任务使用的 CPU 核心数为 4
  #SBATCH -t 1:00:00                # 任务运行的最长时间为 1 小时
  #!SBATCH --qos=debug              # 作业使用的 QoS 为 debug （QoS系统尚未启用）
  
  # 输入要执行的命令，例如 ./hello 或 python test.py 等
  source ~/.zshrc
  conda activate /envs/tbw_fp
  python -V                    
  python -c "print('Hello, world!')"
  ```

  脚本运行结果不会显示在命令行中，请查看脚本中指定的输出文件。

### 取消任务

  ```shell
  scancel [id]
  ```
	
	`id`为任务的ID，可以通过`squeue`命令查看。

!!! warning "一次使用一张虚拟显卡"
    A100上的虚拟显卡由NVIDIA MIG实现，分割后的虚拟显卡也称为MIG实例。由于该系统的限制，多张MIG实例无法同时使用，因此通过slurm系统申请系统MIG资源时最多只可以申请1张，否则将会造成资源的浪费。
    如果一个MIG实例无法满足实验要求，请申请更高规格的MIG实例，或完整的A100显卡（`7c40gb`实例）

!!! note "无需指定设备"
    使用Slurm系统无需通过`CUDA_VISIBLE_DEVICES`指定使用的设备。设备由Slurm系统自动分配。
