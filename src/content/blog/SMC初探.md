---
title: "SMC初探"
description: "简单总结一下SMC"
pubDate: "March 10 2025"
image: /images/12.jpg
categories:
  - tech
tags:
  - 逆向技术
---

## SMC学习笔记

### 1.解释

**SMC（Self-Modified Code，自修改代码）**是一类特殊的代码技术，即在运行时**修改**自身代码，从而使得程序实际行为与反汇编结果不符，同时修改前的代码段数据也可能非合法指令，从而无法被反汇编器识别。然后在**动态运行**程序时对代码进行**解密**，达到程序**正常运行**的效果。
SMC的实现方式有很多种，可以通过修改PE文件的Section Header、使用API Hook实现代码加密和解密、使用VMProtect等第三方加密工具等。

SMC是一种**局部**代码加密技术，可以提高程序的安全性

### 2.原理

SMC的基本原理是在编译可执行文件时，将需要加密的代码区段（例如函数、代码块等）单独编译成一个section（段），并将其标记为**可读、可写、不可执行**（readable, writable, non-executable），然后通过某种方式在程序运行时将这个section解密为可执行代码，并将其标记为**可读、不可写、可执行**（readable, non-writable, executable）。这样，攻击者就无法在内存中找到加密的代码，从而无法直接执行或修改加密的代码。

### 3.主要步骤

1. 读取PE文件并找到需要加密的代码段。
2. 将代码段的内容进行(异或)加密，并更新到内存中的代码段。
3. 重定向代码段的内存地址，使得加密后的代码能够正确执行。
4. 执行加密后的代码段。

#### 例题复现

这里以[网鼎杯 2020 青龙组]jocker为例

ida打开，定位到main，发现堆栈不平衡报错

![image-20250310212410667](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250310212659534.png)

Options-General-Stack pointer显示sp

在两处call \_\_Z7encryptPc按Alt+K修改SP变化值为0，按c重新分析就恢复正常

![image-20250310212748504](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250310212748559.png)

逻辑很清晰，看`wrong`和`sub_401641`函数

![image-20250310212813840](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250310212813890.png)

![image-20250310212823679](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250310212823729.png)

简单加密验证逻辑，解密代码：

```c
#include<stdio.h>
unsigned int unk_4030C0[24] = {
	0x00000066, 0x0000006B, 0x00000063, 0x00000064, 0x0000007F, 0x00000061, 0x00000067, 0x00000064,
	0x0000003B, 0x00000056, 0x0000006B, 0x00000061, 0x0000007B, 0x00000026, 0x0000003B, 0x00000050,
	0x00000063, 0x0000005F, 0x0000004D, 0x0000005A, 0x00000071, 0x0000000C, 0x00000037, 0x00000066
};

void inverse_wrong(unsigned char*a1){

	for (int i = 0; i <= 23; ++i )
	{
		if ( (i & 1) != 0 )
			a1[i] += i;
		else
			a1[i] ^= i;
	}
}

int main(){
	unsigned char ciphertext[24]={};
	for(int i=0;i<24;i++){
		ciphertext[i]=unk_4030C0[i]&0xff;
	}
	inverse_wrong(ciphertext);
	for(int i=0;i<24;i++){
		printf("%c",ciphertext[i]);
	}

	return 0;
}
//flag{fak3_alw35_sp_me!!}
```

运行出来时假的flag，再往下看`encrypt`函数

报错，点进去是乱码，结合导入了`VirtualProtect`模块判断是SMC

![image-20250310213004848](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250310213004971.png)

`if ( encrypt(Destination) )`处下断点，程序执行到这里时双击encrypt进去修复函数

![image-20250310213204308](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250310213204367.png)

![image-20250310213258878](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250310213258926.png)

逻辑很清晰，执行解密代码发现只有一半的flag：

```c
//flag{d07abccf8a410c
```

再看`sub_40159A`函数（需手动修复）

![image-20250310213518833](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250310213518883.png)

flag总长是24位，还差5位，刚好就是`%tp&:`这5个字符与某个值异或就是flag后5位，因为flag最后一位是`'}'`所以`':'^'}'`的值就是异或的值

exp:

```c
#include<stdio.h>
#include<string.h>
unsigned int unk_403040[19] = {
	0x0000000E, 0x0000000D, 0x00000009, 0x00000006, 0x00000013, 0x00000005, 0x00000058, 0x00000056,
	0x0000003E, 0x00000006, 0x0000000C, 0x0000003C, 0x0000001F, 0x00000057, 0x00000014, 0x0000006B,
	0x00000057, 0x00000059, 0x0000000D
};

int main(){
	char flag[24]={};
	char Buffer[]={"hahahaha_do_you_find_me?"};
	for(int i=0;i<19;i++){
		flag[i]=Buffer[i]^(unk_403040[i]&0xff);
		printf("%c",flag[i]);
	}
	char v3[]={"%tp&:"};
	int xor_number=':'^'}';
	for(int i=0;i<5;i++){
		v3[i]^=xor_number;
	}
	printf("%s",v3);
	return 0;
}
//flag{d07abccf8a410cb37a}
```

参考资料：

[探究SMC局部代码加密技术以及在CTF中的运用 - SecPulse.COM | 安全脉搏](https://www.secpulse.com/archives/197285.html)

[文章 - CTF Reverse逆向学习之SMC动态代码加密技术 - 先知社区](https://xz.aliyun.com/news/14551)
