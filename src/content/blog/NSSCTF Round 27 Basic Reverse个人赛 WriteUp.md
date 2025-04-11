---
title: "NSSCTF Round 27 Basic Reverse个人赛 WriteUp"
description: " "
pubDate: "March 16 2025"
image: /images/9.jpg
categories:
  - tech
tags:
  - CTF
  - WP
---

## NSSCTF Round 27 Basic Reverse个人赛 WriteUp

难度偏简单，但题型丰富

### easyre

有一些常规花指令，nop掉就行

![image-20250316185116877](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithubimage-20250316185116877.png)

核心逻辑：

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  int v3; // eax
  int v4; // eax
  int v6; // [esp-8h] [ebp-970h]
  char v7; // [esp+0h] [ebp-968h]
  char v8; // [esp+0h] [ebp-968h]
  int i; // [esp+310h] [ebp-658h]
  unsigned int v10; // [esp+31Ch] [ebp-64Ch]
  char v11[520]; // [esp+328h] [ebp-640h] BYREF
  char Str[520]; // [esp+530h] [ebp-438h] BYREF
  _BYTE v13[264]; // [esp+738h] [ebp-230h] BYREF
  char v14[32]; // [esp+840h] [ebp-128h]
  int a1[65]; // [esp+860h] [ebp-108h] BYREF
  int savedregs; // [esp+968h] [ebp+0h] BYREF

  j_memset(a1, 0, 0x100u);
  v14[0] = 65;
  v14[1] = -15;
  v14[2] = -53;
  v14[3] = 125;
  v14[4] = 8;
  v14[5] = 8;
  v14[6] = 92;
  v14[7] = 105;
  v14[8] = -23;
  v14[9] = -7;
  v14[10] = 53;
  v14[11] = 88;
  v14[12] = 89;
  v14[13] = 104;
  v14[14] = -62;
  v14[15] = 18;
  v14[16] = -63;
  v14[17] = -39;
  v14[18] = -69;
  v14[19] = 47;
  v14[20] = 109;
  v14[21] = 17;
  v14[22] = -124;
  v14[23] = 0;
  strcpy(v13, "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/");
  j_memset(&v13[65], 0, 0xBFu);
  j_memset(Str, 0, 0x200u);
  j_memset(v11, 0, 0x200u);
  printf("please give me your flag:\n", v7);
  scanf("%24s", (char)Str);
  v10 = j_strlen(Str);
  v3 = j_strlen(v13);
  sub_DD1870((int)a1, (int)v13, v3);
  sub_DD1A00((int)a1, (int)v11, v10);
  sub_DD1118((int)Str, (int)v11, v10);
  for ( i = 0; i < 24; ++i )
  {
    if ( Str[i] != v14[i] )
    {
      printf("you are wrong\n", v8);
      exit(0);
    }
  }
  printf("right\n", v8);
  v4 = system("pause");
  sub_DD125D(1);
  v6 = v4;
  sub_DD11F9((int)&savedregs, (int)&dword_DD202C);
  return v6;
}
```

```c
unsigned int __cdecl sub_DD17D0(int a1, int a2, unsigned int a3)
{
  unsigned int result; // eax
  unsigned int i; // [esp+D0h] [ebp-8h]

  sub_DD1343(&unk_DDD014);
  for ( i = 0; ; ++i )
  {
    result = i;
    if ( i >= a3 )
      break;
    *(_BYTE *)(i + a1) ^= *(_BYTE *)(i + a2);
  }
  return result;
}
```

直接动调到`sub_DD17D0`拿值就行，然后异或

exp：

```c
#include<stdio.h>
int main(){
	unsigned char flag[24] = {
		0x0F, 0xA2, 0x98, 0x3E, 0x5C, 0x4E, 0x27, 0x3D, 0x81, 0x90, 0x46, 0x07, 0x68, 0x3B, 0x9D, 0x60,
		0xA2, 0xED, 0xE4, 0x1C, 0x2C, 0x42, 0xDD, 0x7D
	};
	unsigned char data[23] = {
		0x41, 0xF1, 0xCB, 0x7D, 0x08, 0x08, 0x5C, 0x69, 0xE9, 0xF9, 0x35, 0x58, 0x59, 0x68, 0xC2, 0x12,
		0xC1, 0xD9, 0xBB, 0x2F, 0x6D, 0x11, 0x84
	};

	for(int i=0;i<24;i++){
		flag[i]^=data[i];
		printf("%c",flag[i]);
	}

	return 0;
}
//NSSCTF{This_1S_rc4_3ASY}
```

### EzCpp

cpp写的exe，定位到main

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char *v3; // rdi
  __int64 i; // rcx
  __int64 v5; // rax
  __int64 v6; // rax
  __int64 v7; // rax
  __int64 v8; // rax
  __int64 v9; // rax
  __int64 v10; // rax
  __int64 v11; // rax
  char v13[32]; // [rsp+0h] [rbp-20h] BYREF
  char v14; // [rsp+20h] [rbp+0h] BYREF
  char Str[44]; // [rsp+28h] [rbp+8h] BYREF
  unsigned int v16; // [rsp+54h] [rbp+34h]

  v3 = &v14;
  for ( i = 30i64; i; --i )
  {
    *(_DWORD *)v3 = 0xCCCCCCCC;
    v3 += 4;
  }
  sub_7FF7B64A1618(byte_7FF7B64BB0F2);
  v5 = sub_7FF7B64A10C3(std::cout, "Do you Know CPP");
  std::ostream::operator<<(v5, sub_7FF7B64A1046);
  v6 = sub_7FF7B64A10C3(std::cout, "Length Is 23");
  std::ostream::operator<<(v6, sub_7FF7B64A1046);
  v7 = sub_7FF7B64A10C3(std::cout, "Place Input Flag");
  v8 = sub_7FF7B64A10C3(v7, "\n");
  std::ostream::operator<<(v8, sub_7FF7B64A1046);
  std::istream::getline(std::cin, Str, 24i64);
  v16 = j_strlen(Str);
  if ( v16 != 23 )
  {
    v9 = sub_7FF7B64A10C3(std::cerr, "Length Is Error");
    std::ostream::operator<<(v9, sub_7FF7B64A1046);
    v10 = std::ostream::operator<<(std::cout, v16);
    std::ostream::operator<<(v10, sub_7FF7B64A1046);
    system("pause");
    exit(1);
  }
  if ( (unsigned int)check(Str) )
    v11 = sub_7FF7B64A10C3(std::cout, "WOW! You konw cpp , verry good! , flag is right");
  else
    v11 = sub_7FF7B64A10C3(std::cout, "Oh no! , You should study cpp , flag is wrong");
  std::ostream::operator<<(v11, sub_7FF7B64A1046);
  system("pause");
  sub_7FF7B64A1523((__int64)v13, (__int64)&unk_7FF7B64B1190);
  return 0;
}
```

直接看`check`（自己改的名）

```c
__int64 __fastcall sub_7FF7B64A6980(const char *a1)
{
  char *v1; // rdi
  __int64 i; // rcx
  unsigned __int128 v3; // rax
  __int64 v4; // rax
  unsigned int v5; // edi
  char v7[32]; // [rsp+0h] [rbp-20h] BYREF
  char v8; // [rsp+20h] [rbp+0h] BYREF
  char v9[60]; // [rsp+28h] [rbp+8h] BYREF
  int v10; // [rsp+64h] [rbp+44h]
  __int64 v11; // [rsp+88h] [rbp+68h]
  char v12[32]; // [rsp+A4h] [rbp+84h] BYREF
  int j; // [rsp+C4h] [rbp+A4h]
  char v14[60]; // [rsp+E8h] [rbp+C8h] BYREF
  unsigned int v15; // [rsp+124h] [rbp+104h]
  __int64 v16; // [rsp+208h] [rbp+1E8h]
  __int64 v17; // [rsp+228h] [rbp+208h]
  unsigned int v18; // [rsp+244h] [rbp+224h]
  int v19; // [rsp+254h] [rbp+234h]
  unsigned __int64 v20; // [rsp+258h] [rbp+238h]
  unsigned __int64 key_len; // [rsp+260h] [rbp+240h]

  v1 = &v8;
  for ( i = 102i64; i; --i )
  {
    *(_DWORD *)v1 = -858993460;
    v1 += 4;
  }
  sub_7FF7B64A1618((__int64)&byte_7FF7B64BB0F2);
  sub_7FF7B64A1609((__int64)v9);
  v10 = j_strlen(a1);
  v20 = v10;
  v3 = (unsigned __int64)v10 * (unsigned __int128)4ui64;
  if ( !is_mul_ok(v10, 4ui64) )
    *(_QWORD *)&v3 = -1i64;
  v16 = sub_7FF7B64A12A3(v3, *((__int64 *)&v3 + 1));
  v11 = v16;
  qmemcpy(v12, "harker", 6);
  for ( j = 0; j < v10; ++j )
  {
    v19 = a1[j];
    v20 = j;
    key_len = sub_7FF7B64A10AF((__int64)v12);
    *(_DWORD *)(v11 + 4i64 * j) = v12[v20 % key_len] ^ v19;
    sub_7FF7B64A1221((__int64)v9, *(unsigned __int8 *)(v11 + 4i64 * j));
  }
  base64((__int64)v14, (__int64)v9);
  v4 = sub_7FF7B64A109B((__int64)v14);
  v15 = sub_7FF7B64A1212(v4);
  v17 = v11;
  sub_7FF7B64A12FD(v11);
  if ( v17 )
  {
    v11 = 0x8123i64;
    v20 = 0x8123i64;
  }
  else
  {
    v20 = 0i64;
  }
  v18 = v15;
  sub_7FF7B64A113B(v14);
  sub_7FF7B64A113B(v9);
  v5 = v18;
  sub_7FF7B64A1523((__int64)v7, (__int64)&unk_7FF7B64B1100);
  return v5;
}
```

逻辑比较简单，先循环异或`key`，然后base64加密，密文是`Dg0TDB4xGBEtWlAtPlIAGRwtW1VHEhg=`，先拿base64解密脚本还原加密前的16进制数据。

exp:

```c
#include<stdio.h>
char key[]={"harker"};
int main(){
	unsigned char flag[]={14, 13, 19, 12, 30, 49, 24, 17, 45, 90, 80, 45, 62, 82, 0, 25, 28, 45, 91, 85, 71, 18, 24};
	for(int i=0;i<23;i++){
		flag[i]^=key[i%6];
		printf("%c",flag[i]);
	}
	return 0;
}
//flag{Cpp_15_V3rry_345y}
```

### EzProcessStruct

windows内核逆向，还好不复杂

```c
int __stdcall sub_401000(int a1, int a2)
{
  unsigned int v2; // ebx
  int v3; // ecx
  int v4; // esi
  int v5; // esi
  wchar_t *v6; // edi
  wchar_t *v7; // eax
  unsigned int v8; // esi
  PEPROCESS v9; // edi
  char v10; // dl
  int v11; // eax
  const char *v12; // eax
  __int16 *v14; // [esp-Ch] [ebp-27Ch]
  const wchar_t *v15; // [esp-8h] [ebp-278h]
  size_t v16; // [esp-4h] [ebp-274h]
  PEPROCESS Process; // [esp+10h] [ebp-260h] BYREF
  char v18; // [esp+17h] [ebp-259h]
  struct _KAPC_STATE ApcState; // [esp+18h] [ebp-258h] BYREF
  __int16 Dest[256]; // [esp+30h] [ebp-240h] BYREF
  int v21[9]; // [esp+230h] [ebp-40h] BYREF
  char v22[16]; // [esp+254h] [ebp-1Ch] BYREF
  char v23[8]; // [esp+264h] [ebp-Ch] BYREF

  v2 = 0;
  Process = 0;
  PsLookupProcessByProcessId((HANDLE)0xE6C, &Process);
  memset(&ApcState, 0, sizeof(ApcState));
  KeStackAttachProcess(Process, &ApcState);
  qmemcpy(v21, "SkISV6U@b5Q1_6cwejUqc4Iaf5Q~ejQtNTA>", sizeof(v21));
  v3 = *((_DWORD *)Process + 106);
  v4 = *(_DWORD *)(v3 + 16);
  DbgPrint("%d.%d.%d", *(_DWORD *)(v3 + 164), *(_DWORD *)(v3 + 168), *(unsigned __int16 *)(v3 + 172));
  v5 = v4 + 64;
  if ( v5 )
  {
    memset(Dest, 0, sizeof(Dest));
    v6 = wcsrchr(*(const wchar_t **)(v5 + 4), 0x5Cu);
    v7 = wcsrchr(*(const wchar_t **)(v5 + 4), 0x2Eu);
    v18 = 1;
    strcpy(v22, "you found flag!");
    v8 = v7 - v6 - 1;
    strcpy(v23, "wrong");
    if ( v6 )
    {
      if ( v7 && v6 < v7 )
      {
        v16 = v7 - v6 - 1;
        v15 = v6 + 1;
        v14 = Dest;
        _mm_lfence();
        wcsncpy((wchar_t *)v14, v15, v16);
        if ( 2 * v8 >= 0x200 )
          sub_401268();
        Dest[v8] = 0;
        DbgPrint("%ws\n", Dest);
        if ( !v8 )
          goto LABEL_12;
        v9 = Process;
        v10 = v18;
        do
        {
          v11 = (unsigned __int16)(Dest[v2] ^ *(_WORD *)(*((_DWORD *)v9 + 0x6A) + 0xA4) ^ *(_WORD *)(*((_DWORD *)v9 + 0x6A) + 0xA8));
          Dest[v2] = v11;
          if ( v10 )
            v10 = v11 != *((char *)v21 + v2) ? 0 : v10;
          ++v2;
        }
        while ( v2 < v8 );
        v12 = v23;
        if ( v10 )
LABEL_12:
          v12 = v22;
        DbgPrint("%s\n", v12);
      }
    }
  }
  KeUnstackDetachProcess(&ApcState);
  *(_DWORD *)(a1 + 52) = sub_401230;
  return 0;
}
```

核心加密逻辑就是异或

`v11 = (unsigned __int16)(Dest[v2] ^ *(_WORD *)(*((_DWORD *)v9 + 0x6A) + 0xA4) ^ *(_WORD *)(*((_DWORD *)v9 + 0x6A) + 0xA8));`

`dest`就是flag，主要看v9

结合题目描述可知异或的值是7，再异或回来就行，最后还要base64解码一次

![image-20250316190046343](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithubimage-20250316190046343.png)

```c
#include<stdio.h>
#include<string.h>
int main(){
	char flag[]={"SkISV6U@b5Q1_6cwejUqc4Iaf5Q~ejQtNTA>"};
	for(int i=0;i<strlen(flag);i++){
		flag[i]^=7;
	}
	printf("%s",flag);
	return 0;
}
//TlNTQ1RGe2V6X1dpbmRvd3Nfa2VybmVsISF9
//NSSCTF{ez_Windows_kernel!!}
```

### ezminiprogram

微信小程序逆向，给了个`index.wxml`和`__APP__.wxapkg`

第一次做这种题，查资料才知道可以用`unwxapkg.py`解包

注意`index.wxml`里给的信息，那么就找`onInputChange`

```html
<!-- index.wxml -->
<navigation-bar title="Weixin" back="{{false}}" color="black" background="#FFF"></navigation-bar>
<scroll-view class="scrollarea" scroll-y="true" scroll-with-animation="true" style="height: 100vh">
  <view class="container">
    <view animation="{{animationData}}" class="input-container">
      <input class="cool-input" placeholder="请输入内容..." bindinput="onInputChange" value="{{inputValue}}" />
      <icon type="search" class="input-icon"></icon>
    </view>
    <button
      animation="{{animationData}}"
      class="check-button"
      bindtap="onCheck"
      style="position: relative; left: -1rpx; top: -981rpx"
    >
      Check
    </button>
  </view>
</scroll-view>

```

解包后查看文件找关键逻辑，在`index.appservice.js`找到了验证代码

![image-20250316190637576](https://gcore.jsdelivr.net/gh/Q7h2q9/photohouse/picgoandgithubimage-20250316190637576.png)

找在线网站美化一下：

```js
__wxCodeSpace__.batchAddCompiledTemplate((G, R, D, Q, gdc, X, Y, Z, RG = {}) => {
  let P = RG.P || function (a) {
    return typeof a === "function" ? a : () => {};
  };
  return {
    "pages/index/index": (() => {
      let H = {};
      let S;
      let I = (P) => {
        if (!S)
          S = Object.assign({}, H);
        return S[P];
      };
      H[""] = (R, C, D, U) => {
        let L = R.c;
        let M = R.m;
        let O = R.r;
        let A = {
          animationData: Array.from({ length: 2 }),
          inputValue: Array.from({ length: 1 })
        };
        let K = U === true;
        let b = (C) => {};
        let f = (C) => {};
        let g = (C) => {};
        let e = (C, T, E) => {
          E("input", {}, (N, C) => {
            R.devArgs(N).A = [":class", "placeholder", "bindinput", "value",];
            if (C)
              L(N, "cool-input");
            if (C)
              O(N, "placeholder", "请输入内容...");
            if (C)
              O(N, "bindinput", "onInputChange");
            if (C || K || U.inputValue)
              O(N, "value", D.inputValue);
            A.inputValue[0] = (D, E, T) => {
              O(N, "value", D.inputValue);
              E(N);
            };
          }, f);
          E("icon", {}, (N, C) => {
            R.devArgs(N).A = [":class", "type",];
            if (C)
              L(N, "input-icon");
            if (C)
              O(N, "type", "search");
          }, g);
        };
        let h = (C, T) => {
          C ? T("Check") : T();
        };
        let d = (C, T, E) => {
          E("view", {}, (N, C) => {
            R.devArgs(N).A = [":class", "animation",];
            if (C)
              L(N, "input-container");
            if (C || K || U.animationData)
              O(N, "animation", D.animationData);
            A.animationData[0] = (D, E, T) => {
              O(N, "animation", D.animationData);
              E(N);
            };
          }, e);
          E("button", {}, (N, C) => {
            R.devArgs(N).A = [":class", ":style", "animation", "bindtap",];
            if (C)
              L(N, "check-button");
            if (C)
              R.y(N, "position: relative; left: -1rpx; top: -981rpx");
            if (C || K || U.animationData)
              O(N, "animation", D.animationData);
            A.animationData[1] = (D, E, T) => {
              O(N, "animation", D.animationData);
              E(N);
            };
            if (C)
              O(N, "bindtap", "onCheck");
          }, h);
        };
        let c = (C, T, E) => {
          E("view", {}, (N, C) => {
            R.devArgs(N).A = [":class",];
            if (C)
              L(N, "container");
          }, d);
        };
        let a = (C, T, E) => {
          E("navigation-bar", {}, (N, C) => {
            R.devArgs(N).A = ["title", "back", "color", "background",];
            if (C)
              O(N, "title", "Weixin");
            if (C || K || undefined)
              O(N, "back", false);
            if (C)
              O(N, "color", "black");
            if (C)
              O(N, "background", "#FFF");
          }, b);
          E("scroll-view", {}, (N, C) => {
            R.devArgs(N).A = [":class", ":style", "scroll-y", "scroll-with-animation",];
            if (C)
              L(N, "scrollarea");
            if (C)
              R.y(N, "height: 100vh;");
            if (C)
              O(N, "scroll-y", "true");
            if (C)
              O(N, "scroll-with-animation", "true");
          }, c);
        }; ;
        return {
          C: a,
          B: A
        };
      };
      return Object.assign((R) => {
        return H[R];
      }, {
        _: H
      });
    })(),
  };
}); ;
__wxRoute = "pages/index/index";
__wxRouteBegin = true;
__wxAppCurrentFile__ = "pages/index/index.js";
define("pages/index/index.js", (require, module, exports, window, document, frames, self, location, navigator, localStorage, history, Caches, screen, alert, confirm, prompt, XMLHttpRequest, WebSocket, Reporter, webkit, WeixinJSCore) => {
  "use strict";
  Page({
    data: {
      inputValue: "",
      animationData: {}
    },
    onLoad() {
      let t = wx.createAnimation({
        duration: 1e3,
        timingFunction: "ease"
      });
      t.opacity(1).translateY(0).step(), this.setData({
        animationData: t.export()
      });
    },
    onInputChange(t) {
      this.setData({
        inputValue: t.detail.value
      });
    },
    generateSbox(t) {
      for (var a = [], n = t.length, e = [], o = 0; o < 256; o++) a.push(o), e.push(t.charCodeAt(o % n));
      for (let i = 0, r = 0; r < 256; r++) {
        var s, u;
        s = [a[i = (i + a[r] + e[r]) % 256], a[r]], a[r] = s[0], a[i] = s[1], u = [a[(i + 1) % 256], a[(r + 1) % 256]], a[(r + 1) % 256] = u[0], a[(i + 1) % 256] = u[1];
      }
      return a;
    },
    onCheck() {
      let t = this.data.inputValue;
      if (t.trim() !== "") {
        for (var a = t.split(""), n = this.generateSbox("NSSCTF2025"), e = 0, o = 0, i = 0; i < a.length; i++) {
          let r = [n[e = (e + n[o = (o + n[i % 256]) % 256]) % 256], n[o]];
          n[o] = r[0], n[e] = r[1];
          let s = n[(n[o] + n[e]) % 256];
          a[i] = String.fromCharCode(a[i].charCodeAt(0) ^ s);
        }
        let u = a.map((t) => {
          return t.charCodeAt(0);
        });
        console.log(u);
        JSON.stringify(u) === JSON.stringify([216, 156, 159, 86, 8, 143, 254, 92, 113, 3, 228, 74, 37, 80, 146, 68, 71, 42, 137, 132, 170, 85, 13, 196, 226, 152, 120, 176, 184, 36, 195, 233, 123, 230, 89, 10, 121, 180, 5, 219])
          ? wx.showToast({
              title: "太强了！",
              icon: "success",
              duration: 2e3
            })
          : wx.showToast({
              title: "再试试喔",
              icon: "error",
              duration: 2e3
            });
      }
      else {
        wx.showToast({
          title: "请输入内容",
          icon: "none",
          duration: 2e3
        });
      }
    }
  });
}, {
  isPage: true,
  isComponent: true,
  currentFile: "pages/index/index.js"
});
require("pages/index/index.js");
```

魔改RC4（初始化多了一个交换，加解密部分的i、j赋值改变）

exp:

```c
#include <stdio.h>
#include <string.h>

// 初始化 S 盒
void rc4_init(unsigned char *sbox, const unsigned char *key, int keylen) {
	int i, j = 0;
	int temp;
	for (i = 0; i < 256; ++i) {
		sbox[i] = i;
	}
	for (int r = 0; r < 256; r++) {
		i = (i + sbox[r] + key[r%keylen]) % 256;
		temp = sbox[r];
		sbox[r] = sbox[i];
		sbox[i] = temp;

		// 交换位置
		temp = sbox[(i + 1) % 256];
		sbox[(i + 1) % 256] = sbox[(r + 1) % 256];
		sbox[(r + 1) % 256] = temp;
	}
}

// 加密/解密数据
void rc4_crypt(unsigned char *s, unsigned char *data, int len) {
	int i = 0, j = 0, t = 0;
	for (int k = 0; k < len; k++) {
		i = (i + s[k%256]) % 256;
		j = (j + s[i]) % 256;
		unsigned char tmp = s[i];
		s[i] = s[j];
		s[j] = tmp;
		// 交换 s[i] 和 s[j]
		t = (s[i] + s[j]) % 256;
		data[k] ^= s[t];
	}
}

int main() {
	// 密钥
	const char *key = "NSSCTF2025";
	int keylen = strlen(key);

	// 状态数组（S盒）
	unsigned char s[256];

	// 明/密文
	unsigned char v4[] = {216, 156, 159, 86, 8, 143,
		254, 92, 113, 3, 228, 74, 37, 80,
		146, 68, 71, 42, 137, 132, 170, 85,
		13, 196, 226, 152, 120, 176, 184, 36,
		195, 233, 123, 230, 89, 10, 121, 180, 5, 219};
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
//NSSCTF{c6e67111c1aadd1bdc4dad6d99c254e7}
```
