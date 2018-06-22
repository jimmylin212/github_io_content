---
title: "為 Golang CLI 打造漂亮的 Help Message"
date: 2018-06-22T13:28:30+08:00
draft: true
author: "Jimmy Lin"
isCJKLanguage: true
tags: ["Golang", "Backend"]
categories: ["Back-end Development"]
---

Golang 提供了簡單的方法產生 Help Message 以及支援使用者輸入，不過如果你的 CLI Tool 功能較多，或是有階層性的，那要怎麼辦？我們可以利用 [Cobra](https://github.com/spf13/cobra) 以及 [Viper](https://github.com/spf13/viper)，這兩個其實是完全不相關的 module，可以單獨使用，不過我覺得結合再一起真的是非常不錯。Cobra 是用來產生漂亮的 CLI 介面以及支援處理使用者的輸入；Viper 則是負責處理 config 檔。想像一下，一個常見的 CLI，程式輸入不外乎就是從這兩個入口進來，所以一起來看一下吧！

{{< figure src="/images/cover/0010.png" width="" height="" >}}

## Cobra
雖然 Cobra 的 ReadMe 提供了一些教學，不過有些東西並沒有寫得非常清楚，而且這些教學最終的結果都只是印出一些無用的訊息，對於要建構一個大型一點的或複雜一點的 CLI 幫助不大，另外有些基本用法可能太基本，所以並沒有提到，蠻可惜的，這邊就補充一下。

本來 Golang 的程式入口都在 `main()`，不過利用了 Cobra，程式的入口變成一個一個的 command，再由 command 去呼叫他所需要的 function，所以本來的 `main()` 就單純許多。要開始使用 Cobra，可能有兩種情況：

1. 程式的功能都已經完善，只是想要加上 CLI 的輸入
2. 程式都還沒開始開發，想要從 CLI 輸入開始

就假設我們的專案叫做「RealProject」
```go
// 

```

## Viper