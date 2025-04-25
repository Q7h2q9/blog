---
title: "blowfish算法加解密原理及代码实现"
description: "应用密码学布置了一个设计对称加密系统的作业，趁此机会了解一下Blowfish算法"
pubDate: "April 26 2025"
image: "https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250426000239602.png"
categories:
  - tech
tags:
  - 加密算法
---

## blowfish算法加解密原理及代码实现

### 概述

**Blowfish**是一个**对称密钥加密分组密码算法**，由布鲁斯·施奈尔于1993年设计，现已应用在多种加密产品。**Blowfish算法由于分组长度太小已被认为不安全**，施奈尔更建议在现代应用中使用[Twofish](https://zh.wikipedia.org/wiki/Twofish "Twofish")密码。<sup>[[2]](https://zh.wikipedia.org/wiki/Blowfish#cite_note-schneier-interview-dec-2007-2)</sup>

> 施奈尔设计的Blowfish算法用途广泛，意在替代老旧的DES及避免其他算法的问题与限制。Blowfish刚刚研发出的时候，大部分其他加密算法是专利所有的或属于商业(政府)机密，所以发展起来非常受限制。施奈尔则声明Blowfish的使用没有任何限制，任何国家任何人任何时候都可以随意使用Blowfish算法。
>
> Blowfish的设计目标如下：
>  快速：Blowfish在32位微处理器中的加密速率为每个字节26个时钟循环。
>  紧凑：Blowfish可以在不到5kb的内存中执行。
>  简单：Blowfish只使用基本运算，如加法、异或和表格查阅，因此很容易设计与实现。
>  安全：Blowfish是变长密钥的，最长448位，使其既灵活又安全。
> Blowfish适合密钥长期不变的情形（如通信链路加密），而不适合密钥经常更换的情形（如分组加密）。
>
> ‍

- BlowFish是一个对称区块加密算法。每次加密数据为 **64位(** 八个字节)
- Blowfish 的密钥长度是可变，支持32-448位
- BlowFish是由一个16轮循环的Feistel结构进行加密的。

### 加解密原理

Blowfish用变长密钥加密64位块，分成如下两个部分：
（1）子密钥生成：这个过程将最长448位的密钥转换成总长为**4168**位的子密钥。
（2）数据加密：这个过程将简单函数迭代16次，每一轮包含密钥相关置换和密钥与数据相关替换。

#### 子密钥生成

##### 初始化S盒与P盒

Blowfish 的密钥生成过程从 P-盒和 S-盒的初始化开始。

- **P-盒**：Blowfish 使用 18 个 32 位的 P-盒（P0 到 P17）。每个 P-盒存储一个 32 位的值。P-盒的初始值来自一个**固定的常量数组**。
- **S-盒**：Blowfish 使用 4 个 S-盒，每个 S-盒有 256 个 32 位的条目。每个 S-盒的条目也是从一个**常量数组**中初始化的。

对于P盒和S盒的初始值来源，这里要先了解**Blowfish 常量**

- **常量源**：Blowfish 的常量源是一个字符串，该字符串是由 64 位的常数值组成。这个常数值是由将 π (圆周率) 的二进制小数部分转换成 32 位块而得来的。具体来说，Schneier 取了 $π$ 的小数部分的每 **32** 位，依次生成每个 P-盒和 S-盒的初始值。
- **初始常量序列**：这个常量序列作为初始化的基础，在 P-盒和 S-盒中使用。具体来说，P-盒中的初始值是基于这个序列的前 18 个 32 位常数值，而 S-盒中的初始值则使用后续的常数值来填充每个 256 条目。

例如，P盒的前几位

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithubimage-20250425224625-5gti2ci.png)

以`0.14159265358979323846`​为例通过在线网站[小数转换16进制](https://www.sojson.com/hexconvert.html)测试一下：

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250426000239602.png)

和上面的P盒初始值一样，通过这种方式，先后初始化了P盒和S盒

##### 用key修改S盒与P盒

将pbox中的数据与`key`​进行**循环异或**然后设置到`key_pbox`​，注意每一次取key的32位，代码展现如下

```c
int j = 0;
	for (int i = 0; i < BLOWFISH_NUM_ROUNDS + 2; i++) {
		uint32_t data = 0;
		for (int k = 0; k < 4; k++) {
			data = (data << 8) | key[j];
			j = (j + 1) % keyLen;
		}
		ctx->P[i] ^= data;
	}
```

使用当前的`key_pbox`​与`s_box`​对一个64位0数据进行加密。 也就是 两个int 类型数据 8个字节 输出的结果 重新修改到key_pbox 与 key_sbox中。

key_sbox就是当前的使用s_box对64位数据进行加密，产生的输出修改的key_sbox中。代码展现如下

```c
uint32_t L = 0, R = 0;
	for (int i = 0; i < BLOWFISH_NUM_ROUNDS + 2; i += 2) {
		Blowfish_EncryptBlock(ctx, &L, &R);
		ctx->P[i] = L;
		ctx->P[i + 1] = R;
	}
	for (int i = 0; i < 4; i++) {
		for (int j = 0; j < 256; j += 2) {
			Blowfish_EncryptBlock(ctx, &L, &R);
			ctx->S[i][j] = L;
			ctx->S[i][j + 1] = R;
		}
	}
```

注意这里说的key_sbox就是当前的使用s_box与64位数据进行加密操作，输出到key_sbox中。

#### 加解密原理

##### 加密逻辑

先看图

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithubimage-20250426002856-012ipg1.png)

大概的流程就是将P(原始数据)分成左右两部分`L与R`​，先拿`L`​和$K_r$做异或操作，得出的结果调用**F函数**，最后将F函数的输出结果和`R`​进行异或操作。

调换左右部分的位置，继续进行这样的操作，总共进行**16轮**就得到了最终的加密结果。

其中，这里的$K_r$ 就是经`key`​的作用变换得到的`key_pbox`​，范围是从$K_1$ 到 $K_{18}$ 。总共有**18个**密钥组成的数组。 每个密钥的长度是**32**位。

注意最后一轮是把$K_{18}和K_{17}$分别与$L和R$做异或得到加密结果。

##### F函数

函数F的功能如下：

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithubimage-20250426003927-9rudlrk.png)

##### 解密逻辑

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithubimage-20250426004044-fzgtzoq.png)

### 代码(C语言)

```c
#include <stdint.h>
#include <string.h>
#include <stdlib.h>
#include <stdio.h>
#include <time.h> // 用于计时
/*注意：
使用的 clock() 函数（来自 <time.h>）返回的是CPU时间（process time），它只计算程序真正执行指令花费的时间，而I/O操作（例如读写文件）往往不会算进去。
而解密函数可能因为：
解密过程太快（几KB文件，几个块一解，几百万次CPU指令不到）
文件很小（如几十字节、几百字节），解密逻辑瞬间完成
所以 clock() 的值可能还没来得及“跳动”，最终计算结果就是 0 毫秒（四舍五入）。
*/
#include <windows.h>
#include <wincrypt.h>//用于生成随机密钥
#define KEY_SIZE 16  // 128-bit 密钥
#define BLOWFISH_NUM_ROUNDS 16
static const uint32_t _BLOWFISH_SBox [ 4 ] [ 256 ] =
{
	{
		0xd1310ba6l, 0x98dfb5acl, 0x2ffd72dbl, 0xd01adfb7l,
		0xb8e1afedl, 0x6a267e96l, 0xba7c9045l, 0xf12c7f99l,
		0x24a19947l, 0xb3916cf7l, 0x0801f2e2l, 0x858efc16l,
		0x636920d8l, 0x71574e69l, 0xa458fea3l, 0xf4933d7el,
		0x0d95748fl, 0x728eb658l, 0x718bcd58l, 0x82154aeel,
		0x7b54a41dl, 0xc25a59b5l, 0x9c30d539l, 0x2af26013l,
		0xc5d1b023l, 0x286085f0l, 0xca417918l, 0xb8db38efl,
		0x8e79dcb0l, 0x603a180el, 0x6c9e0e8bl, 0xb01e8a3el,
		0xd71577c1l, 0xbd314b27l, 0x78af2fdal, 0x55605c60l,
		0xe65525f3l, 0xaa55ab94l, 0x57489862l, 0x63e81440l,
		0x55ca396al, 0x2aab10b6l, 0xb4cc5c34l, 0x1141e8cel,
		0xa15486afl, 0x7c72e993l, 0xb3ee1411l, 0x636fbc2al,
		0x2ba9c55dl, 0x741831f6l, 0xce5c3e16l, 0x9b87931el,
		0xafd6ba33l, 0x6c24cf5cl, 0x7a325381l, 0x28958677l,
		0x3b8f4898l, 0x6b4bb9afl, 0xc4bfe81bl, 0x66282193l,
		0x61d809ccl, 0xfb21a991l, 0x487cac60l, 0x5dec8032l,
		0xef845d5dl, 0xe98575b1l, 0xdc262302l, 0xeb651b88l,
		0x23893e81l, 0xd396acc5l, 0x0f6d6ff3l, 0x83f44239l,
		0x2e0b4482l, 0xa4842004l, 0x69c8f04al, 0x9e1f9b5el,
		0x21c66842l, 0xf6e96c9al, 0x670c9c61l, 0xabd388f0l,
		0x6a51a0d2l, 0xd8542f68l, 0x960fa728l, 0xab5133a3l,
		0x6eef0b6cl, 0x137a3be4l, 0xba3bf050l, 0x7efb2a98l,
		0xa1f1651dl, 0x39af0176l, 0x66ca593el, 0x82430e88l,
		0x8cee8619l, 0x456f9fb4l, 0x7d84a5c3l, 0x3b8b5ebel,
		0xe06f75d8l, 0x85c12073l, 0x401a449fl, 0x56c16aa6l,
		0x4ed3aa62l, 0x363f7706l, 0x1bfedf72l, 0x429b023dl,
		0x37d0d724l, 0xd00a1248l, 0xdb0fead3l, 0x49f1c09bl,
		0x075372c9l, 0x80991b7bl, 0x25d479d8l, 0xf6e8def7l,
		0xe3fe501al, 0xb6794c3bl, 0x976ce0bdl, 0x04c006bal,
		0xc1a94fb6l, 0x409f60c4l, 0x5e5c9ec2l, 0x196a2463l,
		0x68fb6fafl, 0x3e6c53b5l, 0x1339b2ebl, 0x3b52ec6fl,
		0x6dfc511fl, 0x9b30952cl, 0xcc814544l, 0xaf5ebd09l,
		0xbee3d004l, 0xde334afdl, 0x660f2807l, 0x192e4bb3l,
		0xc0cba857l, 0x45c8740fl, 0xd20b5f39l, 0xb9d3fbdbl,
		0x5579c0bdl, 0x1a60320al, 0xd6a100c6l, 0x402c7279l,
		0x679f25fel, 0xfb1fa3ccl, 0x8ea5e9f8l, 0xdb3222f8l,
		0x3c7516dfl, 0xfd616b15l, 0x2f501ec8l, 0xad0552abl,
		0x323db5fal, 0xfd238760l, 0x53317b48l, 0x3e00df82l,
		0x9e5c57bbl, 0xca6f8ca0l, 0x1a87562el, 0xdf1769dbl,
		0xd542a8f6l, 0x287effc3l, 0xac6732c6l, 0x8c4f5573l,
		0x695b27b0l, 0xbbca58c8l, 0xe1ffa35dl, 0xb8f011a0l,
		0x10fa3d98l, 0xfd2183b8l, 0x4afcb56cl, 0x2dd1d35bl,
		0x9a53e479l, 0xb6f84565l, 0xd28e49bcl, 0x4bfb9790l,
		0xe1ddf2dal, 0xa4cb7e33l, 0x62fb1341l, 0xcee4c6e8l,
		0xef20cadal, 0x36774c01l, 0xd07e9efel, 0x2bf11fb4l,
		0x95dbda4dl, 0xae909198l, 0xeaad8e71l, 0x6b93d5a0l,
		0xd08ed1d0l, 0xafc725e0l, 0x8e3c5b2fl, 0x8e7594b7l,
		0x8ff6e2fbl, 0xf2122b64l, 0x8888b812l, 0x900df01cl,
		0x4fad5ea0l, 0x688fc31cl, 0xd1cff191l, 0xb3a8c1adl,
		0x2f2f2218l, 0xbe0e1777l, 0xea752dfel, 0x8b021fa1l,
		0xe5a0cc0fl, 0xb56f74e8l, 0x18acf3d6l, 0xce89e299l,
		0xb4a84fe0l, 0xfd13e0b7l, 0x7cc43b81l, 0xd2ada8d9l,
		0x165fa266l, 0x80957705l, 0x93cc7314l, 0x211a1477l,
		0xe6ad2065l, 0x77b5fa86l, 0xc75442f5l, 0xfb9d35cfl,
		0xebcdaf0cl, 0x7b3e89a0l, 0xd6411bd3l, 0xae1e7e49l,
		0x00250e2dl, 0x2071b35el, 0x226800bbl, 0x57b8e0afl,
		0x2464369bl, 0xf009b91el, 0x5563911dl, 0x59dfa6aal,
		0x78c14389l, 0xd95a537fl, 0x207d5ba2l, 0x02e5b9c5l,
		0x83260376l, 0x6295cfa9l, 0x11c81968l, 0x4e734a41l,
		0xb3472dcal, 0x7b14a94al, 0x1b510052l, 0x9a532915l,
		0xd60f573fl, 0xbc9bc6e4l, 0x2b60a476l, 0x81e67400l,
		0x08ba6fb5l, 0x571be91fl, 0xf296ec6bl, 0x2a0dd915l,
		0xb6636521l, 0xe7b9f9b6l, 0xff34052el, 0xc5855664l,
		0x53b02d5dl, 0xa99f8fa1l, 0x08ba4799l, 0x6e85076al
	},
	{
		0x4b7a70e9l, 0xb5b32944l, 0xdb75092el, 0xc4192623l,
		0xad6ea6b0l, 0x49a7df7dl, 0x9cee60b8l, 0x8fedb266l,
		0xecaa8c71l, 0x699a17ffl, 0x5664526cl, 0xc2b19ee1l,
		0x193602a5l, 0x75094c29l, 0xa0591340l, 0xe4183a3el,
		0x3f54989al, 0x5b429d65l, 0x6b8fe4d6l, 0x99f73fd6l,
		0xa1d29c07l, 0xefe830f5l, 0x4d2d38e6l, 0xf0255dc1l,
		0x4cdd2086l, 0x8470eb26l, 0x6382e9c6l, 0x021ecc5el,
		0x09686b3fl, 0x3ebaefc9l, 0x3c971814l, 0x6b6a70a1l,
		0x687f3584l, 0x52a0e286l, 0xb79c5305l, 0xaa500737l,
		0x3e07841cl, 0x7fdeae5cl, 0x8e7d44ecl, 0x5716f2b8l,
		0xb03ada37l, 0xf0500c0dl, 0xf01c1f04l, 0x0200b3ffl,
		0xae0cf51al, 0x3cb574b2l, 0x25837a58l, 0xdc0921bdl,
		0xd19113f9l, 0x7ca92ff6l, 0x94324773l, 0x22f54701l,
		0x3ae5e581l, 0x37c2dadcl, 0xc8b57634l, 0x9af3dda7l,
		0xa9446146l, 0x0fd0030el, 0xecc8c73el, 0xa4751e41l,
		0xe238cd99l, 0x3bea0e2fl, 0x3280bba1l, 0x183eb331l,
		0x4e548b38l, 0x4f6db908l, 0x6f420d03l, 0xf60a04bfl,
		0x2cb81290l, 0x24977c79l, 0x5679b072l, 0xbcaf89afl,
		0xde9a771fl, 0xd9930810l, 0xb38bae12l, 0xdccf3f2el,
		0x5512721fl, 0x2e6b7124l, 0x501adde6l, 0x9f84cd87l,
		0x7a584718l, 0x7408da17l, 0xbc9f9abcl, 0xe94b7d8cl,
		0xec7aec3al, 0xdb851dfal, 0x63094366l, 0xc464c3d2l,
		0xef1c1847l, 0x3215d908l, 0xdd433b37l, 0x24c2ba16l,
		0x12a14d43l, 0x2a65c451l, 0x50940002l, 0x133ae4ddl,
		0x71dff89el, 0x10314e55l, 0x81ac77d6l, 0x5f11199bl,
		0x043556f1l, 0xd7a3c76bl, 0x3c11183bl, 0x5924a509l,
		0xf28fe6edl, 0x97f1fbfal, 0x9ebabf2cl, 0x1e153c6el,
		0x86e34570l, 0xeae96fb1l, 0x860e5e0al, 0x5a3e2ab3l,
		0x771fe71cl, 0x4e3d06fal, 0x2965dcb9l, 0x99e71d0fl,
		0x803e89d6l, 0x5266c825l, 0x2e4cc978l, 0x9c10b36al,
		0xc6150ebal, 0x94e2ea78l, 0xa5fc3c53l, 0x1e0a2df4l,
		0xf2f74ea7l, 0x361d2b3dl, 0x1939260fl, 0x19c27960l,
		0x5223a708l, 0xf71312b6l, 0xebadfe6el, 0xeac31f66l,
		0xe3bc4595l, 0xa67bc883l, 0xb17f37d1l, 0x018cff28l,
		0xc332ddefl, 0xbe6c5aa5l, 0x65582185l, 0x68ab9802l,
		0xeecea50fl, 0xdb2f953bl, 0x2aef7dadl, 0x5b6e2f84l,
		0x1521b628l, 0x29076170l, 0xecdd4775l, 0x619f1510l,
		0x13cca830l, 0xeb61bd96l, 0x0334fe1el, 0xaa0363cfl,
		0xb5735c90l, 0x4c70a239l, 0xd59e9e0bl, 0xcbaade14l,
		0xeecc86bcl, 0x60622ca7l, 0x9cab5cabl, 0xb2f3846el,
		0x648b1eafl, 0x19bdf0cal, 0xa02369b9l, 0x655abb50l,
		0x40685a32l, 0x3c2ab4b3l, 0x319ee9d5l, 0xc021b8f7l,
		0x9b540b19l, 0x875fa099l, 0x95f7997el, 0x623d7da8l,
		0xf837889al, 0x97e32d77l, 0x11ed935fl, 0x16681281l,
		0x0e358829l, 0xc7e61fd6l, 0x96dedfa1l, 0x7858ba99l,
		0x57f584a5l, 0x1b227263l, 0x9b83c3ffl, 0x1ac24696l,
		0xcdb30aebl, 0x532e3054l, 0x8fd948e4l, 0x6dbc3128l,
		0x58ebf2efl, 0x34c6ffeal, 0xfe28ed61l, 0xee7c3c73l,
		0x5d4a14d9l, 0xe864b7e3l, 0x42105d14l, 0x203e13e0l,
		0x45eee2b6l, 0xa3aaabeal, 0xdb6c4f15l, 0xfacb4fd0l,
		0xc742f442l, 0xef6abbb5l, 0x654f3b1dl, 0x41cd2105l,
		0xd81e799el, 0x86854dc7l, 0xe44b476al, 0x3d816250l,
		0xcf62a1f2l, 0x5b8d2646l, 0xfc8883a0l, 0xc1c7b6a3l,
		0x7f1524c3l, 0x69cb7492l, 0x47848a0bl, 0x5692b285l,
		0x095bbf00l, 0xad19489dl, 0x1462b174l, 0x23820e00l,
		0x58428d2al, 0x0c55f5eal, 0x1dadf43el, 0x233f7061l,
		0x3372f092l, 0x8d937e41l, 0xd65fecf1l, 0x6c223bdbl,
		0x7cde3759l, 0xcbee7460l, 0x4085f2a7l, 0xce77326el,
		0xa6078084l, 0x19f8509el, 0xe8efd855l, 0x61d99735l,
		0xa969a7aal, 0xc50c06c2l, 0x5a04abfcl, 0x800bcadcl,
		0x9e447a2el, 0xc3453484l, 0xfdd56705l, 0x0e1e9ec9l,
		0xdb73dbd3l, 0x105588cdl, 0x675fda79l, 0xe3674340l,
		0xc5c43465l, 0x713e38d8l, 0x3d28f89el, 0xf16dff20l,
		0x153e21e7l, 0x8fb03d4al, 0xe6e39f2bl, 0xdb83adf7l
	},
	{
		0xe93d5a68l, 0x948140f7l, 0xf64c261cl, 0x94692934l,
		0x411520f7l, 0x7602d4f7l, 0xbcf46b2el, 0xd4a20068l,
		0xd4082471l, 0x3320f46al, 0x43b7d4b7l, 0x500061afl,
		0x1e39f62el, 0x97244546l, 0x14214f74l, 0xbf8b8840l,
		0x4d95fc1dl, 0x96b591afl, 0x70f4ddd3l, 0x66a02f45l,
		0xbfbc09ecl, 0x03bd9785l, 0x7fac6dd0l, 0x31cb8504l,
		0x96eb27b3l, 0x55fd3941l, 0xda2547e6l, 0xabca0a9al,
		0x28507825l, 0x530429f4l, 0x0a2c86dal, 0xe9b66dfbl,
		0x68dc1462l, 0xd7486900l, 0x680ec0a4l, 0x27a18deel,
		0x4f3ffea2l, 0xe887ad8cl, 0xb58ce006l, 0x7af4d6b6l,
		0xaace1e7cl, 0xd3375fecl, 0xce78a399l, 0x406b2a42l,
		0x20fe9e35l, 0xd9f385b9l, 0xee39d7abl, 0x3b124e8bl,
		0x1dc9faf7l, 0x4b6d1856l, 0x26a36631l, 0xeae397b2l,
		0x3a6efa74l, 0xdd5b4332l, 0x6841e7f7l, 0xca7820fbl,
		0xfb0af54el, 0xd8feb397l, 0x454056acl, 0xba489527l,
		0x55533a3al, 0x20838d87l, 0xfe6ba9b7l, 0xd096954bl,
		0x55a867bcl, 0xa1159a58l, 0xcca92963l, 0x99e1db33l,
		0xa62a4a56l, 0x3f3125f9l, 0x5ef47e1cl, 0x9029317cl,
		0xfdf8e802l, 0x04272f70l, 0x80bb155cl, 0x05282ce3l,
		0x95c11548l, 0xe4c66d22l, 0x48c1133fl, 0xc70f86dcl,
		0x07f9c9eel, 0x41041f0fl, 0x404779a4l, 0x5d886e17l,
		0x325f51ebl, 0xd59bc0d1l, 0xf2bcc18fl, 0x41113564l,
		0x257b7834l, 0x602a9c60l, 0xdff8e8a3l, 0x1f636c1bl,
		0x0e12b4c2l, 0x02e1329el, 0xaf664fd1l, 0xcad18115l,
		0x6b2395e0l, 0x333e92e1l, 0x3b240b62l, 0xeebeb922l,
		0x85b2a20el, 0xe6ba0d99l, 0xde720c8cl, 0x2da2f728l,
		0xd0127845l, 0x95b794fdl, 0x647d0862l, 0xe7ccf5f0l,
		0x5449a36fl, 0x877d48fal, 0xc39dfd27l, 0xf33e8d1el,
		0x0a476341l, 0x992eff74l, 0x3a6f6eabl, 0xf4f8fd37l,
		0xa812dc60l, 0xa1ebddf8l, 0x991be14cl, 0xdb6e6b0dl,
		0xc67b5510l, 0x6d672c37l, 0x2765d43bl, 0xdcd0e804l,
		0xf1290dc7l, 0xcc00ffa3l, 0xb5390f92l, 0x690fed0bl,
		0x667b9ffbl, 0xcedb7d9cl, 0xa091cf0bl, 0xd9155ea3l,
		0xbb132f88l, 0x515bad24l, 0x7b9479bfl, 0x763bd6ebl,
		0x37392eb3l, 0xcc115979l, 0x8026e297l, 0xf42e312dl,
		0x6842ada7l, 0xc66a2b3bl, 0x12754cccl, 0x782ef11cl,
		0x6a124237l, 0xb79251e7l, 0x06a1bbe6l, 0x4bfb6350l,
		0x1a6b1018l, 0x11caedfal, 0x3d25bdd8l, 0xe2e1c3c9l,
		0x44421659l, 0x0a121386l, 0xd90cec6el, 0xd5abea2al,
		0x64af674el, 0xda86a85fl, 0xbebfe988l, 0x64e4c3fel,
		0x9dbc8057l, 0xf0f7c086l, 0x60787bf8l, 0x6003604dl,
		0xd1fd8346l, 0xf6381fb0l, 0x7745ae04l, 0xd736fcccl,
		0x83426b33l, 0xf01eab71l, 0xb0804187l, 0x3c005e5fl,
		0x77a057bel, 0xbde8ae24l, 0x55464299l, 0xbf582e61l,
		0x4e58f48fl, 0xf2ddfda2l, 0xf474ef38l, 0x8789bdc2l,
		0x5366f9c3l, 0xc8b38e74l, 0xb475f255l, 0x46fcd9b9l,
		0x7aeb2661l, 0x8b1ddf84l, 0x846a0e79l, 0x915f95e2l,
		0x466e598el, 0x20b45770l, 0x8cd55591l, 0xc902de4cl,
		0xb90bace1l, 0xbb8205d0l, 0x11a86248l, 0x7574a99el,
		0xb77f19b6l, 0xe0a9dc09l, 0x662d09a1l, 0xc4324633l,
		0xe85a1f02l, 0x09f0be8cl, 0x4a99a025l, 0x1d6efe10l,
		0x1ab93d1dl, 0x0ba5a4dfl, 0xa186f20fl, 0x2868f169l,
		0xdcb7da83l, 0x573906fel, 0xa1e2ce9bl, 0x4fcd7f52l,
		0x50115e01l, 0xa70683fal, 0xa002b5c4l, 0x0de6d027l,
		0x9af88c27l, 0x773f8641l, 0xc3604c06l, 0x61a806b5l,
		0xf0177a28l, 0xc0f586e0l, 0x006058aal, 0x30dc7d62l,
		0x11e69ed7l, 0x2338ea63l, 0x53c2dd94l, 0xc2c21634l,
		0xbbcbee56l, 0x90bcb6del, 0xebfc7da1l, 0xce591d76l,
		0x6f05e409l, 0x4b7c0188l, 0x39720a3dl, 0x7c927c24l,
		0x86e3725fl, 0x724d9db9l, 0x1ac15bb4l, 0xd39eb8fcl,
		0xed545578l, 0x08fca5b5l, 0xd83d7cd3l, 0x4dad0fc4l,
		0x1e50ef5el, 0xb161e6f8l, 0xa28514d9l, 0x6c51133cl,
		0x6fd5c7e7l, 0x56e14ec4l, 0x362abfcel, 0xddc6c837l,
		0xd79a3234l, 0x92638212l, 0x670efa8el, 0x406000e0l
	},
	{
		0x3a39ce37l, 0xd3faf5cfl, 0xabc27737l, 0x5ac52d1bl,
		0x5cb0679el, 0x4fa33742l, 0xd3822740l, 0x99bc9bbel,
		0xd5118e9dl, 0xbf0f7315l, 0xd62d1c7el, 0xc700c47bl,
		0xb78c1b6bl, 0x21a19045l, 0xb26eb1bel, 0x6a366eb4l,
		0x5748ab2fl, 0xbc946e79l, 0xc6a376d2l, 0x6549c2c8l,
		0x530ff8eel, 0x468dde7dl, 0xd5730a1dl, 0x4cd04dc6l,
		0x2939bbdbl, 0xa9ba4650l, 0xac9526e8l, 0xbe5ee304l,
		0xa1fad5f0l, 0x6a2d519al, 0x63ef8ce2l, 0x9a86ee22l,
		0xc089c2b8l, 0x43242ef6l, 0xa51e03aal, 0x9cf2d0a4l,
		0x83c061bal, 0x9be96a4dl, 0x8fe51550l, 0xba645bd6l,
		0x2826a2f9l, 0xa73a3ae1l, 0x4ba99586l, 0xef5562e9l,
		0xc72fefd3l, 0xf752f7dal, 0x3f046f69l, 0x77fa0a59l,
		0x80e4a915l, 0x87b08601l, 0x9b09e6adl, 0x3b3ee593l,
		0xe990fd5al, 0x9e34d797l, 0x2cf0b7d9l, 0x022b8b51l,
		0x96d5ac3al, 0x017da67dl, 0xd1cf3ed6l, 0x7c7d2d28l,
		0x1f9f25cfl, 0xadf2b89bl, 0x5ad6b472l, 0x5a88f54cl,
		0xe029ac71l, 0xe019a5e6l, 0x47b0acfdl, 0xed93fa9bl,
		0xe8d3c48dl, 0x283b57ccl, 0xf8d56629l, 0x79132e28l,
		0x785f0191l, 0xed756055l, 0xf7960e44l, 0xe3d35e8cl,
		0x15056dd4l, 0x88f46dbal, 0x03a16125l, 0x0564f0bdl,
		0xc3eb9e15l, 0x3c9057a2l, 0x97271aecl, 0xa93a072al,
		0x1b3f6d9bl, 0x1e6321f5l, 0xf59c66fbl, 0x26dcf319l,
		0x7533d928l, 0xb155fdf5l, 0x03563482l, 0x8aba3cbbl,
		0x28517711l, 0xc20ad9f8l, 0xabcc5167l, 0xccad925fl,
		0x4de81751l, 0x3830dc8el, 0x379d5862l, 0x9320f991l,
		0xea7a90c2l, 0xfb3e7bcel, 0x5121ce64l, 0x774fbe32l,
		0xa8b6e37el, 0xc3293d46l, 0x48de5369l, 0x6413e680l,
		0xa2ae0810l, 0xdd6db224l, 0x69852dfdl, 0x09072166l,
		0xb39a460al, 0x6445c0ddl, 0x586cdecfl, 0x1c20c8ael,
		0x5bbef7ddl, 0x1b588d40l, 0xccd2017fl, 0x6bb4e3bbl,
		0xdda26a7el, 0x3a59ff45l, 0x3e350a44l, 0xbcb4cdd5l,
		0x72eacea8l, 0xfa6484bbl, 0x8d6612ael, 0xbf3c6f47l,
		0xd29be463l, 0x542f5d9el, 0xaec2771bl, 0xf64e6370l,
		0x740e0d8dl, 0xe75b1357l, 0xf8721671l, 0xaf537d5dl,
		0x4040cb08l, 0x4eb4e2ccl, 0x34d2466al, 0x0115af84l,
		0xe1b00428l, 0x95983a1dl, 0x06b89fb4l, 0xce6ea048l,
		0x6f3f3b82l, 0x3520ab82l, 0x011a1d4bl, 0x277227f8l,
		0x611560b1l, 0xe7933fdcl, 0xbb3a792bl, 0x344525bdl,
		0xa08839e1l, 0x51ce794bl, 0x2f32c9b7l, 0xa01fbac9l,
		0xe01cc87el, 0xbcc7d1f6l, 0xcf0111c3l, 0xa1e8aac7l,
		0x1a908749l, 0xd44fbd9al, 0xd0dadecbl, 0xd50ada38l,
		0x0339c32al, 0xc6913667l, 0x8df9317cl, 0xe0b12b4fl,
		0xf79e59b7l, 0x43f5bb3al, 0xf2d519ffl, 0x27d9459cl,
		0xbf97222cl, 0x15e6fc2al, 0x0f91fc71l, 0x9b941525l,
		0xfae59361l, 0xceb69cebl, 0xc2a86459l, 0x12baa8d1l,
		0xb6c1075el, 0xe3056a0cl, 0x10d25065l, 0xcb03a442l,
		0xe0ec6e0el, 0x1698db3bl, 0x4c98a0bel, 0x3278e964l,
		0x9f1f9532l, 0xe0d392dfl, 0xd3a0342bl, 0x8971f21el,
		0x1b0a7441l, 0x4ba3348cl, 0xc5be7120l, 0xc37632d8l,
		0xdf359f8dl, 0x9b992f2el, 0xe60b6f47l, 0x0fe3f11dl,
		0xe54cda54l, 0x1edad891l, 0xce6279cfl, 0xcd3e7e6fl,
		0x1618b166l, 0xfd2c1d05l, 0x848fd2c5l, 0xf6fb2299l,
		0xf523f357l, 0xa6327623l, 0x93a83531l, 0x56cccd02l,
		0xacf08162l, 0x5a75ebb5l, 0x6e163697l, 0x88d273ccl,
		0xde966292l, 0x81b949d0l, 0x4c50901bl, 0x71c65614l,
		0xe6c6c7bdl, 0x327a140al, 0x45e1d006l, 0xc3f27b9al,
		0xc9aa53fdl, 0x62a80f00l, 0xbb25bfe2l, 0x35bdd2f6l,
		0x71126905l, 0xb2040222l, 0xb6cbcf7cl, 0xcd769c2bl,
		0x53113ec0l, 0x1640e3d3l, 0x38abbd60l, 0x2547adf0l,
		0xba38209cl, 0xf746ce76l, 0x77afa1c5l, 0x20756060l,
		0x85cbfe4el, 0x8ae88dd8l, 0x7aaaf9b0l, 0x4cf9aa7el,
		0x1948c25cl, 0x02fb8a8cl, 0x01c36ae4l, 0xd6ebe1f9l,
		0x90d4f869l, 0xa65cdea0l, 0x3f09252dl, 0xc208e69fl,
		0xb74e6132l, 0xce77e25bl, 0x578fdfe3l, 0x3ac372e6l
	}
};

/** @internal Original P-Array (hexdigits of pi) */

static const uint32_t _BLOWFISH_P [ 18 ] =
{
	0x243f6a88l, 0x85a308d3l, 0x13198a2el,
	0x03707344l, 0xa4093822l, 0x299f31d0l,
	0x082efa98l, 0xec4e6c89l, 0x452821e6l,
	0x38d01377l, 0xbe5466cfl, 0x34e90c6cl,
	0xc0ac29b7l, 0xc97c50ddl, 0x3f84d5b5l,
	0xb5470917l, 0x9216d5d9l, 0x8979fb1bl
};

typedef struct {
	uint32_t P[BLOWFISH_NUM_ROUNDS + 2];
	uint32_t S[4][256];
} BlowfishContext;

// ------------------- F 函数 -------------------
uint32_t Blowfish_F(BlowfishContext *ctx, uint32_t x) {
	uint8_t a = (x >> 24) & 0xFF;
	uint8_t b = (x >> 16) & 0xFF;
	uint8_t c = (x >> 8) & 0xFF;
	uint8_t d = x & 0xFF;

	return ((ctx->S[0][a] + ctx->S[1][b]) ^ ctx->S[2][c]) + ctx->S[3][d];
}

// ------------------- 加密一个块 -------------------
void Blowfish_EncryptBlock(BlowfishContext *ctx, uint32_t *left, uint32_t *right) {
	uint32_t L = *left;
	uint32_t R = *right;

	for (int i = 0; i < BLOWFISH_NUM_ROUNDS; i++) {
		L ^= ctx->P[i];
		R ^= Blowfish_F(ctx, L);
		uint32_t temp = L;
		L = R;
		R = temp;
	}

	uint32_t temp = L;
	L = R;
	R = temp;

	R ^= ctx->P[BLOWFISH_NUM_ROUNDS];
	L ^= ctx->P[BLOWFISH_NUM_ROUNDS + 1];

	*left = L;
	*right = R;
}

// ------------------- 解密一个块 -------------------
void Blowfish_DecryptBlock(BlowfishContext *ctx, uint32_t *left, uint32_t *right) {
	uint32_t L = *left;
	uint32_t R = *right;

	for (int i = BLOWFISH_NUM_ROUNDS + 1; i > 1; i--) {
		L ^= ctx->P[i];
		R ^= Blowfish_F(ctx, L);
		uint32_t temp = L;
		L = R;
		R = temp;
	}

	uint32_t temp = L;
	L = R;
	R = temp;

	R ^= ctx->P[1];
	L ^= ctx->P[0];

	*left = L;
	*right = R;
}

// ------------------- 初始化密钥 -------------------
void Blowfish_KeyExpansion(BlowfishContext *ctx, const uint8_t *key, int keyLen) {
	// Step 1: 初始化 P 和 S（你需要补全初始值）
	// 示例（你可以从 Blowfish 官方标准表里复制过来）

	// ctx->P[0] = 0x243F6A88;
	// ctx->S[0][0] = 0xD1310BA6;
	// ...
	memcpy(ctx->P, _BLOWFISH_P, sizeof(_BLOWFISH_P));
	memcpy(ctx->S, _BLOWFISH_SBox, sizeof(_BLOWFISH_SBox));

	// Step 2: 将密钥 XOR 到 P 盒中
	int j = 0;
	for (int i = 0; i < BLOWFISH_NUM_ROUNDS + 2; i++) {
		uint32_t data = 0;
		for (int k = 0; k < 4; k++) {
			data = (data << 8) | key[j];
			j = (j + 1) % keyLen;
		}
		ctx->P[i] ^= data;
	}

	// Step 3: 生成最终的 P/S 盒内容
	uint32_t L = 0, R = 0;
	for (int i = 0; i < BLOWFISH_NUM_ROUNDS + 2; i += 2) {
		Blowfish_EncryptBlock(ctx, &L, &R);
		ctx->P[i] = L;
		ctx->P[i + 1] = R;
	}
	for (int i = 0; i < 4; i++) {
		for (int j = 0; j < 256; j += 2) {
			Blowfish_EncryptBlock(ctx, &L, &R);
			ctx->S[i][j] = L;
			ctx->S[i][j + 1] = R;
		}
	}
}

void Blowfish_Encrypttxt_ECB(BlowfishContext *ctx, const uint8_t *input, uint8_t *output, size_t length) {
	size_t blocks = (length + 7) / 8;

	for (size_t i = 0; i < blocks; ++i) {
		uint32_t left = 0, right = 0;

		// 拆分成两个 32 位整数
		for (int j = 0; j < 4; j++) {
			size_t idx = i * 8 + j;
			left = (left << 8) | (idx < length ? input[idx] : 0);  // padding with 0
		}
		for (int j = 4; j < 8; j++) {
			size_t idx = i * 8 + j;
			right = (right << 8) | (idx < length ? input[idx] : 0);
		}

		Blowfish_EncryptBlock(ctx, &left, &right);

		// 写回加密结果
		output[i * 8 + 0] = (left >> 24) & 0xFF;
		output[i * 8 + 1] = (left >> 16) & 0xFF;
		output[i * 8 + 2] = (left >> 8) & 0xFF;
		output[i * 8 + 3] = left & 0xFF;
		output[i * 8 + 4] = (right >> 24) & 0xFF;
		output[i * 8 + 5] = (right >> 16) & 0xFF;
		output[i * 8 + 6] = (right >> 8) & 0xFF;
		output[i * 8 + 7] = right & 0xFF;
	}
}

void Blowfish_Decrypttxt_ECB(BlowfishContext *ctx, const uint8_t *input, uint8_t *output, size_t length) {
	if (length % 8 != 0) {
		fprintf(stderr, "Length must be multiple of 8 for ECB decryption\n");
		return;
	}

	for (size_t i = 0; i < length; i += 8) {
		uint32_t left = 0, right = 0;

		left |= input[i + 0] << 24;
		left |= input[i + 1] << 16;
		left |= input[i + 2] << 8;
		left |= input[i + 3];

		right |= input[i + 4] << 24;
		right |= input[i + 5] << 16;
		right |= input[i + 6] << 8;
		right |= input[i + 7];

		Blowfish_DecryptBlock(ctx, &left, &right);

		output[i + 0] = (left >> 24) & 0xFF;
		output[i + 1] = (left >> 16) & 0xFF;
		output[i + 2] = (left >> 8) & 0xFF;
		output[i + 3] = left & 0xFF;
		output[i + 4] = (right >> 24) & 0xFF;
		output[i + 5] = (right >> 16) & 0xFF;
		output[i + 6] = (right >> 8) & 0xFF;
		output[i + 7] = right & 0xFF;
	}
}

int Blowfish_EncryptFile_ECB(BlowfishContext *ctx, const char *inFile, const char *outFile) {
	clock_t start_time = clock();  // 🔸 开始时间
	FILE *fin = fopen(inFile, "rb");
	FILE *fout = fopen(outFile, "wb");

	if (!fin || !fout) {
		perror("fopen");
		return -1;
	}

	// 获取原始文件长度
	fseek(fin, 0, SEEK_END);
	uint64_t orig_len = ftell(fin);
	fseek(fin, 0, SEEK_SET);

	// 写入 8 字节原始长度（小端）
	fwrite(&orig_len, sizeof(uint64_t), 1, fout);

	// 分块读取并加密
	uint8_t buffer[8] = {0};
	size_t read;

	while ((read = fread(buffer, 1, 8, fin)) > 0) {
		// 补0（仅最后一块可能 < 8）
		if (read < 8) {
			memset(buffer + read, 0, 8 - read);
		}

		uint32_t left = (buffer[0] << 24) | (buffer[1] << 16) | (buffer[2] << 8) | buffer[3];
		uint32_t right = (buffer[4] << 24) | (buffer[5] << 16) | (buffer[6] << 8) | buffer[7];

		Blowfish_EncryptBlock(ctx, &left, &right);

		uint8_t outbuf[8] = {
			(left >> 24) & 0xFF, (left >> 16) & 0xFF, (left >> 8) & 0xFF, left & 0xFF,
			(right >> 24) & 0xFF, (right >> 16) & 0xFF, (right >> 8) & 0xFF, right & 0xFF
		};

		fwrite(outbuf, 1, 8, fout);
	}

	fclose(fin);
	fclose(fout);
	clock_t end_time = clock();  // 🔸 结束时间

	double duration = (double)(end_time - start_time) * 1000.0 / CLOCKS_PER_SEC;
	printf("加密耗时：%.2f 毫秒\n", duration);
	return 0;
}

int Blowfish_DecryptFile_ECB(BlowfishContext *ctx, const char *inFile, const char *outFile) {
	clock_t start_time = clock();  // 🔸 开始时间
	FILE *fin = fopen(inFile, "rb");
	FILE *fout = fopen(outFile, "wb");

	if (!fin || !fout) {
		perror("fopen");
		return -1;
	}

	// 读取原始长度
	uint64_t orig_len = 0;
	fread(&orig_len, sizeof(uint64_t), 1, fin);

	uint8_t buffer[8];
	size_t total_written = 0;

	while (fread(buffer, 1, 8, fin) == 8) {
		uint32_t left = (buffer[0] << 24) | (buffer[1] << 16) | (buffer[2] << 8) | buffer[3];
		uint32_t right = (buffer[4] << 24) | (buffer[5] << 16) | (buffer[6] << 8) | buffer[7];

		Blowfish_DecryptBlock(ctx, &left, &right);

		uint8_t outbuf[8] = {
			(left >> 24) & 0xFF, (left >> 16) & 0xFF, (left >> 8) & 0xFF, left & 0xFF,
			(right >> 24) & 0xFF, (right >> 16) & 0xFF, (right >> 8) & 0xFF, right & 0xFF
		};

		// 控制只写原始长度的数据（处理最后一块 padding）
		size_t to_write = (orig_len - total_written >= 8) ? 8 : (orig_len - total_written);
		fwrite(outbuf, 1, to_write, fout);
		total_written += to_write;
	}

	fclose(fin);
	fclose(fout);

	clock_t end_time = clock();  // 🔸 结束时间

	double duration = (double)(end_time - start_time) * 1000.0 / CLOCKS_PER_SEC;
	printf("解密耗时：%.2f 毫秒\n", duration);
	return 0;
}

void Blowfish_Encrypttxt_CBC(BlowfishContext *ctx, const uint8_t *input, uint8_t *output, size_t length, const uint8_t *iv) {
	uint32_t left, right;
	uint8_t previous_block[8];
	memcpy(previous_block, iv, 8);  // 初始化IV

	for (size_t i = 0; i < length; i += 8) {
		// 拆分明文块
		left = 0;
		right = 0;

		for (int j = 0; j < 4; j++) {
			left = (left << 8) | (i + j < length ? input[i + j] : 0);
		}
		for (int j = 4; j < 8; j++) {
			right = (right << 8) | (i + j < length ? input[i + j] : 0);
		}

		// CBC：当前明文块与上一个密文块（或IV）异或
		left ^= (previous_block[0] << 24) | (previous_block[1] << 16) | (previous_block[2] << 8) | previous_block[3];
		right ^= (previous_block[4] << 24) | (previous_block[5] << 16) | (previous_block[6] << 8) | previous_block[7];

		// 加密
		Blowfish_EncryptBlock(ctx, &left, &right);

		// 保存密文块并更新IV（当前密文块）
		for (int j = 0; j < 4; j++) {
			output[i + j] = (left >> (24 - j * 8)) & 0xFF;
		}
		for (int j = 4; j < 8; j++) {
			output[i + j] = (right >> (24 - (j - 4) * 8)) & 0xFF;
		}

		memcpy(previous_block, &output[i], 8);  // 更新IV为当前密文块
	}
}

void Blowfish_Decrypttxt_CBC(BlowfishContext *ctx, const uint8_t *input, uint8_t *output, size_t length, const uint8_t *iv) {
	uint32_t left, right;
	uint8_t previous_block[8];
	memcpy(previous_block, iv, 8);  // 初始化IV

	for (size_t i = 0; i < length; i += 8) {
		// 拆分密文块
		left = 0;
		right = 0;

		for (int j = 0; j < 4; j++) {
			left = (left << 8) | input[i + j];
		}
		for (int j = 4; j < 8; j++) {
			right = (right << 8) | input[i + j];
		}

		// 解密
		Blowfish_DecryptBlock(ctx, &left, &right);

		// CBC：解密后与上一个密文块（或IV）异或
		left ^= (previous_block[0] << 24) | (previous_block[1] << 16) | (previous_block[2] << 8) | previous_block[3];
		right ^= (previous_block[4] << 24) | (previous_block[5] << 16) | (previous_block[6] << 8) | previous_block[7];

		// 保存解密后的明文
		for (int j = 0; j < 4; j++) {
			output[i + j] = (left >> (24 - j * 8)) & 0xFF;
		}
		for (int j = 4; j < 8; j++) {
			output[i + j] = (right >> (24 - (j - 4) * 8)) & 0xFF;
		}

		memcpy(previous_block, &input[i], 8);  // 更新IV为当前密文块
	}
}

int Blowfish_EncryptFile_CBC(BlowfishContext *ctx, const char *inFile, const char *outFile, const uint8_t *iv) {
	clock_t start_time = clock();  // 🔸 开始时间
	FILE *fin = fopen(inFile, "rb");
	FILE *fout = fopen(outFile, "wb");

	if (!fin || !fout) {
		perror("fopen");
		return -1;
	}

	// 获取原始文件长度
	fseek(fin, 0, SEEK_END);
	uint64_t orig_len = ftell(fin);
	fseek(fin, 0, SEEK_SET);

	// 写入 8 字节原始长度（小端）
	fwrite(&orig_len, sizeof(uint64_t), 1, fout);

	// 初始化 IV
	uint8_t buffer[8] = {0};
	size_t read;
	uint8_t prev_block[8];
	memcpy(prev_block, iv, 8);  // 初始向量

	while ((read = fread(buffer, 1, 8, fin)) > 0) {
		// 补0（仅最后一块可能 < 8）
		if (read < 8) {
			memset(buffer + read, 0, 8 - read);
		}

		// CBC模式：先将块与前一个块（或IV）异或
		for (int i = 0; i < 8; i++) {
			buffer[i] ^= prev_block[i];
		}

		uint32_t left = (buffer[0] << 24) | (buffer[1] << 16) | (buffer[2] << 8) | buffer[3];
		uint32_t right = (buffer[4] << 24) | (buffer[5] << 16) | (buffer[6] << 8) | buffer[7];

		Blowfish_EncryptBlock(ctx, &left, &right);

		uint8_t outbuf[8] = {
			(left >> 24) & 0xFF, (left >> 16) & 0xFF, (left >> 8) & 0xFF, left & 0xFF,
			(right >> 24) & 0xFF, (right >> 16) & 0xFF, (right >> 8) & 0xFF, right & 0xFF
		};

		fwrite(outbuf, 1, 8, fout);
		memcpy(prev_block, outbuf, 8);  // 更新前一个块为当前加密块
	}

	fclose(fin);
	fclose(fout);

	clock_t end_time = clock();  // 🔸 结束时间

	double duration = (double)(end_time - start_time) * 1000.0 / CLOCKS_PER_SEC;
	printf("加密耗时：%.2f 毫秒\n", duration);
	return 0;
}

int Blowfish_DecryptFile_CBC(BlowfishContext *ctx, const char *inFile, const char *outFile, const uint8_t *iv) {
	clock_t start_time = clock();  // 🔸 开始时间
	FILE *fin = fopen(inFile, "rb");
	FILE *fout = fopen(outFile, "wb");

	if (!fin || !fout) {
		perror("fopen");
		return -1;
	}

	// 读取原始长度
	uint64_t orig_len = 0;
	fread(&orig_len, sizeof(uint64_t), 1, fin);

	uint8_t buffer[8];
	size_t total_written = 0;
	uint8_t prev_block[8];
	memcpy(prev_block, iv, 8);  // 初始向量

	while (fread(buffer, 1, 8, fin) == 8) {
		uint32_t left = (buffer[0] << 24) | (buffer[1] << 16) | (buffer[2] << 8) | buffer[3];
		uint32_t right = (buffer[4] << 24) | (buffer[5] << 16) | (buffer[6] << 8) | buffer[7];

		Blowfish_DecryptBlock(ctx, &left, &right);

		// 解密后与前一个块（或IV）异或
		uint8_t outbuf[8] = {
			(left >> 24) & 0xFF, (left >> 16) & 0xFF, (left >> 8) & 0xFF, left & 0xFF,
			(right >> 24) & 0xFF, (right >> 16) & 0xFF, (right >> 8) & 0xFF, right & 0xFF
		};

		for (int i = 0; i < 8; i++) {
			outbuf[i] ^= prev_block[i];
		}

		// 控制只写原始长度的数据（处理最后一块 padding）
		size_t to_write = (orig_len - total_written >= 8) ? 8 : (orig_len - total_written);
		fwrite(outbuf, 1, to_write, fout);
		total_written += to_write;

		memcpy(prev_block, buffer, 8);  // 更新前一个块为当前加密块
	}

	fclose(fin);
	fclose(fout);

	clock_t end_time = clock();  // 🔸 结束时间

	double duration = (double)(end_time - start_time) * 1000.0 / CLOCKS_PER_SEC;
	printf("解密耗时：%.2f 毫秒\n", duration);
	return 0;
}

int main() {

	//ECB模式
	BlowfishContext ctx;
	const uint8_t key[] = "secretkey";
	Blowfish_KeyExpansion(&ctx, key, strlen((char *)key));

	const char *plaintext = "Hello Blowfish!";
	size_t len = strlen(plaintext);
	size_t paddedLen = ((len + 7) / 8) * 8;
	//填充
	uint8_t *encrypted = calloc(paddedLen, 1);
	uint8_t *decrypted = calloc(paddedLen + 1, 1); // +1 for null terminator

	Blowfish_Encrypttxt_ECB(&ctx, (const uint8_t *)plaintext, encrypted, len);
	for(int i=0;i<len;i++){
		printf("0x%x,",encrypted[i]);
	}
	printf("\n");
	Blowfish_Decrypttxt_ECB(&ctx, encrypted, decrypted, paddedLen);

	printf("Original : %s\n", plaintext);
	printf("Decrypted: %s\n", decrypted);

	free(encrypted);
	free(decrypted);

	//CBC模式
	/*
	BlowfishContext ctx;
	const uint8_t key[] = "my_secret_key";
	Blowfish_KeyExpansion(&ctx, key, strlen((char *)key));

	const uint8_t iv[] = {0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07};  // 使用一个固定的 IV 示例
	const uint8_t input[] = "This is a test message for Blowfish CBC mode.";
	size_t input_len = strlen((char *)input);
	uint8_t encrypted[128] = {0};
	uint8_t decrypted[128] = {0};

	// 加密
	Blowfish_Encrypttxt_CBC(&ctx, input, encrypted, input_len, iv);

	printf("Encrypted: ");
	for (size_t i = 0; i < input_len; i++) {
		printf("0x%02X,", encrypted[i]);
	}
	printf("\n");

	// 解密
	Blowfish_Decrypttxt_CBC(&ctx, encrypted, decrypted, input_len, iv);

	printf("Decrypted: %s\n", decrypted);
	*/
	return 0;
}

```

### 总结

从上面的流程可以看出，blowfish在生成最终的P_box和S-box需要耗费一定的时间，但是一旦生成完毕，或者说密钥不变的情况下，blowfish还是很快速的一种分组加密方法。

> 每个新的密钥都需要进行大概4 KB文本的预处理，和其他分组密码算法相比，这个会很慢
>
> 那么慢有没有好处呢？
>
> 当然有，因为对于一个正常应用来说，是不会经常更换密钥的。所以预处理只会生成一次。在后面使用的时候就会很快了。
>
> 而对于恶意攻击者来说，每次尝试新的密钥都需要进行漫长的预处理，所以对攻击者来说要破解blowfish算法是非常不划算的。所以blowfish是可以抵御字典攻击的。

因此，Blowfish算法不适合频繁变更密钥的应用场景

此外，随着时间推移，64 位分组长度已被认为不安全，它容易受到`生日攻击`​，特别是在HTTPS这样的大数据量加密场景中

贴一个GPT的解释

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250426004723540.png)

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250426004744663.png)

生日攻击利用概率特性，在输出空间中找到碰撞所需的尝试次数远低于空间大小，因此**64 位加密分组长度在大数据量下很容易出现碰撞风险**，被认为不再安全。

### 参考资料

[Blowfish加密算法详解-CSDN博客](https://blog.csdn.net/qq_40279192/article/details/107600004)

[BlowFish加解密原理与代码实现 - iBinary - 博客园](https://www.cnblogs.com/iBinary/p/14883752.html)

[Blowfish - 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/Blowfish)

[密码学系列之:blowfish对称密钥分组算法 - 知乎](https://zhuanlan.zhihu.com/p/382470653)

[blowfish-api/blowfish.c at master · tombonner/blowfish-api](https://github.com/tombonner/blowfish-api/blob/master/blowfish.c)

‍
