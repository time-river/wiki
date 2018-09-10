<!-- TITLE: 用户态的系统调用过程 -->
<!-- SUBTITLE: A quick summary of User Space -->

# 如何使用C语言进行系统调用[1][1]
在AMD64平台的Linux下，使用C标准库中的`printf`写一个“hello world”：

```c
#include <stdio.h>

int main(void) {
    printf("hello world!\n");
    return 0;
}
```

编译，`strace`追踪系统调用（syscall）可以看到：

```bash
$ gcc hello.c -o hello
$ strace ./hello
...
write(1, "hello world!\n", 13hello world!
)          = 13
```

可以看到`pritnf`实际上是`write`，因此“hello world”可以这么写：

```c
#include <unistd.h>

int main(void) {
    write(1, "hello world\n", 13);
    return 0;
}
```

可是`write`做了什么呢？更近一步地，可以使用`syscall`+`SYS_write`代替`write`：

```c
#include <unistd.h>
#include <sys/syscall.h>

int main(void) {
    syscall(SYS_write, 1, "hello world!\n", 13);
    return 0;
}
```

而`syscall`，在glibc中，由一段汇编代码定义：

```asm
/*  sysdeps/unix/sysv/linux/x86_64/syscall.S */
.text
ENTRY (syscall)
	movq %rdi, %rax		/* Syscall number -> rax.  */
	movq %rsi, %rdi		/* shift arg1 - arg5.  */
	movq %rdx, %rsi
	movq %rcx, %rdx
	movq %r8, %r10
	movq %r9, %r8
	movq 8(%rsp),%r9	/* arg6 is on the stack.  */
	syscall			/* Do the system call.  */
	cmpq $-4095, %rax	/* Check %rax for error.  */
	jae SYSCALL_ERROR_LABEL	/* Jump to error handler if error.  */
	ret			/* Return to caller.  */

PSEUDO_END (syscall)
```

因此，使用`syscall`汇编指令的GCC代码这么写：

```c
int main(void) {
    register int   syscall_no asm("rax") = 1;
    register int   arg1       asm("rdi") = 1;
    register char* arg2       asm("rsi") = "hello world!\n";
    register int   arg3       asm("rdx") = 13;
    asm("syscall");
    return 0;
}
```


# References
- [How to make a system call in C][1]
- [Hello world in C inline assembly][2]

[1]: https://jameshfisher.com/2018/02/19/how-to-syscall-in-c.html "How to make a system call in C"
[2]: https://jameshfisher.com/2018/02/20/c-inline-assembly-hello-world.html "Hello world in C inline assembly"