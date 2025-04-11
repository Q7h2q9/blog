---
title: "rc4算法"
description: "标准rc4加解密（用的同一套代码）"
pubDate: "October 22 2024"
image: /images/10.png
categories:
  - tech
tags:
  - 加密算法
---

## rc4

在密码学中，**RC4**（来自Rivest Cipher 4的缩写）是一种**流加密算法**，**密钥长度可变**。它加解密使用相同的密钥，因此也属于**对称加密算法**。

RC4是流密码streamcipher中的一种，为序列密码。RC4加密算法是Ron Rivest在1987年设计出的密钥长度可变的加密算法簇。起初该算法是商业机密，直到1994年，它才公诸于众。由于RC4具有算法简单，运算速度快，软硬件实现都十分容易等优点，使其在一些协议和标准里得到了广泛应用。

### 关键变量

1、**密钥流**：RC4算法的关键是根据明文和密钥生成相应的密钥流，密钥流的长度和明文的长度是对应的，也就是说明文的长度是500字节，那么密钥流也是500字节。当然，加密生成的密文也是500字节，因为**密文第i字节=明文第i字节^密钥流第i字节**；

    2、**状态向量S**：长度为256，S[0],S[1].....S[255]。每个单元都是一个字节，算法运行的任何时候，S都包括0-255的8比特数的排列组合，只不过值的位置发生了变换；

    3、**临时向量T**：长度也为256，每个单元也是一个字节。如果密钥的长度是256字节，就直接把密钥的值赋给T，否则，轮转地将密钥的每个字节赋给T；

    4、**密钥K**：长度为1-256字节，注意密钥的长度keylen与明文长度、密钥流的长度没有必然关系，通常密钥的长度为16字节（128比特）。

### 完整代码：

- c

```c++
#include <stdio.h>
#include <string.h>

// 初始化 S 盒
void rc4_init(unsigned char *s, const unsigned char *key, int keylen) {
	int i, j = 0;
	for (i = 0; i < 256; ++i) {
		s[i] = i;
	}
	for (i = 0; i < 256; ++i) {
		j = (j + s[i] + key[i % keylen]) % 256;
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

	// 明/密文
	unsigned char v4[] = {0x91,0x86,0x1b,0x2d,0x9e,0x6f
		,0x57,0x62,0x77,0xf3,0xf2,0xee,0xd2,0x92,
		0x22,0x14,0x41,0xee,0x57,0x2d,0x80,0x28,
		0x4a,0x69,0x6c,0x4f,0x80,0x77,0x7c,0x26,
		0xb5,0x9b,0x38,0x75,0x2e,0xcf,0x5,0x82,0x63,0xf6,0xb,0x89,0xa6,};
	int len = sizeof(v4) / sizeof(v4[0]);
	// 初始化 S 盒
	rc4_init(s, (const unsigned char *)key, keylen);

	// 加/解密数据
	rc4_crypt(s, v4, len);
	// 输出解密后的数据
	for (int i = 0; i < len; i++) {
		printf("%c", v4[i]);
	}
	printf("\n");

	return 0;
}
```

- python

```py
def KSA(key):
    """ Key-Scheduling Algorithm (KSA) """
    S = list(range(256))
    j = 0
    for i in range(256):
        j = (j + S[i] + key[i % len(key)]) % 256
        S[i], S[j] = S[j], S[i]
    return S

def PRGA(S):
    """ Pseudo-Random Generation Algorithm (PRGA) """
    i, j = 0, 0
    while True:
        i = (i + 1) % 256
        j = (j + S[i]) % 256
        S[i], S[j] = S[j], S[i]
        K = S[(S[i] + S[j]) % 256]
        yield K

def RC4(key, text):
    """ RC4 encryption/decryption """
    S = KSA(key)
    keystream = PRGA(S)
    res = []
    for char in text:
        res.append(char ^ next(keystream))
    return bytes(res)

key = b'example_key'
plaintext = b'hello world'

ciphertext = RC4(key, plaintext)
print(ciphertext)
```

参考资料：[RC4加密解密算法\_rc4解密-CSDN博客](https://blog.csdn.net/huangyimo/article/details/82980364)
