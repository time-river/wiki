<!-- TITLE: Relationship -->
<!-- SUBTITLE: A quick summary of Relationship -->

# Overview
Linux使用`struct task_struct`数据结构来关联所有与进程有关的数据和结构，定义了*pid*、*tgid*、*pgid*、*sid*四种进程类型。

Note：
* *pid* —— Process identifier，进程标识符。
* *tid* —— Thread identifier，线程标识符。
* *tgid* —— Thread group identifier，线程组标识符。
* *pgid* —— Process group identifier，进程组标识符。
* *sid* —— Session identifier，会话标识符。

在v4.12中，与四种进程类型有关的系统调用有以下8个：

| | |
| --: | --: |
| getpid | gettid |
| getppid | getpgrp |
| getpgid | setpgid |
| getsid | setsid |

# Introduction
- *pid* —— 在特定PID namespaces中，Linux使用*pid*唯一标识一个进程。使用`fork / clone`创建一个进程均会被分配一个新的且唯一的*pid*。
- *tid* —— Linux的线程由Native POSIX Thread Library（NPTL）实现，本质上是用进程模拟线程，因此内核中并没有*tid*这种数据，它由*pid*表示。线程是通过带有`CLONE_THREAD`*flag*参数`clone`建立的进程。
- *tgid* —— 同一个进程通过`clone`建立了的线程处于同一个线程组，该线程组的ID叫做*tgid*，线程组组长的*tgid*与其*pid*相同。若一个进程没有线程，则其*tgid*与*pid*也相同。
- *pgid* —— 进程组是一个或多个进程的集合，每个进程都属一个进程组，拥有相同的*pgid*。进程组组长（process group leader）的*pid*是进程组的ID，用以识别进程，可通过`setpgrp`为进程设置*pgid*。
- *sid* —— 几个进程组使用`setsid`可以合并成为一个会话组，会话组内的所有进程拥有相同的*sid*。
# Pre-internal
## PID namespace
PID namespaces隔离了进程ID，指的是不同的进程在不同的PID namespaces可以有着相同的PID。这个特性是在主机之间迁移容器的先决条件；只有一个PID namespaces的时候，在保持PID不变（保持PID不变是一种需求）的情况下将其迁移到另一个主机可能会失败，因为目标节点上可能存在相同PID的进程。

PID namespaces具有层级顺序，一个PID namespaces可以具有多个child PID namespaces，每一个PID namespaces都可以看做parent PID namespaces的一个局部视图。一个新的PID namespaces *B*从当前PID namespaces *A*被创建后，处于当前PID namespaces *A*的进程可以看到*B*中的所有进程，但反之不成立。

![PID Namespaces](/uploads/2018/pid-namespaces.png "PID Namespaces")

新的PID namespaces通过带有`CLONE_NEWPID`*flag*参数的`clone`系统调用创建。新namespace种第一个进程的PID是1，它是这个namespace的init进程，属于此namespace的孤儿进程将会被它收养；若这个进程死亡，则整个namespace都会被中止。

Note: reference to [1][1][2][2][3][3]

## Overview of `struct task_struct`

TODO: `real_parent` vs `parent` 
# Internal
## `getpid` & `gettid`
```
/*
  * sys_getpid - return the thread group id of the current process
  * 
  * Note, despite the name, this returns the tgid not the pid.  The tgid and
  * the pid are identical unless CLONE_THREAD was specified on clone() in
  * which case the tgid is the same in all threads of the same group.
  */
SYSCALL_DEFINE0(getpid)
 - task_tgid_vnr(current)
		- ns = task_active_pid_ns(current)
    - task = current->group_leader
    - pid = task->pids[PIDTYPE_PID].pid
    - return pid->numbers[ns->level].nr
```

```
/* Thread ID - the internal kernel "pid" */
SYSCALL_DEFINE0(gettid)
 - task_pid_vnr(current)
    - ns = task_active_pid_ns(current)
    - pid = current->pids[PIDTYPE_PID].pid
    - return pid->numbers[ns->level].nr
```

# References
[1]: https://lwn.net/Articles/259217/ "LWN.net: PID namespaces in the 2.6.24 kernel"
[2]: http://man7.org/linux/man-pages/man7/pid_namespaces.7.html "Linux Programmer's Manual: pid_namespaces"
[3]: https://cse.yeditepe.edu.tr/~kserdaroglu/spring2014/cse331/termproject/BOOKS/ProfessionalLinuxKernelArchitecture-WolfgangMauerer.pdf "Professional Linux Kernel Architecture - Linux"