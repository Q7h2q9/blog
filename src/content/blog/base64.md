---
title: "base64（含变表）"
description: "base64加解密时通常会遇到换表的情况，该脚本可根据自己需求调整码表"
pubDate: "October 22 2024"
image: /images/5.jpg
categories:
  - tech
tags:
  - 编码算法
---

## base64

### 1.索引表

首先有一个长度为64的索引表（常规base64都是用下面这个）

```py
"ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"
```

当然这个索引表也可以换，例如

```py
"WHydo3sThiS7ABLElO0k5trange+CZfVIGRvup81NKQbjmPzU4MDc9Y6q2XwFxJ/"
```

### 2.具体转换步骤

- 第一步，将待转换的字符串每三个字节分为一组，每个字节占8bit，那么共有24个二进制位。
- 第二步，将上面的24个二进制位每6个一组，共分为4组。
- 第三步，在每组前面添加两个0，每组由6个变为8个二进制位，总共32个二进制位，即四个字节。
- 第四步，每个字节的值作为索引，在索引表获得对应的值。

eg：![图片](https://cdn.jsdelivr.net/gh/quanfanghe/photohouse/picgoandgithub/202411281141588.png)

也就是 3x8=>4x6，然后对着索引表获得值

这是编码过程，解码就是倒着来，先对着索引表获得对应的下标，然后4x6=>3x8（要先把补的那两个0去掉）

加解密过程如下（含变表）：

```python
# coding:utf-8

s = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"
# s = "WHydo3sThiS7ABLElO0k5trange+CZfVIGRvup81NKQbjmPzU4MDc9Y6q2XwFxJ/"

def My_base64_encode(inputs):
    # 将字符串转化为2进制
    bin_str = []
    for i in inputs:
        x = str(bin(ord(i))).replace('0b', '')
        bin_str.append('{:0>8}'.format(x))

    # 输出的字符串
    outputs = ""
    # 不够三倍数，需补齐的次数
    nums = 0
    while bin_str:
        # 每次取三个字符的二进制
        temp_list = bin_str[:3]
        if len(temp_list) != 3:
            nums = 3 - len(temp_list)
            while len(temp_list) < 3:
                temp_list += ['0' * 8]
        temp_str = "".join(temp_list)

        # 将三个8字节的二进制转换为4个十进制
        temp_str_list = []
        for i in range(0, 4):
            temp_str_list.append(int(temp_str[i*6:(i+1)*6], 2))

        if nums:
            temp_str_list = temp_str_list[0:4 - nums]

        for i in temp_str_list:
            outputs += s[i]

        bin_str = bin_str[3:]

    outputs += nums * '='
    print("Encrypted String:\n%s" % outputs)
    return outputs

def My_base64_decode(inputs):
    # 将字符串转化为2进制
    bin_str = []
    for i in inputs:
        if i != '=':
            x = str(bin(s.index(i))).replace('0b', '')
            bin_str.append('{:0>6}'.format(x))

    # 输出的字符串
    outputs = ""
    hex_outputs = ""  # 用于存储十六进制形式的输出
    byte_array = []  # 用于存储字节数组
    nums = inputs.count('=')
    while bin_str:
        temp_list = bin_str[:4]
        temp_str = "".join(temp_list)

        # 补足8位字节
        if len(temp_str) % 8 != 0:
            temp_str = temp_str[0:-1 * nums * 2]

        # 将四个6字节的二进制转换为三个字符
        for i in range(0, int(len(temp_str) / 8)):
            char = chr(int(temp_str[i*8:(i+1)*8], 2))
            outputs += char
            # 添加到十六进制输出
            hex_outputs += '{:02X} '.format(ord(char))
            # 添加到字节数组
            byte_array.append(ord(char))

        bin_str = bin_str[4:]

    print("Decrypted String:\n%s" % outputs)
    #print("Hexadecimal Representation of Decrypted String:\n%s" % hex_outputs.strip())输出解码后的十六进制
    #print("Byte Array of Decrypted String:\n%s" % byte_array)输出解码后的字节
    return byte_array

print()
print("     *************************************")
print("     *    (1)encode         (2)decode    *")
print("     *************************************")
print()

num = input("Please select the operation you want to perform:\n")
if num == "1":
    input_str = input("Please enter a string that needs to be encrypted: \n")
    encoded_str = My_base64_encode(input_str)
else:
    input_str = input("Please enter a string that needs to be decrypted: \n")
    decoded_bytes = My_base64_decode(input_str)
```

参考资料：[彻底弄懂base64的编码与解码原理\_base64解码-CSDN博客](https://blog.csdn.net/Jernnifer_mao/article/details/133981806)
