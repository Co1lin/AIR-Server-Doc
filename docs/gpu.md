# 通过 slurm 系统使用 GPU 资源

!!! tip "Slurm 系统"
    **Slurm 任务调度工具 **，是一个用于 Linux 和 Unix 内核系统的免费、开源的任务调度工具，被世界范围内的超级计算机和计算集群广泛采用。它提供了三个关键功能：

    1. 为用户分配一定时间的专享或非专享的资源 (计算机节点)，以供用户执行工作
    2. 它提供了一个框架，用于启动、执行、监测在节点上运行着的任务 (通常是并行的任务，例如 MPI)
    3. 为任务队列合理地分配资源

## Slurm 系统的基本概念

### 程序：蛋糕配方

Slurm 系统是一个任务调度系统，意味着 Slurm 系统安排程序运行的位置（服务器），协调程序占用的资源，但不会参与程序的具体运行过程。任何程序都可以在 Slurm 系统中运行！连接 VPN ，登陆 `10.0.0.251` ，然后试一试使用 Slurm 系统：

```shell
srun whoami
```

### 数据：蛋糕原料

Slurm 系统内包含多个服务器，但是所有服务器的 `/home` 目录都由存储节点提供，是一个共享的分区，不用考虑数据在服务器之间拷贝同步的问题。执行以下命令尝试一下吧！

````shell
echo 'Hi!' > ~/tmp
hostname && cat ~/tmp
srun hostname && cat ~/tmp
````

### Slurm 节点：蛋糕工厂

在 Slurm 系统中，**节点**指可以独立运行程序的服务器，所有服务器都可以执行用户提交的程序。目前 slurm 系统内共有 5 个节点：

* 登录节点 `air-server` ：连接 VPN 后 ssh 登陆 `10.0.0.251` 

  * 跳板节点上配备 2 张 A100 GPU 供调试，该 GPU 使用无需通过 slurm 系统。

* 运算节点：共有 4 个，分别是

  * `air-node-02`（配备 8 张 A100 GPU）
  * `air-node-03`（配备 6 张 A100 GPU）
  * `air-node-04`（配备 6 张 A100 GPU）
  * `air-node-05`（配备 8 张 A100 GPU）

  使用运算结点上的 GPU 资源，需要通过 slurm 系统提交**任务**

### 任务：装载配方的信封

**任务**是对程序以及程序运行占用资源的封装，每一个任务有的单独的编号（`JOBID`）。

任务在 slurm 中的生命周期如下：

1. 任务 被用户**创建**并**提交**至 slurm 系统
2. 任务 被计算优先级，**进入等待队列**相应位置
3. 任务 被移除等待队列并**分配资源**，当 满足调度条件（可用资源充足，使用总量限制未达到，资源和时间上限在限制范围内）时
4. 任务 在被分配的计算节点**开始执行**
5. 任务 **结束**，关键信息计入审计系统后被销毁

### Slurm 系统：配方信件中转站

最后，Slurm干的事情就是把任务（信）分发到 Slurm 节点（工厂），Slurm 节点执行任务指定的程序（工厂拆开信封开始生产）。

下面介绍使用 Slurm 系统的具体使用方法。

## Slurm 系统的基本使用方法

!!! warning 要使用 conda 虚拟环境，需要在执行 slurm 相关命令前执行 `conda activate xxx`

!!! tips 我要跑 3 天 12 小时内跑完的程序，使用 2 张 GPU，任务名字叫 `hard working`

```shell
srun --gres=gpu:a100:2 --time 3-12:00:00 --job-name "hard working" sleep 10000 
# 真正用的时候还是不要跑 sleep 了
```

!!! tips 我要跑 3 天 12 小时内跑完的程序，使用 2 张 GPU，当任务开始执行、结束时发邮件通知我

```shell
srun --gres=gpu:a100:2 --time 3-12:00:00 --mail-user abc@ef.gh --mail-type BEGIN,END
```

!!! tips 我要跑 3 天 12 小时内跑完的程序，使用 2 张 GPU，还要使用 100GB 内存

```shell
srun --gres=gpu:a100:2 --time 3-12:00:00 --mem 100000 python ...
```

!!! tips 我要跑 3 天 12 小时内跑完的程序，使用 2 张 GPU，还要使用 80 个核

```shell
srun --gres=gpu:a100:2 --time 3-12:00:00 --mem 100000 --cpus-per-task 80 python ...
```

!!! tips 我要使用 2 张 GPU，通过交互方式运行 python，3 小时（180分钟）就行

```shell
srun --gres=gpu:a100:2 --time 180 --pty python
```

!!! tips 我还没想好要跑什么程序，但是我得需要 1 张 GPU，3 天内能跑完

```shell
salloc --time=3-0 --gres=gpu:a100:1
# 进入新的 shell，这个时候分配的 3 天时间就开始计时了

# 当你想好后
srun python ...
# 还是没想好，先用交互模式试试手
srun --pty python

# 任务都跑完了，早点结束吧
exit
# Ctrl+d 也行
```

!!! tips 我想用一张 GPU 同时跑两个程序

```shell
srun --time=3-0 --gres=gpu:a100:1 --pty bash
# 进入新的 bash 环境，这个时候分配的 3 天时间就开始计时了

# 执行第一个程序，程序输出到 a.out 文件中
python a.py 1>a.out 2>&1 &
# 执行第二个程序，程序输出到 b.out 文件中
python b.py 1>b.out 2>&1 &
# 当然用 tmux / screen 也可以，不过注意 srun 本身在 tmux 中时就不要再用 tmux 了

# 任务都跑完了，早点结束吧
exit
# Ctrl+d 也行
```

!!! tips 我想一次提交100个任务排着队，有空就慢慢跑；第 i 个任务就跑 `python run.py i`

创建文件 `batch.sh` （文件名可改）

```shell
#!/bin/bash
#SBATCH --job-name example                  # 任务名叫 example
#SBATCH --array 0-99                        # 提交 100 个子任务，序号分别为 0,1,2,...99
#SBATCH --gres gpu:a100:1                   # 每个子任务都用一张 A100 GPU
#SBATCH --time 1-1:00:00                    # 子任务 1 天 1 小时就能跑完
#SBATCH --output %A_%a.out                  # 100个程序输出重定向到 [任务id]_[子任务序号].out
#SBATCH --mail-user example@gmail.com       # 这些程序开始、结束、异常突出的时候都发邮件告诉我
#SBATCH --mail-type ALL                     

# 任务 ID 通过 SLURM_ARRAY_TASK_ID 环境变量访问
# 上述行指定参数将传递给 sbatch 作为命令行参数
# 中间不可以有非 #SBATCH 开头的行

# 执行 sbatch 命令前先通过 conda activate [env_name] 进入环境

python run.py ${SLURM_ARRAY_TASK_ID}
```

## 任务优先级与调度策略

TODO

## 任务限制

**每个用户在集群中最多同时运行 6 个任务**；等待队列中的任务数量不受限制。

**每个任务最长运行时间为 5 天。**

## srun: 提交任务并前台运行

```shell
srun --gres=gpu:a100:GPU_COUNT --time=d-hh:mm:ss [--mem=20000] CMD
```
CMD 为程序正常执行时的命令。使用 `srun` 命令提交任务后，程序输出到 `stdout` 与 `stderr` ，在控制台中可以看到程序的所有输出。

如需后台运行任务请使用 `tmux` / `screen` 等工具或使用下方 `sbatch` 命令提交任务。

参数说明：

* `--gres`: **G**eneric **RES**ources，申请使用的 GPU 类型与数量
    * 目前集群中只有 NVIDIA A100 40GB 一种GPU，用 a100 指代
    * 参数格式：`gpu:a100:GPU_COUNT`
* `--time`: 程序运行的最长运行时间
    * 默认值为1分钟
    * 如果任务需要运行更长时间，请联系集群管理员
    * **超过最长运行时间的任务会被强行终止**
    * 参数格式： `d-hh:mm:ss`
* `--nodelist`: （可选）指定程序运行的节点机，未指定时将由系统自动分配。目前可选的节点机有：
    * `air-node-02`
    * `air-node-03`
    * `air-node-04`
    * `air-node-05`
* `--cpus-per-task`: （可选）任务请求使用的 CPU 核数
    * 默认情况下，每申请一张GPU会配给 10 的 CPU 核心；因此除非程序需要使用大量 CPU 核心，无需指定此参数
    * 参数格式：整数

* `--mem`: （可选）任务请求使用的内存
    * 默认情况下，每申请一张GPU会配给 20GB 的内存；因此除非程序需要使用大量内存，无需指定此参数
    * 参数格式：整数，单位为 MB
* `--mail-user`: （可选）任务状态变更时，接收更新邮件的邮箱
* `--mail-type`: （可选）在任务状态发生哪些变化时，发送更新邮件
    * `BEGIN`: 任务从等待队列中取出，开始运行时
    * `END`: 任务正常结束时
    * `FAILURE`: 任务异常退出时
    * `ALL`: 以上三种情况均发送状态更新邮件


!!! info "参数位置"
    提供给 `srun` 命令的参数应当置于程序命令 CMD 之前，否则会被认为是提供给 CMD 的运行参数。

!!! danger "运行时间"
    为了合理使用GPU资源，请提交任务时指定一个合理的最长运行时间。请注意，过长的运行时间可能导致调度的优先级降低，详情请见下方调度策略说明。

!!! info "容器应用支持"
    目前集群中所有节点已经支持容器应用。
    提交运行容器的任务需要在使用 `srun` 命令使用额外的一些参数，具体使用方法请参考[这里](https://co1lin.github.io/AIR-Server-Doc/gpu/)。
    需要运行容器的任务只能通过 `srun` 提交（不能通过 `sbatch`）。

## squeue: 查看任务队列

```shell
squeue
```

显示正在运行与正在排队的任务。样例输出：

```shell
JOBIDJOBID     USER    GROUP       NAME              STATE     QOS                 TIME        TIME_LIMIT  NODELIST    TRES_PER_NODPENDING_TIM s REASON              PRIORITY
4437 4437      wenh    AIot        bash              PENDING   normal              0:00        5-00:00:00              gpu:a100:2       254244 s AssocGrpGRESRunMinut10800655
4245 4245      chenxx  DISCOVER    bash              PENDING   normal              0:00        5-00:00:00              gpu:a100:8       529623 s BadConstraints      0
4244 4244      liyang  DISCOVER    bash              PENDING   normal              0:00        5-00:00:00              gpu:a100:8       530287 s BadConstraints      0
4504 4504      wangzy  JJ_Group    sd02w12adap30-hcl-RUNNING   normal              17:17:06    3-00:00:00  air-node-03 gpu:a100:2            1 s None                480769
4484 4484      yuqy    JJ_Group    advcl-sd02w12adap3RUNNING   normal              20:17:26    3-00:00:00  air-node-03 gpu:a100:2            0 s None                480769
4515 4515      tb5zhh  DISCOVER    interactive       RUNNING   normal              5:09:52     1-00:00:00  air-node-02 gpu:8                 1 s None                346153
```

输出表格字段含义：

- `JOBID`: 任务序号
- `USER`: 任务提交用户
- `GROUP`: 任务提交用户所属实验室名称
- `NAME`: 任务名称。默认值为可执行文件名 (`bash`, `python` 等)，使用 `sbatch` 命令提交任务可以自定义任务名
- `QOS`: TODO
- `STATE`: 任务状态，常见任务状态如下：
    - `RUNNING`: 任务正在运行
    - `PENDING`: 任务正在队列中等待分配资源
    - `COMPLETING`: 任务正在终止
    - 出现其他状态时请联系管理员
- `TIME`: 任务已运行时长
- `TIME_LIMIT`: 任务最长运行时间。**超过最长运行时间的任务会被强行终止**
- `NODE_LIST`: 任务使用的运算节点
- `TRES_PER_NODE`: 任务申请使用的资源数量
- `PENDING_TIME`: 任务在队列中等待时长，单位为秒
- `REASON`: 任务排队原因
- `PRIORITY`: TODO

## scontrol: 查看正在运行任务的状态

```shell
scontrol show job JOBID
```

该命令返回任务的详细状态。在该命令的输出中，能够找到关于任务的所有信息：提交时间、开始时间、运行时长、申请资源、排队时长、排队原因等。

## sinfo: 查看集群运算节点的状态

```shell
sinfo
```

该命令返回集群中运算节点的名称、节点资源总量、节点资源已使用量等信息。样例输出如下：

```shell
NODELIST            STATE       AVAIL GRES                          CPUS(A/I/O/T)       GRES_USED                          MEMORY              ALLOCMEM            REASON              CPUS(A/I/O/T)
air-node-02         mixed       up    gpu:a100:8                    80/80/0/160         gpu:a100:8(IDX:0-7)                320000              160000              none                80/80/0/160
air-node-03         mixed       up    gpu:a100:6                    40/80/0/120         gpu:a100:4(IDX:0-1,3-4)            240000              80000               none                40/80/0/120
air-node-04         idle        up    gpu:a100:6                    0/120/0/120         gpu:a100:0(IDX:N/A)                240000              0                   none                0/120/0/120
air-node-05         idle        up    gpu:a100:8                    0/160/0/160         gpu:a100:0(IDX:N/A)                320000              0                   none                0/160/0/160
```

输出表格字段含义：

- `NODELIST`: 节点名称
- `STATE`: 节点状态
    - `idle`: 节点空闲
    - `mixed`: 节点正在运行任务，但是仍有可用资源
        - 请注意，此处所指资源包括CPU、内存、GPU等，此状态不保证有空闲的 GPU 卡
    - `alloc`: 节点所有资源均被占用
    - `draining`, `drained`: 无法在该节点上提交新任务，但是会等待节点上现有任务运行结束。
        - 出现此状态原因可由 `REASON` 字段查看，通常为节点维护或调试
        - 现有任务结束前为 `draining` ，全部现有任务结束后为 `drained`
    - `completing`: 节点上所有任务均处于`COMPLETING`状态
        - 此状态是暂时的，如果长时间保持此状态请联系管理员
    - `fail`: 节点不可用
        - 出现此状态请联系管理员
- `AVAIL`: 节点状态（简略版）
    - `up`: 可用
    - `down`, `drain`: 不可用
- `GRES`: 节点可用 GPU 资源
- `GRES_USED`: 节点已分配 GPU 资源数量及序号
- `MEMORY`: 节点可用内存，单位为MB
- `ALLOC_MEM`: 节点已分配内存，单位为MB
- `REASON`: 节点异常状态原因

## scancel: 取消任务

```shell
scancel JOBID
```

取消任务。只能取消自己提交的任务。

**`scancel` 操作对象为任务组 ID 时，会取消任务组中所有任务。详情请见下方 `sbatch` 说明。**

该操作不可逆，执行前请再三确认。

## sbatch: 批量提交后台任务

```shell
sbatch run.sh
```

根据 `run.sh` 向 Slurm 中批量提交任务，执行后立即返回。如果任务在 `conda` 环境中执行，在运行 `sbatch` 前应当先进入该 `conda` 环境。

`run.sh` 第一行**必须为** `#!/bin/bash`.

`sbatch` 的命令行参数可以通过两种方式指定：

1. 置于 `sbatch` 命令后，如 `sbatch --gres=gpu:a100:1 run.sh`
2. 置于 `run.sh` 开头，以 shell 注释的方式提供，如 `#SBATCH --gres=gpu:a100:1` ；所有 `#SBATCH` 行须相邻。


参数说明：

* `--job-name`: 提交的任务名称。会出现在 `squeue` 命令的 `NAME` 字段内，默认为可执行文件名。
* `--gres`: 任务申请的 GPU 资源，格式同 `srun` 中 `--gres`
* `--time`: 任务最长运行时间，格式同 `srun` 中 `--time`
* `--nodelist`: （可选）指定任务运行的节点机，格式同 `srun` 中 `--nodelist`
* `--array`: （可选）提交批量任务（任务组）。
    * 使用此选项时，会一次性将多个任务提交至 Slurm 系统。
    * 每一个任务组与普通任务的地位是等同的，任务组本身有一个 ID (`ARRAYID`)，这个 ID 可以通过 `squeue` 命令查看。**对这个 ID 使用 `scancel` 会取消任务组中所有任务**
    * 任务组中的每个任务会有一个组内序号（`ARRAYINDEX`），在各任务内通过环境变量 `SLURM_ARRAY_TASK_ID` 访问。需要单独去取消组内任务时，请使用 `scancel ARRAYID_ARRAYINDEX` 
    * 此选项提供的值，即为任务组中任务的序号。例如 `0-15` 会提交 16 个任务；这16个任务内环境变量 `SLURM_ARRAY_TASK_ID` 的值分别为 0, 1, 2, ... 15。
    * 允许的参数格式：`0-15` (0, 1, 2, ... 15), `0,1,2,3` , `0,1,6-8`, `0-15:4` (0, 4, 8, 12), `0-15%4` (提交 16 个任务，但是只能有 4 个任务同时运行)
    * 最小任务序号为 0
* `--output`: （可选）任务的输出文件
    * 对于单任务，`%j` 表示任务 ID 占位符。如果任务 ID 为 10086， `--output %j.out` 表示任务输出到 `10086.out` 
    * 对于多任务，`%A` 表示任务组ID，`%a` 表示组内序号。如果任务组 ID 为 10086，某任务组内序号为 4 ，`--output %A_%a.out` 表示任务输出到 `10086_4.out`
    * 对于单任务，默认输出为 `slurm-%j.out`
    * 对于多任务，默认输出为 `slurm-%A_%a.out`
    * 如果指定通知邮箱，任务结束时，结束通知邮件会将输出文件最后 100 行随邮件发送至邮箱
* `--mail-user`: （可选）任务状态变更时，接收更新邮件的邮箱。同 `srun` 中 `--mail-user` 
* `--mail-type`: （可选）在任务状态发生哪些变化时，发送更新邮件。同 `srun` 中 `--mail-user` 
    * `BEGIN`: 任务从等待队列中取出，开始运行时
    * `END`: 任务正常结束时
    * `FAILURE`: 任务异常退出时
    * `ALL`: 以上三种情况均发送状态更新邮件

样例 `run.sh` 脚本如下：

```shell
#!/bin/bash
#SBATCH --job-name example                  # 任务在 squeue 中显示任务名为 example
#SBATCH --output %A_%a.out                  # 任务输出重定向至 [任务组id]_[组内序号].out
#SBATCH --gres gpu:a100:1                   # 任务申请使用一张 A100 GPU
#SBATCH --time 1-1:00:00                    # 任务最长运行 1 天 1 小时，超时任务将被杀死
#SBATCH --array 0-15                        # 提交 16 个任务，组内序号分别为 0,1,2,...15
#SBATCH --mail-user example@gmail.com       # 将任务状态更新以邮件形式发送至 example@gmail.com
#SBATCH --mail-type ALL                     # 任务开始运行、正常结束、异常退出时均发送邮件通知

# 任务 ID 通过 SLURM_ARRAY_TASK_ID 环境变量访问
# 上述行指定参数将传递给 sbatch 作为命令行参数
# 中间不可以有非 #SBATCH 开头的行

# 执行 sbatch 命令前先通过 conda activate [env_name] 进入环境
# 执行程序
echo ${SLURM_ARRAY_TASK_ID}
python -V
python -c "print ('Hello, world!')"
```
