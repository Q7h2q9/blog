---
title: "SM4算法"
description: "SM4算法简介"
pubDate: "December 17 2024"
image: /images/11.jpg
categories:
  - tech
tags:
  - 加密算法
---

## 1.SM4算法介绍

引用百度百科的介绍:

> **SM4.0**（原名**SMS4.0**）是中华人民共和国政府]采用的一种[分组密码标准，由国家密码管理局于2012年3月21日发布。相关标准为“GM/T 0002-2012《SM4分组密码算法》（原SMS4分组密码算法）”。
>
> 在商用密码体系中，SM4主要用于数据加密，其算法公开，分组长度与密钥长度均为128bit，加密算法与密钥扩展算法都采用32轮非线性迭代结构，S盒为固定的8bit输入8bit输出。

## 2. 算法流程

SM4涉及异或、移位以及盒变换等操作，它分为**加解密**以及**密钥扩展**两个模块

加密流程（左）和密钥扩展（右）如下图所示

![img](https://img2020.cnblogs.com/blog/1215563/202012/1215563-20201214211048328-557839971.jpg)

另外，加解密和密钥扩展用到的S盒如下

![image-20241217151122051](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202412171511237.png)

```c
const uint8 Sbox[256] = {
0xd6,0x90,0xe9,0xfe,0xcc,0xe1,0x3d,0xb7,0x16,0xb6,0x14,0xc2,0x28,0xfb,0x2c,0x05,
0x2b,0x67,0x9a,0x76,0x2a,0xbe,0x04,0xc3,0xaa,0x44,0x13,0x26,0x49,0x86,0x06,0x99,
0x9c,0x42,0x50,0xf4,0x91,0xef,0x98,0x7a,0x33,0x54,0x0b,0x43,0xed,0xcf,0xac,0x62,
0xe4,0xb3,0x1c,0xa9,0xc9,0x08,0xe8,0x95,0x80,0xdf,0x94,0xfa,0x75,0x8f,0x3f,0xa6,
0x47,0x07,0xa7,0xfc,0xf3,0x73,0x17,0xba,0x83,0x59,0x3c,0x19,0xe6,0x85,0x4f,0xa8,
0x68,0x6b,0x81,0xb2,0x71,0x64,0xda,0x8b,0xf8,0xeb,0x0f,0x4b,0x70,0x56,0x9d,0x35,
0x1e,0x24,0x0e,0x5e,0x63,0x58,0xd1,0xa2,0x25,0x22,0x7c,0x3b,0x01,0x21,0x78,0x87,
0xd4,0x00,0x46,0x57,0x9f,0xd3,0x27,0x52,0x4c,0x36,0x02,0xe7,0xa0,0xc4,0xc8,0x9e,
0xea,0xbf,0x8a,0xd2,0x40,0xc7,0x38,0xb5,0xa3,0xf7,0xf2,0xce,0xf9,0x61,0x15,0xa1,
0xe0,0xae,0x5d,0xa4,0x9b,0x34,0x1a,0x55,0xad,0x93,0x32,0x30,0xf5,0x8c,0xb1,0xe3,
0x1d,0xf6,0xe2,0x2e,0x82,0x66,0xca,0x60,0xc0,0x29,0x23,0xab,0x0d,0x53,0x4e,0x6f,
0xd5,0xdb,0x37,0x45,0xde,0xfd,0x8e,0x2f,0x03,0xff,0x6a,0x72,0x6d,0x6c,0x5b,0x51,
0x8d,0x1b,0xaf,0x92,0xbb,0xdd,0xbc,0x7f,0x11,0xd9,0x5c,0x41,0x1f,0x10,0x5a,0xd8,
0x0a,0xc1,0x31,0x88,0xa5,0xcd,0x7b,0xbd,0x2d,0x74,0xd0,0x12,0xb8,0xe5,0xb4,0xb0,
0x89,0x69,0x97,0x4a,0x0c,0x96,0x77,0x7e,0x65,0xb9,0xf1,0x09,0xc5,0x6e,0xc6,0x84,
0x18,0xf0,0x7d,0xec,0x3a,0xdc,0x4d,0x20,0x79,0xee,0x5f,0x3e,0xd7,0xcb,0x39,0x48
};
```

### 2.1 加解密

- 输入的明文长度为128bit，先将其拆分为4个32bit的数据

$$
X_i,X_{i+1},X_{i+2},X_{i+3}
$$

- 其中，$X_{i}$先不做处理，将$X_{i+1}$,$X_{i+2}$,$X_{i+3}$与**轮密钥$rk_i$**进行**异或**操作，并将结果作为盒运算的输入(32bit)，即

$$
SboxInput=x_{i+1}⊕x_{i+2}⊕x_{i+3}⊕rk_i
$$

- 将`SboxInput`拆分为4个8bit数据，分别进行**盒运算**，并将盒运算后的4个8bit重新合并成一个32bit数据**`SboxOutput`**。

- 对`SboxOutput`分别进行5种**移位操作**(左移2位，左移10位，不移位，左移18位，左移24位)，将5种结果与$X_i$进行**异或操作**，其结果作为一轮加密的输出**($X_i+4$)**

```py
output1=(SboxOutput<<2)^(SboxOutput<<10)^(SboxOutput)^(SboxOutput<<18)^(SboxOutput<<24)^x_i
```

- 得到$X_{i+4}$，至此，完成了一轮加密，在实际加密过程中，该过程要进行32轮，每一轮使用**不同**的轮密钥$rk_i$参与加密，每一轮的$rk_i$由**密钥扩展**部分生成
- 经过32轮加密后，将生成的最后4个32bit数据($x_{35}$,$x_{34}$,$x_{33}$,$x_{32}$，最后要进行反序变换，也就是按$x_{35}$~$x_{32}$的顺序)合并成一个128bit的数据`output`，作为加密后的结果。

### 2.2 密钥扩展

密钥扩展和加解密部分差不多，都要经过拆分，盒变换，移位，异或等操作

**(i为轮次，初始为0)**

- 输入初始密钥长度为128bit,先将其拆分为4个32bit的数据

$$
MK_i,MK_{i+1},MK_{i+2},MK_{i+3}
$$

- 对每一个32bit的数据$MK_i$，将其与128bit的系统参数`FK`的$FK_i$进行异或，得到$K_i$,$K_{i+1}$,$K_{i+2}$,$K_{i+3}$，
- $K_i$不动，将$K_{i+1}$,$K_{i+2}$,$K_{i+3}$与**固定参数$CK_i$**进行**异或**操作，并将结果作为盒运算的输入(32bit)，即

$$
SboxInput=K_{i+1}⊕K_{i+2}⊕K_{i+3}⊕CK_i
$$

- 将`SboxInput`拆分为4个8bit数据，分别进行**盒运算**，并将盒运算后的4个8bit重新合并成一个32bit数据**`SboxOutput`**。

- 对`SboxOutput`分别进行3种**移位操作**(左移13位，不移位，左移23位)，将3种结果与$K_i$进行**异或操作**，其结果作为一轮加密的输出**($K_i+4$)**

```py
output1=(SboxOutput<<13)^(SboxOutput)^(SboxOutput<<23)^K_i
```

- 得到$rk_i=k_{i+4}$，至此，完成了一轮密钥生成，在实际加密过程中，该过程要进行32轮，每一轮使用**不同**的固定参数$CK_i$参与加密
- 执行完 32 轮后，便可获得 32 个用于加解密的 $rk_i$

### 2.3 逆运算

对于解密，一个简单的方法是将轮密钥$rk_i$逆序后再执行一次 32 轮的加密运算，即将密文投入加密函数，并且第 0 轮使用 $rk_{31}$作为轮密钥，第 i 轮使用 $rk_{31-i}$作为轮密钥，最后获得的结果便是加密前的密文

下面直接贴[SM4加密算法原理和简单实现（java） - kentle - 博客园](https://www.cnblogs.com/kentle/p/14135865.html)的解释，讲得很清楚

![image-20241217151008903](https://gcore.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202412171510964.png)

## 3.加密模式

### 1. **ECB（电子密码本模式）**

**电子密码本模式（ECB）**是最简单的加密模式。它将明文分成固定大小的块，每个块独立地加密，得到相应的密文块。SM4的块大小是128比特，因此每次加密128比特的明文数据。

适用于一些对安全性要求不高的小规模加密应用，但由于其较低的安全性，不推荐用于敏感数据的加密。

### 2. **CBC（密码分组链接模式）**

**密码分组链接模式（CBC）**将每个明文块与前一个密文块进行异或处理后再加密。第一个明文块与一个初始化向量（IV）进行异或。这样可以避免相同的明文块产生相同的密文块。

适合加密需要防止模式分析、且对并行性要求不高的数据流。常用于文件加密、数据传输加密等。

### 3. **CFB（加密反馈模式）**

**加密反馈模式（CFB）**类似于CBC模式，但它使用先前的密文块与明文块进行异或，通常有几种不同的反馈长度（如CFB-1, CFB-8, CFB-128）。在SM4中，CFB模式通常以128比特（CFB-128）进行实现。

适合流加密的场景，如实时视频或音频流的加密、流媒体加密等。

### 4. **CTR（计数器模式）**

**计数器模式（CTR）**是利用计数器（Counter）和加密算法生成密钥流的模式。每次加密操作中，计数器与密钥一起进行加密，生成的密钥流与明文进行异或得到密文。每个加密块使用不同的计数器值，从而避免了重复的加密操作。

适合高速、大规模的数据加密，如磁盘加密、文件系统加密等，并且可以支持并行加密，效率较高。

### 5. **OFB（输出反馈模式）**

**输出反馈模式（OFB）**与CFB模式类似，但与CFB不同的是，OFB使用加密算法的输出作为下一个反馈输入，而不是将密文作为反馈输入。这样每次反馈时，都会加密一个固定的值来生成密钥流，并与明文进行异或。

适合实时加密和流加密等场景，如加密网络流量等。

### 6. **XTS（扩展标准磁盘加密模式）**

**XTS模式**是一种专门用于磁盘加密的模式，尤其适用于加密大数据块（如磁盘分区）。XTS模式结合了CTR模式和Tweakable Block Cipher的原理，能够为每个数据块生成一个独特的密钥流，使得即使在加密过程中发生密文重复，也不会暴露数据。

适用于磁盘加密和大数据块加密场景，如存储设备加密、云存储加密等。

## 代码实现

### java

此处仅将 SM4 简单实现，而实际运用的时候，还需考虑各种工作模式（例如 OFB 或是 CFB）以及输入分组长度不是 128bit 的整数倍时需要添加的填充（例如 PKCS #7）。此处的代码仅用于展示 SM4 加解密过程的原理，输入的加密数据长度仅支持 128bit（长度为 16 的 `byte` 数组）

```java
public class SM4 {
    int[] key_r;

    /* 初始化轮密钥 */
    SM4(byte[] key) {
        this.key_r = keyGenerate(key);
    }

    /* 密钥拓展 */
    private int[] keyGenerate(byte[] key) {
        int[] key_r = new int[32];//轮密钥rk_i
        int[] key_temp = new int[4];
        int box_in, box_out;//盒变换输入输出
        final int[] FK = {0xa3b1bac6, 0x56aa3350, 0x677d9197, 0xb27022dc};
        final int[] CK = {
                0x00070e15, 0x1c232a31, 0x383f464d, 0x545b6269,
                0x70777e85, 0x8c939aa1, 0xa8afb6bd, 0xc4cbd2d9,
                0xe0e7eef5, 0xfc030a11, 0x181f262d, 0x343b4249,
                0x50575e65, 0x6c737a81, 0x888f969d, 0xa4abb2b9,
                0xc0c7ced5, 0xdce3eaf1, 0xf8ff060d, 0x141b2229,
                0x30373e45, 0x4c535a61, 0x686f767d, 0x848b9299,
                0xa0a7aeb5, 0xbcc3cad1, 0xd8dfe6ed, 0xf4fb0209,
                0x10171e25, 0x2c333a41, 0x484f565d, 0x646b7279
        };
        //将输入的密钥每32比特合并，并异或FK
        for (int i = 0; i < 4; i++) {
            key_temp[i] = jointBytes(key[4 * i], key[4 * i + 1], key[4 * i + 2], key[4 * i + 3]);
            key_temp[i] = key_temp[i] ^ FK[i];
        }
        //32轮密钥拓展
        for (int i = 0; i < 32; i++) {
            box_in = key_temp[1] ^ key_temp[2] ^ key_temp[3] ^ CK[i];
            box_out = sBox(box_in);
            key_r[i] = key_temp[0] ^ box_out ^ shift(box_out, 13) ^ shift(box_out, 23);
            key_temp[0] = key_temp[1];
            key_temp[1] = key_temp[2];
            key_temp[2] = key_temp[3];
            key_temp[3] = key_r[i];
        }
        return key_r;
    }

    /* 加解密主模块 */
    private static byte[] sm4Main(byte[] input, int[] key_r, int mod) {
        int[] text = new int[4];//32比特字
        //将输入以32比特分组
        for (int i = 0; i < 4; i++) {
            text[i] = jointBytes(input[4 * i], input[4 * i + 1], input[4 * i + 2], input[4 * i + 3]);
        }
        int box_input, box_output;//盒变换输入和输出
        for (int i = 0; i < 32; i++) {
            int index = (mod == 0) ? i : (31 - i);//通过改变key_r的顺序改变模式
            box_input = text[1] ^ text[2] ^ text[3] ^ key_r[index];
            box_output = sBox(box_input);
            int temp = text[0] ^ box_output ^ shift(box_output, 2) ^ shift(box_output, 10) ^ shift(box_output, 18) ^ shift(box_output, 24);
            text[0] = text[1];
            text[1] = text[2];
            text[2] = text[3];
            text[3] = temp;
        }
        byte[] output = new byte[16];//输出
        //将结果的32比特字拆分
        for (int i = 0; i < 4; i++) {
            System.arraycopy(splitInt(text[3 - i]), 0, output, 4 * i, 4);
        }
        return output;
    }

    /* 加密 */
    public byte[] encrypt(byte[] plaintext) {
        return sm4Main(plaintext, key_r, 0);
    }

    /* 解密 */
    public byte[] decrypt(byte[] ciphertext) {
        return sm4Main(ciphertext, key_r, 1);
    }

    /* 将32比特数拆分成4个8比特数 */
    private static byte[] splitInt(int n) {
        return new byte[]{(byte) (n >>> 24), (byte) (n >>> 16), (byte) (n >>> 8), (byte) n};
    }

    /* 将4个8比特数合并成32比特数 */
    private static int jointBytes(byte byte_0, byte byte_1, byte byte_2, byte byte_3) {
        return ((byte_0 & 0xFF) << 24) | ((byte_1 & 0xFF) << 16) | ((byte_2 & 0xFF) << 8) | (byte_3 & 0xFF);
    }

    /* S盒变换 */
    private static int sBox(int box_input) {
        //s盒的参数
        final int[] SBOX = {
                0xD6, 0x90, 0xE9, 0xFE, 0xCC, 0xE1, 0x3D, 0xB7, 0x16, 0xB6, 0x14, 0xC2, 0x28, 0xFB, 0x2C, 0x05, 0x2B, 0x67, 0x9A,
                0x76, 0x2A, 0xBE, 0x04, 0xC3, 0xAA, 0x44, 0x13, 0x26, 0x49, 0x86, 0x06, 0x99, 0x9C, 0x42, 0x50, 0xF4, 0x91, 0xEF,
                0x98, 0x7A, 0x33, 0x54, 0x0B, 0x43, 0xED, 0xCF, 0xAC, 0x62, 0xE4, 0xB3, 0x1C, 0xA9, 0xC9, 0x08, 0xE8, 0x95, 0x80,
                0xDF, 0x94, 0xFA, 0x75, 0x8F, 0x3F, 0xA6, 0x47, 0x07, 0xA7, 0xFC, 0xF3, 0x73, 0x17, 0xBA, 0x83, 0x59, 0x3C, 0x19,
                0xE6, 0x85, 0x4F, 0xA8, 0x68, 0x6B, 0x81, 0xB2, 0x71, 0x64, 0xDA, 0x8B, 0xF8, 0xEB, 0x0F, 0x4B, 0x70, 0x56, 0x9D,
                0x35, 0x1E, 0x24, 0x0E, 0x5E, 0x63, 0x58, 0xD1, 0xA2, 0x25, 0x22, 0x7C, 0x3B, 0x01, 0x21, 0x78, 0x87, 0xD4, 0x00,
                0x46, 0x57, 0x9F, 0xD3, 0x27, 0x52, 0x4C, 0x36, 0x02, 0xE7, 0xA0, 0xC4, 0xC8, 0x9E, 0xEA, 0xBF, 0x8A, 0xD2, 0x40,
                0xC7, 0x38, 0xB5, 0xA3, 0xF7, 0xF2, 0xCE, 0xF9, 0x61, 0x15, 0xA1, 0xE0, 0xAE, 0x5D, 0xA4, 0x9B, 0x34, 0x1A, 0x55,
                0xAD, 0x93, 0x32, 0x30, 0xF5, 0x8C, 0xB1, 0xE3, 0x1D, 0xF6, 0xE2, 0x2E, 0x82, 0x66, 0xCA, 0x60, 0xC0, 0x29, 0x23,
                0xAB, 0x0D, 0x53, 0x4E, 0x6F, 0xD5, 0xDB, 0x37, 0x45, 0xDE, 0xFD, 0x8E, 0x2F, 0x03, 0xFF, 0x6A, 0x72, 0x6D, 0x6C,
                0x5B, 0x51, 0x8D, 0x1B, 0xAF, 0x92, 0xBB, 0xDD, 0xBC, 0x7F, 0x11, 0xD9, 0x5C, 0x41, 0x1F, 0x10, 0x5A, 0xD8, 0x0A,
                0xC1, 0x31, 0x88, 0xA5, 0xCD, 0x7B, 0xBD, 0x2D, 0x74, 0xD0, 0x12, 0xB8, 0xE5, 0xB4, 0xB0, 0x89, 0x69, 0x97, 0x4A,
                0x0C, 0x96, 0x77, 0x7E, 0x65, 0xB9, 0xF1, 0x09, 0xC5, 0x6E, 0xC6, 0x84, 0x18, 0xF0, 0x7D, 0xEC, 0x3A, 0xDC, 0x4D,
                0x20, 0x79, 0xEE, 0x5F, 0x3E, 0xD7, 0xCB, 0x39, 0x48
        };

        byte[] temp = splitInt(box_input);//拆分32比特数
        byte[] output = new byte[4];//单个盒变换输出
        //盒变换
        for (int i = 0; i < 4; i++) {
            output[i] = (byte) SBOX[temp[i] & 0xFF];
        }
        //将4个8位字节合并为一个字作为盒变换输出
        return jointBytes(output[0], output[1], output[2], output[3]);
    }

    /* 将input左移n位 */
    private static int shift(int input, int n) {
        return (input >>> (32 - n)) | (input << n);
    }
}
```

### c/c++

sm4.h

```c
#include <stdio.h>
#include <stdio.h>
#include <stdlib.h>
#include <memory.h>
#include  <time.h>
#include <windows.h>
#include <process.h>
#ifndef _SM4_H_
#define _SM4_H_
#ifdef __cplusplus
extern "C" {
#endif

/**@brief  ECB模式的SM4加密
 * @param[in]  pKey					密钥
 * @param[in]  KeyLen				密钥长度，16字节。
 * @param[in]  pInData				输入数据
 * @param[in]  inDataLen			输入数据长度
 * @param[out]  pOutData			输出数据
 * @param[out]  pOutDataLen			输出数据长度
 * @return
 * @remarks
 *
 */
int SM4_ECB_Encrypt( unsigned char *pKey,
                      unsigned int KeyLen,
                      unsigned char *pInData,
                      unsigned int inDataLen,
                      unsigned char *pOutData,
                      unsigned int *pOutDataLen);

/**@brief  ECB模式的SM4解密
 * @param[in]  pKey				密钥
 * @param[in]  KeyLen			密钥长度，16字节。
 * @param[in]  pInData			输入数据
 * @param[in]  inDataLen		输入数据长度
 * @param[out]  pOutData		输出数据
 * @param[out]  pOutDataLen		输出数据长度
 * @return
 * @remarks
 *
 */
int SM4_ECB_Decrypt(  unsigned char *pKey,
                      unsigned int KeyLen,
                      unsigned char *pInData,
                      unsigned int inDataLen,
                      unsigned char *pOutData,
                      unsigned int *pOutDataLen);

/**@brief  CBC模式的SM4加密
 * @param[in]  pKey				密钥
 * @param[in]  KeyLen			密钥长度，16字节。
 * @param[in]  pIV				初始向量
 * @param[in]  ivLen			初始向量，16字节。
 * @param[in]  pInData			输入数据
 * @param[in]  inDataLen		输入数据长度
 * @param[out]  pOutData		输出数据
 * @param[out]  pOutDataLen		输出数据长度
 * @return
 * @remarks
 *
 */
int SM4_CBC_Encrypt( unsigned char *pKey,
                     unsigned int KeyLen,
                     unsigned char *pIV,
                     unsigned int ivLen,
                     unsigned char *pInData,
                     unsigned int inDataLen,
                     unsigned char *pOutData,
                     unsigned int *pOutDataLen);

/**@brief  CBC模式的SM4解密
 * @param[in]  pKey				密钥
 * @param[in]  KeyLen			密钥长度，16字节。
 * @param[in]  pIV				初始向量
 * @param[in]  ivLen			初始向量，16字节。
 * @param[in]  pInData			输入数据
 * @param[in]  inDataLen		输入数据长度
 * @param[out]  pOutData		输出数据
 * @param[out]  pOutDataLen		输出数据长度
 * @return
 * @remarks
 *
 */
int SM4_CBC_Decrypt(unsigned char *pKey,
                    unsigned int KeyLen,
                    unsigned char *pIV,
                    unsigned int ivLen,
                    unsigned char *pInData,
                    unsigned int inDataLen,
                    unsigned char *pOutData,
                    unsigned int *pOutDataLen);

void SM4Mac(unsigned char *InData, int InLen, unsigned char *Key, unsigned char *IV, unsigned char *Mac);
#ifdef __cplusplus
}
#endif

#endif // _SM4_H_

```

sm4.c

```c
//#include "stdafx.h"
#include "sm4.h"

#define SM4_ROUND            32

static unsigned int FK[4]={
    0xA3B1BAC6,0x56AA3350,0x677D9197,0xB27022DC
};

static unsigned int CK[SM4_ROUND]={
    0x00070e15, 0x1c232a31, 0x383f464d, 0x545b6269,
    0x70777e85, 0x8c939aa1, 0xa8afb6bd, 0xc4cbd2d9,
    0xe0e7eef5, 0xfc030a11, 0x181f262d, 0x343b4249,
    0x50575e65, 0x6c737a81, 0x888f969d, 0xa4abb2b9,
    0xc0c7ced5, 0xdce3eaf1, 0xf8ff060d, 0x141b2229,
    0x30373e45, 0x4c535a61, 0x686f767d, 0x848b9299,
    0xa0a7aeb5, 0xbcc3cad1, 0xd8dfe6ed, 0xf4fb0209,
    0x10171e25, 0x2c333a41, 0x484f565d, 0x646b7279
};

static unsigned char Sbox[256]={
    0xd6,0x90,0xe9,0xfe,0xcc,0xe1,0x3d,0xb7,0x16,0xb6,0x14,0xc2,0x28,0xfb,0x2c,0x05,
    0x2b,0x67,0x9a,0x76,0x2a,0xbe,0x04,0xc3,0xaa,0x44,0x13,0x26,0x49,0x86,0x06,0x99,
    0x9c,0x42,0x50,0xf4,0x91,0xef,0x98,0x7a,0x33,0x54,0x0b,0x43,0xed,0xcf,0xac,0x62,
    0xe4,0xb3,0x1c,0xa9,0xc9,0x08,0xe8,0x95,0x80,0xdf,0x94,0xfa,0x75,0x8f,0x3f,0xa6,
    0x47,0x07,0xa7,0xfc,0xf3,0x73,0x17,0xba,0x83,0x59,0x3c,0x19,0xe6,0x85,0x4f,0xa8,
    0x68,0x6b,0x81,0xb2,0x71,0x64,0xda,0x8b,0xf8,0xeb,0x0f,0x4b,0x70,0x56,0x9d,0x35,
    0x1e,0x24,0x0e,0x5e,0x63,0x58,0xd1,0xa2,0x25,0x22,0x7c,0x3b,0x01,0x21,0x78,0x87,
    0xd4,0x00,0x46,0x57,0x9f,0xd3,0x27,0x52,0x4c,0x36,0x02,0xe7,0xa0,0xc4,0xc8,0x9e,
    0xea,0xbf,0x8a,0xd2,0x40,0xc7,0x38,0xb5,0xa3,0xf7,0xf2,0xce,0xf9,0x61,0x15,0xa1,
    0xe0,0xae,0x5d,0xa4,0x9b,0x34,0x1a,0x55,0xad,0x93,0x32,0x30,0xf5,0x8c,0xb1,0xe3,
    0x1d,0xf6,0xe2,0x2e,0x82,0x66,0xca,0x60,0xc0,0x29,0x23,0xab,0x0d,0x53,0x4e,0x6f,
    0xd5,0xdb,0x37,0x45,0xde,0xfd,0x8e,0x2f,0x03,0xff,0x6a,0x72,0x6d,0x6c,0x5b,0x51,
    0x8d,0x1b,0xaf,0x92,0xbb,0xdd,0xbc,0x7f,0x11,0xd9,0x5c,0x41,0x1f,0x10,0x5a,0xd8,
    0x0a,0xc1,0x31,0x88,0xa5,0xcd,0x7b,0xbd,0x2d,0x74,0xd0,0x12,0xb8,0xe5,0xb4,0xb0,
    0x89,0x69,0x97,0x4a,0x0c,0x96,0x77,0x7e,0x65,0xb9,0xf1,0x09,0xc5,0x6e,0xc6,0x84,
    0x18,0xf0,0x7d,0xec,0x3a,0xdc,0x4d,0x20,0x79,0xee,0x5f,0x3e,0xd7,0xcb,0x39,0x48
};

#define ROL(x,y)    ((x)<<(y) |    (x)>>(32-(y)))

unsigned int SMS4_T1(unsigned int    dwA)
{
    unsigned char    a0[4]={0};
    unsigned char    b0[4]={0};
    unsigned int    dwB=0;
    unsigned int    dwC=0;
    int                i=0;
/*
    for (i=0;i<4;i++)
    {
        a0[i] = (unsigned char)((dwA>>(i*8)) & 0xff);
        b0[i] = Sbox[a0[i]];
        dwB  |= (b0[i]<<(i*8));
    }
*/
		a0[0] = (unsigned char)((dwA) & 0xff);
    b0[0] = Sbox[a0[0]];
    dwB  |= (b0[0]);

    a0[1] = (unsigned char)((dwA>>(8)) & 0xff);
    b0[1] = Sbox[a0[1]];
    dwB  |= (b0[1]<<(8));

    a0[2] = (unsigned char)((dwA>>(16)) & 0xff);
    b0[2] = Sbox[a0[2]];
    dwB  |= (b0[2]<<(16));

    a0[3] = (unsigned char)((dwA>>(24)) & 0xff);
    b0[3] = Sbox[a0[3]];
    dwB  |= (b0[3]<<(24));          // 2013-08-13

    dwC=dwB^ROL(dwB,2)^ROL(dwB,10)^ROL(dwB,18)^ROL(dwB,24);

    return dwC;
}

unsigned int SMS4_T2(unsigned int    dwA)
{
    unsigned char    a0[4]={0};
    unsigned char    b0[4]={0};
    unsigned int    dwB=0;
    unsigned int    dwC=0;
    int        i=0;
/*
    for (i=0;i<4;i++)
    {
        a0[i] = (unsigned char)((dwA>>(i*8)) & 0xff);
        b0[i] = Sbox[a0[i]];
        dwB  |= (b0[i]<<(i*8));
    }
*/
		a0[0] = (unsigned char)((dwA) & 0xff);
    b0[0] = Sbox[a0[0]];
    dwB  |= (b0[0]);

    a0[1] = (unsigned char)((dwA>>(8)) & 0xff);
    b0[1] = Sbox[a0[1]];
    dwB  |= (b0[1]<<(8));

    a0[2] = (unsigned char)((dwA>>(16)) & 0xff);
    b0[2] = Sbox[a0[2]];
    dwB  |= (b0[2]<<(16));

    a0[3] = (unsigned char)((dwA>>(24)) & 0xff);
    b0[3] = Sbox[a0[3]];
    dwB  |= (b0[3]<<(24));          // 2013-08-13

    dwC=dwB^ROL(dwB,13)^ROL(dwB,23);

    return dwC;
}

/* MK[4] is the Encrypt Key, rk[32] is Round Key */
void SMS4_Key_Expansion(unsigned int MK[],    unsigned int rk[])
{
    unsigned int    K[4]={0};
    int        i=0;

    for (i=0;i<4;i++)
    {
        K[i]    =    MK[i]    ^    FK[i];
    }

    for (i=0;i<SM4_ROUND;i++)
    {
        K[i%4]^=SMS4_T2(K[(i+1)%4]^K[(i+2)%4]^K[(i+3)%4]^CK[i]);
        rk[i]=K[i%4];
    }
}

/* X[4] is PlainText, rk[32] is round Key, Y[4] is CipherText */
void SMS4_ECB_Encryption_Core(unsigned int X[], unsigned int rk[], unsigned int Y[])
{
    unsigned int    tempX[4]={0};
    int                i=0;
/*
    for (i=0;i<4;i++)
    {
        tempX[i]=X[i];
    }
*/
		tempX[0]=X[0];
    tempX[1]=X[1];
    tempX[2]=X[2];
    tempX[3]=X[3];
/*
    for (i=0;i<SM4_ROUND;i++)
    {
        tempX[i%4]^=SMS4_T1(tempX[(i+1)%4]^tempX[(i+2)%4]^tempX[(i+3)%4]^rk[i]);
    }
*/
		tempX[0]^=SMS4_T1(tempX[1]^tempX[2]^tempX[3]^rk[0]);
		tempX[1]^=SMS4_T1(tempX[2]^tempX[3]^tempX[0]^rk[1]);
		tempX[2]^=SMS4_T1(tempX[3]^tempX[0]^tempX[1]^rk[2]);
		tempX[3]^=SMS4_T1(tempX[0]^tempX[1]^tempX[2]^rk[3]);

		tempX[0]^=SMS4_T1(tempX[1]^tempX[2]^tempX[3]^rk[4]);
		tempX[1]^=SMS4_T1(tempX[2]^tempX[3]^tempX[0]^rk[5]);
		tempX[2]^=SMS4_T1(tempX[3]^tempX[0]^tempX[1]^rk[6]);
		tempX[3]^=SMS4_T1(tempX[0]^tempX[1]^tempX[2]^rk[7]);

		tempX[0]^=SMS4_T1(tempX[1]^tempX[2]^tempX[3]^rk[8]);
		tempX[1]^=SMS4_T1(tempX[2]^tempX[3]^tempX[0]^rk[9]);
		tempX[2]^=SMS4_T1(tempX[3]^tempX[0]^tempX[1]^rk[10]);
		tempX[3]^=SMS4_T1(tempX[0]^tempX[1]^tempX[2]^rk[11]);

		tempX[0]^=SMS4_T1(tempX[1]^tempX[2]^tempX[3]^rk[12]);
		tempX[1]^=SMS4_T1(tempX[2]^tempX[3]^tempX[0]^rk[13]);
		tempX[2]^=SMS4_T1(tempX[3]^tempX[0]^tempX[1]^rk[14]);
		tempX[3]^=SMS4_T1(tempX[0]^tempX[1]^tempX[2]^rk[15]);

		tempX[0]^=SMS4_T1(tempX[1]^tempX[2]^tempX[3]^rk[16]);
		tempX[1]^=SMS4_T1(tempX[2]^tempX[3]^tempX[0]^rk[17]);
		tempX[2]^=SMS4_T1(tempX[3]^tempX[0]^tempX[1]^rk[18]);
		tempX[3]^=SMS4_T1(tempX[0]^tempX[1]^tempX[2]^rk[19]);

		tempX[0]^=SMS4_T1(tempX[1]^tempX[2]^tempX[3]^rk[20]);
		tempX[1]^=SMS4_T1(tempX[2]^tempX[3]^tempX[0]^rk[21]);
		tempX[2]^=SMS4_T1(tempX[3]^tempX[0]^tempX[1]^rk[22]);
		tempX[3]^=SMS4_T1(tempX[0]^tempX[1]^tempX[2]^rk[23]);

		tempX[0]^=SMS4_T1(tempX[1]^tempX[2]^tempX[3]^rk[24]);
		tempX[1]^=SMS4_T1(tempX[2]^tempX[3]^tempX[0]^rk[25]);
		tempX[2]^=SMS4_T1(tempX[3]^tempX[0]^tempX[1]^rk[26]);
		tempX[3]^=SMS4_T1(tempX[0]^tempX[1]^tempX[2]^rk[27]);

		tempX[0]^=SMS4_T1(tempX[1]^tempX[2]^tempX[3]^rk[28]);
		tempX[1]^=SMS4_T1(tempX[2]^tempX[3]^tempX[0]^rk[29]);
		tempX[2]^=SMS4_T1(tempX[3]^tempX[0]^tempX[1]^rk[30]);
		tempX[3]^=SMS4_T1(tempX[0]^tempX[1]^tempX[2]^rk[31]);
/*    for (i=0;i<4;i++)
    {
        Y[i]=tempX[3-i];
    }*/
    Y[0]=tempX[3];
    Y[1]=tempX[2];
    Y[2]=tempX[1];
    Y[3]=tempX[0]; // 2013-08-19
}

/* X[4] is PlainText, rk[32] is round Key, Y[4] is CipherText */
void SMS4_ECB_Decryption_Core(unsigned int X[], unsigned int rk[], unsigned int Y[])
{
    unsigned int    tempX[4]={0};
    int                i=0;

    /*    for (i=0;i<4;i++)
    {
        tempX[i]=X[i];
    }
		*/
		tempX[0]=X[0];
    tempX[1]=X[1];
    tempX[2]=X[2];
    tempX[3]=X[3]; // 2013-08-19

/*    for (i=0;i<SM4_ROUND;i++)
    {
        tempX[i%4]^=SMS4_T1(tempX[(i+1)%4]^tempX[(i+2)%4]^tempX[(i+3)%4]^rk[(31-i)]);
    }
*/
		tempX[0]^=SMS4_T1(tempX[1]^tempX[2]^tempX[3]^rk[31]);
		tempX[1]^=SMS4_T1(tempX[2]^tempX[3]^tempX[0]^rk[30]);
		tempX[2]^=SMS4_T1(tempX[3]^tempX[0]^tempX[1]^rk[29]);
		tempX[3]^=SMS4_T1(tempX[0]^tempX[1]^tempX[2]^rk[28]);

		tempX[0]^=SMS4_T1(tempX[1]^tempX[2]^tempX[3]^rk[27]);
		tempX[1]^=SMS4_T1(tempX[2]^tempX[3]^tempX[0]^rk[26]);
		tempX[2]^=SMS4_T1(tempX[3]^tempX[0]^tempX[1]^rk[25]);
		tempX[3]^=SMS4_T1(tempX[0]^tempX[1]^tempX[2]^rk[24]);

		tempX[0]^=SMS4_T1(tempX[1]^tempX[2]^tempX[3]^rk[23]);
		tempX[1]^=SMS4_T1(tempX[2]^tempX[3]^tempX[0]^rk[22]);
		tempX[2]^=SMS4_T1(tempX[3]^tempX[0]^tempX[1]^rk[21]);
		tempX[3]^=SMS4_T1(tempX[0]^tempX[1]^tempX[2]^rk[20]);

		tempX[0]^=SMS4_T1(tempX[1]^tempX[2]^tempX[3]^rk[19]);
		tempX[1]^=SMS4_T1(tempX[2]^tempX[3]^tempX[0]^rk[18]);
		tempX[2]^=SMS4_T1(tempX[3]^tempX[0]^tempX[1]^rk[17]);
		tempX[3]^=SMS4_T1(tempX[0]^tempX[1]^tempX[2]^rk[16]);

		tempX[0]^=SMS4_T1(tempX[1]^tempX[2]^tempX[3]^rk[15]);
		tempX[1]^=SMS4_T1(tempX[2]^tempX[3]^tempX[0]^rk[14]);
		tempX[2]^=SMS4_T1(tempX[3]^tempX[0]^tempX[1]^rk[13]);
		tempX[3]^=SMS4_T1(tempX[0]^tempX[1]^tempX[2]^rk[12]);

		tempX[0]^=SMS4_T1(tempX[1]^tempX[2]^tempX[3]^rk[11]);
		tempX[1]^=SMS4_T1(tempX[2]^tempX[3]^tempX[0]^rk[10]);
		tempX[2]^=SMS4_T1(tempX[3]^tempX[0]^tempX[1]^rk[9]);
		tempX[3]^=SMS4_T1(tempX[0]^tempX[1]^tempX[2]^rk[8]);

		tempX[0]^=SMS4_T1(tempX[1]^tempX[2]^tempX[3]^rk[7]);
		tempX[1]^=SMS4_T1(tempX[2]^tempX[3]^tempX[0]^rk[6]);
		tempX[2]^=SMS4_T1(tempX[3]^tempX[0]^tempX[1]^rk[5]);
		tempX[3]^=SMS4_T1(tempX[0]^tempX[1]^tempX[2]^rk[4]);

		tempX[0]^=SMS4_T1(tempX[1]^tempX[2]^tempX[3]^rk[3]);
		tempX[1]^=SMS4_T1(tempX[2]^tempX[3]^tempX[0]^rk[2]);
		tempX[2]^=SMS4_T1(tempX[3]^tempX[0]^tempX[1]^rk[1]);
		tempX[3]^=SMS4_T1(tempX[0]^tempX[1]^tempX[2]^rk[0]);// 2013-08-19
/*    for (i=0;i<4;i++)
    {
        Y[i]=tempX[3-i];
    }*/
    Y[0]=tempX[3];
    Y[1]=tempX[2];
    Y[2]=tempX[1];
    Y[3]=tempX[0]; // 2013-08-19
}

void SMS4_convert_to_network_order(unsigned int* src,unsigned int* dst,int count)
{
	int i=0;

	for ( ; i<count; i++ )
	{
		unsigned char* ps = (unsigned char*)(src+i);
		unsigned char* pd = (unsigned char*)(dst+i);

		pd[0] = ps[3];
		pd[1] = ps[2];
		pd[2] = ps[1];
		pd[3] = ps[0];
	}
}

void SMS4_convert_to_host_order(unsigned int* src,unsigned int* dst,int count)
{
	SMS4_convert_to_network_order(src,dst,count);
}

void SMS4_ECB_Encryption(unsigned char plaintext[16], unsigned char key[16], unsigned char ciphertext[16])
{
	unsigned int _pt[4];
	unsigned int _ky[4];
	unsigned int _ct[4];
	unsigned int _rk[32];

	SMS4_convert_to_network_order((unsigned int*)plaintext,_pt,4);
	SMS4_convert_to_network_order((unsigned int*)key,_ky,4);

	SMS4_Key_Expansion(_ky,_rk);
	SMS4_ECB_Encryption_Core(_pt,_rk,_ct);

	SMS4_convert_to_host_order(_ct,(unsigned int*)ciphertext,4);
}

void Key_Expansion_init( unsigned char key[16], unsigned int rk[32])
{
	unsigned int _ky[4];

	SMS4_convert_to_network_order((unsigned int*)key,_ky,4);
	SMS4_Key_Expansion (_ky,rk);
}

void SMS4_ECB_EncryptionEx(unsigned char plaintext[16], unsigned int key[32], unsigned char ciphertext[16])
{
	unsigned int _pt[4];
	unsigned int _ct[4];

	SMS4_convert_to_network_order((unsigned int*)plaintext,_pt,4);
	SMS4_ECB_Encryption_Core(_pt,key,_ct);
	SMS4_convert_to_host_order(_ct,(unsigned int*)ciphertext,4);
}

void SMS4_ECB_Decryption(unsigned char ciphertext[16], unsigned char key[16], unsigned char plaintext[16])
{
	unsigned int _ct[4];
	unsigned int _ky[4];
	unsigned int _pt[4];
	unsigned int _rk[32];

	SMS4_convert_to_network_order((unsigned int*)ciphertext,_ct,4);
	SMS4_convert_to_network_order((unsigned int*)key,_ky,4);

	SMS4_Key_Expansion(_ky,_rk);
	SMS4_ECB_Decryption_Core(_ct,_rk,_pt);

	SMS4_convert_to_host_order(_pt,(unsigned int*)plaintext,4);
}

void SMS4_ECB_DecryptionEx(unsigned char ciphertext[16], unsigned int key[32], unsigned char plaintext[16])
{
	unsigned int _ct[4];
	unsigned int _pt[4];

	SMS4_convert_to_network_order((unsigned int*)ciphertext,_ct,4);
	SMS4_ECB_Decryption_Core(_ct,key,_pt);
	SMS4_convert_to_host_order(_pt,(unsigned int*)plaintext,4);
}

void SMS4_CBC_Encryption(unsigned char plaintext[16], unsigned char key[16], unsigned char iv[16], unsigned char ciphertext[16])
{
    unsigned char plaintextNew[16];
    int i = 0;
    for (i = 0; i < 16; i ++)
    {
        plaintextNew[i] = plaintext[i] ^ iv[i];
    }

    SMS4_ECB_Encryption(plaintextNew, key, ciphertext);
}

void SMS4_CBC_Decryption(unsigned char ciphertext[16], unsigned char key[16], unsigned char iv[16], unsigned char plaintext[16])
{
    unsigned char plaintextTemp[16];
    int i = 0;
    SMS4_ECB_Decryption(ciphertext, key, plaintextTemp);
    for (i = 0; i < 16; i ++)
    {
        plaintext[i] = plaintextTemp[i] ^ iv[i];
    }
}

void SMS4_CBC_EncryptionEx(unsigned char plaintext[16], unsigned int key[32], unsigned char iv[16], unsigned char ciphertext[16])
{
    unsigned char plaintextNew[16];
    int i = 0;
    for (i = 0; i < 16; i ++)
    {
        plaintextNew[i] = plaintext[i] ^ iv[i];
    }

    SMS4_ECB_EncryptionEx(plaintextNew, key, ciphertext);
}

void SMS4_CBC_DecryptionEx(unsigned char ciphertext[16], unsigned int key[32], unsigned char iv[16], unsigned char plaintext[16])
{
    unsigned char plaintextTemp[16];
    int i = 0;
    SMS4_ECB_DecryptionEx(ciphertext, key, plaintextTemp);
    for (i = 0; i < 16; i ++)
    {
        plaintext[i] = plaintextTemp[i] ^ iv[i];
    }
}

/**@brief  ECB模式的SMS4加密
 * @param[in]  pKey         密钥
 * @param[in]  KeyLen    密钥长度，16字节。
 * @param[in]  pInData    输入数据
 * @param[in]  inDataLen    输入数据长度
 * @param[out]  pOutData    输出数据
 * @param[out]  pOutDataLen    输出数据长度
 * @return
 * @remarks
 *
 */
int SM4_ECB_Encrypt( unsigned char *pKey,
                      unsigned int KeyLen,
                      unsigned char *pInData,
                      unsigned int inDataLen,
                      unsigned char *pOutData,
                      unsigned int *pOutDataLen)
{
    int i = 0;
    //int rv = 0;
    int loop = 0;
    unsigned int rk[32];

    *pOutDataLen = 0;

    if(KeyLen != 16)
    {
        return 1;
    }
    if(inDataLen % 16 != 0)
    {
        return 1;
    }

    Key_Expansion_init(pKey,rk);
    loop = inDataLen / 16;
    for (i = 0; i < loop; i ++)
    {
        //SMS4_ECB_Encryption(pInData + i * 16, pKey, pOutData + i * 16);
        SMS4_ECB_EncryptionEx(pInData + i * 16, rk, pOutData + i * 16);
    }
    *pOutDataLen = inDataLen;
    return 0;
}

/**@brief  ECB模式的SM4解密
 * @param[in]  pKey         密钥
 * @param[in]  KeyLen    密钥长度，16字节。
 * @param[in]  pInData    输入数据
 * @param[in]  inDataLen    输入数据长度
 * @param[out]  pOutData    输出数据
 * @param[out]  pOutDataLen    输出数据长度
 * @return
 * @remarks
 *
 */
int SM4_ECB_Decrypt(  unsigned char *pKey,
                      unsigned int KeyLen,
                      unsigned char *pInData,
                      unsigned int inDataLen,
                      unsigned char *pOutData,
                      unsigned int *pOutDataLen)
{
    int i = 0;
    // int rv = 0;
    int loop = 0;
    unsigned int rk[32];

    *pOutDataLen = 0;
    if(KeyLen != 16)
    {
        return 1;
    }
    if(inDataLen % 16 != 0)
    {
        return 1;
    }

    Key_Expansion_init(pKey,rk);
    loop = inDataLen / 16;
    for (i = 0; i < loop; i ++)
    {
        //SMS4_ECB_Decryption(pInData + i * 16, pKey, pOutData + i * 16);
	 SMS4_ECB_DecryptionEx(pInData + i * 16, rk, pOutData + i * 16);
    }
    *pOutDataLen = inDataLen;
    return 0;
}

/**@brief  CBC模式的SM4加密
 * @param[in]  pKey         密钥
 * @param[in]  KeyLen    密钥长度，16字节。
 * @param[in]  pIV         初始向量
 * @param[in]  ivLen    初始向量，16字节。
 * @param[in]  pInData    输入数据
 * @param[in]  inDataLen    输入数据长度
 * @param[out]  pOutData    输出数据
 * @param[out]  pOutDataLen    输出数据长度
 * @return
 * @remarks
 *
 */
int SM4_CBC_Encrypt( unsigned char *pKey,
                     unsigned int KeyLen,
                     unsigned char *pIV,
                     unsigned int ivLen,
                     unsigned char *pInData,
                     unsigned int inDataLen,
                     unsigned char *pOutData,
                     unsigned int *pOutDataLen)
{
    int i = 0;
    // int rv = 0;
    int loop = 0;
    unsigned char *pIVTemp = NULL;
    unsigned int rk[32];

    *pOutDataLen = 0;
    if(KeyLen != 16)
    {
        return 1;
    }
    if(inDataLen % 16 != 0)
    {
        return 1;
    }
    if(ivLen != 16)
    {
        return 1;
    }
    Key_Expansion_init(pKey,rk);
    loop = inDataLen / 16;
    pIVTemp = pIV;
    for (i = 0; i < loop; i ++)
    {
        SMS4_CBC_EncryptionEx(pInData + i * 16, rk, pIVTemp, pOutData + i * 16);
        pIVTemp = pOutData + i * 16;
    }
    *pOutDataLen = inDataLen;
    return 0;
}

/**@brief  CBC模式的SM4解密
 * @param[in]  pKey         密钥
 * @param[in]  KeyLen    密钥长度，16字节。
 * @param[in]  pIV         初始向量
 * @param[in]  ivLen    初始向量，16字节。
 * @param[in]  pInData    输入数据
 * @param[in]  inDataLen    输入数据长度
 * @param[out]  pOutData    输出数据
 * @param[out]  pOutDataLen    输出数据长度
 * @return
 * @remarks
 *
 */
int SM4_CBC_Decrypt(unsigned char *pKey,
                    unsigned int KeyLen,
                    unsigned char *pIV,
                    unsigned int ivLen,
                    unsigned char *pInData,
                    unsigned int inDataLen,
                    unsigned char *pOutData,
                    unsigned int *pOutDataLen)
{
    int i = 0;
    // int rv = 0;
    int loop = 0;
    unsigned char *pIVTemp = NULL;
    unsigned int rk[32];

    *pOutDataLen = 0;
    if(KeyLen != 16)
    {
        return 1;
    }
    if(inDataLen % 16 != 0)
    {
        return 1;
    }
    if(ivLen != 16)
    {
        return 1;
    }
    Key_Expansion_init(pKey,rk);
    loop = inDataLen / 16;
    pIVTemp = pIV;
    for (i = 0; i < loop; i ++)
    {
        SMS4_CBC_DecryptionEx(pInData + i * 16, rk, pIVTemp, pOutData + i * 16);
        pIVTemp = pInData + i * 16;
    }
    *pOutDataLen = inDataLen;
    return 0;
}

void SM4Mac(unsigned char *InData, int InLen, unsigned char *Key, unsigned char *IV, unsigned char *Mac)
{
	unsigned char		PacketData[128];
	int		NewLen;
	unsigned char		Pad[16] = { 0x80,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00 };

	memcpy(PacketData, InData, InLen);
	if (InLen % 16)
	{
		memcpy(PacketData + InLen, Pad, 16 - InLen % 16);
		NewLen = InLen + (16 - InLen % 16);
	}
	else
	{
		memcpy(PacketData + InLen, Pad, 16);
		NewLen = InLen + 16;
	}

	//SM4 CBC加密
	SM4_CBC_Encrypt(Key, 16, IV, 16, PacketData, NewLen, PacketData, &NewLen);
	memcpy(Mac, PacketData + NewLen - 16, 4);
	return;
}

```

### python

```py
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Author  : 河北雪域网络科技有限公司 A.Star
# @contact: astar@snowland.ltd
# @site: www.snowland.ltd
# @file: SM4.py
# @time: 2018/9/21 15:25
# @Software: PyCharm

import copy
import time
from functools import reduce

# Expanded SM4 S-boxes    Sbox table: 8bits input convert to 8 bits output
SboxTable = [
    0xd6, 0x90, 0xe9, 0xfe, 0xcc, 0xe1, 0x3d, 0xb7, 0x16, 0xb6, 0x14, 0xc2, 0x28, 0xfb, 0x2c, 0x05,
    0x2b, 0x67, 0x9a, 0x76, 0x2a, 0xbe, 0x04, 0xc3, 0xaa, 0x44, 0x13, 0x26, 0x49, 0x86, 0x06, 0x99,
    0x9c, 0x42, 0x50, 0xf4, 0x91, 0xef, 0x98, 0x7a, 0x33, 0x54, 0x0b, 0x43, 0xed, 0xcf, 0xac, 0x62,
    0xe4, 0xb3, 0x1c, 0xa9, 0xc9, 0x08, 0xe8, 0x95, 0x80, 0xdf, 0x94, 0xfa, 0x75, 0x8f, 0x3f, 0xa6,
    0x47, 0x07, 0xa7, 0xfc, 0xf3, 0x73, 0x17, 0xba, 0x83, 0x59, 0x3c, 0x19, 0xe6, 0x85, 0x4f, 0xa8,
    0x68, 0x6b, 0x81, 0xb2, 0x71, 0x64, 0xda, 0x8b, 0xf8, 0xeb, 0x0f, 0x4b, 0x70, 0x56, 0x9d, 0x35,
    0x1e, 0x24, 0x0e, 0x5e, 0x63, 0x58, 0xd1, 0xa2, 0x25, 0x22, 0x7c, 0x3b, 0x01, 0x21, 0x78, 0x87,
    0xd4, 0x00, 0x46, 0x57, 0x9f, 0xd3, 0x27, 0x52, 0x4c, 0x36, 0x02, 0xe7, 0xa0, 0xc4, 0xc8, 0x9e,
    0xea, 0xbf, 0x8a, 0xd2, 0x40, 0xc7, 0x38, 0xb5, 0xa3, 0xf7, 0xf2, 0xce, 0xf9, 0x61, 0x15, 0xa1,
    0xe0, 0xae, 0x5d, 0xa4, 0x9b, 0x34, 0x1a, 0x55, 0xad, 0x93, 0x32, 0x30, 0xf5, 0x8c, 0xb1, 0xe3,
    0x1d, 0xf6, 0xe2, 0x2e, 0x82, 0x66, 0xca, 0x60, 0xc0, 0x29, 0x23, 0xab, 0x0d, 0x53, 0x4e, 0x6f,
    0xd5, 0xdb, 0x37, 0x45, 0xde, 0xfd, 0x8e, 0x2f, 0x03, 0xff, 0x6a, 0x72, 0x6d, 0x6c, 0x5b, 0x51,
    0x8d, 0x1b, 0xaf, 0x92, 0xbb, 0xdd, 0xbc, 0x7f, 0x11, 0xd9, 0x5c, 0x41, 0x1f, 0x10, 0x5a, 0xd8,
    0x0a, 0xc1, 0x31, 0x88, 0xa5, 0xcd, 0x7b, 0xbd, 0x2d, 0x74, 0xd0, 0x12, 0xb8, 0xe5, 0xb4, 0xb0,
    0x89, 0x69, 0x97, 0x4a, 0x0c, 0x96, 0x77, 0x7e, 0x65, 0xb9, 0xf1, 0x09, 0xc5, 0x6e, 0xc6, 0x84,
    0x18, 0xf0, 0x7d, 0xec, 0x3a, 0xdc, 0x4d, 0x20, 0x79, 0xee, 0x5f, 0x3e, 0xd7, 0xcb, 0x39, 0x48,
]

# System parameter
FK = [0xa3b1bac6, 0x56aa3350, 0x677d9197, 0xb27022dc]

# fixed parameter
CK = [
    0x00070e15, 0x1c232a31, 0x383f464d, 0x545b6269,
    0x70777e85, 0x8c939aa1, 0xa8afb6bd, 0xc4cbd2d9,
    0xe0e7eef5, 0xfc030a11, 0x181f262d, 0x343b4249,
    0x50575e65, 0x6c737a81, 0x888f969d, 0xa4abb2b9,
    0xc0c7ced5, 0xdce3eaf1, 0xf8ff060d, 0x141b2229,
    0x30373e45, 0x4c535a61, 0x686f767d, 0x848b9299,
    0xa0a7aeb5, 0xbcc3cad1, 0xd8dfe6ed, 0xf4fb0209,
    0x10171e25, 0x2c333a41, 0x484f565d, 0x646b7279
]

ENCRYPT = 0
DECRYPT = 1

def GET_UINT32_BE(key_data):
    return int((key_data[0] << 24) | (key_data[1] << 16) | (key_data[2] << 8) | (key_data[3]))

def PUT_UINT32_BE(n):
    return [int((n >> 24) & 0xff), int((n >> 16) & 0xff), int((n >> 8) & 0xff), int((n) & 0xff)]

# rotate shift left marco definition
def SHL(x, n):
    return int(int(x << n) & 0xffffffff)

def ROTL(x, n):
    return SHL(x, n) | int((x >> (32 - n)) & 0xffffffff)

def XOR(a, b):
    return list(map(lambda x, y: x ^ y, a, b))

# look up in SboxTable and get the related value.
# args:    [in] inch: 0x00~0xFF (8 bits unsigned value).
def sm4Sbox(idx):
    return SboxTable[idx]

# Calculating round encryption key.
# args:    [in] a: a is a 32 bits unsigned value;
# return: sk[i]: i{0,1,2,3,...31}.
def sm4CalciRK(ka):
    a = PUT_UINT32_BE(ka)
    b = [sm4Sbox(i) for i in a]
    bb = GET_UINT32_BE(b)
    rk = bb ^ (ROTL(bb, 13)) ^ (ROTL(bb, 23))
    return rk

# private F(Lt) function:
# "T algorithm" == "L algorithm" + "t algorithm".
# args:    [in] a: a is a 32 bits unsigned value;
# return: c: c is calculated with line algorithm "L" and nonline algorithm "t"
def sm4Lt(ka):
    a = PUT_UINT32_BE(ka)
    b = [sm4Sbox(i) for i in a]
    bb = GET_UINT32_BE(b)
    return bb ^ (ROTL(bb, 2)) ^ (ROTL(bb, 10)) ^ (ROTL(bb, 18)) ^ (ROTL(bb, 24))

# private F function:
# Calculating and getting encryption/decryption contents.
# args:    [in] x0: original contents;
# args:    [in] x1: original contents;
# args:    [in] x2: original contents;
# args:    [in] x3: original contents;
# args:    [in] rk: encryption/decryption key;
# return the contents of encryption/decryption contents.
def sm4F(x0, x1, x2, x3, rk):
    return (x0 ^ sm4Lt(x1 ^ x2 ^ x3 ^ rk))

class Sm4(object):
    def __init__(self):
        self.sk = [0] * 32
        self.mode = ENCRYPT

    def sm4_set_key(self, key_data, mode):
        self.sm4_setkey(key_data, mode)

    def sm4_setkey(self, key, mode):
        MK = [0, 0, 0, 0]
        k = [0] * 36
        MK[0] = GET_UINT32_BE(key[0:4])
        MK[1] = GET_UINT32_BE(key[4:8])
        MK[2] = GET_UINT32_BE(key[8:12])
        MK[3] = GET_UINT32_BE(key[12:16])
        k[0:4] = XOR(MK, FK)
        for i in range(32):
            k[i + 4] = k[i] ^ (sm4CalciRK(k[i + 1] ^ k[i + 2] ^ k[i + 3] ^ CK[i]))
        self.sk = k[4:]
        self.mode = mode
        if mode == DECRYPT:
            self.sk.reverse()

    def sm4_one_round(self, sk, in_put):
        item = [GET_UINT32_BE(in_put[0:4]), GET_UINT32_BE(in_put[4:8]), GET_UINT32_BE(in_put[8:12]),
                GET_UINT32_BE(in_put[12:16])]
        res = reduce(lambda x, y: [x[1], x[2], x[3], sm4F(x[0], x[1], x[2], x[3], y)], sk, item)
        res.reverse()
        rev = map(PUT_UINT32_BE, res)
        out_put = []
        [out_put.extend(_) for _ in rev]
        return out_put

    def sm4_crypt_ecb(self, input_data):
        # SM4-ECB block encryption/decryption
        output_data = []
        tmp = [input_data[i:i + 16] for i in range(0, len(input_data), 16)]
        [output_data.extend(each) for each in map(lambda x: self.sm4_one_round(self.sk, x), tmp)]
        return output_data

    def sm4_crypt_cbc(self, iv, input_data):
        # SM4-CBC buffer encryption/decryption
        length = len(input_data)
        i = 0
        output_data = []
        tmp_input = [0] * 16
        if self.mode == ENCRYPT:
            while length > 0:
                tmp_input[0:16] = XOR(input_data[i:i + 16], iv[0:16])
                output_data += self.sm4_one_round(self.sk, tmp_input[0:16])
                iv = copy.deepcopy(output_data[i:i + 16])
                i += 16
                length -= 16
        else:
            while length > 0:
                output_data += self.sm4_one_round(self.sk, input_data[i:i + 16])
                output_data[i:i + 16] = XOR(output_data[i:i + 16], iv[0:16])
                iv = copy.deepcopy(input_data[i:i + 16])
                i += 16
                length -= 16
        return output_data

def sm4_crypt_ecb(mode, key, data):
    sm4_d = Sm4()
    sm4_d.sm4_set_key(key, mode)
    en_data = sm4_d.sm4_crypt_ecb(data)
    return en_data

def sm4_crypt_cbc(mode, key, iv, data):
    sm4_d = Sm4()
    sm4_d.sm4_set_key(key, mode)
    en_data = sm4_d.sm4_crypt_cbc(iv, data)
    return en_data

if __name__ == "__main__":
    # log_init()

    key_data = [0x5a] * 16
    input_data = [0x6b] * 128
    iv_data = [0] * 16

    sm4_d = Sm4()
    sm4_d.sm4_set_key(key_data, ENCRYPT)

    en_data = sm4_d.sm4_crypt_ecb(input_data)
    print(en_data, "en_data:")
    sm4_d.sm4_set_key(key_data, DECRYPT)
    de_data = sm4_d.sm4_crypt_ecb(en_data)
    print(de_data, "de_data:")
    if de_data == input_data:
        print("ecb check pass")
    else:
        print("ecb check fail")
        raise BaseException("error")

    sm4_d.sm4_set_key(key_data, ENCRYPT)
    en_data = sm4_d.sm4_crypt_cbc(iv_data, input_data)
    print(en_data, "en_data:")
    sm4_d.sm4_set_key(key_data, DECRYPT)
    de_data = sm4_d.sm4_crypt_cbc(iv_data, en_data)
    print(de_data, "de_data:")
    if de_data == input_data:
        print("cbc check pass")
    else:
        print("cbc check fail")
        raise BaseException("error")
    # file test
    file_path = r"test2.txt"
    ecb_path_en = r"test2k_ecb_en.txt"
    ecb_path_de = r"test2k_ecb_de.txt"
    cbc_path_en = r"test2k_cbc_en.txt"
    cbc_path_de = r"test2k_cbc_de.txt"

    key_data = [0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08,
                0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08]
    iv_data = [0x5a] * 16

    with open(file_path, 'rb') as f:
        file_data = f.read()
    # file_data_list = [ord(x) for x in file_data]
    file_data_list = list(file_data)
    # 1. ECB
    sm4_d = Sm4()
    sm4_d.sm4_set_key(key_data, ENCRYPT)
    en_data = sm4_d.sm4_crypt_ecb(file_data_list)
    with open(ecb_path_en, 'w+b') as f:
        f.write(bytes(en_data))

    sm4_d.sm4_set_key(key_data, DECRYPT)
    de_data = sm4_d.sm4_crypt_ecb(en_data)
    with open(ecb_path_de, 'w+b') as f:
        f.write(bytes(de_data))
    if de_data == file_data_list:
        print("file decode pass")
    else:
        print("file decode fail")
        raise BaseException('error')

    # 2. CBC
    sm4_d.sm4_set_key(key_data, ENCRYPT)
    en_data = sm4_d.sm4_crypt_cbc(iv_data, file_data_list)
    with open(cbc_path_en, 'w+b') as f:
        f.write(bytes(en_data))

    sm4_d.sm4_set_key(key_data, DECRYPT)
    de_data = sm4_d.sm4_crypt_cbc(iv_data, en_data)
    with open(cbc_path_de, 'w+b') as f:
        f.write(bytes(de_data))
    if de_data == file_data_list:
        print("file decode pass")
    else:
        print("file decode fail")
        raise BaseException("error")
    # log_end()
```

参考资料：

[SM4加密算法原理和简单实现（java） - kentle - 博客园](https://www.cnblogs.com/kentle/p/14135865.html)

[gmbz.org.cn/main/viewfile/20180108015408199368.html](http://www.gmbz.org.cn/main/viewfile/20180108015408199368.html)
