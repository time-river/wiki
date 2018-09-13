<!-- TITLE: Credentials -->
<!-- SUBTITLE: A quick summary of Credentials -->

# Overview
Credentials在Linux中用于访问控制（Access Control），基于*uid*、*gid*、*sid*，是Linux几种安全措施的一部分。同时，仅用于进程（task）中的Capabilities提供了更细化的权限控制机制。[1]

[Note][2]：
* *uid* —— User Identifier，用户标识符，用于辨识用户。又分为*euid*、*ruid*、*suid*、*fsuid*。
  	- *ruid* —— Real UID，真实用户ID，一般称之为*uid*。
  	- *euid* —— Effective UID，有效用户ID。
  	-  *suid* —— Saved UID，暂存用户ID。
  	-  *fsuid* —— File System UID，文件系统用户ID。
* *gid* —— Group Identifier，用户组标识符，用户辨识用户组。也又分为*egid*、*rgid*、*sgid*、*fsgid*。

在v4.12中，与此有关的系统调用有以下16个：

| | |
| --: | --: | 
| `getuid` | `setuid` |
| `geteuid` | `seteuid` |
| `getresuid` | `setresuid` |
| `getgid` | `setgid` |
| `getegid` | `setegid` |
| `getresgid` | `setresgid` |
| `getfsuid` | `getfsgid` |
| `capget` | `catset` |
# Pre-Internal
## user namespace
> [A new approach to user namespaces][3]梗概：
> 容器可以被看做一种轻量级的虚拟化技术。因为与宿主机共享内核，所以比真正的虚拟机运行效率更高。但必须提供一种机制，把全局可见的资源封装进命名空间中，对容器展现只属于自己的那部分资源（比如进程ID、文件系统、网络接口）的视图。用户命名空间（user namespace）可以被认为是user/group ID以及相关权限的封装，它允许容器的所有者进程（不一定为root用户）在容器内部以root的身份运行，同时将容器内用户与系统的其余部分隔离。那么，同一个进程怎样才能在不同的上下文中有不同的*uid*呢？
> Eric Biederman提交了一组patch解决了这个问题。这组patch中定义了两种新的数据类型`kuid_t`（Kernel UID）与`kgid_t`（Kernel GID）。Kernel UID用于描述进程在内核中的身份，而不管它在容器中可能采用的任何*uid*；它是用于大多数特权检查的值；并且进程并没有办法知道它的值。
> 为了kernel ID与user ID、group ID在不同的命名空间中做转换，除了知道Kernel ID之外，还需要知道特定的namespace ID，因此有了下面的一组用于*uid*的转换函数（*gid*同样存在）：
> ```c
    kuid_t make_kuid(struct user_namespace *from, uid_t uid);
    uid_t from_kuid(struct user_namespace *to, kuid_t kuid);
    uid_t from_kuid_munged(struct user_namespace *to, kuid_t kuid);
    bool kuid_has_mapping(struct user_namespace *ns, kuid_t uid);
		```
> 在Kernel与user ID、group ID之间建立映射是一种特权操作，需要`CAP_SETUID, CAP_SETGID`标志。

# Internal


# Reference
[1]: https://www.kernel.org/doc/html/v4.17/security/credentials.html#task-credentials "Credentials in Linux#task-credentials"
[2]: https://zh.wikipedia.org/wiki/%E7%94%A8%E6%88%B7ID "用户ID"
[3]: https://lwn.net/Articles/491310/ "A new approach to user namespaces"