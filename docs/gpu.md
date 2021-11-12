# 通过 slurm 系统使用 GPU 资源

当前集群中，A100 显卡通过 slurm 系统分配使用；3090 显卡可以直接使用。

!!! tip "Slurm 系统"
    **Slurm 任务调度工具 **，是一个用于 Linux 和 Unix 内核系统的免费、开源的任务调度工具，被世界范围内的超级计算机和计算集群广泛采用。它提供了三个关键功能：

    1. 为用户分配一定时间的专享或非专享的资源 (计算机节点)，以供用户执行工作
    2. 它提供了一个框架，用于启动、执行、监测在节点上运行着的任务 (通常是并行的任务，例如 MPI)
    3. 为任务队列合理地分配资源

## 节点与任务

在 slurm 系统中，**节点**指可以独立运行程序的服务器。目前 slurm 系统内共有4个节点：

* 登录节点 `air-server` ：集群用户可以通过 `ssh` 连接 `103.242.175.247` 登录。跳板节点上配备 2 张 A100 GPU 供调试，该 GPU 使用无需通过 slurm 系统。
* 运算节点：共有 3 个，分别是 `air-node-01`（配备 6 张 A100 GPU），`air-node-02`（配备 2 张 A100 GPU），`air-node-03`（配备 6 张 A100 GPU）。集群用户需要通过 slurm 系统提交任务使用运算结点上的 GPU 资源，使用方式见<a href="ref">下方</a>。

**任务**是对程序以及程序运行占用资源的抽象，每一个任务有的单独的编号（`JOBID`）。用户将程序提交至 slurm 系统后，任务队列会加入一个新任务；当集群中有可用的资源时由 slurm 系统自动分配资源与任务绑定，并从任务队列中取出并开始运行。

## slurm 常用操作 <span id="ref" />

### srun: 提交任务并前台运行

```shell
srun --gres=gpu:a100:GPU_COUNT --time=d-hh:mm:ss [--mem=20000] CMD
```
CMD 为程序正常执行时的命令。使用 `srun` 命令提交任务后，程序输出到 `stdout` 与 `stderr` ，在控制台中可以看到程序的所有输出。

如需后台运行任务请使用 `tmux` / `screen` 等工具或使用下方 `sbatch`命令提交任务。

参数说明：

* `--gres`: **G**eneric **RES**ources，申请使用的 GPU 类型与数量
  * 目前集群中只有 NVIDIA A100 40GB 一种GPU，用 a100 指代
  * 参数格式：`gpu:a100:GPU_COUNT`
* `--time`: 程序运行的最长运行时间
  * 默认值为1分钟，最大值为7天
  * 如果任务需要运行更长时间，请联系集群管理员
  * **超过最长运行时间的任务会被强行终止**
  * 参数格式： `d-hh:mm:ss`
* `--mem`: （可选）任务请求使用的内存
  
  * 默认情况下，每申请一张GPU会配给 20GB 的内存；因此除非程序需要使用大量内存，无需指定此参数
  * 参数格式：整数，单位为 MB
  
  !!! info "参数位置"
    提供给 `srun` 命令的参数应当置于程序命令 CMD 之前，否则会被认为是提供给 CMD 的运行参数。
  
  !!! danger "运行时间"
    为了合理使用GPU资源，请提交任务时指定一个合理的最长运行时间。请注意，过长的运行时间可能导致调度的优先级降低，详情请见下方调度策略说明。

### squeue: 查看任务队列

```shell
squeue
```

显示正在运行与正在排队的任务。样例输出：

```shell
# JOBID     USER    NAME      STATE     TIME      TIME_LIMIT NODELIST    TRES_PER_NODE       
# 149       tb5zhh  bash      RUNNING   5:46:33   UNLIMITED  air-node-03 gpu:a100:1          
# 150       tb5zhh  bash      RUNNING   5:46:30   UNLIMITED  air-node-03 gpu:a100:1          
# 154       lpf     python    RUNNING   2:22:36   UNLIMITED  air-node-03 gpu:a100:4 
```

输出表格字段含义：

- `JOBID`: 任务序号

- `USER`: 任务提交用户
- `NAME`: 任务名称。默认值为可执行文件名 (`bash`, `python` 等)，使用 `sbatch` 命令提交任务可以自定义任务名
- `STATE`: 任务状态，常见任务状态如下：
  - `RUNNING`: 任务正在运行
  - `PENDING`: 任务正在队列中等待分配资源
  - `COMPLETING`: 任务正在终止
  - 出现其他状态时请联系管理员
- `TIME`: 任务已运行时长
- `TIME_LIMIT`: 任务最长运行时间。**超过最长运行时间的任务会被强行终止**
- `NODE_LIST`: 任务使用的运算节点
- `TRES_PER_NODE`: 任务申请使用的资源数量
- `MIN_MEMORY`: 任务申请使用的最小内存
- `MEM_PER_TRES`: 为每一张 GPU 卡配给的内存
- `PENDING_TIME`: 任务在队列中等待时长，单位为秒
- `REASON`: 任务排队原因

```shell
squeue --start
```

显示正在排队的任务，以及任务开始运行的最晚时间。

请注意，根据调度算法，`START_TIME` 字段在任务进入队列时即可确定并**不会改变**，任何时候队列中的任务只会**早于 `START_TIME` 开始运行**。

```shell
JOBID	PARTITION     NAME     USER ST          START_TIME  NODES SCHEDNODES           NODELIST(REASON)
1207     debug   python   wangym PD 2021-11-14T09:29:49      1 (null)               (Priority)
1208     debug   python   wangym PD 2021-11-14T09:29:49      1 (null)               (Priority)
1209     debug   python   wangym PD 2021-11-14T09:29:49      1 (null)               (Priority)
1210     debug   python   wangym PD 2021-11-14T09:29:49      1 (null)               (Priority)
1211     debug clipmean   wangzy PD 2021-11-14T09:29:49      1 (null)               (Priority)
1162     debug  python3   liyang PD 2021-11-14T23:12:54      1 (null)               (Resources)
```

### 调度策略

整体而言，Slurm 系统按照进入队列的顺序来调度任务，但有一种情况允许后进入队列的任务先开始运行（插队）。当且仅当新进入队列的任务满足：

1. 申请使用的资源能够被当前集群所满足（集群上有空闲的 GPU）
2. 申请运行的最长时间不会导致队列现有任务开始时间推迟（不影响正在排队的任务）

时，该新任务将允许优先被调度。

  !!! info "参数位置"
    我们可以通过俄罗斯方块游戏来理解这个调度策略：集群可以形象化理解为一个固定宽度的桶，水平方向代表集群资源整体数量，竖直方向代表时间；每一个任务都是一个长方形（任务块），长方形的横边长为任务申请资源数量，竖边长为任务指定的最长运行时间。
    随着时间的推移，桶中的任务会逐渐下降，任务块接触桶底并逐渐在竖直方向上缩短，表示任务正在运行，并正在接近提交时指定的 deadline。
    任务进入调度队列的过程，就是每一个任务块进入集群桶中的过程。Slurm 系统会为每一个任务安排一个最靠近桶底的位置：当任务较少，可用资源较多，任务直接被安排在桶底，任务就可以马上开始执行；当可用资源无法满足任务需求时，任务会被安排在尽可能接近桶底的位置。值得说明的是，一旦任务进入桶中，任务在时间轴上的绝对位置**会被固定**，不会因为队列中的新任务而改变；这意味着一旦进入队列，任务就有一个最晚开始执行的时间节点。
    上述提到的插队规则，形象的理解就是：对于新进入桶中的任务块，如果在尽可能靠近桶底的位置上仍有能够放下该任务块的空位，则可以将任务置于该空位，这样一方面不会影响已经在桶中的任务，任务本身运行时间也能提前。
    这里的调度规则，也是希望大家合理指定任务最长运行时间的原因。最长运行时间越短，任务块的竖直长度越短，就越有可能被安排在距离桶底进的位置，从而越早开始执行。

### scontrol: 查看正在运行任务的状态

```shell
scontrol show job JOBID
```

该命令返回任务的详细状态。在该命令的输出中，能够找到关于任务的所有信息：提交时间、开始时间、运行时长、申请资源、排队时长、排队原因等。

### sinfo: 查看集群运算节点的状态

```shell
sinfo
```

该命令返回集群中运算节点的名称、节点资源总量、节点资源已使用量等信息。样例输出如下：

```shell
NODELIST            STATE       AVAIL GRES                          GRES_USED                          MEMORY              ALLOCMEM            REASON              
air-node-01         mixed       up    gpu:a100:8                    gpu:a100:7(IDX:1-7)                402800              140000              none                
air-node-03         mixed       up    gpu:a100:6                    gpu:a100:6(IDX:0-5)                402800              120000              none                
air-node-04         mixed       up    gpu:a100:8                    gpu:a100:8(IDX:0-7)                402800              285760              none
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

### scancel: 取消任务

```shell
scancel JOBID
```

取消任务。只能取消自己提交的任务。

**`scancel` 操作对象为任务组 ID 时，会取消任务组中所有任务。详情请见下方 `sbatch` 说明。**

该操作不可逆，执行前请再三确认。

### sbatch: 批量提交后台任务

```shell
sbatch run.sh
```

根据 `run.sh` 向 Slurm 中批量提交任务，执行后立即返回。如果任务在 `conda` 环境中执行，在运行 `sbatch` 前应当先进入该 `conda` 环境。

样例脚本第一行**必须为** `#!/bin/bash`.

`sbatch` 的命令行参数可以通过两种方式指定：

1. 置于 `sbatch` 命令后，如 `sbatch --gres=gpu:a100:1 run.sh`
2. 置于 `run.sh` 开头，以 shell 注释的方式提供，如 `#SBATCH --gres=gpu:a100:1` ；所有 `#SBATCH` 行须相邻。


参数说明：

* `--job-name`: 提交的任务名称。会出现在 `squeue` 命令的 `NAME` 字段内，默认为可执行文件名。
* `--array`: 提交批量任务（任务组）。
  * 使用此选项时，会一次性将多个任务提交至 Slurm 系统。
  * 每一个任务组与普通任务的地位是等同的，任务组本身有一个 ID (`ARRAYID`)。**对这个 ID 使用 `scancel` 会取消任务组中所有任务**
  * 任务组中的每个任务会有一个组内序号（`ARRAYINDEX`），通过环境变量 `SLURM_ARRAY_TASK_ID` 访问。需要单独去取消组内任务时，请使用 `scancel ARRAYID_ARRAYINDEX` 
  * 此选项提供的值，即为任务组中任务的序号。例如 `0-15` 会提交 16 个任务；这16个任务内环境变量 `SLURM_ARRAY_TASK_ID` 的值分别为 0, 1, 2, ... 15。
  * 允许的参数格式：`0-15` (0, 1, 2, ... 15), `0,1,2,3` , `0,1,6-8`, `0-15:4` (0, 4, 8, 12), `0-15%4` (提交 16 个任务，但是只能有 4 个任务同时运行)
  * 最小任务序号为 0

* `--output`: 任务的输出文件
  * 对于单任务，`%j` 表示任务 ID 占位符。如果任务 ID 为 10086， `--output %j.out` 表示任务输出到 `10086.out` 
  * 对于多任务，`%A` 表示任务组ID，`%a` 表示组内序号。如果任务组 ID 为 10086，某任务组内序号为 4 ，`--output %A_%a.out` 表示任务输出到 `10086_4.out`
  * 对于单任务，默认输出为 `slurm-%j.out`
  * 对于多任务，默认输出为 `slurm-%A_%a.out`

* `--gres`: 任务申请的 GPU 资源，格式同 `srun` 中 `--gres`
* `--time`: 任务最长运行时间，格式同 `srun` 中 `--time`

样例 `run.sh` 脚本如下：

```shell
#!/bin/bash
#SBATCH --job-name example
#SBATCH --output %A_%a.out   
#SBATCH --gres gpu:a100:1 
#SBATCH --time 1:00:00     
#SBATCH --array 0-15

# 任务 ID 通过 SLURM_ARRAY_TASK_ID 环境变量访问
# 上述行指定参数将传递给 sbatch 作为命令行参数
# 中间不可以有非 #SBATCH 开头的行

# 执行 sbatch 命令前先通过 conda activate [env_name] 进入环境
# 执行程序
echo ${SLURM_ARRAY_TASK_ID}
python -V
python -c "print ('Hello, world!')"
```

