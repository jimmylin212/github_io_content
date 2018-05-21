---
title: "利用 Go 實作 Crawler 並存資料至 Google Firebase"
date: 2018-05-01T00:16:02+08:00
draft: true
author: "Jimmy Lin"
isCJKLanguage: true
tags: ["Golang", "Back-end"]
categories: ["Web App Development"]
---

最近想要寫一個 crawler 抓些第三方網站上的資料並且儲存在 Google Firebase 當中，之前都習慣用 Python 來寫，因為熟悉，開發起來速度快，一下子就可以搞定了。看到 Google Firebase 的教學當中可以使用 Nodejs、Java、Python 以及 Go 來作為 Admin 的開發使用。雖然 Nodejs 現在很流行，但是我覺得以 Go 語言的特性以及效能，未來應該可以狠甩 Nodejs 幾個街口，那就來玩玩 Go 吧。

網路上找找 Go 語言的特性很適合拿來開發 RESTful API，不過可惜的是我這邊並不是開發一個 RESTful API，而是一個單純的 Backend Script，這個 Script 的功能很簡單：

1. 去某個網頁抓特定的 URL，並且下載存成 Excel 檔
2. 讀取 Excel 檔，簡單過濾一些不要的資料
3. 比對現有的 DB，進行新增以及更新
4. 把整包程式打包成一個可執行檔，方便移植到其他平台

那就開始吧。

## Go 開發環境介紹

## 實作 Crawler

## 讀取 Excel 檔案

## 連結 Google Firebase Database

## 打包成可執行檔