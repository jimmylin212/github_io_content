---
title: "從 Nodejs 呼叫 C++"
date: 2017-12-09T10:05:56+08:00
draft: true
---
最近因為工作的關係，需要利用 Nodejs 去呼叫 C++ 寫好的 module，網路上找了找發現還真的有一些方法，不過資料不是很多，找出來也都很片段，甚至不是我需要的，這邊整理一下也是以後比較好參考用。大致分成下面幾種方法：

1. [node-ffi](https://github.com/node-ffi/node-ffi)
2. [N-API](https://nodejs.org/docs/latest/api/n-api.html)
3. [Native Abstractions for Node.js (NaN)](https://github.com/nodejs/nan)

我們最後選了 NaN 來開發，為什麼捨棄前兩個呢？`node-ffi` 是蠻易用的，不過我們看了一下他們的 Git，發現已經有一陣子沒有更新了，對於開發一個 production 的 project 來說，還是想要用較新版而且有在更新的 module；至於 `N-API` 就太新了，2017年 Nodejs 還在某個研討會上面介紹，而且官網也說目前正在試驗階段，不適合拿來 production 使用，另外 Document 也相對較少，開發起來更花時間。

那我們到底想要做到啥事情呢？在別的部門已經有人用 C++ 開發出可以控制更底層的 module，我們的目標就是用 Nodejs 去呼叫那個 module 所 build 出來的 .so 檔，就這麼簡單，不過花了非常多時間研究。

