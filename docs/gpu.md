# 通过 slurm 系统使用 GPU 资源

当前集群中，A100 显卡通过 slurm 系统分配使用；3090 显卡可以直接使用。

!!! tip "Slurm 系统"
    **Slurm 任务调度工具 **，是一个用于 Linux 和 Unix 内核系统的免费、开源的任务调度工具，被世界范围内的超级计算机和计算集群广泛采用。它提供了三个关键功能：

    1. 为用户分配一定时间的专享或非专享的资源 (计算机节点)，以供用户执行工作
    2. 它提供了一个框架，用于启动、执行、监测在节点上运行着的任务 (通常是并行的任务，例如 MPI)
    3. 为任务队列合理地分配资源

## 服务器架构

![all](1.png)

## 节点与任务

在 slurm 系统中，**节点**指可以独立运行程序的服务器。目前 slurm 系统内共有4个节点：

* 登录节点 `air-server` ：集群用户可以通过 `ssh` 连接 `103.242.175.247` 登录。跳板节点上配备 2 张 A100 GPU 供调试，该 GPU 使用无需通过 slurm 系统。
* 运算节点：共有 3 个，分别是 `air-node-01`（配备 6 张 A100 GPU），`air-node-02`（配备 2 张 A100 GPU），`air-node-03`（配备 6 张 A100 GPU）。集群用户需要通过 slurm 系统提交任务使用运算结点上的 GPU 资源，使用方式见<a href="ref">下方</a>。

**任务**是对程序以及程序运行占用资源的抽象，每一个任务有的单独的编号（`JOBID`）。用户将程序提交至 slurm 系统后，任务队列会加入一个新任务；当集群中有可用的资源时由 slurm 系统自动分配资源与任务绑定，并从任务队列中取出并开始运行。

## slurm 常用操作 <a id="ref"></a>

```shell
# 运行任务
srun --gres=gpu:a100:[gpu_count] [cmd]

# 查看任务队列
squeue

## 任务ID     用户    任务名     任务状态    已运行时间  运行时间限制 任务运行主机  任务占用资源
## JOBID     USER    NAME      STATE     TIME      TIME_LIMIT NODELIST    TRES_PER_NODE       
## 149       tb5zhh  bash      RUNNING   5:46:33   UNLIMITED  air-node-03 gpu:a100:1          
## 150       tb5zhh  bash      RUNNING   5:46:30   UNLIMITED  air-node-03 gpu:a100:1          
## 154       lpf     python    RUNNING   2:22:36   UNLIMITED  air-node-03 gpu:a100:4 

# 查看任务详情（已分配资源，排队原因，失败原因）
scontrol show job [JOBID]

# 取消任务
scancel [JOBID]
```

## slurm 进阶使用

### sbatch

`sbatch`命令异步地向`slurm`系统提交任务，执行后立即返回；任务输出由脚本指定。通过`sbatch`可以快速地向`slurm`提交大量任务。

```shell
sbatch slurm.sh
```

样例`slurm.sh`脚本如下

```shell
#!/bin/bash
#SBATCH --job-name example # 作业名为 example
#SBATCH --output %a.out    # 屏幕上的输出文件重定向到 [JOBID].out
#SBATCH --gres gpu:a100:1  # 使用 1 张 A100 显卡
#SBATCH --time 1:00:00     # 任务运行的最长时间为 1 小时
#SBATCH --array 0-15       # 可选，提交组合任务（此处提交了16个任务）
                           # 任务 ID 通过 SLURM_ARRAY_TASK_ID 环境变量访问
#!SBATCH --qos debug       # 任务的优先级为 debug （QoS 系统尚未启用）
# 上述行指定参数将传递给 sbatch 作为命令行参数
# 中间不可以有非 #SBATCH 开头的行

# 执行 sbatch 命令前先通过 conda activate [env_name] 进入环境
# 执行程序
echo ${SLURM_ARRAY_TASK_ID}
python -V
python -c "print ('Hello, world!')"
```
