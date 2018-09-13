<!-- TITLE: Credentials -->
<!-- SUBTITLE: A quick summary of Credentials -->

# Overview
Credentials在Linux中用于访问控制（Access Control），基于*uid*、*gid*、*sid*，是Linux几种安全措施的一部分。同时，仅用于进程（task）中的Capabilities提供了更细化的权限控制机制。[1]

[Note][2]：
* *uid* —— User Identifier，用户标识符，用于辨识用户。又分为*euid*、*ruid*、*suid*、*fsuid*。
  	- *ruid* —— Real UID，真实用户ID，一般称之为*uid*。
		- *euid* —— Effective UID，有效用户ID。
		- *suid* —— Saved UID，暂存用户ID。
		- *fsuid* —— File System UID，文件系统用户ID。
* *gid* —— Group Identifier，用户组标识符，用户辨识用户组。也又分为*egid*、*rgid*、*sgid*、*fsgid*。

# Background



# Reference
[1]: https://www.kernel.org/doc/html/v4.17/security/credentials.html#task-credentials "Credentials in Linux#task-credentials"
[2]: https://zh.wikipedia.org/wiki/%E7%94%A8%E6%88%B7ID "用户ID"