---
title: "利用 Golang 上傳檔案至 GraphQL Server"
date: 2018-06-14T00:16:02+08:00
draft: true
author: "Jimmy Lin"
isCJKLanguage: true
tags: ["Golang", "GraphQL", "Backend"]
categories: ["Web App Development", "Back-end Development"]
---

我們目前做的網站提供上傳以及下載的功能，另外想要提供一個 offline CLI Tool 可以讓使用者更方便的上傳或下載檔案，也有可能可以整合這個 CLI tool 到他們自己的 CI\CD 流程當中。因為是獨立的 tool，想要用 Golang 來完成，順便可以了解一下這個語言的特性。

在 server 上我們用了下面這些 modules：Nextjs、Koa、Koa-Router、、GraphQL、Mongoose、

- Database: MongoDB