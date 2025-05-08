---
title: "Python解包与(再)打包"
description: "吾爱上偶然看到一篇关于Python解包与再打包的文章，跟着教程自己写一个例子走一遍"
pubDate: "5 8 2025"
image: "https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithubimage-20250508195607-p0qkz80.png"
categories:
  - tech
tags:
  - 逆向技术
  - Python
---

## Python解包与(再)打包

### 1.环境准备

* Python3.8

> 测试过py3.11，发现pyc反编译回py时不能完整还原源码，GPT的解释如下：
>
> 你用的是 Python 3.11a7（还处于 alpha 阶段），PyLingual 不一定完全支持这个字节码版本，导致反编译错误。（conda给的环境emmmm）

* 安装的库

  ```bash
  pip install lxml
  pip install lief
  ```

* 使用的工具

  [pyinstaller/pyinstaller](https://github.com/pyinstaller/pyinstaller)（打包）

  [PyInstaller Extractor](https://github.com/extremecoders-re/pyinstxtractor)（解包以及检查Python版本）

  [Pyinstaller Repacker](https://github.com/pyinstxtractor/pyinstaller-repacker)（解包以及再打包）

  [PyLingual](https://pylingual.io/)（在线反编译器，很强）

‍

### 2.测试程序

以两个py文件作为测试用例

`test.py`​

```py
from MyRC4 import*
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad
import base64
if __name__ == "__main__":
    print("Welcome to the Encryption Program")
    print("1. AES Encryption")
    print("2. Base64 Encoding")
    print("3. RC4 Encryption")
    print("4. Exit")
    print("Please select an option (1-4):")
    content=input("Enter your choice: ")
    plaintext="test.py"
    key = b'0123456789abcdef'
    if content == "1":
        print("You have selected option 1")
        # Encrypt the plaintext
        cipher = AES.new(key, AES.MODE_CBC)
        ciphertext = cipher.encrypt(pad(plaintext.encode(), AES.block_size))
        print("AES Encoded Ciphertext:", ciphertext.hex())
    elif content == "2":
        print("You have selected option 2")
        #base64 encode the ciphertext
        ciphertext = base64.b64encode(plaintext.encode()).decode()
        print("Base64 Encoded Ciphertext:", ciphertext)
    elif content == "3":
        print("You have selected option 3")
        # RC4 encryption
        initialize_sbox(key)
        ciphertext = rc4(key, plaintext.encode())
        print("RC4 Encoded Ciphertext:", ciphertext.hex())
    elif content == "4":
        print("You have selected option 4")
        print("oioioioioioiiiooii")
    else:
        print("Invalid choice. Please select a valid option.")


```

`MyRC4,py`​

```py
def initialize_sbox(key):
    """
    Initialize the S-box using the provided key.
    """
    sbox = list(range(256))
    key_length = len(key)
    
    # Key-scheduling algorithm (KSA)
    j = 0
    for i in range(256):
        j = (j + sbox[i] + key[i % key_length]) % 256
        sbox[i], sbox[j] = sbox[j], sbox[i]
    
    return sbox

def rc4(key, plaintext):
    """
    Perform RC4 encryption/decryption on the plaintext using the provided key.
    """
    sbox = initialize_sbox(key)
    i = j = 0
    keystream = []
    
    # Pseudo-random generation algorithm (PRGA)
    for char in plaintext:
        i = (i + 1) % 256
        j = (j + sbox[i]) % 256
        sbox[i], sbox[j] = sbox[j], sbox[i]
        keystream.append(sbox[(sbox[i] + sbox[j]) % 256])
    
    # XOR the plaintext with the keystream to get the ciphertext
    ciphertext = bytes([char ^ keystream[i]^0xf for i, char in enumerate(plaintext)])
    
    return ciphertext
```

注意这里进行了魔改，最后异或时多异或了个`0xf`​，目标是** 对打包后的exe进行解包然后拿到关键代码patch后再编译打包成新的exe，也就是去掉`0xf **`

#### 2.1打包

先打包成exe

```bash
pyinstaller -F test.py
```

会在当前目录下生成两个文件夹`build`​和`dist`​

`dist`​文件下有打包好的`test.exe`​

#### 2.2 解包、patch

使用`pyinstxtractor.py`​可以查看exe对应的Python版本，执行命令

```bash
python pyinstxtractor.py test.exe
```

确认版本后，切换到对应的Python环境，装库

```bash
pip install lxml
pip install lief
```

解包（注意参数`extract`​）

```bash
python .\pyinst-repacker.py extract test.exe
```

出现`Done`​表示解包成功

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithubimage-20250508172943-ezbwyad.png)

在`test.exe`​同级目录下生成`test.exe-repacker`​文件夹，

在`dist\test.exe-repacker\FILES`​中可以看到和使用`pyinstxtractor`​解包后一样的文件内容，在`PYZ.pyz`​文件夹里是导入模块。

找到`test.pyc`​和导入的`MyRC4`​模块，使用**PyLingual网站进行反编译**

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithubimage-20250508192556-tu634yp.png)

下载or复制，并进行修改，这里选择下载，得到`MyRC4.pyc_Decompiled.py`​，然后修改去掉`^0xf`​，保存，执行以下命令进行编译：

```bash
python -m py_compile MyRC4.pyc_Decompiled.py
```

在当前目录下会生成一个`__pycache__`​文件夹，里面有`MyRC4.cpython-38.pyc`​，修改名字为`MyRC4.pyc`​，并替换原来的对应文件，保存，执行以下命令**打包**

```bash
python .\pyinst-repacker.py build dist\test.exe-repacker
```

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithubimage-20250508193336-jblcmnc.png)

显示Done说明打包成功，重新运行，发现MyRC4已经被修改为标准RC4加密

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithubimage-20250508193443-e93lh86.png)

over

### 3.总结

总结一下思路和用到的命令

* 要使用Pyinstaller Repack，需要安装两个库

```bash
pip install lxml
pip install lief
```

* Python源码打包成exe

```bash
pyinstaller -F main.py
```

* 查看Python版本(也可以解包用于分析)

```bash
python pyinstxtractor.py test.exe
```

* 解包

```bash
python .\pyinst-repacker.py extract test.exe
#注意参数：extract
```

* 编译：.py→.pyc

```bash
python -m py_compile MyRC4.pyc_Decompiled.py
```

* 再打包(替换对应的pyc文件后)

```bash
python .\pyinst-repacker.py build dist\test.exe-repacker
#注意参数：build
```

* 另外

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithubimage-20250508194401-itx5xi6.png)

‍

### 4.参考资料

**[Pyinstaller Repack 指南 - 吾爱破解 - 52pojie.cn](https://www.52pojie.cn//thread-2025482-1-1.html)**

[PyLingual](https://pylingual.io/)

[pyinstxtractor/pyinstaller-repacker： Pyinstaller 重新打包程序](https://github.com/pyinstxtractor/pyinstaller-repacker)

[extremecoders-re/pyinstxtractor：PyInstaller 提取器](https://github.com/extremecoders-re/pyinstxtractor)
