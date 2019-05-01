---
layout: post
title: 认识容器技术背后的namespace和cgroup
date: 2017-09-12
tag: MicroService
---

Docker是现在最popular的容器技术的实践。支撑它的是两大关于linux内核的技术-namespace和cgroup。我们来一点一点地把逻辑理顺。

## 实现Docker/容器的前提

Docker是在host上开辟出了一块新的“空间”。这个空间，有自己的计算资源，自己的文件，自己的网络接口等，与host相对独立的。这个空间基本上不知道host的存在，但是host要知道这个空间的存在，以调配资源给这个空间。要实现这样的目的，我们要有两个前提：

1.要有能把各种资源从host独立出来的技术
2.要有使得host能够调控容器的技术

第一点对应的是namespace资源隔离技术，第二点对应的是cgroups资源限制技术。这两大技术就是docker的两大基石。

## Namespace资源隔离技术

如果要自己实现一个资源隔离的容器，第一点想到的是对文件系统进行隔离，使得容器能够使用自己的文件而无须依赖host。为了在分布式的环境下进行通信和定位，容器必然需要一个独立的 IP、端口、路由等等，自然就想到了网络的隔离，和主机名的独立。然后进程间的通信需要隔离，用户和用户组也需要隔离。最后运行在容器中PID自然也需要跟host中的PID进行隔离。

我们基本上完成了一个容器所需要做的六项隔离，Linux 内核中就提供了这六种 namespace 隔离的系统调用，如下表所示。

| Namespace | 系统调用参数   | 隔离内容                 |
| --- | --- | --- |
| UTS       | CLONE_NEWUTS | 主机名与域名              |
| IPC       | CLONE_NEWIPC | 信号量、消息队列和共享内存  |
| PID       | CLONE_NEWPID | 进程编号                 |
| Network   | CLONE_NEWNET | 网络设备、网络栈、端口等等  |
| Mount     | CLONE_NEWNS  | 挂载点（文件系统）         |
| User      | CLONE_NEWUSER | 用户和用户组             |

在同一个 namespace 下的进程可以感知彼此的变化，而对外界的进程一无所知。这样就可以让容器中的进程产生错觉，仿佛自己置身于一个独立的系统环境中，以此达到独立和隔离的目的。

### 与namespace有关的操作

1. 通过 clone() 创建新进程的同时创建 namespace

    使用 clone() 来创建一个独立 namespace 的进程是最常见做法，它的调用方式如下。

    ```
    int clone(int (*child_func)(void *), void *child_stack, int flags, void *arg);
    ```

2. 查看 /proc/[pid]/ns 文件

    从 3.8 版本的内核开始，用户就可以在 /proc/[pid]/ns 文件下看到指向不同 namespace 号的文件。相同的namespace号代表着同一个namespace。

3. 自建一个container

    参考 [creating containers](http://crosbymichael.com/creating-containers-part-1.html), 自己一步一步地进行六大资源的隔离。


## cgroups资源限制技术

cgroups技术不仅可以限制被 namespace 隔离起来的资源，还可以为资源设置权重、计算使用量、操控进程启停等等。cgroups的official定义是：

>cgroups 是 Linux 内核提供的一种机制，这种机制可以根据特定的行为，把一系列系统任务及其子任务整合（或分隔）到按资源划分等级的不同组内，从而为系统资源管理提供一个统一的框架。

本质上来说，cgroups 是内核附加在程序上的一系列钩子（hooks），通过程序运行时对资源的调度触发相应的钩子以达到资源追踪和限制的目的。

### cgroups的四大功能

* 资源限制（Resource Limitation）：cgroups 可以对进程组使用的资源总额进行限制。如设定应用运行时使用内存的上限，一旦超过这个配额就发出 OOM（Out of Memory）。
* 优先级分配（Prioritization）：通过分配的 CPU 时间片数量及硬盘 IO 带宽大小，实际上就相当于控制了进程运行的优先级。
* 资源统计（Accounting）： cgroups 可以统计系统的资源使用量，如 CPU 使用时长、内存用量等等，这个功能非常适用于计费。
* 进程控制（Control）：cgroups 可以对进程组执行挂起、恢复等操作。

### cgroups中的概念理解

1. **task（任务）**：cgroups 的术语中，task 就表示系统的一个进程。
2. **cgroup（控制组）**：cgroups 中的资源控制都以 cgroup 为单位实现。cgroup 表示按某种资源控制标准划分而成的任务组，包含一个或多个**子系统**。一个任务可以加入某个 cgroup，也可以从某个 cgroup 迁移到另外一个 cgroup。
3. **subsystem（子系统）**：cgroups 中的 subsystem 就是一个资源调度控制器（Resource Controller）。比如 CPU 子系统可以控制 CPU 时间分配，内存子系统可以限制 cgroup 内存使用量。
    典型的子系统介绍如下：
    * cpu 子系统，主要限制进程的 cpu 使用率。
    * cpuacct 子系统，可以统计 cgroups 中的进程的 cpu 使用报告。
    * cpuset 子系统，可以为 cgroups 中的进程分配单独的 cpu 节点或者内存节点。
    * memory 子系统，可以限制进程的 memory 使用量。
    * blkio 子系统，可以限制进程的块设备 io。
    * devices 子系统，可以控制进程能够访问某些设备。
    * net_cls 子系统，可以标记 cgroups 中进程的网络数据包，然后可以使用 tc 模块（traffic control）对数据包进行控制。
    * freezer 子系统，可以挂起或者恢复 cgroups 中的进程。
    * ...


4. **hierarchy（层级树）**: 一个hierarchy可以理解为一棵**cgroup树**，树的每个节点就是一个进程组，每棵树都会与零到多个subsystem关联。在一颗树里面，每个进程只能属于一个节点（进程组）。系统中可以有很多颗cgroup树，每棵树都和不同的subsystem关联，一个进程可以属于多颗树，即一个进程可以属于多个进程组，只是这些进程组和不同的subsystem关联。目前Linux支持12种subsystem，如果不考虑不与任何subsystem关联的情况（systemd就属于这种情况)。

### 通过实践来理解概念

#### 查看某个进程属于哪些cgroup

首先我们要知道 /proc 文件夹中保存着当前所有的进程的pid。我们可以进入 /proc/1,它是init的进程。

```
cat /proc/1/cgroup     #显示进程1所属的cgroups
```

```
11:cpuset:/
10:perf_event:/
9:devices:/init.scope
8:blkio:/
7:pids:/init.scope
6:hugetlb:/
5:net_cls,net_prio:/
4:cpu,cpuacct:/
3:freezer:/
2:memory:/
1:name=systemd:/init.scope
```
每一行包含用冒号隔开的三列，他们的意思分别是

1. cgroup树的ID， 和/proc/cgroups文件中的ID一一对应。

2. 和cgroup树绑定的所有subsystem，多个subsystem之间用逗号隔开。这里name=systemd表示没有和任何subsystem绑定，只是给他起了个名字叫systemd。

3. 进程在cgroup树中的路径，即进程所属的cgroup，这个路径是相对于挂载点的相对路径。

上边的解释，可能一开始还不知所云，那么我们再来看另外一个存着cgroups的文件夹，看完就明白了。

#### 查看有哪些cgroups

进入 /sys/fs/cgroup

```
cd /sys/fs/cgroup && ls
```
```
systemd  
perf_event  
net_cls,net_prio  
memory   
freezer  
cpuset   
cpu,cpuacct  
blkio
pids     
net_prio    
net_cls           
hugetlb  
devices  
cpuacct  
cpu
```
这里的每一个都是一个cgroup tree，是针对一个或者多个subsystem建立的hierachy。之前我们看到对应进程1，它的memory的cgroup是/。这意味着它属于memory这个cgropu tree的root节点这个cgroup。而它的pids的cgroup是init.scope.我们来看看pids文件夹下有什么。

```
ls /sys/fs/cgroup/pids/
```

```
cgroup.clone_children  init.scope      system.slice
cgroup.procs           notify_on_release  tasks
cgroup.sane_behavior   release_agent      user.slice
```

我们看到这里确实有个 init.scope.它其实是这个pids cgroup tree的次节点。

/sys/fs/cgroup/pids/ 还有有趣的东西。如果你打开cgroup.proc，就能看到有哪些进程是属于这个cgroup的。而tasks指的是属于这个cgroup的线程。
看到这里，我们就能把进程和cgroup联系起来了，也能理解cgropus中的四大重要概念了。那么，我们如何来建立一个cgroup呢？

#### 新建一个cgroup

其实很简单。

* 方案1: 建立一个全新cgroup

```
#准备需要的目录
dev@ubuntu:~$ mkdir cgroup && cd cgroup
dev@ubuntu:~/cgroup$ mkdir demo

#由于name=demo的cgroup树不存在，所以系统会创建一颗新的cgroup树，然后挂载到demo目录
dev@ubuntu:~/cgroup$ sudo mount -t cgroup -o none,name=demo demo ./demo

#挂载点所在目录就是这颗cgroup树的root cgroup，在root cgroup下面，系统生成了一些默认文件
dev@ubuntu:~/cgroup$ ls ./demo/
cgroup.clone_children  cgroup.procs  cgroup.sane_behavior  notify_on_release  release_agent  tasks

#cgroup.procs里包含系统中的所有进程
dev@ubuntu:~/cgroup$ wc -l ./demo/cgroup.procs
131 ./demo/cgroup.procs
```
 * 方案2: 在一个已经存在的cgroup下建立子节点/子cgroup。

 ```
#从这里的输出可以看到，pids已经被挂载在了/sys/fs/cgroup/pids，这是systemd做的
dev@dev:~$ mount |grep pids
cgroup on /sys/fs/cgroup/pids type cgroup (rw,nosuid,nodev,noexec,relatime,pids)

#进入目录/sys/fs/cgroup/pids/并新建一个目录，即创建了一个子cgroup
dev@dev:~$ cd /sys/fs/cgroup/pids/
dev@dev:/sys/fs/cgroup/pids$ sudo mkdir test

#再来看看test目录下的文件,因为是在一个cgroup下创建的文件夹，系统自动将其认为是一个子cgroup，
#并给它按照父节点的设置添加了默认设置

dev@dev:/sys/fs/cgroup/pids$ cd test
dev@dev:/sys/fs/cgroup/pids/test$ ls
cgroup.clone_children  cgroup.procs  notify_on_release  pids.current  pids.max  tasks

```
#### 改变cgroup和进程的设置

```
#要改变一个cgroup的设置，只要直接修改文件夹下的文件内容就好了。非常简单粗暴。
dev@dev:/sys/fs/cgroup/pids/test$ echo 1 > pids.max

#要把一个进程加入一个cgroup也很容易。
dev@ubuntu:~/cgroup/demo/test$ sudo sh -c 'echo 1421 > cgroup.procs'


```

上边讲了这么多，对应cgroup应该了解了大体轮廓了。还有一个东西要讲，就是hierarchy和subsystem的组合规则。

### hierarchy和subsystem的组合规则

* 规则 1： 同一个 hierarchy 可以附加一个或多个 subsystem。

* 一个 subsystem 可以附加到多个 hierarchy，当且仅当这些 hierarchy 只有这唯一一个 subsystem。

* 一个 task 不能属于同一个 hierarchy 的不同 cgroup, 否则就乱了。

* 进程（task）在 fork 自身时创建的子任务（child task）默认与原 task 在同一个 cgroup 中，但是 child task 允许被移动到不同的 cgroup 中。即 fork 完成后，父子进程间是完全独立的。


-----------------------
参考资料：
* https://www.infoq.cn/article/docker-kernel-knowledge-namespace-resource-isolation
* https://www.infoq.cn/article/docker-kernel-knowledge-cgroups-resource-isolation
* https://segmentfault.com/a/1190000006917884
