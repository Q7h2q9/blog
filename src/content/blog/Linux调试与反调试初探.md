---
title: "Linux调试与反调试初探"
description: "ps:初入Linux调试，内容疏浅，如有不对的地方请指出。:)"
pubDate: "April 01 2025"
image: /images/image1.png
categories:
  - tech
tags:
  - Hacks
  - 逆向技术
---

## Linux调试与反调试初探

### 补一下基础知识

#### enum

`enum`枚举是 C 语言中的一种基本数据类型，用于定义一组具有离散值的常量，

```c
enum DAY
{
      MON=1, TUE, WED, THU, FRI, SAT, SUN
};
```

第一个枚举成员的默认值为整型的 0，后续枚举成员的值在前一个成员上加 1。我们在这个实例中把第一个枚举成员的值定义为 1，第二个就为 2，以此类推

```c
#include <stdio.h>

enum DAY
{
      MON=1, TUE, WED, THU, FRI, SAT, SUN
};

int main()
{
    enum DAY day;
    day = WED;
    printf("%d",day);
    return 0;
}
```

#### fork()

**在父进程中**：`fork()` 返回 **子进程的 PID**（进程标识符），即一个正整数。如果 `fork()` 成功，父进程会接收到子进程的 PID。

**在子进程中**：`fork()` 返回 **0**，即子进程在创建后会接收到返回值为 `0`。

**如果发生错误**：`fork()` 返回 **-1**，并且会设置 `errno` 以指示具体的错误原因。常见的错误包括系统资源不足（如进程表已满）或达到进程数限制。

测试代码：

```c
#include <stdio.h>
#include <unistd.h>

int main() {
    pid_t pid = fork();

    if (pid == -1) {
        // 错误处理
        perror("fork failed");
    } else if (pid == 0) {
        // 子进程
        printf("This is the child process.\n");
    } else {
        // 父进程
        printf("This is the parent process. Child PID: %d\n", pid);
    }

    return 0;
}

```

运行结果：

![image-20250402143018344](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithubimage-20250402143018344.png)

#### execl函数

**函数原型**：

```c
#include <unistd.h>

int execl(const char *path, const char *arg, ... /* (char  *) NULL -->必须以NULL结尾*/);

```

| 参数   | 说明                                                        |
| ------ | ----------------------------------------------------------- |
| `path` | 要执行的程序路径（可以是绝对路径或相对路径）                |
| `arg`  | `argv[0]`，通常是程序名（传统上与 `path` 的最后一部分相同） |
| `...`  | 传递给新程序的参数，**必须以 `NULL` 结尾**                  |

**返回值**

- **成功**：不返回，因为进程的代码已经被替换。
- **失败**：返回 `-1`，并设置 `errno` 来指示错误。

示例代码：

```c
#include <stdio.h>
#include <unistd.h>

int main() {
    printf("Before execl\n");

    // 用 /bin/ls 替换当前进程
    execl("/bin/ls", "ls", "-l", NULL);

    // 只有在 execl 失败时才会执行这行
    perror("execl failed");
    printf("After execl\n");
    // 这行代码不会被执行，因为 execl 成功后，当前进程会被替换
    return 1;
}

```

![image-20250402151933194](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithubimage-20250402151933194.png)

注意：

`execl` 成功执行后，当前进程的代码会被新程序完全替换，**不会继续执行原代码**。
但如果 `execl` 失败，例如目标程序不存在或权限不足，函数才会返回 `-1`，然后执行 `perror` 等后续代码。

#### syscall函数

`syscall`函数是Linux内核中提供给用户空间程序使用的接口，也称为系统调用函数。它可以使用户空间程序向内核发出请求，并获取内核返回结果。`syscall`函数是操作系统与应用程序之间进行通信的桥梁**linux syscall函数**，是Linux系统中最基本、最底层的API之一。

**函数原型**

```c
NAME
       syscall - 间接系统调用

SYNOPSIS
       #define _GNU_SOURCE
       #include <unistd.h>
       #include <sys/syscall.h>                 /* For SYS_xxx definitions */

       int syscall(int number, ...);
```

**参数**

`number`：要调用的系统调用号（syscall number）。

后面的可变参数 `...` 是系统调用的参数，具体数量和类型取决于所调用的系统调用。

**返回值**

`syscall` 返回值通常是系统调用的返回值：

- 成功时返回 0 或正整数（取决于具体的系统调用）。
- 失败时返回 `-1`，并设置 `errno`。

![掌握Linux syscall函数，深入解析内核机制掌握Linux syscall函数，深入解析内核机制](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub1682856477141_1.jpg)

在Linux内核中**linux syscall函数**，所有的系统调用都是由内核中断实现的。当用户空间程序调用syscall函数时，会触发一个软中断（也称为系统调用中断），然后内核会根据传入参数的不同来执行相应的操作，并将结果传递给用户空间程序。

### ptrace函数

函数原型：

```c
NAME
    ptrace - process trace
SYNOPSIS
       #include <sys/ptrace.h>
       long ptrace(enum __ptrace_request request, pid_t pid,
                   void *addr, void *data);
```

**参数**：

`request` —— 指定 `ptrace` 进行的操作（如 **附加进程**、**读取内存**、**修改寄存器**）。

`pid` —— 目标进程的 PID（即要调试的子进程）。

`addr` —— 访问的内存地址（部分操作忽略此参数）。

`data` —— 额外数据（可读写内存、修改寄存器等）。

#### request参数

request的不同参数决定了系统调用的功能

| PTRACE_TRACEME                             | **子进程调用**，表示允许父进程调试自己，此时剩下的pid、addr、data参数都没有实际意义可以全部为0。这个选项只能用在被调试的进程中，也是被调试的子进程唯一能用的request选项，其他的都只能用父进程调试器使用 |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| PTRACE**\*PEEK**TEXT, PTRACE**\*PEEK**DATA | 从内存地址中**读取**一个字节，内存地址由addr给出。                                                                                                                                                      |
| PTRACE\_**PEEK**USER                       | 从USER区域中**读取**一个字节，偏移量为addr。                                                                                                                                                            |
| PTRACE**\*POKE**TEXT, PTRACE**\*POKE**DATA | 往内存地址中**写入**一个字节。内存地址由addr给出。                                                                                                                                                      |
| PTRACE\_**POKE**USER                       | 往USER区域中**写入**一个字节。偏移量为addr。                                                                                                                                                            |
| PTRACE_SYSCALL,                            | 让进程在每次系统调用前后**暂停**                                                                                                                                                                        |
| PTRACE\_**KILL**                           | **杀掉子进程**，使它退出。                                                                                                                                                                              |
| PTRACE_SINGLESTEP                          | 设置单步执行标志，**单步执行**一条指令                                                                                                                                                                  |
| PTRACE\_**GETREGS**                        | **获取**寄存器状态（x86 上 `struct user_regs_struct`）                                                                                                                                                  |
| PTRACE\_**SETREGS**                        | **设置**寄存器状态                                                                                                                                                                                      |
| PTRACE\_**ATTACH**                         | attach到一个指定的进程，**使其成为当前进程跟踪的子进程**，而子进程的行为等同于它进行了一次PTRACE_TRACEME操作，可想而知，gdb的`attach`命令使用这个参数选项实现的。                                       |
| PTRACE\_**DETACH**                         | 结束跟踪，**分离**进程，使其恢复正常执行                                                                                                                                                                |
| PTRACE_CONT                                | 继续运行之前停止的子进程，也可以向子进程发送指定的信号，这个其实就相当于gdb中的`continue`命令                                                                                                           |

**返回值**

- **成功时**：大多数 `ptrace` 操作返回 `0`，但某些请求可能会返回具体的数据（如 `PTRACE_PEEKDATA` 读取的值）。

- **失败时**：返回 `-1`，并设置 `errno`，表示发生错误。

![img](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20191201144906-ab752400-1406-1.png)

gdb调试的本质实际上就是父进程使用**ptrace**函数对子进程进行一系列的命令操作

示例代码1：

```c
#include <stdio.h>
#include <sys/ptrace.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>

int main() {
    pid_t pid = fork();  // 创建子进程

    if (pid == 0) {
        // 子进程执行 PTRACE_TRACEME，表示自己允许被调试
        ptrace(PTRACE_TRACEME, 0, NULL, NULL);
        printf("Child process running...\n");
        execl("/bin/ls", "ls", NULL); // 执行 ls 命令
    } else {
        // 父进程等待子进程停止，并继续调试
        wait(NULL);
        printf("Parent: Tracing child process (PID: %d)...\n", pid);
        ptrace(PTRACE_CONT, pid, NULL, NULL);  // 继续执行子进程
        wait(NULL);
    }
    return 0;
}
```

![image-20250402152756388](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithubimage-20250402152756388.png)

示例代码2：

```c
#include <sys/ptrace.h>
#include <sys/wait.h>
#include <sys/reg.h>   /* For constants ORIG_EAX etc */
#include <sys/user.h>
#include <sys/syscall.h> /* SYS_write */
#include <stdio.h>
int main() {
    pid_t child;
    long orig_rax;
    int status;
    int iscalling = 0;
    struct user_regs_struct regs;

    child = fork();
    if(child == 0)
    {
        ptrace(PTRACE_TRACEME, 0, 0);//发送信号给父进程表示已做好准备被调试
        execl("/bin/ls", "ls", "-l", "-h", 0);
    }
    else
    {
        while(1)
        {
            wait(&status);//等待子进程发来信号或者子进程退出
            if(WIFEXITED(status))
            //WIFEXITED函数(宏)用来检查子进程是被ptrace暂停的还是准备退出
            {
                break;
            }
            orig_rax = ptrace(PTRACE_PEEKUSER, child, 8 * ORIG_RAX, 0);
            //获取rax值从而判断将要执行的系统调用号
            if(orig_rax == SYS_write)
            {//如果系统调用是write
                ptrace(PTRACE_GETREGS, child, 0, &regs);
                if(!iscalling)
                {
                    iscalling = 1;
                    //打印出系统调用write的各个参数内容
                    printf("SYS_write call with %p, %p, %p\n",
                            regs.rdi, regs.rsi, regs.rdx);
                }
                else
                {
                    printf("SYS_write call return %p\n", regs.rax);
                    iscalling = 0;
                }
            }

            ptrace(PTRACE_SYSCALL, child, 0, 0);
            //PTRACE_SYSCALL,其作用是使内核在子进程进入和退出系统调用时都将其暂停
            //得到处于本次调用之后下次调用之前的状态
        }
    }
    return 0;
}
```

![image-20250402160738108](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithubimage-20250402160738108.png)

可以看到，每一次进行系统调用前以及调用后的寄存器内容都发生的变化，并且输出了`ls -l -h`的内容

#### 反调试案例

![image-20250402160911347](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithubimage-20250402160911347.png)

![image-20250402161251540](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithubimage-20250402161251540.png)

#### 如何绕过

一般有以下几种操作：

1. 打patch，把有关ptrace函数的部分**nop**掉
2. 利用hook技术，把ptrace函数给**替换成自定义的ptrace函数**，从而可以**任意指定它的返回值**
3. 充分利用gdb的catch命令，`catch syscall ptrace`会在发生ptrace调用的时候停下，因此在第二次停住的时候`set $rax=0`，从而绕过程序中`ptrace(PTRACE_TRACEME, 0, 0, 0) ==-1`的判断

在 GDB 中，可以使用 `catch syscall ptrace` 来捕获 `ptrace` 系统调用，这样每当程序调用 `ptrace`，GDB 都会自动暂停，并让你检查寄存器和调用参数。

![image-20250402162602830](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithubimage-20250402162602830.png)

注意第一次调用时显示的信息是`call to syscall ptrace`，而第二次是`returned from syscall ptrace`

程序实际上是执行了 `ptrace(PTRACE_TRACEME)` 系统调用并等待父进程（调试器）的控制。`continue` 命令让程序继续执行，直到 `ptrace` 系统调用完成并返回，之后程序可以继续执行后续的操作。

成功绕过

### 参考资料：

[掌握Linux syscall函数，深入解析内核机制 | 《Linux就该这么学》](https://www.linuxprobe.com/zwlhssrjxnhj.html)

[Linux系统——fork()函数详解(看这一篇就够了！！！)\_fork函数-CSDN博客](https://blog.csdn.net/cckluv/article/details/109169941)

[linux下syscall函数-CSDN博客](https://blog.csdn.net/swj9099/article/details/77890964)

[Linux逆向之调试&反调试-先知社区](https://xz.aliyun.com/news/6478?time__1311=YqIxgQG%3D0%3DqWqGNDQiiQdD%3DeHTFxBjD8oD&u_atoken=6355ada0db6b52885799866b7feddda5&u_asig=0a472f8317435720196854801e0035)

[Linux平台反调试技术概览-CSDN博客](https://blog.csdn.net/stonesharp/article/details/8211526)

[Anti-debugging Skills in APK | WooYun知识库](http://drops.xmd5.com/static/drops/mobile-16969.html)
