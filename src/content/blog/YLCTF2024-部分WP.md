---
title: "YLCTF2024部分WP"
description: "YLCTF2024部分WP"
pubDate: "October 21 2024"
image: /images/15.jpg
categories:
  - tech
tags:
  - CTF
  - WP
---

## misc

### 签到

![image-20241012005512328](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410120055368.png)

```
YLCTF{W3lc0m3_T0_Yuan1ooCtf}
```

### hide_png

打开图片，发现是用点......画出的flag，先用stegsolve处理一下

![image-20241012005736588](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410120057816.png)

![image-20241012005814166](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410120058226.png)

发现还是看不清，太宽了，截图到word里面压缩一下，清楚多了（勉强能看）

![image-20241012005902327](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410120059465.png)

```
YLCTF{a27f2d1a-9176-42cf-a2b6-1c87b17b98dc}
```

### pngorzip

猜测是LSB隐写，拿到图片放stegsolve提取数据，发现zip文件头'PK'

![image-20241012010156548](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410120101603.png)

提取数据text，改后缀为zip,然后放010查看，发现从3f 3f 3f 3f 00处往上就是zip的数据，复制16进制到txt文件然后再导入16进制再改后缀为zip,拿到压缩包

![image-20241012010352757](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410120103811.png)

![image-20241012010536074](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410120105126.png)

发现最后的114514????，明显是人为加上去的，以及压缩包解压需要密码，确定是掩码攻击，得到密码114514giao

![image-20241012010738390](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410120107454.png)

解压得flag

```
YLCTF{d359d6e4-740a-49cf-83eb-5b0308f09c8c}
```

## Web

###Disal

![image-20241012011257534](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410120112671.png)

![img](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410120110379.png)

![image-20241012011118078](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410120111153.png)

![image-20241012011223855](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410120112939.png)

```
YLCTF{a7560a21-6d6b-439e-8cab-4d4ecd0a5ddf}
```

## Crypto

### BREAK

![image-20241012011503414](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410120115490.png)

```
YLCTF{fbb6186c-6603-11ef-ba80-deb857dc15be}
```

### signrsa

![image-20241012011711487](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410120117704.png)

```
YLCTF{539d14cd-da2f-47d1-bb29-38d0f80c62da}
```

### ezrsa

![image-20241012011832654](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410120118740.png)

```
YLCTF{121ee5e5-12b2-4204-8b5f-a45052176dc4}
```

## Reverse

### xor

die分析文件，发现是upx加壳，工具脱壳

![image-20241012012151999](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410120121047.png)

![image-20241012012251253](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410120122306.png)

定位到main，逻辑就是将flag字符串的每一位和28进行异或输出。那么将结果再与28异或就能还原

```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  int v3; // edx
  int v4; // ecx
  int v5; // r8d
  int v6; // r9d
  int i; // [rsp+4h] [rbp-Ch]
  __int64 v9; // [rsp+8h] [rbp-8h]

  v9 = getenv("GZCTF_FLAG", argv, envp);
  for ( i = 0; i <= 43; ++i )
  {
    v4 = i;
    v3 = *(unsigned __int8 *)(i + v9) ^ 28;
    *(_BYTE *)(i + v9) ^= 28u;
  }
  printf((unsigned int)"%s", v9, v3, v4, v5, v6);
  return 0;
}
```

```c
#include<stdio.h>

int main(){
	int v9[]={0x45,0x50,0x5f,0x48,0x5a,0x67,0x2a,0x24,0x79,0x7f,0x7d,0x24,0x2e,0x2a,0x31,0x2e,0x7d,0x29,0x29,0x31,0x28,0x2c,0x7f,0x2e,0x31,0x24,0x7a,0x7e,0x78,0x31,0x2c,0x7a,0x2c,0x7e,0x2c,0x2c,0x2b,0x24,0x24,0x78,0x2b,0x2a,0x61,0x1c, };
	for (int i = 0; i <= 43; ++i )
	{
		*(int *)(i + v9) ^= 28u;
		printf("%c",v9[i]);
	}
	return 0;
}
```

```c
//YLCTF{68ea826-2a55-402-8fbd-0f0b00788d76}
```

### ezgo

没做过go语言逆向的题，上网查才知道要用到ghidra

在ghidra中定位到main

![image-20241012012741271](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410120127346.png)

找到关键代码

```go
  for (lVar2 = 0; lVar2 < auVar3._0_8_; lVar2 = lVar2 + 1) {
    *(byte *)(lVar1 + lVar2) = *(byte *)(lVar1 + lVar2) ^ (char)lVar2 + 0x35U;
    local_28 = &DAT_00486080;
    puStack_20 = runtime.staticuint64s + *(byte *)(lVar1 + lVar2);
    fmt.Fprintln(1,1,puStack_20,&local_28);
  }
```

大概意思是将lVar1数组的每一位和(下标的值+0x35)取异或后输出，倒过来写脚本就行

```c
#include<stdio.h>
int main(){
	int data[] = {
		108, 122, 116, 108, 127, 65, 13, 13, 13, 12, 92, 36, 39, 115, 110,
		38, 33, 39, 118, 101, 125, 124, 45, 127, 96, 44, 122, 97, 105, 127,
		97, 98, 101, 50, 50, 110, 109, 98, 111, 105, 100, 60, 34
	};
	for(int i=0;i<(int)(sizeof(data)/sizeof(int));i++){
		*(int *)(data+i)^=0x35u+i;
		printf("%c",data[i]);
	}
	return 0;
}
```

```c
//YLCTF{6102cdf1-bda1-46f3-b518-260de648459b}
```

### xorplus

找到关键代码，经典rc4加密，不过还是要看看有没有变种

```c
  v72 = __readfsqword(0x28u);
  memset(v8, 0, sizeof(v8));
  strcpy(v9, "welcometoylctf");
  s = getenv("GZCTF_FLAG");
  v7 = strlen(s);
  v3 = strlen(v9);                              // 15
  rc4_init((__int64)v8, (__int64)v9, v3);
  rc4_crypt((__int64)v8, (__int64)s, v7);
  for ( i = 0; v7 > i; ++i )
    printf("0x%x,", (unsigned __int8)s[i]);     // 0x91,0x86,0x1b,0x2d,0x9e,0x6f,0x57,0x62,0x77,0xf3,0xf2,0xee,0xd2,0x92,0x22,0x14,0x41,0xee,0x57,0x2d,0x80,0x28,0x4a,0x69,0x6c,0x4f,0x80,0x77,0x7c,0x26,0xb5,0x9b,0x38,0x75,0x2e,0xcf,0x5,0x82,0x63,0xf6,0xb,0x89,0xa6,
  return 0;
```

rc4加密，rc4_init()点进去看看，相比正常初始化多了个1300

![image-20241012013242247](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410120132324.png)

再看看rc4_crypt()，相比正常加密多了个+20

![image-20241012013354600](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410120133710.png)

写脚本

```c
#include <stdio.h>
#include <string.h>

// 初始化 S 盒
void rc4_init(unsigned char *s, const unsigned char *key, int keylen) {
	int i, j = 0;
	for (i = 0; i < 256; ++i) {
		s[i] = i;
	}
	for (i = 0; i < 256; ++i) {
		j = (j + s[i] + key[i % keylen]+1300) % 256;
		// 交换 s[i] 和 s[j]
		unsigned char tmp = s[i];
		s[i] = s[j];
		s[j] = tmp;
	}
}

// 加密/解密数据
void rc4_crypt(unsigned char *s, unsigned char *data, int len) {
	int i = 0, j = 0, t = 0;
	for (int k = 0; k < len; k++) {
		i = (i + 1) % 256;
		j = (j + s[i]) % 256;
		// 交换 s[i] 和 s[j]
		unsigned char tmp = s[i];
		s[i] = s[j];
		s[j] = tmp;

		t = (s[i] + s[j]) % 256;
		data[k] ^= s[t];
	}
}

int main() {
	// 密钥
	const char *key = "welcometoylctf";
	int keylen = strlen(key);

	// 状态数组（S盒）
	unsigned char s[256];

	// 密文
	unsigned char v4[] = {0x91,0x86,0x1b,0x2d,0x9e,0x6f
		,0x57,0x62,0x77,0xf3,0xf2,0xee,0xd2,0x92,
		0x22,0x14,0x41,0xee,0x57,0x2d,0x80,0x28,
		0x4a,0x69,0x6c,0x4f,0x80,0x77,0x7c,0x26,
		0xb5,0x9b,0x38,0x75,0x2e,0xcf,0x5,0x82,0x63,0xf6,0xb,0x89,0xa6,};
	int len = sizeof(v4) / sizeof(v4[0]);
	for(int i=0;i<len;i++){
		v4[i]-=20;
	}
	// 初始化 S 盒
	rc4_init(s, (const unsigned char *)key, keylen);

	// 解密数据
	rc4_crypt(s, v4, len);

	// 输出解密后的数据
	for (int i = 0; i < len; i++) {
		printf("%c", v4[i]);
	}
	printf("\n");

	return 0;
}
```

```c
//YLCTF{2e51ca62-0209-47fd-8bef-656563a6856e}
```

###math

查壳

![image-20241012161949618](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410121619713.png)

找到主程序

![image-20241012162019541](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410121620587.png)

再看check，基本确定这是一个9x9的矩阵，根据用户的输入在对应位置填数字，当满足每一行每一列9个数字的和都为45时输出flag

![image-20241012162105021](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410121621095.png)

查看init()，即矩阵初始化状态

![image-20241012162255133](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410121622357.png)

棋盘根据dword_558E900FF048数组的值按照一定逻辑进行初始化，点进去却发现dword_558E900FF048没有在本地初始化，之前一直以为ELF不能动调，网上一搜才知道ida可以远程调试

![image-20241012162348242](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410121623280.png)

（这里附上动调的配置教程，参考[使用IDAPro + KaliLinux进行动态调试调试ELF - 思泉 | Jev0n](https://jev0n.com/2020/02/16/63.html))

- 0x01 具体步骤

  1.把idapro安装目录中dbgsrv目录下的linux_server或者linux_server64放到linux中（根据自己要调试的程序选择哪个版本的，放置的路径随意）
  ![01.jpg](https://i.loli.net/2020/02/16/kSQcmbBYZCxOyAp.jpg)
  linux_server：用于调试32位Linux应用程序。
  linux_server64：用于调试64位Linux应用程序。

  2.打开终端，提权运行对应服务程序
  使用cd命令打开服务程序所在目录，chmod a+x linux_server64 进行提权，然后运行 linux_server64。具体[linux下chmod +x的意思？为什么要进行chmod +x](https://blog.csdn.net/u012106306/article/details/80436911)可以看这篇文章

这里附上我配置的情况，然后开始远程连接

![image-20241012163525265](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410121635343.png)

![image-20241012163647576](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410121636712.png)

ifconfig先找到自己的ip

然后在ida中选择Remote Linux debugger

![image-20241012163759862](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410121637894.png)

Application和Input file的路径为linux中要调试程序的完整路径，Directory直接输入属性中的父文件夹(除去程序的完整路径)

![image-20241012163949394](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410121639431.png)

这里积极拒绝是因为写wp的时候虚拟机没有连，如果连上再点开始调试就会出现以下界面

![image-20241012164219115](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410121642430.png)

这时可以看到dword_558E900FF048的内容了

![image-20241012164406634](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410121644669.png)

提取数据，写脚本拿到num的初始状态

```c
#include<stdio.h>
int dword_558E900FF048[80] = {
	0, 2, 0, 0, 3, 0, 0, 0, 4, 0, 0, 5, 0, 0, 6, 0, 0, 0, 7, 0, 0, 8, 0, 0, 9, 0,
	1, 0, 0, 2, 0, 0, 3, 0, 0, 0, 4, 0, 0, 5, 0, 0, 6, 7, 0, 0, 8, 0, 0, 9, 0, 0,
	0, 0, 1, 0, 0, 2, 0, 0, 3, 4, 0, 0, 5, 0, 0, 6, 0, 0, 0, 7, 0, 0, 8, 0, 0, 9
};
int num[81];
int init()
{
	int v0; // rax
	int v1; // edx
	int v2; // edx
	int v3; // edx
	int i; // [rsp+Ch] [rbp-14h]
	int j; // [rsp+10h] [rbp-10h]
	int k; // [rsp+10h] [rbp-10h]
	int m; // [rsp+10h] [rbp-10h]
	int v9; // [rsp+14h] [rbp-Ch]
	int v10; // [rsp+18h] [rbp-8h]

	v9 = 0;
	v10 = 1;
	for ( i = 0; i <= 8; ++i )
	{
		v0 = dword_558E900FF048[9 * i - 9];
		if ( v0 == 1 || (dword_558E900FF048[9 * i - 9]== 4) )
		{
			for ( j = 0; j <= 8; j += 3 )
			{
				v1 = v10++;
//				v0 = &num;
				*((int *)&num + 9 * i + j) = v1;
			}
		}
		else
		{
			v0 = *((int *)&num + 9 * i - 9);
			if ( v0 == 4 )
			{
				for ( k = 1; k <= 8; k += 3 )
				{
					v2 = v10++;
//					v0 = &num;
					*((int *)&num + 9 * i + k) = v2;
				}
			}
			else
			{
				v0 = v9;
				for ( m = v9; m <= 8; m += 3 )
				{
					v3 = v10++;
//					v0 = &num;
					*((int *)&num + 9 * i + m) = v3;
				}
			}
		}
		if ( ++v9 == 3 )
			v9 = 1;
		if ( v10 == 10 )
			v10 = 1;
	}
	return (int)v0;
}
int main(){
	init();
	for(int i=0;i<9;i++){
		for(int j=0;j<9;j++){
			printf("%d",*((int *)&num + 9 * i + j));
		}
		printf("\n");
	}
	return 0;
}
```

![image-20241012164548038](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410121645172.png)

程序再将用户通过输入的内容按顺序填到0处，最后判断每行每列加起来是否都为45。数独刚好也满足这个条件，拿解数独的脚本解就可以了

```c
#include<stdio.h>
#include <stdbool.h>
char grid[] =
{
	1, 0, 0, 2, 0, 0, 3, 0, 0,
	0, 4, 0, 0, 5, 0, 0, 6, 0,
	0, 0, 7, 0, 0, 8, 0, 0, 9,
	0, 1, 0, 0, 2, 0, 0, 3, 0,
	0, 0, 4, 0, 0, 5, 0, 0, 6,
	7, 0, 0, 8, 0, 0, 9, 0, 0,
	0, 0, 1, 0, 0, 2, 0, 0, 3,
	4, 0, 0, 5, 0, 0, 6, 0, 0,
	0, 7, 0, 0, 8, 0, 0, 9, 0

};
#define length 60
int flag[length] = { 0 };
int index = 0;

bool isValid(int row, int col, char val, char grid[]) {
	for (int j = 0; j < 9; j++) {
		if (grid[row * 9 + j] == val) {
			return false;
		}
	}
	for (int i = 0; i < 9; i++) {
		if (grid[i * 9 + col] == val) {
			return false;
		}
	}

	int startRow = (row / 3) * 3;
	int startCol = (col / 3) * 3;
	for (int i = startRow; i < startRow + 3; i++) { // 判断9方格里是否重复
		for (int j = startCol; j < startCol + 3; j++) {
			if (grid[i * 9 + j] == val) {
				return false;
			}
		}
	}
	return true;
}
bool backtracking(char grid[]) {
	for (int i = 0; i < 9; i++) {        // 遍历行
		for (int j = 0; j < 9; j++) { // 遍历列
			if (grid[i * 9 + j] != 0) continue;
			for (char k = 1; k <= 9; k++) {     // (i, j) 这个位置放k是否合适
				if (isValid(i, j, k, grid)) {
					grid[i * 9 + j] = k;                // 放置k
					if (backtracking(grid))
					{
						flag[index] = k;
						index++;
						return true; // 如果找到合适一组立刻返回
					}

					grid[i * 9 + j] = 0;              // 回溯，撤销k
				}
			}
			return false;                           // 9个数都试完了，都不行，那么就返回false
		}
	}
	return true; // 遍历完没有返回false，说明找到了合适棋盘位置了
}

int main() {
	backtracking(grid);
	printf("\n");
	//输出数独表
	for (int i = 0; i < 9; i++) {
		for (int j = 0; j < 9; j++) {
			printf("%d", grid[i * 9 + j]);
		}
		printf("\n");
	}
	//输出要填的数字
	for (int i = length - 1; i >= 0; i--) {
		printf("%d ", flag[i]);
	}
	return 0;
}
```

![d29403fd-63c5-4361-8409-924a4581b86a](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410121634367.png)

因此填

```python
#5 6 4 9 7 8 8 9 3 7 1 2 2 3 1 6 4 5 5 8 9 6 7 4 3 9 7 1 8 2 6 2 3 4 1 5 9 8 6 7 5 4 2 3 9 1 8 7 6 5 4 3 2 1
```

就可以拿到flag

![image-20241012164924227](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410121649391.png)

```c
//YLCTF{236632af-3683-4358-9047-8e2f1fd1c9f2}
```

###calc

打开源码

![image-20241012165248899](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410121652997.png)

之前没做过c语言混淆的题，只见过一些python代码混淆的题。问了ai才知道是利用宏定义来混淆代码，再问知道了VS有预编译功能可以直接查看代码

![image-20241012165426925](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410121654083.png)

鼠标放到

![image-20241012165624589](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410121656841.png)

拿到源码后修复一下得到源码，就是一个基于栈的逆波兰计算器

```c
typedef struct Stack {
    double* top; double* low; int size;
}stack; void init(stack* s) {
    s->low = (double*)malloc((sizeof(double))) s->top = s->low; s->size = 100;
}void push(stack* s, double e) {
    *(s->top) = e; s->top++;
}void pop(stack* s, double* e) {
    *e = *--(s->top);
}int main() {
    setbuf((__acrt_iob_func(0)), 0); setbuf((__acrt_iob_func(1)), 0);
    stack s;
    char ch;
    double d, e char num[100];
    int i = 0; init(&s);
    puts("input data , end of '#'");
    scanf("%s", &ch);
    while (ch != '#') {
        while (ch >= '0' && ch <= '9') {
            num[i] = ch;
            scanf("%c", &ch);
            if (ch == ' ') {
                d = atof(num);
                push(&s, d);
                i = 0;
                break
            }
        }
        switch (ch) {
        case'+':
        pop(&s\uff0c & d);
        pop(&s, &e);
        push(&s, e + d);
        break;
        case'-':
        pop(&s, &d);
        pop(&s, &e);
        push(&s, e - d);
        break;
        case'*':
        pop(&s, &d);
        pop(&s, &e);
        push(&s, e * d);
        break;
        case'/':
            pop(&s, &d);
            pop(&s, &e);
            push(&s, e / d)
            break;
        }
        scanf("%c", &ch);
    }
        pop(&s, &d);
        if (d == 125) {
        printf("%s", getenv("GZCTF_FLAG"));
    }
}
```

根据源码，当用户输入经计算结果为125使输出flag

那么构造使结果为125的算式就行

```
5 5 *5 *#
```

![image-20241012170426136](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202410121704200.png)

```c
//YLCTF{d96b0136-cd4d-4e3b-83f6-c63c88a6c3e7}
```
