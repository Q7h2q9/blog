---
title: "手脱UPX脱壳初探"
description: "不知道为什么半天就能搞定的事以前花一周都搞不定"
pubDate: "April 27 2025"
image: /images/5.jpg
categories:
  - tech
tags:
  - 逆向技术
  - 壳
---

## 手脱UPX脱壳初探

PS:仅面向做题总结经验，底层细节还有待深挖

测试用例：

- buuctf 新年快乐
- [NewStarCTF 2023 公开赛道]咳

- BaseCTF2024 UPX mini,UPX,UPX PRO,UPX PRO MAX

用到的工具：

upxf.exe，upx.exe(upx-5.0.0-win64)

### 0.前言

#### UPX

UPX（Ultimate Packer for eXecutables）是一种常用的可执行文件压缩工具，它通过对程序进行压缩来减小文件体积。

**UPX壳是一个开源的压缩壳，** 是指在可执行文件中通过UPX压缩后，所形成的一种**加壳结构**。简单来说，它是一个封装了压缩程序的“外壳”，在执行时会先解压缩程序的内容，再执行原始程序。

主要用于

- **软件保护**：通过压缩程序，降低被逆向工程和反汇编的概率。
- **减小文件大小**：尤其在网络传输时，压缩后的文件更加节省带宽。

官网：[upx/upx: UPX - the Ultimate Packer for eXecutables](https://github.com/upx/upx)

通过使用官方发布的`release`​(upx.exe)可以对可执行文件进行压缩，并且使ida不能正常识别。下面以几道题和一个demo为例说明

### 1.demo

#### 加壳

以一个简单的**test.c**程序为例，编译后用`upx`​加壳和去壳，用`Die、ida、010editor和xdbg`​观察其变化

```c
#include<stdio.h>

int xor(int a,int b){
	return a^b;
}

int main(){

	int a=0x30;
	int b=0x12;
	printf("异或后的值是：\t");
	printf("%x",xor(a,b));
	return 0;;
}
```

如图，ida正常分析

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427191615962.png)

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427191741141.png)

然后执行命令加壳

```bash
upx.exe test.exe
```

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427191845146.png)

查壳如下

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427191858288.png)

用ida分析，打开时报错，发现打开时函数很少，且有很大一段内容没有被识别出来

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427191941397.png)

在010中发现明显多了一部分内容，标志位是UPX

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427192209745.png)

#### 去壳

最常用的方法就是`怎么加壳就怎么去壳`

执行命令

```bash
upx.exe -d test.exe
```

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427192926096.png)

再用工具查看，和没加壳的一样。

但是，做题过程中常常遇到UPX变种，出题人给exe加壳还修改了其他内容，比如UPX的标志位、overlay_offset值等。此时upx工具不再有效，需要进行手动脱壳。

#### 手动脱壳

这里**使用xdbg应用ESP定律进行脱壳**

首先要先在加壳后的程序中定位到原程序的入口点。使用x64dbg打开后直接运行一步，发现停在了pushad上。该指令将所有寄存器的值压栈，而在UPX的执行流程里，这一步之后会加载UPX的解压代码用于将原始程序解压。

> upx的工作原理其实是这样的：首先将程序压缩。所谓的压缩包括两方面，一方面在程序的开头或者其他合适的地方插入一段代码，另一方面是将程序的其他地方做压缩。压缩也可以叫做加密，因为压缩后的程序比较难看懂，主要是和原来的代码有很大的不同。最大的表现也就是他的主要作用就是程序本身变小了。变小之后的程序在传输方面有很大的优势。其次就是在程序执行时，实时的对程序解压缩。解压缩功能是在第一步时插入的代码完成的功能。联起来就是：upx可以完成代码的压缩和实时解压执行。且不会影响程序的执行效率。
>
> upx和普通的压缩，解压不同点就算在于upx是实时解压缩的。实时解压的原理可以使用一下图形表示：
>
> ```python
> graph LR;
>   1-->2-->3-->4-->5-->6;
> ```
>
> 假设1是upx插入的代码，2，3，4是压缩后的代码。5，6是随便的什么东西。程序从1开始执行。而1的功能是将2，3，4解压缩为7，8，9。7，8，9就是2，3，4在压缩之前的形式。
>
> ```python
> graph LR;
>   1-->7-->8-->9-->5-->6;
> ```
>
> 连起来就是：
>
> ```python
> graph LR;
>   1==>2-.->3-.->4-.->5;
>   7==>8==>9==>5==>6;
>   2-->解密-->7;
>   3-->解密-->8;
>   4-->解密-->9;
> ```

注意要在`选项`​-`首选项`​中把系统断点打开，调试器就可以在`pushad`​处断下

> 在解压过程中，**UPX壳代码常常会用** **​`pushad`​**​ **/** **​`popad`​**​ **这种指令保存和恢复寄存器状态**。

> - UPX壳起始处的动作比较明显，一般是做一些基本检查（比如判断是不是自己解包过了），然后**马上保存现场（pushad）** 。
> - 由于xdbg在加载完模块后，会根据设置自动打一些"系统断点"（如入口点、TLS、系统API），这样你有机会**在UPX壳的早期代码里介入**。
> - **pushad指令因为非常标志性**（很少一上来就手动push这么多），所以特别容易成为壳保护程序的一个分水岭。

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427193446860.png)

**具体操作**

- F9运行两次，来到pushad处

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427194223752.png)

f8步过，走完push，RSP的值变为`0x6CCCFFFA38`

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427194252394.png)

右键RSP-在转储/内存中跟随，右键内存对应的数据-断点-`硬件-访问/存取`​-双字(四字节)

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427194718920.png)

f9执行，程序成功断在popad处，说明UPX的解压过程已经结束，现在在进行一些清理恢复工作。

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427194748211.png)

f8步过，有一个循环，在其后下断点，f9过去，来到`jmp`​处，步进。这里就是真正的**OEP(Original Entry Point)**

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427194821814.png)

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427195127063.png)

找到入口点后使用Scylla工具脱壳

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427195234028.png)

点击`转储/dump`​，会在当前文件夹下生成一个`test_dmp.exe`​文件，

然后点击IAT自动搜索（可能会出现以下情况，选择'是'）

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427195506497.png)

再点击获取导入，最后修复转储，选择刚刚导出的`test_dmp.exe`​文件，在当前文件夹下生成`test_dump_SCY.exe`​文件。

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427195541137.png)

脱壳到此结束

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427195623434.png)

#### 禁用ASLR

但是！程序无法正常运行，并且用ida打开还是不对，怎么回事？

问题出在**64位程序**普遍开了 **ASLR**（Address Space Layout Randomization)，地址空间随机化。使得每次PE文件记载到虚拟内存的起始地址不一样, 而且栈和堆起始地址也不一样。

> 微软从windows vista/windows server 2008（kernel version 6.0）开始采用ASLR技术，主要目的是为了防止缓冲区溢出
>
> ASLR技术会使PE文件每次加载到内存的起始地址随机变化，并且进程的栈和堆的起始地址也会随机改变。
>
> 该技术需要操作系统和编译工具的双重支持（主要是操作系统的支持，编译工具主要作用是生成支持ASLR的PE格式）
>
> 若不想使用ASLR功能，可以在VS编译的时候将“**配置属性-&gt;链接器-&gt;高级-&gt;随机基址”的值修改为否即可**

使用010或者CFF Explore工具打开test.exe文件，**修改NT_Header下的Optional Header下的DllCharacteristics值，**

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427202147360.png)

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427202202002.png)

当前是0x0160，二进制是`0000 0001 0110 0000`

每一位代表一个特性，主要的：

| 位     | 功能            | 含义                       |
| ------ | --------------- | -------------------------- |
| 0x0100 | NXCompat        | 支持DEP（内存保护）        |
| 0x0040 | **DynamicBase** | 支持地址空间随机化（ASLR） |
| 0x0020 | NoIsolation     | 不使用隔离                 |

要做的就是把 `0x0040`​ 这位去掉，也就是：

```plaintext
0x0160 - 0x0040 = 0x0120
```

即把0x0160改为0x0120即可

注意，不同的文件这里的值可能不同，但要去掉ASLR都是减去0x40

修改后保存，重新用xdbg走一遍

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427202646980.png)

可以发现RSP的值已经不是之前那一长串儿了

然后走一样的流程就行，IDA能够正常分析，但是还是有问题——程序无法运行👇👇

注意：如果遇到IAT表导入部分错误需要修复IAT表(造成无法运行等的错误)，参考文章：[通过x64dbg脚本功能修复IAT表](https://bbs.kanxue.com/thread-273896.htm)

(尝试了没有成功，我的xdbg不能运行某系汇编？它报错说无法识别指令：Loop，有没有大佬教教🥲。我的xdbg是在吾爱上下载的`x64dbg 中文版`​)代码如下

```asm
$ArrayPtr = 475000
$ArrayEnd = 475120

Loop:

cmp dword:[$ArrayPtr], 0    // 跳过模块空隙
je Next

EIP = dword:[$ArrayPtr]
RunToParty 1                // 执行到系统模块断下
dword:[$ArrayPtr] = EIP     // 取当前地址

Next:

$ArrayPtr += 4              // 指向下一块地址
cmp $ArrayPtr, $ArrayEnd    // 判断是否结束
jne Loop
ret
```

附一张图(小白初次学习，啥都不懂)

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427211208207.png)

---

### 2.实战

#### buuctf 新年快乐

PE32，upx壳

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427211826674.png)

还是ESP定律，32位程序能直接显示`pushad`

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427211957135.png)

找到OEP，dump+修IAT表

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427212024909.png)

其实这里直接拿dump后的文件丢IDA分析是可以的(会有一个报错，不过无伤大雅)，只是不知道为什么不论是否修没修IAT，在IDA中都是显示`UPX0:`​段而不是正常的`text:`​段

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427212257831.png)

修复IAT表后的如下(一模一样，也许本来就不需要修？**但是不修就没法正常运行，只是IDA能正常静态分析！** )：

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427212430386.png)

脱壳结束，可以正常分析了

#### [NewStarCTF 2023 公开赛道]咳

PE64，UPX壳

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427212908388.png)

一样的方法，具体过程就略过了

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427213019567.png)

其中`KE_dump_SCY.exe`​和`KE_dump.exe`​都不能正常运行，但不影响IDA正常分析，动调估计还是不行。

运行不了原因如下

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427213445916.png)

IAT表有问题，但是笔者还不知道怎么修/(ㄒoㄒ)/~~

#### BaseCTF2024 UPX mini,UPX,UPX PRO,UPX PRO MAX

连放4道，主要说考点

- `UPX mini`​就是`upx.exe file.exe`​加壳来的，走常规流程就行。但是手脱得到的dump和修补后的程序依然无法运行，还是上面的问题👆

- `UPX`​在加壳的基础上修改了标志位，这里是小写`upx`​，正常的应该是大写`UPX`​，修改后就能正常脱壳。`UPX_dump.exe`​无法运行，`UPX_dump_SCY.exe`​能正常运行，说明IAT表修复成功。

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427214125216.png)

- ​`UPX PRO`​是一个upx打包的**ELF文件**

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427214458137.png)

考点是`overlay_offset`​的值应该是F4而不是4F

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427214629795.png)

- UPX PRO MAX

UPX标志位全改成00了，只能手脱

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427214815298.png)

​`UPX PRO MAX_dump.exe`​和`UPX PRO MAX_dump_SCY.exe`​都不能运行，看官方WP应该也是静态分析

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250427214943472.png)

### 3.总结

本文以一个demo和几道题为例，介绍使用`xdbg`​和ESP定律手脱`upx`​壳的过程

手脱过程中遇到了IAT表不能自动修复从而导致脱壳后的程序只能静态分析，无法正常运行的情况。参考一篇文章的讲解([通过x64dbg脚本功能修复IAT表](https://bbs.kanxue.com/thread-273896.htm))但遇到了其他问题

PS：还是不够清爽，再看看其他文章找找方法

### 4.参考资料

主要：

- [[原创] 借助 x64dbg 的 UPX 手工脱壳-加壳脱壳-看雪-安全社区|安全招聘|kanxue.com](https://bbs.kanxue.com/thread-268159.htm)
- [修改PE文件头删除ASLR\_怎么修改pe文件让他支持aslr-CSDN博客](https://blog.csdn.net/qq_33976344/article/details/115016181#:~:text=%E7%86%9F%E7%9F%A5PE%E6%96%87%E4%BB%B6%E5%A4%B4%E6%A0%BC%E5%BC%8F%E7%9A%84%E9%80%86%E5%90%91%E7%8E%A9%E5%AE%B6%2C%20%E5%8F%AF%E4%BB%A5%E9%80%9A%E8%BF%87%E4%BF%AE%E6%94%B9%E4%B8%80%E4%B8%AA%E6%A0%87%E5%BF%97%E4%BD%8D%E6%9D%A5%E8%BE%BE%E5%88%B0%E5%88%A0%E9%99%A4%E7%BC%96%E8%AF%91%E5%A5%BD%E7%9A%84%E5%8F%AF%E6%89%A7%E8%A1%8C%E6%96%87%E4%BB%B6%E7%9A%84ASLR%E5%8A%9F%E8%83%BD.%20%E6%9C%89%20IMAGE_DLLCHARACTERISTICS_DYNAMIC_BASE%20%E6%A0%87%E5%BF%97%2C%208140%E6%94%B9%E5%88%B08100%E5%8D%B3%E5%8F%AF%E5%88%A0%E9%99%A4ASLR.,Hex%20Editor%E6%89%93%E5%BC%80%E6%A0%B9%E6%8D%AEPE%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F%E6%89%BE%E5%88%B0%E5%81%8F%E7%A7%BB%20%280x136%29%E5%A4%84%E4%BF%AE%E6%94%B9.%20%E6%B3%A8%E6%84%8F%E6%AF%8F%E4%B8%AAPE%E6%96%87%E4%BB%B6%E4%B8%8D%E4%B8%80%E5%AE%9A%E7%9B%B8%E5%90%8C%2C%20%E9%9C%80%E8%A6%81PE-view%E6%9F%A5%E7%9C%8B%2C%20%E6%88%96%E8%80%85CFF%E7%9B%B4%E6%8E%A5%E7%9C%8B%E6%94%B9%E5%8D%B3%E5%8F%AF.%20%E6%96%87%E7%AB%A0%E6%B5%8F%E8%A7%88%E9%98%85%E8%AF%BB796%E6%AC%A1%E3%80%82)
- 👉[完美UPX脱壳-之投怀送抱篇（适合所有变形） - 吾爱破解 - 52pojie.cn](https://www.52pojie.cn/thread-1673206-1-1.html)👈

其他：

- [细讲UPX壳 | CN-SEC 中文网](https://cn-sec.com/archives/3517798.html)

- [深入解析 UPX 压缩与脱壳：从原理到实战-CSDN博客](https://blog.csdn.net/m0_57836225/article/details/145504179)
- [UPX - 维基百科，自由的百科全书](https://zh.wikipedia.org/zh-cn/UPX)
- [关闭地址随机化ASLR - 简书](https://www.jianshu.com/p/91b2b6665e64)
- [破解入门（五）-----实战"ESP定律法"脱壳-CSDN博客](https://blog.csdn.net/qiurisuixiang/article/details/7649799)
