---
title: "TEA家族算法"
description: "之前都是面向CTF-reverse学的TEA算法，属于是只知道找算法特征然后无脑一把梭，现在来好好学习总结一下"
pubDate: "September 28 2024"
image: /images/image4.jpg
categories:
  - tech
tags:
  - 加密算法
---

## TEA

引用百度百科的介绍：

[TEA算法](https://baike.baidu.com/item/TEA算法/10167844?fromModule=lemma_inlink)由剑桥大学计算机实验室的David Wheeler和Roger Needham于1994年发明。它是一种分组[密码算法](https://baike.baidu.com/item/密码算法/231826?fromModule=lemma_inlink)，其明文密文块为**64位（8字节）**，[密钥](https://baike.baidu.com/item/密钥/101144?fromModule=lemma_inlink)长度为**128位（16字节）**。TEA算法利用不断增加的**Delta**([黄金分割率](https://baike.baidu.com/item/黄金分割率/953816?fromModule=lemma_inlink))值作为变化，使得每轮的加密是不同，该加密算法的迭代次数可以改变，建议的迭代次数为32轮。

TEA 采用与 DES 算法类似的 **Feistel** 结构，迭代的每次循环使用加法和移位操作，对明文和密钥进行扩散和混乱，实现明文的**非线性变换**。TEA 密钥长度和迭代次数都是 DES 的两倍，抗“试错法”攻击的强度不低于 DES 算法。算法以**32bits** 的字为运算单位，而不是耗费计算能力的逐位运算。算法没有采用 DES 那样的转换矩阵，它安全、高效、占用存储空间少，非常适合在嵌入式系统中应用, 据说 QQ 就是使用 16 轮迭代的 TEA 算法。

简单来说，就是又轻量又快

### 加密原理

原理图如下（k[]就是密钥key数组）

![img](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202411281706433.png)

加密时，先把明文块（64位）分成两个32位的无符号整数

**1.**将其中一个无符号整数(设为v0)作为输入端生成对另一个无符号整数(设为v1)进行加密的内容，该过程包括：

- (v0<<4)+k[0]
- sum+=Delta; v0+sum（sum初始值为0）
- (v0>>5)+k[1]

以上计算得到的3个值相互异或。将异或后的值与v1相加，相加后的值给v1

用代码表示如下

```c
v1+=((v0<<4)+k[0])^(v0+sum)^((v0>>5)+k[1]);
```

**2.**将得到的v1作为下一次加密的输入端，生成对v0进行加密的内容，该过程包括：

- (v1<<4)+k[2]
- sum+=Delta;v1+sum
- (v1>>5)+k[3]

同样地，以上计算得到的3个值相互异或。将异或后的值与v0相加，相加后的值给v0

```c
v0+=((v1<<4)+k[2])^(v1+sum)^((v1>>5)+k[3]);
```

依次类推，回到步骤1，直到进行32轮加密

完整加密代码如下：

```c
void encrypt (uint32_t *v,uint32_t *k ){
	uint32_t v0=v[0],v1=v[1],sum=0,i;
	uint32_t delta=0x9e3779b9;//（通常默认delta的值为0x9e3779b9，但它是多少其实不重要）
	uint32_t k0=k[0],k1=k[1],k2=k[2],k3=k[3];
	for(i=0;i<32;i++){
		sum+=delta;
		v0+=((v1<<4)+k0)^(v1+sum)^((v1>>5)+k1);
		v1+=((v0<<4)+k2)^(v0+sum)^((v0>>5)+k3);
	}
	v[0]=v0;v[1]=v1;
}
```

### 解密原理

由于异或的可逆性，解密过程就是把加密的顺序颠倒一下，+变成-就行

重点是将加密的

```c
sum+=delta;
v0+=((v1<<4)+k0)^(v1+sum)^((v1>>5)+k1);
v1+=((v0<<4)+k2)^(v0+sum)^((v0>>5)+k3);
```

颠倒成

```c
v1-=((v0<<4)+k2)^(v0+sum)^((v0>>5)+k3);
v0-=((v1<<4)+k0)^(v1+sum)^((v1>>5)+k1);
sum-=delta;
```

完整解密代码如下

```c

void decrypt (uint32_t *v,uint32_t *k){
	uint32_t v0=v[0],v1=v[1],sum=0xC6EF3720,i;//这里的sum是0x9e3779b9*32后截取32位的结果，截取很重要。
	uint32_t delta=0x9e3779b9;
	uint32_t k0=k[0],k1=k[1],k2=k[2],k3=k[3];
	for (i=0;i<32;i++){
		v1-=((v0<<4)+k2)^(v0+sum)^((v0>>5)+k3);
		v0-=((v1<<4)+k0)^(v1+sum)^((v1>>5)+k1);
		sum-=delta;
	}
	v[0]=v0;v[1]=v1;
```

main举例

```c

int main()
{
	uint32_t v[2]={1,2},k[4]={2,2,3,4};
	printf("加密前的数据：%u %u\n",v[0],v[1]);	//%u 以十进制形式输出无符号整数
	encrypt(v,k);
	printf("加密后数据：%u %u\n",v[0],v[1]);
	decrypt(v,k);
	printf("解密后数据：%u %u\n",v[0],v[1]);
	return 0;
}
```

### 实战

以极客大挑战的DH爱喝茶为例（只关注加密部分）

```c
unsigned int __cdecl enc(unsigned int *a1, int *a2)
{
  unsigned int result; // eax
  unsigned int v3; // [esp+8h] [ebp-28h]
  unsigned int v4; // [esp+Ch] [ebp-24h]
  int v5; // [esp+10h] [ebp-20h]
  unsigned int i; // [esp+14h] [ebp-1Ch]
  int v7; // [esp+1Ch] [ebp-14h]
  int v8; // [esp+20h] [ebp-10h]

  v3 = *a1;
  v4 = a1[1];
  v5 = 0;
  v7 = *a2;
  v8 = a2[1];
  for ( i = 0; i <= 0x1F; ++i )
  {
    v5 += (unsigned __int8)(v8 ^ v7) - 0x6789ABCE;
    v3 += (v4 + v5) ^ (16 * v4 + v7) ^ ((v4 >> 5) + v8);
    v4 += (v3 + v5) ^ (16 * v3 + a2[2]) ^ ((v3 >> 5) + a2[3]);
  }
  *a1 = v3;
  result = v4;
  a1[1] = v4;
  return result;
}

//调用部分
v6[0] = 0x56789ABC;
v6[1] = 0x6789ABCD;
v6[2] = 0x789ABCDE;
v6[3] = 0x89ABCDEF;
puts("plz treat DH with a cup of tea:");
fgets(s, 33, stdin);
s[strcspn(s, "\n")] = 0;
for ( i = 0; i <= 3; ++i )
{
  	v6[i] = __ROL4__(v6[i], 6);//对v6[i]左循环位移6位，相当于右移2位
    enc((unsigned int *)&s[8 * i], v6);//魔改TEA加密内容
}
```

关注enc函数，可以发现它在原TEA的基础上做了一些魔改，但不影响解密，仍然是+变成-，-变成+，异或还是异或，Delta它是多少就是多少，这个不重要

核心代码如下

```c
int v5[] = {0xecaa140, 0xeca8fc0, 0xeca8fc0, 0xeca8fc0};//外层有4次调用enc，每次调用后最后得到的v5不一样

for (i = 0; i <= 0x1F; ++i)
{
	v4 -= (v3 + v5) ^ (16 * v3 + a2[2]) ^ ((v3 >> 5) + a2[3]);
	v3 -= (v4 + v5) ^ (16 * v4 + v7) ^ ((v4 >> 5) + v8);
	v5 -= (unsigned __int8)(v8 ^ v7) - 0x6789ABCE;
}
```

## XTEA

之后 TEA 算法被发现存在缺陷，作为回应，设计者提出了一个 TEA 的升级版本——**XTEA**（有时也被称为"tean"）

**XTEA**是TEA的升级版，也被称作为Corrected Block TEA，它增加了更多的**密钥表，移位和异或操作**等等，设计者是Roger Needham, David Wheeler

XTEA 跟 TEA 使用了**相同的简单运算**，但它采用了截然不同的顺序，为了阻止密钥表攻击，四个子密钥（在加密过程中，原 128 位的密钥被拆分为 4 个 32 位的子密钥）采用了一种不太正规的方式进行混合，但速度**更慢了**。

![img](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202411281744628.png)

加密算法如下：

```c
#include<stdio.h>
#include<stdint.h>

void XTEA_encipher(unsigned int num_rounds, uint32_t v[2], uint32_t const key[4]){
	unsigned int i;
	uint32_t v0=v[0],v1=v[1],sum=0,delta=0x9E3779B9;
	for(i=0;i<num_rounds;i++){
		v0+=(((v1<<4)^(v1>>5))+v1)^(sum+key[sum&3]);
		sum+=delta;
		v1+=(((v0<<4)^(v0>>5))+v0)^(sum+key[(sum>>11)&3]);
	}
	v[0]=v0;v[1]=v1;
}

void XTEA_decipher(unsigned int num_rounds,uint32_t v[2],uint32_t const key[4]){
	unsigned int i;
	uint32_t v0=v[0],v1=v[1],delta=0x9E3779B9,sum=delta*num_rounds;
	for(i=0;i<num_rounds;i++){
	v1-=(((v0<<4)^(v0>>5))+v0)^(sum+key[(sum>>11)&3]);
	sum-=delta;
	v0-=(((v1<<4)^(v1>>5))+v1)^(sum+key[sum&3]);
	}
	v[0]=v0;v[1]=v1;
}
```

```py
from ctypes import *
def encrypt(v,k):
	v0=c_uint32(v[0])
	v1=c_uint32(v[1])
	sum1=c_uint32(0)
	delta=0x9e3779b9
	for i in range(32):
		v0.value+=(((v1.value<<4)^(v1.value>>5))+v1.value)^(sum1.value+k[sum1.value&3])
		sum1.value+=delta
		v1.value+=(((v0.value<<4)^(v0.value>>5))+v0.value)^(sum1.value+k[(sum1.value>>11)&3])
	return v0.value,v1.value

def decrypt(v,k):
	v0=c_uint32(v[0])
	v1=c_uint32(v[1])
	delta=0x9e3779b9
	sum1=c_uint32(delta*32)
	for i in range(32):
		v1.value-=(((v0.value<<4)^(v0.value>>5))+v0.value)^(sum1.value+k[(sum1.value>>11)&3])
		sum1.value-=delta
		v0.value-=(((v1.value<<4)^(v1.value>>5))+v1.value)^(sum1.value+k[sum1.value&3])
	return v0.value,v1.value

if __name__=='__main__':
	a=[1,2]
	k=[2,2,3,4]
	print("加密前数据:",a)
	res=encrypt(a,k)
	print("加密后的数据:",res)
	res=decrypt(res,k)
	print("解密后数据:",res)
```

## XXTEA

XXTEA，又称Corrected Block TEA，是XTEA的升级版 ，设计者是Roger Needham, David Wheeler。

特点：原字符串长度（明文）可以不是4的倍数了，是目前TEA系列中最安全的算法，但性能较上两种有所降低。

加密流程：

![img](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202411281748689.jpeg)

示例代码：

```c
#include <stdio.h>
#include <stdint.h>
#define DELTA 0x9e3779b9
#define MX (((z>>5^y<<2) + (y>>3^z<<4)) ^ ((sum^y) + (key[(p&3)^e] ^ z)))

void btea(uint32_t *v, int n, uint32_t const key[4])
{
    uint32_t y, z, sum;
    unsigned p, rounds, e;
    if (n > 1)            /* Coding Part */
    {
        rounds = 6 + 52/n;
        sum = 0;
        z = v[n-1];
        do
        {
            sum += DELTA;
            e = (sum >> 2) & 3;
            for (p=0; p<n-1; p++)
            {
                y = v[p+1];
                z = v[p] += MX;
            }
            y = v[0];
            z = v[n-1] += MX;
        }
        while (--rounds);
    }
    else if (n < -1)      /* Decoding Part */
    {
        n = -n;
        rounds = 6 + 52/n;
        sum = rounds*DELTA;
        y = v[0];
        do
        {
            e = (sum >> 2) & 3;
            for (p=n-1; p>0; p--)
            {
                z = v[p-1];
                y = v[p] -= MX;
            }
            z = v[n-1];
            y = v[0] -= MX;
            sum -= DELTA;
        }
        while (--rounds);
    }
}

int main()
{
    uint32_t v[2]= {1,2};
    uint32_t const k[4]= {2,2,3,4};
    int n= 2; //n的绝对值表示v的长度，取正表示加密，取负表示解密
    // v为要加密的数据是两个32位无符号整数
    // k为加密解密密钥，为4个32位无符号整数，即密钥长度为128位
    printf("加密前原始数据：%u %u\n",v[0],v[1]);
    btea(v, n, k);
    printf("加密后的数据：%u %u\n",v[0],v[1]);
    btea(v, -n, k);
    printf("解密后的数据：%u %u\n",v[0],v[1]);
    return 0;
}
```

单独的加密代码：

```c

void XXTEA_encrypto(unsigned int v[8], unsigned int key[4]) {
    unsigned int sum = 0;
    unsigned int y, z, p, rounds, e;
    int n = 8; // 数组v的长度
    rounds = 6 + 52 / n;
    y = v[0];
    sum = 0; // 加密时从0开始加delta

    while (rounds--) {
        sum += delta;
        e = sum >> 2 & 3;
        for (p = n - 1; p > 0; p--) {
            z = v[p - 1];
            y = v[p] = v[p] + ((((z >> 5) ^ (y << 2)) + ((y >> 3) ^ (z << 4))) ^
                              ((key[(p & 3) ^ e] ^ z) + (y ^ sum)));
        }
        z = v[n - 1];
        y = v[0] = v[0] + (((key[(p ^ e) & 3] ^ z) + (y ^ sum)) ^
                          (((y << 2) ^ (z >> 5)) + ((z << 4) ^ (y >> 3))));
    }
}
```

另外的解密代码：

```c
#include <stdio.h>
#include <stdlib.h>
#define delta 0x9e3779b9

void XXTEA_decrypto(unsigned int v[8], unsigned int key[4]) {
    unsigned int sum = 0;
    unsigned int y, z, p, rounds, e;
    int n = 8; // 数组v的长度
    rounds = 6 + 52 / n;
    y = v[0];
    sum = rounds * delta;
    do {
        e = sum >> 2 & 3;
        for (p = n - 1; p > 0; p--) {
            z = v[p - 1];
            v[p] -= ((((z >> 5) ^ (y << 2)) + ((y >> 3) ^ (z << 4))) ^
                     ((key[(p & 3) ^ e] ^ z) + (y ^ sum)));
            y = v[p];
        }
        z = v[n - 1];
        v[0] -= (((key[(p ^ e) & 3] ^ z) + (y ^ sum)) ^
                 (((y << 2) ^ (z >> 5)) + ((z << 4) ^ (y >> 3))));
        y = v[0];
        sum -= delta;
    } while (--rounds);
    // 打印结果（如果需要）
    for (int i = 0; i < n; i++) {
        printf("%c%c%c%c", *((char *)&v[i] + 0), *((char *)&v[i] + 1),
               *((char *)&v[i] + 2), *((char *)&v[i] + 3));
        // 或者使用下面这行来以不同的字节顺序打印
        // printf("%c%c%c%c", *((char *)&v[i] + 3), *((char *)&v[i] + 2), *((char *)&v[i] + 1), *((char *)&v[i] + 0));
    }
}
```

python代码示例

```python
import struct

_DELTA = 0x9E3779B9

def _long2str(v, w):
    n = (len(v) - 1) << 2
    if w:
        m = v[-1]
        if (m < n - 3) or (m > n): return b''
        n = m
    s = struct.pack('<%iL' % len(v), *v)
    return s[:n] if w else s

def _str2long(s, w):
    n = len(s)
    m = (4 - (n & 3) & 3) + n
    s = s.ljust(m, b"\0")  # 确保是 bytes 类型
    v = list(struct.unpack('<%iL' % (m >> 2), s))
    if w: v.append(n)
    return v

def encrypt(data, key):
    if not data: return data
    v = _str2long(data, True)
    k = _str2long(key.ljust(16, b"\0"), False)
    n = len(v) - 1
    z = v[n]
    y = v[0]
    sum = 0
    q = 6 + 52 // (n + 1)
    while q > 0:
        sum = (sum + _DELTA) & 0xffffffff
        e = sum >> 2 & 3
        for p in range(n):
            y = v[p + 1]
            v[p] = (v[p] + ((z >> 5 ^ y << 2) + (y >> 3 ^ z << 4) ^ (sum ^ y) + (k[p & 3 ^ e] ^ z))) & 0xffffffff
            z = v[p]
        y = v[0]
        v[n] = (v[n] + ((z >> 5 ^ y << 2) + (y >> 3 ^ z << 4) ^ (sum ^ y) + (k[n & 3 ^ e] ^ z))) & 0xffffffff
        z = v[n]
        q -= 1
    return _long2str(v, False)

def decrypt(data, key):
    if not data: return data
    v = _str2long(data, False)
    k = _str2long(key.ljust(16, b"\0"), False)
    n = len(v) - 1
    z = v[n]
    y = v[0]
    q = 6 + 52 // (n + 1)
    sum = (q * _DELTA) & 0xffffffff
    while sum != 0:
        e = sum >> 2 & 3
        for p in range(n, 0, -1):
            z = v[p - 1]
            v[p] = (v[p] - ((z >> 5 ^ y << 2) + (y >> 3 ^ z << 4) ^ (sum ^ y) + (k[p & 3 ^ e] ^ z))) & 0xffffffff
            y = v[p]
        z = v[n]
        v[0] = (v[0] - ((z >> 5 ^ y << 2) + (y >> 3 ^ z << 4) ^ (sum ^ y) + (k[0 & 3 ^ e] ^ z))) & 0xffffffff
        y = v[0]
        sum = (sum - _DELTA) & 0xffffffff
    return _long2str(v, True)

if __name__ == "__main__":
    key = b"flag"  # Python3 需要 bytes
    data1 = [0xbc, 0xa5, 0xce, 0x40, 0xf4, 0xb2, 0xb2, 0xe7,
             0xa9, 0x12, 0x9d, 0x12, 0xae, 0x10, 0xc8, 0x5b,
             0x3d, 0xd7, 0x06, 0x1d, 0xdc, 0x70, 0xf8, 0xdc]

    s = bytes(data1)  # 直接转换为 bytes
    s = decrypt(s, key)
    print(repr(s))
```

## 参考资料：

[TEA/XTEA/XXTEA系列算法 - zpchcbd - 博客园](https://www.cnblogs.com/zpchcbd/p/15974293.html)

[解析 TEA 加密算法(C语言、python)：\_tea加密-CSDN博客](https://blog.csdn.net/xiao__1bai/article/details/123307059)
