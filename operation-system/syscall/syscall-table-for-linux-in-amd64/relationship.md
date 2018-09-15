<!-- TITLE: Relationship -->
<!-- SUBTITLE: A quick summary of Relationship -->

# Overview

# Introduction

# Pre-internal
## PID namespace
PID namespaces隔离了进程ID，指的是不同的进程在不同的PID namespaces可以有着相同的PID。这个特性是在主机之间迁移容器的先决条件；只有一个PID namespaces的时候，在保持PID不变（保持PID不变是一种需求）的情况下将其迁移到另一个主机可能会失败，因为目标节点上可能存在相同PID的进程。

PID namespaces具有层级顺序，一个PID namespaces可以具有多个child PID namespaces，每一个PID namespaces都可以看做parent PID namespaces的一个视图。一个新的PID namespaces *B*从当前PID namespaces *A*被创建后，处于当前PID namespaces *A*的进程可以看到*B*中的所有进程，但反之不成立。

![PID Namespaces](/uploads/2018/pid-namespaces.png "PID Namespaces")

新的PID namespaces通过带有`CLONE_NEWPID`*flag*参数的`clone`系统调用创建。新namespace种第一个进程的PID是1，它是这个namespace的init进程，属于此namespace的孤儿进程将会被它收养；若这个进程死亡，则整个namespace都会被中止。

## Overview of `struct task_struct`

# References
[1]: https://lwn.net/Articles/259217/ "LWN.net: PID namespaces in the 2.6.24 kernel"
[2]: http://man7.org/linux/man-pages/man7/pid_namespaces.7.html "Linux Programmer's Manual: pid_namespaces"