# GPU

正在引入Slurm调度系统，在不久的将来在所有机器上都可以使用Slurm提交任务。

## AIR1 & AIR2

AIR1服务器配备8张RTX 3090 24GB显卡；AIR2服务器配备10张RTX 3090 24GB显卡。

在运行实验前，请通过以下指令指定需要使用的GPU卡的编号（编号从0开始）

```shell
export CUDA_VISIBLE_DEVICES='0,1,2' # 指定程序在GPU#0，GPU#1，GPU#2上运行
```

!!! warning "避免程序异常占用显存"
    请定时检查程序的运行状态。如果你的程序异常退出，可能会有子进程仍然存活，此时其它程序无法利用该子进程占用的显存。请通过`nvidia-smi`命令检查是否有异常进程占用显存。

## AIR3

AIR3服务器配备3张A100显卡，其中一张显卡按照运算单元3:2:2，显存20GB:10GB:10GB的方式分割成为三张虚拟显卡。完整、20GB、10GB显卡在下文中用`7c40gb`、`3c20g`、`2c10g`指代。

AIR3服务器部署了slurm系统，（这里简单介绍一下slurm）

常用命令如下：

* 查看当前运行&排队的任务

```shell
squeue  # 包括各任务的JOBID
```

* 申请GPU资源后进入shell（交互模式）

```shell
sterm --gres=gpu:7c40g:1  # 申请使用一张完整的A100卡
sterm --gres=gpu:3c20g:1  # 申请使用一张占3/7流处理器，20GB显存的虚拟显卡
sterm --gres=gpu:2c10g:1  # 申请使用一张占2/7流处理器，10GB显存的虚拟显卡
```

* 申请GPU资源执行命令

```shell
srun --gres=gpu:3c20g:1 python run.py  # 申请方式同上，在前台执行命令
```

* 申请GPU资源执行后台脚本 ==TODO==
* 申请取消`JOBID`为`id`的任务

```shell
scancel [id]
```

!!! warning 
    A100上的虚拟显卡由NVIDIA MIG实现，分割后的虚拟显卡也称为MIG实例。由于该系统的限制，多张MIG实例无法同时使用，因此通过slurm系统申请系统MIG资源时最多只可以申请1张，否则将会造成资源的浪费。

!!! note
    使用Slurm系统无需通过`CUDA_VISIBLE_DEVICES`指定使用的设备。设备由Slurm系统自动分配。
