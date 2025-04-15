---
title: "TGCTF2025 (RE)复现"
description: "我要成为复现糕手(哭"
pubDate: "April 15 2025"
image: /image/TGCTF2025.png
categories:
  - tech
tags:
  - 复现
  - CTF
---

## TGCTF2025 (RE)复现

> 又拖队伍后腿了呜呜呜

### 蛇年的本命语言

#### 逻辑：

对输入的字符串进行计数，并转成字符串并与目标串比较。如"abdcba"-->"2211"

遍历输入的字符串，每遇到一次不同字符就给`str`​中添加**该字符**以及该字符**在整个字符串中出现的次数。**

检查`str`​，满足方程式则判对

#### 复现

拿到一个python的exe文件，用`pyinstxtractor.py`​解包后找到`output.pyc`​，找在线网站反编译

得到伪代码，有混淆，简单替换一下名字，代码如下：

```python
# uncompyle6 version 3.9.2
# Python bytecode version base 3.8.0 (3413)
# Decompiled from: Python 3.6.12 (default, Feb  9 2021, 09:19:15)
# [GCC 8.3.0]
# Embedded file name: output.py
from collections import Counter
print("Welcome to HZNUCTF!!!")
print("Plz input the flag:")
input_flag = input()
count_l = Counter(input_flag)
count_str = "".join((str(count_l[i]) for i in count_l))
print("ans1: ", end="")
print(count_str)
if count_str != "111111116257645365477364777645752361":
    print("wrong_wrong!!!")
    exit(1)
str2 = ""
for i in input_flag:
    if count_l[i] > 0:
        str2 += i + str(count_l[i])
        count_l[i] = 0
    else:
        arr = [ord(i) for i in str2]
        t = [
         7 * arr[0] == 504,
         9 * arr[0] - 5 * arr[1] == 403,
         2 * arr[0] - 5 * arr[1] + 10 * arr[2] == 799,
         3 * arr[0] + 8 * arr[1] + 15 * arr[2] + 20 * arr[3] == 2938,
         5 * arr[0] + 15 * arr[1] + 20 * arr[2] - 19 * arr[3] + 1 * arr[4] == 2042,
         7 * arr[0] + 1 * arr[1] + 9 * arr[2] - 11 * arr[3] + 2 * arr[4] + 5 * arr[5] == 1225,
         11 * arr[0] + 22 * arr[1] + 33 * arr[2] + 44 * arr[3] + 55 * arr[4] + 66 * arr[5] - 77 * arr[6] == 7975,
         21 * arr[0] + 23 * arr[1] + 3 * arr[2] + 24 * arr[3] - 55 * arr[4] + 6 * arr[5] - 7 * arr[6] + 15 * arr[7] == 229,
         2 * arr[0] + 26 * arr[1] + 13 * arr[2] + 0 * arr[3] - 65 * arr[4] + 15 * arr[5] + 29 * arr[6] + 1 * arr[7] + 20 * arr[8] == 2107,
         10 * arr[0] + 7 * arr[1] + -9 * arr[2] + 6 * arr[3] + 7 * arr[4] + 1 * arr[5] + 22 * arr[6] + 21 * arr[7] - 22 * arr[8] + 30 * arr[9] == 4037,
         15 * arr[0] + 59 * arr[1] + 56 * arr[2] + 66 * arr[3] + 7 * arr[4] + 1 * arr[5] - 122 * arr[6] + 21 * arr[7] + 32 * arr[8] + 3 * arr[9] - 10 * arr[10] == 4950,
         13 * arr[0] + 66 * arr[1] + 29 * arr[2] + 39 * arr[3] - 33 * arr[4] + 13 * arr[5] - 2 * arr[6] + 42 * arr[7] + 62 * arr[8] + 1 * arr[9] - 10 * arr[10] + 11 * arr[11] == 12544,
         23 * arr[0] + 6 * arr[1] + 29 * arr[2] + 3 * arr[3] - 3 * arr[4] + 63 * arr[5] - 25 * arr[6] + 2 * arr[7] + 32 * arr[8] + 1 * arr[9] - 10 * arr[10] + 11 * arr[11] - 12 * arr[12] == 6585,
         223 * arr[0] + 6 * arr[1] - 29 * arr[2] - 53 * arr[3] - 3 * arr[4] + 3 * arr[5] - 65 * arr[6] + 0 * arr[7] + 36 * arr[8] + 1 * arr[9] - 15 * arr[10] + 16 * arr[11] - 18 * arr[12] + 13 * arr[13] == 6893,
         29 * arr[0] + 13 * arr[1] - 9 * arr[2] - 93 * arr[3] + 33 * arr[4] + 6 * arr[5] + 65 * arr[6] + 1 * arr[7] - 36 * arr[8] + 0 * arr[9] - 16 * arr[10] + 96 * arr[11] - 68 * arr[12] + 33 * arr[13] - 14 * arr[14] == 1883,
         69 * arr[0] + 77 * arr[1] - 93 * arr[2] - 12 * arr[3] + 0 * arr[4] + 0 * arr[5] + 1 * arr[6] + 16 * arr[7] + 36 * arr[8] + 6 * arr[9] + 19 * arr[10] + 66 * arr[11] - 8 * arr[12] + 38 * arr[13] - 16 * arr[14] + 15 * arr[15] == 8257,
         23 * arr[0] + 2 * arr[1] - 3 * arr[2] - 11 * arr[3] + 12 * arr[4] + 24 * arr[5] + 1 * arr[6] + 6 * arr[7] + 14 * arr[8] - 0 * arr[9] + 1 * arr[10] + 68 * arr[11] - 18 * arr[12] + 68 * arr[13] - 26 * arr[14] + 15 * arr[15] - 16 * arr[16] == 5847,
         24 * arr[0] + 0 * arr[1] - 1 * arr[2] - 15 * arr[3] + 13 * arr[4] + 4 * arr[5] + 16 * arr[6] + 67 * arr[7] + 146 * arr[8] - 50 * arr[9] + 16 * arr[10] + 6 * arr[11] - 1 * arr[12] + 69 * arr[13] - 27 * arr[14] + 45 * arr[15] - 6 * arr[16] + 17 * arr[17] == 18257,
         25 * arr[0] + 26 * arr[1] - 89 * arr[2] + 16 * arr[3] + 19 * arr[4] + 44 * arr[5] + 36 * arr[6] + 66 * arr[7] - 150 * arr[8] - 250 * arr[9] + 166 * arr[10] + 126 * arr[11] - 11 * arr[12] + 690 * arr[13] - 207 * arr[14] + 46 * arr[15] + 6 * arr[16] + 7 * arr[17] - 18 * arr[18] == 12591,
         5 * arr[0] + 26 * arr[1] + 8 * arr[2] + 160 * arr[3] + 9 * arr[4] - 4 * arr[5] + 36 * arr[6] + 6 * arr[7] - 15 * arr[8] - 20 * arr[9] + 66 * arr[10] + 16 * arr[11] - 1 * arr[12] + 690 * arr[13] - 20 * arr[14] + 46 * arr[15] + 6 * arr[16] + 7 * arr[17] - 18 * arr[18] + 19 * arr[19] == 52041,
         29 * arr[0] - 26 * arr[1] + 0 * arr[2] + 60 * arr[3] + 90 * arr[4] - 4 * arr[5] + 6 * arr[6] + 6 * arr[7] - 16 * arr[8] - 21 * arr[9] + 69 * arr[10] + 6 * arr[11] - 12 * arr[12] + 69 * arr[13] - 20 * arr[14] - 46 * arr[15] + 65 * arr[16] + 0 * arr[17] - 1 * arr[18] + 39 * arr[19] - 20 * arr[20] == 20253,
         45 * arr[0] - 56 * arr[1] + 10 * arr[2] + 650 * arr[3] - 900 * arr[4] + 44 * arr[5] + 66 * arr[6] - 6 * arr[7] - 6 * arr[8] - 21 * arr[9] + 9 * arr[10] - 6 * arr[11] - 12 * arr[12] + 69 * arr[13] - 2 * arr[14] - 406 * arr[15] + 651 * arr[16] + 2 * arr[17] - 10 * arr[18] + 69 * arr[19] - 0 * arr[20] + 21 * arr[21] == 18768,
         555 * arr[0] - 6666 * arr[1] + 70 * arr[2] + 510 * arr[3] - 90 * arr[4] + 499 * arr[5] + 66 * arr[6] - 66 * arr[7] - 610 * arr[8] - 221 * arr[9] + 9 * arr[10] - 23 * arr[11] - 102 * arr[12] + 6 * arr[13] + 2050 * arr[14] - 406 * arr[15] + 665 * arr[16] + 333 * arr[17] + 100 * arr[18] + 609 * arr[19] + 777 * arr[20] + 201 * arr[21] - 22 * arr[22] == 111844,
         1 * arr[0] - 22 * arr[1] + 333 * arr[2] + 4444 * arr[3] - 5555 * arr[4] + 6666 * arr[5] - 666 * arr[6] + 676 * arr[7] - 660 * arr[8] - 22 * arr[9] + 9 * arr[10] - 73 * arr[11] - 107 * arr[12] + 6 * arr[13] + 250 * arr[14] - 6 * arr[15] + 65 * arr[16] + 39 * arr[17] + 10 * arr[18] + 69 * arr[19] + 777 * arr[20] + 201 * arr[21] - 2 * arr[22] + 23 * arr[23] == 159029,
         520 * arr[0] - 222 * arr[1] + 333 * arr[2] + 4 * arr[3] - 56655 * arr[4] + 6666 * arr[5] + 666 * arr[6] + 66 * arr[7] - 60 * arr[8] - 220 * arr[9] + 99 * arr[10] + 73 * arr[11] + 1007 * arr[12] + 7777 * arr[13] + 2500 * arr[14] + 6666 * arr[15] + 605 * arr[16] + 390 * arr[17] + 100 * arr[18] + 609 * arr[19] + 99999 * arr[20] + 210 * arr[21] + 232 * arr[22] + 23 * arr[23] - 24 * arr[24] == 2762025,
         1323 * arr[0] - 22 * arr[1] + 333 * arr[2] + 4 * arr[3] - 55 * arr[4] + 666 * arr[5] + 666 * arr[6] + 66 * arr[7] - 660 * arr[8] - 220 * arr[9] + 99 * arr[10] + 3 * arr[11] + 100 * arr[12] + 777 * arr[13] + 2500 * arr[14] + 6666 * arr[15] + 605 * arr[16] + 390 * arr[17] + 100 * arr[18] + 609 * arr[19] + 9999 * arr[20] + 210 * arr[21] + 232 * arr[22] + 23 * arr[23] - 24 * arr[24] + 25 * arr[25] == 1551621,
         777 * arr[0] - 22 * arr[1] + 6969 * arr[2] + 4 * arr[3] - 55 * arr[4] + 666 * arr[5] - 6 * arr[6] + 96 * arr[7] - 60 * arr[8] - 220 * arr[9] + 99 * arr[10] + 3 * arr[11] + 100 * arr[12] + 777 * arr[13] + 250 * arr[14] + 666 * arr[15] + 65 * arr[16] + 90 * arr[17] + 100 * arr[18] + 609 * arr[19] + 999 * arr[20] + 21 * arr[21] + 232 * arr[22] + 23 * arr[23] - 24 * arr[24] + 25 * arr[25] - 26 * arr[26] == 948348,
         97 * arr[0] - 22 * arr[1] + 6969 * arr[2] + 4 * arr[3] - 56 * arr[4] + 96 * arr[5] - 6 * arr[6] + 96 * arr[7] - 60 * arr[8] - 20 * arr[9] + 99 * arr[10] + 3 * arr[11] + 10 * arr[12] + 707 * arr[13] + 250 * arr[14] + 666 * arr[15] + -9 * arr[16] + 90 * arr[17] + -2 * arr[18] + 609 * arr[19] + 0 * arr[20] + 21 * arr[21] + 2 * arr[22] + 23 * arr[23] - 24 * arr[24] + 25 * arr[25] - 26 * arr[26] + 27 * arr[27] == 777044,
         177 * arr[0] - 22 * arr[1] + 699 * arr[2] + 64 * arr[3] - 56 * arr[4] - 96 * arr[5] - 66 * arr[6] + 96 * arr[7] - 60 * arr[8] - 20 * arr[9] + 99 * arr[10] + 3 * arr[11] + 10 * arr[12] + 707 * arr[13] + 250 * arr[14] + 666 * arr[15] + -9 * arr[16] + 0 * arr[17] + -2 * arr[18] + 69 * arr[19] + 0 * arr[20] + 21 * arr[21] + 222 * arr[22] + 23 * arr[23] - 224 * arr[24] + 25 * arr[25] - 26 * arr[26] + 27 * arr[27] - 28 * arr[28] == 185016,
         77 * arr[0] - 2 * arr[1] + 6 * arr[2] + 6 * arr[3] - 96 * arr[4] - 9 * arr[5] - 6 * arr[6] + 96 * arr[7] - 0 * arr[8] - 20 * arr[9] + 99 * arr[10] + 3 * arr[11] + 10 * arr[12] + 707 * arr[13] + 250 * arr[14] + 666 * arr[15] + -9 * arr[16] + 0 * arr[17] + -2 * arr[18] + 9 * arr[19] + 0 * arr[20] + 21 * arr[21] + 222 * arr[22] + 23 * arr[23] - 224 * arr[24] + 26 * arr[25] - -58 * arr[26] + 27 * arr[27] - 2 * arr[28] + 29 * arr[29] == 130106]
        if all(t):
            print("Congratulation!!!")
        else:
            print("wrong_wrong!!!")

```

逻辑大概就是

- 对输入的字符串进行计数，并转成字符串并与目标串比较。如"abdcba"-->"2211"

- 遍历输入的字符串，每遇到一次不同字符就给`str`​中添加**该字符**以及该字符**在整个字符串中出现的次数。**

- 检查`str`​，满足方程式则判对

先用z3求解

```python
from z3 import *
from collections import Counter
# 求解方程组
def solve_equations():
    # 创建solver
    s = Solver()

    # 创建30个变量(根据方程组数量)
    val = [Int(f'val_{i}') for i in range(30)]

    # 添加方程约束
    s.add(7 * val[0] == 504)
    s.add(9 * val[0] - 5 * val[1] == 403)
    s.add(2 * val[0] - 5 * val[1] + 10 * val[2] == 799)
    s.add(3 * val[0] + 8 * val[1] + 15 * val[2] + 20 * val[3] == 2938)
    s.add(5 * val[0] + 15 * val[1] + 20 * val[2] - 19 * val[3] + 1 * val[4] == 2042)
    s.add(7 * val[0] + 1 * val[1] + 9 * val[2] - 11 * val[3] + 2 * val[4] + 5 * val[5] == 1225)
    s.add(11 * val[0] + 22 * val[1] + 33 * val[2] + 44 * val[3] + 55 * val[4] + 66 * val[5] - 77 * val[6] == 7975)
    s.add(21 * val[0] + 23 * val[1] + 3 * val[2] + 24 * val[3] - 55 * val[4] + 6 * val[5] - 7 * val[6] + 15 * val[7] == 229)
    s.add(2 * val[0] + 26 * val[1] + 13 * val[2] + 0 * val[3] - 65 * val[4] + 15 * val[5] + 29 * val[6] + 1 * val[7] + 20 * val[8] == 2107)
    s.add(10 * val[0] + 7 * val[1] + -9 * val[2] + 6 * val[3] + 7 * val[4] + 1 * val[5] + 22 * val[6] + 21 * val[7] - 22 * val[8] + 30 * val[9] == 4037)
    s.add(15 * val[0] + 59 * val[1] + 56 * val[2] + 66 * val[3] + 7 * val[4] + 1 * val[5] - 122 * val[6] + 21 * val[7] + 32 * val[8] + 3 * val[9] - 10 * val[10] == 4950)
    s.add(13 * val[0] + 66 * val[1] + 29 * val[2] + 39 * val[3] - 33 * val[4] + 13 * val[5] - 2 * val[6] + 42 * val[7] + 62 * val[8] + 1 * val[9] - 10 * val[10] + 11 * val[11] == 12544)
    s.add(23 * val[0] + 6 * val[1] + 29 * val[2] + 3 * val[3] - 3 * val[4] + 63 * val[5] - 25 * val[6] + 2 * val[7] + 32 * val[8] + 1 * val[9] - 10 * val[10] + 11 * val[11] - 12 * val[12] == 6585)
    s.add(223 * val[0] + 6 * val[1] - 29 * val[2] - 53 * val[3] - 3 * val[4] + 3 * val[5] - 65 * val[6] + 0 * val[7] + 36 * val[8] + 1 * val[9] - 15 * val[10] + 16 * val[11] - 18 * val[12] + 13 * val[13] == 6893)
    s.add(29 * val[0] + 13 * val[1] - 9 * val[2] - 93 * val[3] + 33 * val[4] + 6 * val[5] + 65 * val[6] + 1 * val[7] - 36 * val[8] + 0 * val[9] - 16 * val[10] + 96 * val[11] - 68 * val[12] + 33 * val[13] - 14 * val[14] == 1883)
    s.add(69 * val[0] + 77 * val[1] - 93 * val[2] - 12 * val[3] + 0 * val[4] + 0 * val[5] + 1 * val[6] + 16 * val[7] + 36 * val[8] + 6 * val[9] + 19 * val[10] + 66 * val[11] - 8 * val[12] + 38 * val[13] - 16 * val[14] + 15 * val[15] == 8257)
    s.add(23 * val[0] + 2 * val[1] - 3 * val[2] - 11 * val[3] + 12 * val[4] + 24 * val[5] + 1 * val[6] + 6 * val[7] + 14 * val[8] - 0 * val[9] + 1 * val[10] + 68 * val[11] - 18 * val[12] + 68 * val[13] - 26 * val[14] + 15 * val[15] - 16 * val[16] == 5847)
    s.add(24 * val[0] + 0 * val[1] - 1 * val[2] - 15 * val[3] + 13 * val[4] + 4 * val[5] + 16 * val[6] + 67 * val[7] + 146 * val[8] - 50 * val[9] + 16 * val[10] + 6 * val[11] - 1 * val[12] + 69 * val[13] - 27 * val[14] + 45 * val[15] - 6 * val[16] + 17 * val[17] == 18257)
    s.add(25 * val[0] + 26 * val[1] - 89 * val[2] + 16 * val[3] + 19 * val[4] + 44 * val[5] + 36 * val[6] + 66 * val[7] - 150 * val[8] - 250 * val[9] + 166 * val[10] + 126 * val[11] - 11 * val[12] + 690 * val[13] - 207 * val[14] + 46 * val[15] + 6 * val[16] + 7 * val[17] - 18 * val[18] == 12591)
    s.add(5 * val[0] + 26 * val[1] + 8 * val[2] + 160 * val[3] + 9 * val[4] - 4 * val[5] + 36 * val[6] + 6 * val[7] - 15 * val[8] - 20 * val[9] + 66 * val[10] + 16 * val[11] - 1 * val[12] + 690 * val[13] - 20 * val[14] + 46 * val[15] + 6 * val[16] + 7 * val[17] - 18 * val[18] + 19 * val[19] == 52041)
    s.add(29 * val[0] - 26 * val[1] + 0 * val[2] + 60 * val[3] + 90 * val[4] - 4 * val[5] + 6 * val[6] + 6 * val[7] - 16 * val[8] - 21 * val[9] + 69 * val[10] + 6 * val[11] - 12 * val[12] + 69 * val[13] - 20 * val[14] - 46 * val[15] + 65 * val[16] + 0 * val[17] - 1 * val[18] + 39 * val[19] - 20 * val[20] == 20253)
    s.add(45 * val[0] - 56 * val[1] + 10 * val[2] + 650 * val[3] - 900 * val[4] + 44 * val[5] + 66 * val[6] - 6 * val[7] - 6 * val[8] - 21 * val[9] + 9 * val[10] - 6 * val[11] - 12 * val[12] + 69 * val[13] - 2 * val[14] - 406 * val[15] + 651 * val[16] + 2 * val[17] - 10 * val[18] + 69 * val[19] - 0 * val[20] + 21 * val[21] == 18768)
    s.add(555 * val[0] - 6666 * val[1] + 70 * val[2] + 510 * val[3] - 90 * val[4] + 499 * val[5] + 66 * val[6] - 66 * val[7] - 610 * val[8] - 221 * val[9] + 9 * val[10] - 23 * val[11] - 102 * val[12] + 6 * val[13] + 2050 * val[14] - 406 * val[15] + 665 * val[16] + 333 * val[17] + 100 * val[18] + 609 * val[19] + 777 * val[20] + 201 * val[21] - 22 * val[22] == 111844)
    s.add(1 * val[0] - 22 * val[1] + 333 * val[2] + 4444 * val[3] - 5555 * val[4] + 6666 * val[5] - 666 * val[6] + 676 * val[7] - 660 * val[8] - 22 * val[9] + 9 * val[10] - 73 * val[11] - 107 * val[12] + 6 * val[13] + 250 * val[14] - 6 * val[15] + 65 * val[16] + 39 * val[17] + 10 * val[18] + 69 * val[19] + 777 * val[20] + 201 * val[21] - 2 * val[22] + 23 * val[23] == 159029)
    s.add(520 * val[0] - 222 * val[1] + 333 * val[2] + 4 * val[3] - 56655 * val[4] + 6666 * val[5] + 666 * val[6] + 66 * val[7] - 60 * val[8] - 220 * val[9] + 99 * val[10] + 73 * val[11] + 1007 * val[12] + 7777 * val[13] + 2500 * val[14] + 6666 * val[15] + 605 * val[16] + 390 * val[17] + 100 * val[18] + 609 * val[19] + 99999 * val[20] + 210 * val[21] + 232 * val[22] + 23 * val[23] - 24 * val[24] == 2762025)
    s.add(1323 * val[0] - 22 * val[1] + 333 * val[2] + 4 * val[3] - 55 * val[4] + 666 * val[5] + 666 * val[6] + 66 * val[7] - 660 * val[8] - 220 * val[9] + 99 * val[10] + 3 * val[11] + 100 * val[12] + 777 * val[13] + 2500 * val[14] + 6666 * val[15] + 605 * val[16] + 390 * val[17] + 100 * val[18] + 609 * val[19] + 9999 * val[20] + 210 * val[21] + 232 * val[22] + 23 * val[23] - 24 * val[24] + 25 * val[25] == 1551621)
    s.add(777 * val[0] - 22 * val[1] + 6969 * val[2] + 4 * val[3] - 55 * val[4] + 666 * val[5] - 6 * val[6] + 96 * val[7] - 60 * val[8] - 220 * val[9] + 99 * val[10] + 3 * val[11] + 100 * val[12] + 777 * val[13] + 250 * val[14] + 666 * val[15] + 65 * val[16] + 90 * val[17] + 100 * val[18] + 609 * val[19] + 999 * val[20] + 21 * val[21] + 232 * val[22] + 23 * val[23] - 24 * val[24] + 25 * val[25] - 26 * val[26] == 948348)
    s.add(97 * val[0] - 22 * val[1] + 6969 * val[2] + 4 * val[3] - 56 * val[4] + 96 * val[5] - 6 * val[6] + 96 * val[7] - 60 * val[8] - 20 * val[9] + 99 * val[10] + 3 * val[11] + 10 * val[12] + 707 * val[13] + 250 * val[14] + 666 * val[15] + -9 * val[16] + 90 * val[17] + -2 * val[18] + 609 * val[19] + 0 * val[20] + 21 * val[21] + 2 * val[22] + 23 * val[23] - 24 * val[24] + 25 * val[25] - 26 * val[26] + 27 * val[27] == 777044)
    s.add(177 * val[0] - 22 * val[1] + 699 * val[2] + 64 * val[3] - 56 * val[4] - 96 * val[5] - 66 * val[6] + 96 * val[7] - 60 * val[8] - 20 * val[9] + 99 * val[10] + 3 * val[11] + 10 * val[12] + 707 * val[13] + 250 * val[14] + 666 * val[15] + -9 * val[16] + 0 * val[17] + -2 * val[18] + 69 * val[19] + 0 * val[20] + 21 * val[21] + 222 * val[22] + 23 * val[23] - 224 * val[24] + 25 * val[25] - 26 * val[26] + 27 * val[27] - 28 * val[28] == 185016)
    s.add(77 * val[0] - 2 * val[1] + 6 * val[2] + 6 * val[3] - 96 * val[4] - 9 * val[5] - 6 * val[6] + 96 * val[7] - 0 * val[8] - 20 * val[9] + 99 * val[10] + 3 * val[11] + 10 * val[12] + 707 * val[13] + 250 * val[14] + 666 * val[15] + -9 * val[16] + 0 * val[17] + -2 * val[18] + 9 * val[19] + 0 * val[20] + 21 * val[21] + 222 * val[22] + 23 * val[23] - 224 * val[24] + 26 * val[25] - -58 * val[26] + 27 * val[27] - 2 * val[28] + 29 * val[29] == 130106)

    if s.check() == sat:
        m = s.model()
        # 获取结果
        result = [m[val[i]].as_long() for i in range(30)]
        # 转换为字符
        flag = ''.join(chr(x) for x in result)
        return flag
    else:
        return "No solution found"

def main():
    flag = solve_equations()
    print(flag)
if __name__ == "__main__":
    main()
#H1Z1N1U1C1T1F1{1a6d275f7-463}1
```

拿到的`flag`​中字符的种类以及每个字符出现的**次数**

对着`"111111116257645365477364777645752361"`​，还原得到flag：

HZNUCTF{ad7fa-76a7-ff6a-fffa-7f7d6a}

> PS：拿着`flag`​回去验证发现是`wrong`​，它的逻辑似乎有点问题，按照WP的意思是z3解出每个字符及其出现的次数，然后`"111111116257645365477364777645752361"`​的意思是出现次数为`str[i]`​的字符在`flag`​的第`i`​位，考虑到格式是`HZNUCTF{}`​，那么中间`625764536547736477764575236`​对应的次数刚好和`a6d275f7-463`​中得出的每个字符出现次数刚好一一对应，这样就可以由出现次数反推`flag[i]`​是什么了

‍

### exchange

#### 补基础知识

‍[通俗易懂，十分钟读懂DES，详解DES加密算法原理，DES攻击手段以及3DES原理。Python DES实现源码-CSDN博客](https://blog.csdn.net/Demonslzh/article/details/129129493)

以及一个没有问题的`DES`加解密**c**代码：[des/3des算法C语言实现 - 简书](https://www.jianshu.com/p/850f3e18dddb)

#### 逻辑

获取用户输入，判断前8位是否为`HZNUCTF{`​以及最后一位是否为`}`​，中间部分要求长度为`32`​字节

将字符串的每两位转成对应的十六进制(如："14"-->"3134")，然后将中间两位翻转("3134"-->"3314")，得到长度为`64`​字节的字符串

将字符串进行`DES`​加密，密钥为`HZNUCTF{`​。

加密结果与目标数组比较，不一样输出`wrong`

#### 分析

刚开始给的文件显示无法打开，用`die`​看看发现有upx壳，去壳后就能运行了

> ps:看WP说把某一位改一下也行？不懂

定位到main，逻辑很清晰，简单改一下函数名

```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  char *v3; // rdi
  __int64 i; // rcx
  char v6[32]; // [rsp+0h] [rbp-20h] BYREF
  char v7; // [rsp+20h] [rbp+0h] BYREF
  char *Source; // [rsp+28h] [rbp+8h]
  char Destination[44]; // [rsp+48h] [rbp+28h] BYREF
  char Str1[36]; // [rsp+74h] [rbp+54h] BYREF
  char *v11; // [rsp+98h] [rbp+78h]
  char v12[65]; // [rsp+C0h] [rbp+A0h] BYREF
  int j; // [rsp+124h] [rbp+104h]
  unsigned __int8 v14; // [rsp+144h] [rbp+124h]
  unsigned __int8 v15; // [rsp+164h] [rbp+144h]
  char v16[32]; // [rsp+184h] [rbp+164h] BYREF
  char v17[32]; // [rsp+1A4h] [rbp+184h] BYREF
  char v18; // [rsp+1C4h] [rbp+1A4h]

  v3 = &v7;
  for ( i = 114i64; i; --i )
  {
    *(_DWORD *)v3 = -858993460;
    v3 += 4;
  }
  sub_7FF7E46E13CA((__int64)&unk_7FF7E46F60A6);
  Source = (char *)malloc(0x2Aui64);
  printf("Welcome to HZNUCTF!!!\n");
  printf("Plz input the flag:\n");
  scanf("%s", Source);
  strncpy_s(Destination, 9ui64, Source, 8ui64);
  strncpy_s(Str1, 2ui64, Source + 40, 1ui64);
  if ( !j_strcmp(Destination, "HZNUCTF{") && !j_strcmp(Str1, "}") )
  {
    v11 = (char *)malloc(0x24ui64);
    strncpy_s(v11, 0x24ui64, Source + 8, 0x20ui64);
    memset(v12, 0, sizeof(v12));
    for ( j = 0; j < 32; j += 2 )
    {
      v14 = 0;
      v15 = 0;
      memset(v16, 0, 5ui64);
      memset(v17, 0, 3ui64);
      v14 = v11[j];
      sub_7FF7E46E1393((__int64)v16, "%x", v14);// 转对应的十六进制并替换成字符串
      v15 = v11[j + 1];
      sub_7FF7E46E1393((__int64)v17, "%x", v15);
      j_strcat(v16, v17);
      v18 = v16[1];                             // 交换
      v16[1] = v16[2];
      v16[2] = v18;
      j_strcat(v12, v16);
    }
    check((int)v12, (int)Destination);
    printf("Congratulation!!!\n");
    free(v11);
  }
  free(Source);
  sub_7FF7E46E135C((__int64)v6, (__int64)&unk_7FF7E46ED390);
  return 0;
}
```

首先获取用户输入，然后校验前八位是否为`HZNUCTF{`​以及第41为是否为`}`​。因此flag里面长度为32位

主要看`sub_7FF7E46E1393((__int64)v16, "%x", v14);`​，它传入了flag里面的数据并做了一些处理最后赋值给`v16`​，这里直接通过动调确认它是读入字符，然后转对应的十六进制再拆分开，两个部分分别作为字符串赋给`v16`​。例如"1"-->0x31-->"31"-->0x33,0x31

每次循环处理两个字符并拼接起来，共4个字符，然后将中间两个字符交换

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250415084950239.png)

依次类推，共处理32个字符，生成长度为`64`​字节的字符串`v12`

然后`check`​，传入`v12`​和密钥`Destination`​(`HZNUCTF{`​)

```c
// local variable allocation has failed, the output may be wrong!
void __fastcall sub_7FF7E46E7F30(int a1, int a2)
{
  char *v2; // rdi
  __int64 i; // rcx
  char v4[32]; // [rsp+0h] [rbp-30h] BYREF
  char v5; // [rsp+30h] [rbp+0h] BYREF
  char Src[96]; // [rsp+40h] [rbp+10h] BYREF
  char v7[64]; // [rsp+A0h] [rbp+70h] BYREF
  int j; // [rsp+F4h] [rbp+C4h]
  const void *v9; // [rsp+1F0h] [rbp+1C0h]
  __int64 v10; // [rsp+1F8h] [rbp+1C8h]

  v10 = *(_QWORD *)&a2;
  v9 = *(const void **)&a1;
  v2 = &v5;
  for ( i = 58i64; i; --i )
  {
    *(_DWORD *)v2 = -858993460;
    v2 += 4;
  }
  sub_7FF7E46E13CA((__int64)&unk_7FF7E46F60A6);
  j_memcpy(v7, Src, sizeof(v7));
  sub_7FF7E46E13B6(v10, 8, v9, Src, 8u);
  for ( j = 0; j < 64; ++j )
  {
    if ( (unsigned __int8)Src[j] != dword_7FF7E46F0000[j] )
    {
      printf("wrong_wrong!!!\n");
      exit(1);
    }
  }
  sub_7FF7E46E135C((__int64)v4, (__int64)&unk_7FF7E46ED1F0);
}
```

有验证代码，重点看加密函数`sub_7FF7E46E13B6(v10, 8, v9, Src, 8u);`

```c
void __fastcall sub_7FF7E46E21B0(__int64 key, char a2, const void *flag, void *dest, unsigned int a5)
{
  char *v5; // rdi
  __int64 i; // rcx
  char v7[32]; // [rsp+0h] [rbp-20h] BYREF
  char v8; // [rsp+20h] [rbp+0h] BYREF
  __int64 a1[35]; // [rsp+30h] [rbp+10h] BYREF
  __int64 a2a[4]; // [rsp+148h] [rbp+128h] BYREF
  __int64 v11[25]; // [rsp+168h] [rbp+148h] BYREF
  char v12; // [rsp+234h] [rbp+214h]

  v5 = &v8;
  for ( i = 90i64; i; --i )
  {
    *(_DWORD *)v5 = -858993460;
    v5 += 4;
  }
  sub_7FF7E46E13CA((__int64)&unk_7FF7E46F60A6);
  if ( a5 && key && dest && flag && (a2 == 8 || a2 == 16) )
  {
    j_memmove(dest, flag, (int)(8 * a5));
    v12 = a2;
    if ( a2 == 8 )                              // DES
    {
      j_memcpy(a2a, (const void *)key, 8ui64);
      sub_7FF7E46E1249((__int64)a1, (__int64)a2a);// 密钥调度
      sub_7FF7E46E12DF((__int64)a1, (__int64)dest, a5);// a5=8
    }
    else if ( v12 == 16 )                       // 3DES
    {
      j_memcpy(a2a, (const void *)key, 8ui64);
      j_memcpy(v11, (const void *)(key + 8), 8ui64);
      sub_7FF7E46E1249((__int64)a1, (__int64)a2a);
      sub_7FF7E46E12DF((__int64)a1, (__int64)dest, a5);
      sub_7FF7E46E1249((__int64)a1, (__int64)v11);
      sub_7FF7E46E10A0();
      sub_7FF7E46E1249((__int64)a1, (__int64)a2a);
      sub_7FF7E46E12DF((__int64)a1, (__int64)dest, a5);
    }
  }
  sub_7FF7E46E135C((__int64)v7, (__int64)&unk_7FF7E46ED120);
}
```

经过分析可以确定a2=8时是`DES`​加密，v12=16时是`3DES`​加密，魔改的地方是`totrot`​数组

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250415085733769.png)

标准的应该是

```c
static unsigned char totrot[16] = {
1,2,4,6,8,10,12,14,15,17,19,21,23,25,27,28
};
```

其他都没改

‍

#### 复现(尝试1）

用脚本解密，把`totrot`​换了

```c
#include <string.h>
#include <stdio.h>

#define EN0 0   /* MODE == encrypt */
#define DE1 1   /* MODE == decrypt */

typedef struct
{
	unsigned long ek[32];
	unsigned long dk[32];
} des_ctx;

static unsigned long SP1[64] = {
	0x01010400L, 0x00000000L, 0x00010000L, 0x01010404L,
	0x01010004L, 0x00010404L, 0x00000004L, 0x00010000L,
	0x00000400L, 0x01010400L, 0x01010404L, 0x00000400L,
	0x01000404L, 0x01010004L, 0x01000000L, 0x00000004L,
	0x00000404L, 0x01000400L, 0x01000400L, 0x00010400L,
	0x00010400L, 0x01010000L, 0x01010000L, 0x01000404L,
	0x00010004L, 0x01000004L, 0x01000004L, 0x00010004L,
	0x00000000L, 0x00000404L, 0x00010404L, 0x01000000L,
	0x00010000L, 0x01010404L, 0x00000004L, 0x01010000L,
	0x01010400L, 0x01000000L, 0x01000000L, 0x00000400L,
	0x01010004L, 0x00010000L, 0x00010400L, 0x01000004L,
	0x00000400L, 0x00000004L, 0x01000404L, 0x00010404L,
	0x01010404L, 0x00010004L, 0x01010000L, 0x01000404L,
	0x01000004L, 0x00000404L, 0x00010404L, 0x01010400L,
	0x00000404L, 0x01000400L, 0x01000400L, 0x00000000L,
	0x00010004L, 0x00010400L, 0x00000000L, 0x01010004L };

static unsigned long SP2[64] = {
	0x80108020L, 0x80008000L, 0x00008000L, 0x00108020L,
	0x00100000L, 0x00000020L, 0x80100020L, 0x80008020L,
	0x80000020L, 0x80108020L, 0x80108000L, 0x80000000L,
	0x80008000L, 0x00100000L, 0x00000020L, 0x80100020L,
	0x00108000L, 0x00100020L, 0x80008020L, 0x00000000L,
	0x80000000L, 0x00008000L, 0x00108020L, 0x80100000L,
	0x00100020L, 0x80000020L, 0x00000000L, 0x00108000L,
	0x00008020L, 0x80108000L, 0x80100000L, 0x00008020L,
	0x00000000L, 0x00108020L, 0x80100020L, 0x00100000L,
	0x80008020L, 0x80100000L, 0x80108000L, 0x00008000L,
	0x80100000L, 0x80008000L, 0x00000020L, 0x80108020L,
	0x00108020L, 0x00000020L, 0x00008000L, 0x80000000L,
	0x00008020L, 0x80108000L, 0x00100000L, 0x80000020L,
	0x00100020L, 0x80008020L, 0x80000020L, 0x00100020L,
	0x00108000L, 0x00000000L, 0x80008000L, 0x00008020L,
	0x80000000L, 0x80100020L, 0x80108020L, 0x00108000L };

static unsigned long SP3[64] = {
	0x00000208L, 0x08020200L, 0x00000000L, 0x08020008L,
	0x08000200L, 0x00000000L, 0x00020208L, 0x08000200L,
	0x00020008L, 0x08000008L, 0x08000008L, 0x00020000L,
	0x08020208L, 0x00020008L, 0x08020000L, 0x00000208L,
	0x08000000L, 0x00000008L, 0x08020200L, 0x00000200L,
	0x00020200L, 0x08020000L, 0x08020008L, 0x00020208L,
	0x08000208L, 0x00020200L, 0x00020000L, 0x08000208L,
	0x00000008L, 0x08020208L, 0x00000200L, 0x08000000L,
	0x08020200L, 0x08000000L, 0x00020008L, 0x00000208L,
	0x00020000L, 0x08020200L, 0x08000200L, 0x00000000L,
	0x00000200L, 0x00020008L, 0x08020208L, 0x08000200L,
	0x08000008L, 0x00000200L, 0x00000000L, 0x08020008L,
	0x08000208L, 0x00020000L, 0x08000000L, 0x08020208L,
	0x00000008L, 0x00020208L, 0x00020200L, 0x08000008L,
	0x08020000L, 0x08000208L, 0x00000208L, 0x08020000L,
	0x00020208L, 0x00000008L, 0x08020008L, 0x00020200L };

static unsigned long SP4[64] = {
	0x00802001L, 0x00002081L, 0x00002081L, 0x00000080L,
	0x00802080L, 0x00800081L, 0x00800001L, 0x00002001L,
	0x00000000L, 0x00802000L, 0x00802000L, 0x00802081L,
	0x00000081L, 0x00000000L, 0x00800080L, 0x00800001L,
	0x00000001L, 0x00002000L, 0x00800000L, 0x00802001L,
	0x00000080L, 0x00800000L, 0x00002001L, 0x00002080L,
	0x00800081L, 0x00000001L, 0x00002080L, 0x00800080L,
	0x00002000L, 0x00802080L, 0x00802081L, 0x00000081L,
	0x00800080L, 0x00800001L, 0x00802000L, 0x00802081L,
	0x00000081L, 0x00000000L, 0x00000000L, 0x00802000L,
	0x00002080L, 0x00800080L, 0x00800081L, 0x00000001L,
	0x00802001L, 0x00002081L, 0x00002081L, 0x00000080L,
	0x00802081L, 0x00000081L, 0x00000001L, 0x00002000L,
	0x00800001L, 0x00002001L, 0x00802080L, 0x00800081L,
	0x00002001L, 0x00002080L, 0x00800000L, 0x00802001L,
	0x00000080L, 0x00800000L, 0x00002000L, 0x00802080L };

static unsigned long SP5[64] = {
	0x00000100L, 0x02080100L, 0x02080000L, 0x42000100L,
	0x00080000L, 0x00000100L, 0x40000000L, 0x02080000L,
	0x40080100L, 0x00080000L, 0x02000100L, 0x40080100L,
	0x42000100L, 0x42080000L, 0x00080100L, 0x40000000L,
	0x02000000L, 0x40080000L, 0x40080000L, 0x00000000L,
	0x40000100L, 0x42080100L, 0x42080100L, 0x02000100L,
	0x42080000L, 0x40000100L, 0x00000000L, 0x42000000L,
	0x02080100L, 0x02000000L, 0x42000000L, 0x00080100L,
	0x00080000L, 0x42000100L, 0x00000100L, 0x02000000L,
	0x40000000L, 0x02080000L, 0x42000100L, 0x40080100L,
	0x02000100L, 0x40000000L, 0x42080000L, 0x02080100L,
	0x40080100L, 0x00000100L, 0x02000000L, 0x42080000L,
	0x42080100L, 0x00080100L, 0x42000000L, 0x42080100L,
	0x02080000L, 0x00000000L, 0x40080000L, 0x42000000L,
	0x00080100L, 0x02000100L, 0x40000100L, 0x00080000L,
	0x00000000L, 0x40080000L, 0x02080100L, 0x40000100L };

static unsigned long SP6[64] = {
	0x20000010L, 0x20400000L, 0x00004000L, 0x20404010L,
	0x20400000L, 0x00000010L, 0x20404010L, 0x00400000L,
	0x20004000L, 0x00404010L, 0x00400000L, 0x20000010L,
	0x00400010L, 0x20004000L, 0x20000000L, 0x00004010L,
	0x00000000L, 0x00400010L, 0x20004010L, 0x00004000L,
	0x00404000L, 0x20004010L, 0x00000010L, 0x20400010L,
	0x20400010L, 0x00000000L, 0x00404010L, 0x20404000L,
	0x00004010L, 0x00404000L, 0x20404000L, 0x20000000L,
	0x20004000L, 0x00000010L, 0x20400010L, 0x00404000L,
	0x20404010L, 0x00400000L, 0x00004010L, 0x20000010L,
	0x00400000L, 0x20004000L, 0x20000000L, 0x00004010L,
	0x20000010L, 0x20404010L, 0x00404000L, 0x20400000L,
	0x00404010L, 0x20404000L, 0x00000000L, 0x20400010L,
	0x00000010L, 0x00004000L, 0x20400000L, 0x00404010L,
	0x00004000L, 0x00400010L, 0x20004010L, 0x00000000L,
	0x20404000L, 0x20000000L, 0x00400010L, 0x20004010L };

static unsigned long SP7[64] = {
	0x00200000L, 0x04200002L, 0x04000802L, 0x00000000L,
	0x00000800L, 0x04000802L, 0x00200802L, 0x04200800L,
	0x04200802L, 0x00200000L, 0x00000000L, 0x04000002L,
	0x00000002L, 0x04000000L, 0x04200002L, 0x00000802L,
	0x04000800L, 0x00200802L, 0x00200002L, 0x04000800L,
	0x04000002L, 0x04200000L, 0x04200800L, 0x00200002L,
	0x04200000L, 0x00000800L, 0x00000802L, 0x04200802L,
	0x00200800L, 0x00000002L, 0x04000000L, 0x00200800L,
	0x04000000L, 0x00200800L, 0x00200000L, 0x04000802L,
	0x04000802L, 0x04200002L, 0x04200002L, 0x00000002L,
	0x00200002L, 0x04000000L, 0x04000800L, 0x00200000L,
	0x04200800L, 0x00000802L, 0x00200802L, 0x04200800L,
	0x00000802L, 0x04000002L, 0x04200802L, 0x04200000L,
	0x00200800L, 0x00000000L, 0x00000002L, 0x04200802L,
	0x00000000L, 0x00200802L, 0x04200000L, 0x00000800L,
	0x04000002L, 0x04000800L, 0x00000800L, 0x00200002L };

static unsigned long SP8[64] = {
	0x10001040L, 0x00001000L, 0x00040000L, 0x10041040L,
	0x10000000L, 0x10001040L, 0x00000040L, 0x10000000L,
	0x00040040L, 0x10040000L, 0x10041040L, 0x00041000L,
	0x10041000L, 0x00041040L, 0x00001000L, 0x00000040L,
	0x10040000L, 0x10000040L, 0x10001000L, 0x00001040L,
	0x00041000L, 0x00040040L, 0x10040040L, 0x10041000L,
	0x00001040L, 0x00000000L, 0x00000000L, 0x10040040L,
	0x10000040L, 0x10001000L, 0x00041040L, 0x00040000L,
	0x00041040L, 0x00040000L, 0x10041000L, 0x00001000L,
	0x00000040L, 0x10040040L, 0x00001000L, 0x00041040L,
	0x10001000L, 0x00000040L, 0x10000040L, 0x10040000L,
	0x10040040L, 0x10000000L, 0x00040000L, 0x10001040L,
	0x00000000L, 0x10041040L, 0x00040040L, 0x10000040L,
	0x10040000L, 0x10001000L, 0x10001040L, 0x00000000L,
	0x10041040L, 0x00041000L, 0x00041000L, 0x00001040L,
	0x00001040L, 0x00040040L, 0x10000000L, 0x10041000L };

#define   CRCSEED 0x7529

static void scrunch(unsigned char *, unsigned long *);
static void unscrun(unsigned long *, unsigned char *);
static void desfunc(unsigned long *, unsigned long *);
static void cookey(unsigned long *);
void usekey(register unsigned long *from);

static unsigned long KnL[32] = { 0L };
//static unsigned long KnR[32] = { 0L };
//static unsigned long Kn3[32] = { 0L };
//static unsigned char Df_Key[24] = {
//  0x01,0x23,0x45,0x67,0x89,0xab,0xcd,0xef,
//  0xfe,0xdc,0xba,0x98,0x76,0x54,0x32,0x10,
//  0x89,0xab,0xcd,0xef,0x01,0x23,0x45,0x67 };

static unsigned short bytebit[8]    = {
	0200, 0100, 040, 020, 010, 04, 02, 01 };

static unsigned long bigbyte[24] = {
	0x800000L,  0x400000L,  0x200000L,  0x100000L,
	0x80000L,   0x40000L,   0x20000L,   0x10000L,
	0x8000L,    0x4000L,    0x2000L,    0x1000L,
	0x800L,     0x400L,     0x200L,     0x100L,
	0x80L,      0x40L,      0x20L,      0x10L,
	0x8L,       0x4L,       0x2L,       0x1L    };

/* Use the key schedule specified in the Standard (ANSI X3.92-1981). */

static unsigned char pc1[56] = {
	56, 48, 40, 32, 24, 16,  8,  0, 57, 49, 41, 33, 25, 17,
	9,  1, 58, 50, 42, 34, 26, 18, 10,  2, 59, 51, 43, 35,
	62, 54, 46, 38, 30, 22, 14,  6, 61, 53, 45, 37, 29, 21,
	13,  5, 60, 52, 44, 36, 28, 20, 12,  4, 27, 19, 11,  3 };

//static unsigned char totrot[16] = {
//	1,2,4,6,8,10,12,14,15,17,19,21,23,25,27,28 };
static unsigned char totrot[16] = {
	1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16};
static unsigned char pc2[48] = {
	13, 16, 10, 23,  0,  4,  2, 27, 14,  5, 20,  9,
	22, 18, 11,  3, 25,  7, 15,  6, 26, 19, 12,  1,
	40, 51, 30, 36, 46, 54, 29, 39, 50, 44, 32, 47,
	43, 48, 38, 55, 33, 52, 45, 41, 49, 35, 28, 31 };

void deskey(unsigned char *key,short edf)   /* Thanks to James Gillogly & Phil Karn! */
{
	register int i, j, l, m, n;
	unsigned char pc1m[56], pcr[56];
	unsigned long kn[32];

	for ( j = 0; j < 56; j++ )
	{
		l = pc1[j];
		m = l & 07;
		pc1m[j] = (key[l >> 3] & bytebit[m]) ? 1 : 0;
	}
	for( i = 0; i < 16; i++ )
	{
		if( edf == DE1 ) m = (15 - i) << 1;
		else m = i << 1;
		n = m + 1;
		kn[m] = kn[n] = 0L;
		for( j = 0; j < 28; j++ )
		{
			l = j + totrot[i];
			if( l < 28 ) pcr[j] = pc1m[l];
			else pcr[j] = pc1m[l - 28];
		}
		for( j = 28; j < 56; j++ )
		{
			l = j + totrot[i];
			if( l < 56 ) pcr[j] = pc1m[l];
			else pcr[j] = pc1m[l - 28];
		}
		for( j = 0; j < 24; j++ )
		{
			if( pcr[pc2[j]] ) kn[m] |= bigbyte[j];
			if( pcr[pc2[j+24]] ) kn[n] |= bigbyte[j];
		}
	}
	cookey(kn);
	return;
}

static void cookey(register unsigned long *raw1)
{
	register unsigned long *cook, *raw0;
	unsigned long dough[32];
	register int i;

	cook = dough;
	for( i = 0; i < 16; i++, raw1++ )
	{
		raw0 = raw1++;
		*cook    = (*raw0 & 0x00fc0000L) << 6;
		*cook   |= (*raw0 & 0x00000fc0L) << 10;
		*cook   |= (*raw1 & 0x00fc0000L) >> 10;
		*cook++ |= (*raw1 & 0x00000fc0L) >> 6;
		*cook    = (*raw0 & 0x0003f000L) << 12;
		*cook   |= (*raw0 & 0x0000003fL) << 16;
		*cook   |= (*raw1 & 0x0003f000L) >> 4;
		*cook++ |= (*raw1 & 0x0000003fL);
	}
	usekey(dough);
	return;
}

void cpkey(register unsigned long *into)
{
	register unsigned long *from, *endp;

	from = KnL, endp = &KnL[32];
	while( from < endp ) *into++ = *from++;
	return;
}

void usekey(register unsigned long *from)
{
	register unsigned long *to, *endp;

	to = KnL, endp = &KnL[32];
	while( to < endp ) *to++ = *from++;
	return;
}

void des(unsigned char *inblock, unsigned char *outblock)
{
	unsigned long work[2];

	scrunch(inblock, work);
	desfunc(work, KnL);
	unscrun(work, outblock);
	return;
}

static void scrunch(register unsigned char *outof, register unsigned long *into)
{
	*into    = (*outof++ & 0xffL) << 24;
	*into   |= (*outof++ & 0xffL) << 16;
	*into   |= (*outof++ & 0xffL) << 8;
	*into++ |= (*outof++ & 0xffL);
	*into    = (*outof++ & 0xffL) << 24;
	*into   |= (*outof++ & 0xffL) << 16;
	*into   |= (*outof++ & 0xffL) << 8;
	*into   |= (*outof   & 0xffL);
	return;
}

static void unscrun(register unsigned long *outof, register unsigned char *into)
{
	*into++ = (unsigned char)((*outof >> 24) & 0xffL);
	*into++ = (unsigned char)((*outof >> 16) & 0xffL);
	*into++ = (unsigned char)((*outof >>  8) & 0xffL);
	*into++ = (unsigned char)(*outof++   & 0xffL);
	*into++ = (unsigned char)((*outof >> 24) & 0xffL);
	*into++ = (unsigned char)((*outof >> 16) & 0xffL);
	*into++ = (unsigned char)((*outof >>  8) & 0xffL);
	*into   =  (unsigned char)(*outof    & 0xffL);
	return;
}

static void desfunc(register unsigned long *block,register unsigned long *keys)
{
	register unsigned long fval, work, right, leftt;
	register int round;

	leftt = block[0];
	right = block[1];
	work = ((leftt >> 4) ^ right) & 0x0f0f0f0fL;
	right ^= work;
	leftt ^= (work << 4);
	work = ((leftt >> 16) ^ right) & 0x0000ffffL;
	right ^= work;
	leftt ^= (work << 16);
	work = ((right >> 2) ^ leftt) & 0x33333333L;
	leftt ^= work;
	right ^= (work << 2);
	work = ((right >> 8) ^ leftt) & 0x00ff00ffL;
	leftt ^= work;
	right ^= (work << 8);
	right = ((right << 1) | ((right >> 31) & 1L)) & 0xffffffffL;
	work = (leftt ^ right) & 0xaaaaaaaaL;
	leftt ^= work;
	right ^= work;
	leftt = ((leftt << 1) | ((leftt >> 31) & 1L)) & 0xffffffffL;

	for( round = 0; round < 8; round++ )
	{
		work  = (right << 28) | (right >> 4);
		work ^= *keys++;
		fval  = SP7[ work        & 0x3fL];
		fval |= SP5[(work >>  8) & 0x3fL];
		fval |= SP3[(work >> 16) & 0x3fL];
		fval |= SP1[(work >> 24) & 0x3fL];
		work  = right ^ *keys++;
		fval |= SP8[ work        & 0x3fL];
		fval |= SP6[(work >>  8) & 0x3fL];
		fval |= SP4[(work >> 16) & 0x3fL];
		fval |= SP2[(work >> 24) & 0x3fL];
		leftt ^= fval;
		work  = (leftt << 28) | (leftt >> 4);
		work ^= *keys++;
		fval  = SP7[ work        & 0x3fL];
		fval |= SP5[(work >>  8) & 0x3fL];
		fval |= SP3[(work >> 16) & 0x3fL];
		fval |= SP1[(work >> 24) & 0x3fL];
		work  = leftt ^ *keys++;
		fval |= SP8[ work        & 0x3fL];
		fval |= SP6[(work >>  8) & 0x3fL];
		fval |= SP4[(work >> 16) & 0x3fL];
		fval |= SP2[(work >> 24) & 0x3fL];
		right ^= fval;
	}

	right = (right << 31) | (right >> 1);
	work = (leftt ^ right) & 0xaaaaaaaaL;
	leftt ^= work;
	right ^= work;
	leftt = (leftt << 31) | (leftt >> 1);
	work = ((leftt >> 8) ^ right) & 0x00ff00ffL;
	right ^= work;
	leftt ^= (work << 8);
	work = ((leftt >> 2) ^ right) & 0x33333333L;
	right ^= work;
	leftt ^= (work << 2);
	work = ((right >> 16) ^ leftt) & 0x0000ffffL;
	leftt ^= work;
	right ^= (work << 16);
	work = ((right >> 4) ^ leftt) & 0x0f0f0f0fL;
	leftt ^= work;
	right ^= (work << 4);
	*block++ = right;
	*block = leftt;
	return;
}

/* Validation sets:
*
* Single-length key, single-length plaintext -
* Key    : 0123 4567 89ab cdef
* Plain  : 0123 4567 89ab cde7
* Cipher : c957 4425 6a5e d31d
*****************************************************************/

void des_key(des_ctx *dc, unsigned char *key)
{
	deskey(key,EN0);
	cpkey(dc->ek);
	deskey(key,DE1);
	cpkey(dc->dk);
}

/*Encrypt several blocks in ECB mode. Caller is responsible for short blocks. */
void des_enc(des_ctx *dc, unsigned char *data, int blocks)
{
	unsigned long work[2];
	int i;
	unsigned char *cp;
	cp=data;
	for(i=0;i<blocks;i++)
	{
		scrunch(cp,work);
		desfunc(work,dc->ek);
		unscrun(work,cp);
		cp+=8;
	}
}

void des_dec(des_ctx *dc, unsigned char *data, int blocks)
{
	unsigned long work[2];
	int i;
	unsigned char *cp;
	cp=data;

	for(i=0;i<blocks;i++)
	{
		scrunch(cp,work);
		desfunc(work,dc->dk);
		unscrun(work,cp);
		cp+=8;
	}
}

int des_encrypt(const unsigned char* Key,const unsigned char KeyLen,const unsigned char* PlainContent,unsigned char* CipherContent,const int BlockCount)
{
	des_ctx dc;
	unsigned char  keyEn[8];
	unsigned char  keyDe[8];

	if((BlockCount == 0) || (Key == NULL) || (CipherContent == NULL) || (PlainContent == NULL))
	{
		return -1;
	}

	if((KeyLen != 8) && (KeyLen != 16))
	{
		return -1;
	}

	memcpy(CipherContent, PlainContent,BlockCount*8);

	switch (KeyLen)
	{
		case 8:    // DES
		memcpy(keyEn,Key,8);               //keyEn
		des_key(&dc, keyEn);
		des_enc(&dc, CipherContent, BlockCount);
		break;
		case 16:    // 3DES
		memcpy(keyEn,Key,8);               //keyEn
		memcpy(keyDe, Key+8, 8);      //keyDe
		des_key(&dc, keyEn);
		des_enc(&dc, CipherContent, BlockCount);

		des_key(&dc, keyDe);
		des_dec(&dc, CipherContent, BlockCount);

		des_key(&dc, keyEn);
		des_enc(&dc, CipherContent, BlockCount);
		break;

	default:
		return -1;
	}
	return 0;
}

int des_decrypt(const unsigned char* Key,const unsigned char KeyLen,const unsigned char* CipherContent,unsigned char* PlainContent,const int BlockCount)
{
	des_ctx dc;
	unsigned char keyEn[8];
	unsigned char keyDe[8];

	if((BlockCount == 0) || (Key == NULL) || (CipherContent == NULL) || (PlainContent == NULL))
	{
		return -1;
	}

	if((KeyLen != 8) && (KeyLen != 16))
	{
		return -1;
	}

	memcpy(PlainContent,CipherContent,BlockCount*8);

	switch (KeyLen)
	{
		case 8:    // DES
		memcpy(keyEn,Key,8);               //keyEn
		des_key(&dc, keyEn);
		des_dec(&dc, PlainContent, BlockCount);
		break;
	case 16:
		memcpy(keyEn,Key,8);               //keyEn
		memcpy(keyDe, Key+8, 8);        //keyDe
		des_key(&dc, keyEn);
		des_dec(&dc, PlainContent, BlockCount);

		des_key(&dc, keyDe);
		des_enc(&dc, PlainContent, BlockCount);

		des_key(&dc, keyEn);
		des_dec(&dc, PlainContent, BlockCount);
		break;
	default:
		return -1;
	}
	return 0;
}

int main(void)
{
	unsigned char i;
	unsigned char plain[32]={0};
	unsigned char cipher1[32]={
		0x84,0x8b,0x03,0x22,0x14,0xbe,0xdf,0x75,
		0xb3,0xd5,0x76,0x6f,0xcd,0x2a,0x5d,0xd7,
		0x4d,0xb2,0x5f,0x06,0x98,0x9d,0x3e,0xa8,
		0xf7,0x23,0xf2,0x8b,0xf2,0x54,0x65,0x7a,
	};
	unsigned char cipher2[32]={
		0x20,0xc0,0x87,0x55,0xd6,0x3b,0x46,0x3d,
		0xf7,0xb2,0x7a,0x9d,0xc2,0xcf,0x1a,0xae,
		0x16,0xc7,0x15,0x30,0x8e,0xfd,0x8f,0x9e,
		0xaa,0x39,0xab,0xfe,0x95,0xa7,0x1f,0xf1,
	};
	const unsigned char key[16] = { 0x48, 0x5A, 0x4E, 0x55, 0x43, 0x54, 0x46, 0x7B};
	if(des_decrypt(key,8,cipher1,plain,4) == 0x0)
	{
		printf("sc_des plain data: \n");
		for(i = 0; i < 32; i++)
		{
			printf("%c",plain[i]);
		}
		printf(("\n"));
	}
	if(des_decrypt(key,8,cipher2,plain,4) == 0x0)
	{
		printf("sc_des plain data: \n");
		for(i = 0; i < 32; i++)
		{
			printf("%c",plain[i]);
		}
		printf(("\n"));
	}
	//333936147332632923d96353321d3345636826d26314621d3349330463126348
}

```

拿到`DES`​加密前的字符串后，翻转回来并作为十六进制数转为字符串

```python
enc = "333936147332632923d96353321d3345636826d26314621d3349330463126348"
enc_list = list(enc)

for i in range(1, len(enc_list), 4):
    if i + 1 < len(enc_list):
        enc_list[i], enc_list[i + 1] = enc_list[i + 1], enc_list[i]
    processed_str = "".join(enc_list)
print("HZNUCTF{", end="")

for i in range(0, len(processed_str), 2):
    hex_str = processed_str[i:i+2]
    print(chr(int(hex_str, 16)), end="")
print("}")
#HZNUCTF{391ds2b9-9e31-45f8-ba4a-4904a2d8}
```

#### 复现(尝试2)

注意看核心加密函数

```c
void __fastcall sub_7FF7E46E2090(__int64 a1, __int64 a2, int a3)
{
  char *v3; // rdi
  __int64 i; // rcx
  char v5[32]; // [rsp+0h] [rbp-20h] BYREF
  char v6; // [rsp+20h] [rbp+0h] BYREF
  int v7[7]; // [rsp+28h] [rbp+8h] BYREF
  int j; // [rsp+44h] [rbp+24h]
  unsigned __int8 *a1a; // [rsp+68h] [rbp+48h]

  v3 = &v6;
  for ( i = 26i64; i; --i )
  {
    *(_DWORD *)v3 = -858993460;
    v3 += 4;
  }
  sub_7FF7E46E13CA(byte_7FF7E46F60A6);
  a1a = (unsigned __int8 *)a2;
  for ( j = 0; j < a3; ++j )
  {
    sub_7FF7E46E3500(a1a, v7);                  // 将 a1 中的 8 个字节数据（假设是明文的一部分）以大端字节序转换成两个 32 位的整数，并将它们存储到 a2 中。因此，最终 a2 会包含 2 个整数，每个整数由 4 个字节组成。
    sub_7FF7E46E2510((unsigned int *)v7, (_DWORD *)a1);//核心加密
    sub_7FF7E46E39A0(v7, a1a);
    a1a += 8;
  }
  sub_7FF7E46E135C((__int64)v5, (__int64)&unk_7FF7E46ECF90);
}
```

```c

```

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250415140454003.png)

由于DES是对称的，那么只要把扩展密钥倒过来就能实现解密

同时注意，在密钥调度部分生成了加密的key和解密的key(猜测本来是给3DES用的，DES只取加密的key就行了)，那么可以直接动调拿到解密的key和密文然后直接跑一遍核心加密函数拿到明文

先拿`key`

```c
unsigned long data[32] = {
    0x09221A0A2C0B3C36, 0x09123B0D2829051D, 0x0512011F2C091B18, 0x0712292009292E17,
    0x0514372E090D1703, 0x27100E270915123C, 0x25150D2901050927, 0x2411261813151F32,
    0x34312B3703052031, 0x3C19151B13043A05, 0x3429383023063B3E, 0x380B3F2A03062108,
    0x3009141B260E063D, 0x300B01240E223D3D, 0x11093D14062A1700, 0x1208083E0E22262B,
    0x1208083E0E22262B, 0x11093D14062A1700, 0x300B01240E223D3D, 0x3009141B260E063D,
    0x380B3F2A03062108, 0x3429383023063B3E, 0x3C19151B13043A05, 0x34312B3703052031,
    0x2411261813151F32, 0x25150D2901050927, 0x27100E270915123C, 0x0514372E090D1703,
    0x0712292009292E17, 0x0512011F2C091B18, 0x09123B0D2829051D, 0x09221A0A2C0B3C36
};
```

> DES（Data Encryption Standard）密钥调度的作用是从一个 64 位密钥（其中有效位是 56 位，8 位用于奇偶校验）中，派生出 **16 个子密钥（subkeys）** ，每个子密钥为 **48 位**。

前16个密钥是加密的，后16个是解密的

exp：

```c
#include<stdio.h>
#include<stdlib.h>
#include<windows.h>
unsigned int s1[64] = {
	0x01010400, 0x00000000, 0x00010000, 0x01010404, 0x01010004, 0x00010404, 0x00000004, 0x00010000,
	0x00000400, 0x01010400, 0x01010404, 0x00000400, 0x01000404, 0x01010004, 0x01000000, 0x00000004,
	0x00000404, 0x01000400, 0x01000400, 0x00010400, 0x00010400, 0x01010000, 0x01010000, 0x01000404,
	0x00010004, 0x01000004, 0x01000004, 0x00010004, 0x00000000, 0x00000404, 0x00010404, 0x01000000,
	0x00010000, 0x01010404, 0x00000004, 0x01010000, 0x01010400, 0x01000000, 0x01000000, 0x00000400,
	0x01010004, 0x00010000, 0x00010400, 0x01000004, 0x00000400, 0x00000004, 0x01000404, 0x00010404,
	0x01010404, 0x00010004, 0x01010000, 0x01000404, 0x01000004, 0x00000404, 0x00010404, 0x01010400,
	0x00000404, 0x01000400, 0x01000400, 0x00000000, 0x00010004, 0x00010400, 0x00000000, 0x01010004};
unsigned int s2[64] = {
	0x80108020, 0x80008000, 0x00008000, 0x00108020, 0x00100000, 0x00000020, 0x80100020, 0x80008020,
	0x80000020, 0x80108020, 0x80108000, 0x80000000, 0x80008000, 0x00100000, 0x00000020, 0x80100020,
	0x00108000, 0x00100020, 0x80008020, 0x00000000, 0x80000000, 0x00008000, 0x00108020, 0x80100000,
	0x00100020, 0x80000020, 0x00000000, 0x00108000, 0x00008020, 0x80108000, 0x80100000, 0x00008020,
	0x00000000, 0x00108020, 0x80100020, 0x00100000, 0x80008020, 0x80100000, 0x80108000, 0x00008000,
	0x80100000, 0x80008000, 0x00000020, 0x80108020, 0x00108020, 0x00000020, 0x00008000, 0x80000000,
	0x00008020, 0x80108000, 0x00100000, 0x80000020, 0x00100020, 0x80008020, 0x80000020, 0x00100020,
	0x00108000, 0x00000000, 0x80008000, 0x00008020, 0x80000000, 0x80100020, 0x80108020, 0x00108000};
unsigned int s3[64] = {
	0x00000208, 0x08020200, 0x00000000, 0x08020008, 0x08000200, 0x00000000, 0x00020208, 0x08000200,
	0x00020008, 0x08000008, 0x08000008, 0x00020000, 0x08020208, 0x00020008, 0x08020000, 0x00000208,
	0x08000000, 0x00000008, 0x08020200, 0x00000200, 0x00020200, 0x08020000, 0x08020008, 0x00020208,
	0x08000208, 0x00020200, 0x00020000, 0x08000208, 0x00000008, 0x08020208, 0x00000200, 0x08000000,
	0x08020200, 0x08000000, 0x00020008, 0x00000208, 0x00020000, 0x08020200, 0x08000200, 0x00000000,
	0x00000200, 0x00020008, 0x08020208, 0x08000200, 0x08000008, 0x00000200, 0x00000000, 0x08020008,
	0x08000208, 0x00020000, 0x08000000, 0x08020208, 0x00000008, 0x00020208, 0x00020200, 0x08000008,
	0x08020000, 0x08000208, 0x00000208, 0x08020000, 0x00020208, 0x00000008, 0x08020008, 0x00020200};
unsigned int s4[64] = {
	0x00802001, 0x00002081, 0x00002081, 0x00000080, 0x00802080, 0x00800081, 0x00800001, 0x00002001,
	0x00000000, 0x00802000, 0x00802000, 0x00802081, 0x00000081, 0x00000000, 0x00800080, 0x00800001,
	0x00000001, 0x00002000, 0x00800000, 0x00802001, 0x00000080, 0x00800000, 0x00002001, 0x00002080,
	0x00800081, 0x00000001, 0x00002080, 0x00800080, 0x00002000, 0x00802080, 0x00802081, 0x00000081,
	0x00800080, 0x00800001, 0x00802000, 0x00802081, 0x00000081, 0x00000000, 0x00000000, 0x00802000,
	0x00002080, 0x00800080, 0x00800081, 0x00000001, 0x00802001, 0x00002081, 0x00002081, 0x00000080,
	0x00802081, 0x00000081, 0x00000001, 0x00002000, 0x00800001, 0x00002001, 0x00802080, 0x00800081,
	0x00002001, 0x00002080, 0x00800000, 0x00802001, 0x00000080, 0x00800000, 0x00002000, 0x00802080};
unsigned int s5[64] = {
	0x00000100, 0x02080100, 0x02080000, 0x42000100, 0x00080000, 0x00000100, 0x40000000, 0x02080000,
	0x40080100, 0x00080000, 0x02000100, 0x40080100, 0x42000100, 0x42080000, 0x00080100, 0x40000000,
	0x02000000, 0x40080000, 0x40080000, 0x00000000, 0x40000100, 0x42080100, 0x42080100, 0x02000100,
	0x42080000, 0x40000100, 0x00000000, 0x42000000, 0x02080100, 0x02000000, 0x42000000, 0x00080100,
	0x00080000, 0x42000100, 0x00000100, 0x02000000, 0x40000000, 0x02080000, 0x42000100, 0x40080100,
	0x02000100, 0x40000000, 0x42080000, 0x02080100, 0x40080100, 0x00000100, 0x02000000, 0x42080000,
	0x42080100, 0x00080100, 0x42000000, 0x42080100, 0x02080000, 0x00000000, 0x40080000, 0x42000000,
	0x00080100, 0x02000100, 0x40000100, 0x00080000, 0x00000000, 0x40080000, 0x02080100, 0x40000100};
unsigned int s6[64] = {
	0x20000010, 0x20400000, 0x00004000, 0x20404010, 0x20400000, 0x00000010, 0x20404010, 0x00400000,
	0x20004000, 0x00404010, 0x00400000, 0x20000010, 0x00400010, 0x20004000, 0x20000000, 0x00004010,
	0x00000000, 0x00400010, 0x20004010, 0x00004000, 0x00404000, 0x20004010, 0x00000010, 0x20400010,
	0x20400010, 0x00000000, 0x00404010, 0x20404000, 0x00004010, 0x00404000, 0x20404000, 0x20000000,
	0x20004000, 0x00000010, 0x20400010, 0x00404000, 0x20404010, 0x00400000, 0x00004010, 0x20000010,
	0x00400000, 0x20004000, 0x20000000, 0x00004010, 0x20000010, 0x20404010, 0x00404000, 0x20400000,
	0x00404010, 0x20404000, 0x00000000, 0x20400010, 0x00000010, 0x00004000, 0x20400000, 0x00404010,
	0x00004000, 0x00400010, 0x20004010, 0x00000000, 0x20404000, 0x20000000, 0x00400010, 0x20004010};
unsigned int s7[64] = {
	0x00200000, 0x04200002, 0x04000802, 0x00000000, 0x00000800, 0x04000802, 0x00200802, 0x04200800,
	0x04200802, 0x00200000, 0x00000000, 0x04000002, 0x00000002, 0x04000000, 0x04200002, 0x00000802,
	0x04000800, 0x00200802, 0x00200002, 0x04000800, 0x04000002, 0x04200000, 0x04200800, 0x00200002,
	0x04200000, 0x00000800, 0x00000802, 0x04200802, 0x00200800, 0x00000002, 0x04000000, 0x00200800,
	0x04000000, 0x00200800, 0x00200000, 0x04000802, 0x04000802, 0x04200002, 0x04200002, 0x00000002,
	0x00200002, 0x04000000, 0x04000800, 0x00200000, 0x04200800, 0x00000802, 0x00200802, 0x04200800,
	0x00000802, 0x04000002, 0x04200802, 0x04200000, 0x00200800, 0x00000000, 0x00000002, 0x04200802,
	0x00000000, 0x00200802, 0x04200000, 0x00000800, 0x04000002, 0x04000800, 0x00000800, 0x00200002};
unsigned int s8[64] = {
	0x10001040, 0x00001000, 0x00040000, 0x10041040, 0x10000000, 0x10001040, 0x00000040, 0x10000000,
	0x00040040, 0x10040000, 0x10041040, 0x00041000, 0x10041000, 0x00041040, 0x00001000, 0x00000040,
	0x10040000, 0x10000040, 0x10001000, 0x00001040, 0x00041000, 0x00040040, 0x10040040, 0x10041000,
	0x00001040, 0x00000000, 0x00000000, 0x10040040, 0x10000040, 0x10001000, 0x00041040, 0x00040000,
	0x00041040, 0x00040000, 0x10041000, 0x00001000, 0x00000040, 0x10040040, 0x00001000, 0x00041040,
	0x10001000, 0x00000040, 0x10000040, 0x10040000, 0x10040040, 0x10000000, 0x00040000, 0x10001040,
	0x00000000, 0x10041040, 0x00040040, 0x10000040, 0x10040000, 0x10001000, 0x10001040, 0x00000000,
	0x10041040, 0x00041000, 0x00041000, 0x00001040, 0x00001040, 0x00040040, 0x10000000, 0x10041000};
//unsigned long long key[16]={
//	0x1208083E0E22262B, 0x11093D14062A1700, 0x300B01240E223D3D, 0x3009141B260E063D,
//	0x380B3F2A03062108, 0x3429383023063B3E, 0x3C19151B13043A05, 0x34312B3703052031,
//	0x2411261813151F32, 0x25150D2901050927, 0x27100E270915123C, 0x0514372E090D1703,
//	0x0712292009292E17, 0x0512011F2C091B18, 0x09123B0D2829051D, 0x09221A0A2C0B3C36
//};
unsigned int key[64] = {
	0x2C0B3C36, 0x09221A0A, 0x2829051D, 0x09123B0D, 0x2C091B18, 0x0512011F, 0x09292E17, 0x07122920,
	0x090D1703, 0x0514372E, 0x0915123C, 0x27100E27, 0x01050927, 0x25150D29, 0x13151F32, 0x24112618,
	0x03052031, 0x34312B37, 0x13043A05, 0x3C19151B, 0x23063B3E, 0x34293830, 0x03062108, 0x380B3F2A,
	0x260E063D, 0x3009141B, 0x0E223D3D, 0x300B0124, 0x062A1700, 0x11093D14, 0x0E22262B, 0x1208083E,
	0x0E22262B, 0x1208083E, 0x062A1700, 0x11093D14, 0x0E223D3D, 0x300B0124, 0x260E063D, 0x3009141B,
	0x03062108, 0x380B3F2A, 0x23063B3E, 0x34293830, 0x13043A05, 0x3C19151B, 0x03052031, 0x34312B37,
	0x13151F32, 0x24112618, 0x01050927, 0x25150D29, 0x0915123C, 0x27100E27, 0x090D1703, 0x0514372E,
	0x09292E17, 0x07122920, 0x2C091B18, 0x0512011F, 0x2829051D, 0x09123B0D, 0x2C0B3C36, 0x09221A0A};

void __fastcall des_encrypt(unsigned int *_a1_, DWORD *_a2_)
{
	unsigned int left = _a1_[1];
	unsigned int right = _a1_[0];

	unsigned int temp = (left ^ (right >> 4)) & 0xF0F0F0F;
	left = temp ^ left;
	right = (temp << 4) ^ right;

	temp = (left ^ (right >> 16)) & 0x0000FFFF;
	left = temp ^ left;
	right = (temp << 16) ^ right;

	temp = (right ^ (left >> 2)) & 0x33333333;
	right = temp ^ right;
	left = (temp << 2) ^ left;

	temp = (right ^ (left >> 8)) & 0x00FF00FF;
	right = temp ^ right;
	left = (((temp << 8) ^ left) >> 31) | (2 * ((temp << 8) ^ left));

	temp = (left ^ right) & 0xAAAAAAAA;
	left = temp ^ left;
	right = ((temp ^ right) >> 31) | (2 * (temp ^ right));

	for (int i = 0; i < 8; i++)
	{
		temp = *_a2_++ ^ ((left >> 4) | (left << 28));
		unsigned int result1 = s1[(temp >> 24) & 0x3F] |
		s3[(temp >> 16) & 0x3F] |
		s5[(temp >> 8) & 0x3F] |
		s7[temp & 0x3F];
		temp = *_a2_++ ^ left;

		right ^= s2[(temp >> 24) & 0x3F] |
		s4[(temp >> 16) & 0x3F] |
		s6[(temp >> 8) & 0x3F] |
		s8[temp & 0x3F] |
		result1;

		temp = *_a2_++ ^ ((right >> 4) | (right << 28));
		result1 = s1[(temp >> 24) & 0x3F] |
		s3[(temp >> 16) & 0x3F] |
		s5[(temp >> 8) & 0x3F] |
		s7[temp & 0x3F];

		temp = *_a2_++ ^ right;
		left ^= s2[(temp >> 24) & 0x3F] |
		s4[(temp >> 16) & 0x3F] |
		s6[(temp >> 8) & 0x3F] |
		s8[temp & 0x3F] |
		result1;
	}
	left = (left >> 1) | (left << 31);
	temp = (left ^ right) & 0xAAAAAAAA;
	left = temp ^ left;
	right = ((temp ^ right) >> 1) | ((temp ^ right) << 31);

	temp = (left ^ (right >> 8)) & 0xFF00FF;
	left = temp ^ left;
	right = (temp << 8) ^ right;

	temp = (left ^ (right >> 2)) & 0x33333333;
	left = temp ^ left;
	right = (temp << 2) ^ right;

	temp = (right ^ (left >> 16)) & 0xFFFF;
	right = temp ^ right;
	left = (temp << 16) ^ left;

	temp = (right ^ (left >> 4)) & 0x0F0F0F0F;

	_a1_[0] = (temp << 4) ^ left;
	_a1_[1] = temp ^ right;
}

int main(){
	unsigned char Enc[64]={};
	unsigned char data[64]={
		0x84,0x8b,0x03,0x22,0x14,0xbe,0xdf,0x75,
		0xb3,0xd5,0x76,0x6f,0xcd,0x2a,0x5d,0xd7,
		0x4d,0xb2,0x5f,0x06,0x98,0x9d,0x3e,0xa8,
		0xf7,0x23,0xf2,0x8b,0xf2,0x54,0x65,0x7a,
		0x20,0xc0,0x87,0x55,0xd6,0x3b,0x46,0x3d,
		0xf7,0xb2,0x7a,0x9d,0xc2,0xcf,0x1a,0xae,
		0x16,0xc7,0x15,0x30,0x8e,0xfd,0x8f,0x9e,
		0xaa,0x39,0xab,0xfe,0x95,0xa7,0x1f,0xf1,
	};
	for (int i = 0; i < 64; i++){
		Enc[i] = data[i];
	}
	//4字节一组翻转
	for (int i = 0; i < 64; i += 4)
	{
		unsigned char a,b,c,d;
		a = Enc[i];
		b = Enc[i + 1];
		c = Enc[i + 2];
		d = Enc[i + 3];
		Enc[i] = d;
		Enc[i + 1] = c;
		Enc[i + 2] = b;
		Enc[i + 3] = a;
	}
	//des解密
	for(int i=0;i<64;i+=8){
		des_encrypt((unsigned int*)(Enc+i),(DWORD*)(key+32));
	}
	//4字节一组翻转
	for (int i = 0; i < 64; i += 4)
	{
		unsigned char a = Enc[i], b = Enc[i + 1], c = Enc[i + 2], d = Enc[i + 3];
		Enc[i] = d;
		Enc[i + 1] = c;
		Enc[i + 2] = b;
		Enc[i + 3] = a;
	}
//	for (int i = 0; i < 64; i++){
//		printf("%c",Enc[i]);
//	}
	//333936147332632923d96353321d3345636826d26314621d3349330463126348
	int a,b;
	for(int i=0;i<64;i+=4){
		a=Enc[i+1];
		b=Enc[i+2];
		Enc[i+2]=a;
		Enc[i+1]=b;
	}
//	for (int i = 0; i < 64; i++){
//		printf("%c",Enc[i]);
//	}
	//33393164733262392d396533312d343566382d626134612d3439303461326438
//	for (int i = 0; i < 32; i++){
//		printf("0x");
//		for(int j=0;j<2;j++){
//			printf("%c",Enc[i*2+j]);
//		}
//		printf(",");
//	}
	//0x33,0x39,0x31,0x64,0x73,0x32,0x62,0x39,0x2d,0x39,0x65,0x33,0x31,0x2d,0x34,0x35,0x66,0x38,0x2d,0x62,0x61,0x34,0x61,0x2d,0x34,0x39,0x30,0x34,0x61,0x32,0x64,0x38,
	unsigned char flag[]={
		0x33,0x39,0x31,0x64,0x73,0x32,0x62,0x39,0x2d,0x39,0x65,
		0x33,0x31,0x2d,0x34,0x35,0x66,0x38,0x2d,0x62,0x61,0x34,
		0x61,0x2d,0x34,0x39,0x30,0x34,0x61,0x32,0x64,0x38,
	};
	printf("HZNUCTF{");
	for(int i=0;i<32;i++){
		printf("%c",flag[i]);
	}
	printf("}");
	return 0;
}
//HZNUCTF{391ds2b9-9e31-45f8-ba4a-4904a2d8}
```

‍

### conforand

#### 逻辑&分析

RC4魔改

1. 获取输入

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250415144031892.png)

2. RC4加密，3个魔改点

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250415144108858.png)

- key的魔改，**魔改**成`rand()`​的值与`key[i]`​异或赋给`key[i]`​，注意只`rand()`​一次

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250415144300135.png)

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250415144637531.png)

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250415152850722.png)

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250415144709041.png)

注意不要被这里的`s`​给骗了，它不是s盒(这道题就卡在这，呜呜呜以后一定要仔细分析)

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250415153036756.png)

从这里也可以看出s是key数组

其中种子在`sub_5s5s5s()`​里，是当前时间戳随机

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250415144841378.png)

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250415144912742.png)

另外密钥在`init()`​里

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250415145028543.png)

- sbox初始化轮数是257轮，以及模数也是257（就是原来256的地方全变成了257）

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250415153019232.png)

- RC4加密里本来的`swap`​变成了

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250415145553494.png)

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250415145617630.png)

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250415145928753.png)

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250415145940834.png)

确认3个魔改点后可以爆破求`rand()`​的值

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

void rc4(unsigned char *key, int key_len, unsigned char *data, int data_len) {
	unsigned char s[257];
	unsigned char t[257];
	int j = 0;
	// 使用魔改的初始化 S 盒
	for (int i = 0; i < 257; i++) {
		s[i] = i;
		t[i] = key[i % key_len];
	}

	for (int i = 0; i < 257; i++) {
		j = (j + s[i] + t[i]) % 257;
		int temp = s[i];
		s[i] = s[j];
		s[j] = temp;
	}

	int i = 0, k = 0;
	for (int l = 0; l < data_len; l++) {
		i = (i + 1) % 257;
		k = (k + s[i]) % 257;
		s[i]=s[k];
		s[k] = s[i];
		data[l] = data[l] ^ s[(s[i] + s[k]) % 257];
	}
}

int main() {
	char key[] = "JustDoIt!";  // 已知密钥
	for(int val=0;val<666;val++){
		strcpy(key,"JustDoIt!");

		unsigned char data[] = {
			0x83, 0x1e, 0x9c, 0x48, 0x7a, 0xfa, 0xe8, 0x88,
			0x36, 0xd5,
			0x0a, 0x08, 0xf6, 0xa7, 0x70, 0x0f, 0xfd, 0x67,
			0xdd, 0xd4,
			0x3c, 0xa7, 0xed, 0x8d, 0x51, 0x10, 0xce, 0x6a,
			0x9e, 0x56,
			0x57, 0x83, 0x56, 0xe7, 0x67, 0x9a, 0x67, 0x22,
			0x24, 0x6e,
			0xcd, 0x2f
		};
		for(int j=0;j<9;j++){
			key[j]^=val;
		}
		int data_len = sizeof(data) / sizeof(data[0]);
		rc4((unsigned char*)key,9,data,data_len);
		if (data[0]=='H'&&data[1]=='Z'&&data[2]=='N'&&data[3]=='U') {
			for(int i=0;i<42;i++){
				printf("%c",data[i]);
			}
			printf("\nrand()=%d",val);
			break;
		}
	}
	return 0;
}
//HZNUCTF{489b88-1305-411e-b1f4-88a3070a73}
//rand()=56
```

#### 总结

`d810`​和`angr`​都不能完美恢复这道题的代码，只能硬读去找没被混淆的代码，还有就是要仔细读，不要被`ida`​给的变量名给骗了/(ㄒoㄒ)/~~

> 工具的效果是有限的，面对极致的混淆也束手无策

### index

当时看到.js文件还以为是`js`​逆向，结果是wasm(哭)，虽然知道也不知道怎么做

就看了看官方和大佬的WP

主要是花了不少时间配环境，这台电脑上没装`ghidra`​(

**​`ghidra 装插件之后可以很好的反编译 wasm`​**

先在[`https://github.com/NationalSecurityAgency/ghidra`](https://github.com/NationalSecurityAgency/ghidra)​下了软件

然后[`zackelia/ghidra-dark-theme: Modern dark theme based on the original ghidra-dark`](https://github.com/zackelia/ghidra-dark-theme)​上换了个主题（原版颜色太费眼了）

插件的网址[`https github.com/nneonneo/ghidra-wasm-plugin/`](https://github.com/nneonneo/ghidra-wasm-plugin/)

反编译后的逻辑不难，花点时间就能理清

> 话说怎么像**ida**一样实现**交叉引用**、**esc**回退以及数据提取？每次都要先回`main`再到目标函数，太唐了(主要是因为不会数据提取而放弃了重新完整走一遍复现的想法~~

### 总结

这次的题总体感觉不难，但就是不知道为什么在某几个小地方卡了很久，怀疑是思维的问题----中下难度的题刷多了造成一些思维固化，感觉更应该从逆向者的角度来看每一道题，而不是经验性地每种题有几乎固定的解题流程，这样的提升兴许会更大。

继续沉淀~

‍
