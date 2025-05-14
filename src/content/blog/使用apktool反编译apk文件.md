---
title: "apktool配置及使用"
description: "apktool 常用于反编译 Android 中的资源文件 , 主要是获取 AndroidManifest.xml、res 目录下的图片、布局、style 风格配置等资源文件。注意，当前目录以及文件路径不能包含中文"
pubDate: "5 14 2025"
image: "https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithubimage-20250513134438-ep7u4po.png"
categories:
  - tech
tags:
  - 逆向技术
  - 安卓逆向
---

## 环境配置

### 1.基本环境

jeb(jadx)、雷电9、adb

参考[环境配置](环境配置.md)

### 2.配置及使用apktool

#### 下载

参考：

[Android逆向之旅—反编译利器Apktool使用教程（Apktool的安装使用）建议新手浏览-CSDN博客](https://blog.csdn.net/qq_41866988/article/details/127589900)

[【Android】使用Apktool反编译Apk文件 - R-Bear - 博客园](https://www.cnblogs.com/R-bear/p/18076144)

下载地址：[Install Guide | Apktool](https://apktool.org/docs/install/)

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithubimage-20250513134438-ep7u4po.png)

下载并配置好环境变量

cmd输入`apktool`​输出以下内容代表成功

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250513140338013.png)

‍

#### 使用

测试用例：NewStarCTF2022 艾克体悟题

题目给了一个apk文件，给它补后缀.apk，拖入模拟器里运行

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250513141227947.png)

同时使用`jeb`​分析

根据题目描述要运行`FlagActivity`​

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250513141518782.png)

如何运行参考[入门指南](https://q7h2q9.top/blog/安卓逆向入门指南/)

使用MT管理器获取包名，其实在jeb里也直接看得到

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250513141752379.png)

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250513141825580.png)

执行

```bash
su #需要root权限
am start -n com.droidlearn.activity_travel/com.droidlearn.activity_travel.FlagActivity
```

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250513141955192.png)

题目说明点击1w次就出flag，尝试修改内部逻辑。这里使用`apktool`​进行解包和再打包

##### **解包：**

```bash
apktool d apk.apk #在apk目录下
#！！！！！注意，当前目录以及文件路径不能包含中文！！！！！
```

在当前目录下生成了一个`apk`​文件夹

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250513142238225.png)

在`apk/smali/com/droidlearn/activity_travel`​下找到`FlagActivity$1.smali`​

找到比较部分的代码

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250513142649078.png)

0x2710就是10000，这里改成5。保存

##### 重打包

执行命令，生成新的apk文件，为了便于区分识别，把文件夹名字从`apk`​改成`apk_folder`​

```bash
apktool b apk_folder
```

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250513142949572.png)

输出显示新的apk在`dist`​目录下，为了区分，把新的apk命名问`new_apk.apk`​

这时候直接拖进模拟器是不行的，因为还没有签名

##### 签名apk

> 以下大部分都参考这篇文章：[【Android】使用Apktool反编译Apk文件 - R-Bear - 博客园](https://www.cnblogs.com/R-bear/p/18076144)

1.生成密钥库文件，即`keystore`​文件

使用以下命令生成

```bash
keytool -genkey -alias android_keystore -keyalg RSA -validity 20000 -keystore android.keystore
```

以上各个参数的含义如下：

* ​`-genkey`​：这个参数表示keytool要生成一个新的密钥对和一个自签名的证书
* ​`-alias android_keystore`​：这个参数指定了生成的密钥对和证书的别名（alias），此处别名为android\_keystore，这个别名在密钥库中用于唯一标识这个特定的密钥对和证书
* ​`-keyalg RSA`​：这个参数指定了用于生成密钥对的算法，即RSA算法。RSA是一种广泛使用的非对称加密算法，它使用一对密钥：一个公钥用于加密数据，另一个私钥用于解密数据
* ​`-validity 20000`​：这个参数设置了证书的有效期，以天为单位，此处证书的有效期是20000天
* ​`-keystore android.keystore`​：这个参数指定了密钥库（keystore）的文件名，即android.keystore。密钥库是一个用于存储密钥对和证书的数据库文件

当使用keytool生成密钥对和证书时，命令执行过程需要输入信息：

* 密钥库的密码：此处我填写为 `q7h2q9`​
* 密钥对和证书的所有者姓名、组织单位、城市或地区、省/州/郡、国家代码等，这些信息将被包含在生成的证书中，用于标识证书的所有者。

这里要求输入密钥口令和其他信息，随便输，**口令得记住**

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250513143535404.png)

在当前目录下生成了`android.keystore`​

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250513143600445.png)

2.将刚刚生成的android.keystore文件，拷贝到未签名的apk文件同级目录下，切换到该路径下，输入以下命令，进行签名：

```bash
jarsigner -verbose -keystore android.keystore -signedjar new_apk-signed.apk new_apk.apk android_keystore
```

以上各个参数的含义如下：

* ​`jarsigner`​：这是 Java 开发工具包 (JDK) 中的一个工具，用于对 JAR 文件、APK 文件等进行签名
* ​`-verbose`​：这个参数用于输出详细的签名过程，当使用这个参数时，jarsigner 会显示更多关于签名步骤的信息，这有助于调试和了解签名过程的具体情况
* ​`-keystore android.keystore`​：这个参数指定了密钥库文件的路径和名称，即 android.keystore
* ​`-signedjar new_apk-signed.apk new_apk.apk`​：这个参数指定了签名后的 APK 文件的输出路径和名称，即 `new_apk-signed.apk`​，这个文件是原始 APK 文件  `new_apk.apk`​ 经过签名后的结果
* ​`android_keystore`​：这个参数指定了密钥库中用于签名的密钥对的别名，别名是在使用 keytool 生成密钥对时指定的，它用于在密钥库中唯一标识这个特定的密钥对

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250513145357353.png)

在当前目录下生成了`new_apk-signed.apk`​文件，就是签名后的apk

导入模拟器，运行`FlagActivity`​

点击5次后，出`flag`​

![image](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithub20250513145704285.png)

over

### 3.总结

指令总结：

#！！！！！注意，当前目录以及文件路径不能包含中文！！！！！

```bash
#解包
apktool d apk文件 #在apk目录下
#重打包
apktool b 解包后得到的文件夹名
#生成密钥库文件
keytool -genkey -alias android_keystore -keyalg RSA -validity 20000 -keystore android.keystore
#签名apk
jarsigner -verbose -keystore android.keystore -signedjar 签名后的apk名字 原apk名 android_keystore
```

参考资料：

[【Android】使用Apktool反编译Apk文件 - R-Bear - 博客园](https://www.cnblogs.com/R-bear/p/18076144)

[NewStarCTF 公开赛第一周Writeup | Anyyy安全小站](https://www.anyiblog.top/2022/09/25/20220925/#Re5-%E8%89%BE%E5%85%8B%E4%BD%93%E6%82%9F%E9%A2%98)

‍
