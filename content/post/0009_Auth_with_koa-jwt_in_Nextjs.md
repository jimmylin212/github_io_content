---
title: "使用 JWT 對 Nextjs 服務做身份認證"
date: 2018-06-04T14:58:16+08:00
draft: true
author: "Jimmy Lin"
isCJKLanguage: true
tags: ["Nextjs", "Nodejs"]
categories: ["Web App Development"]
---

最近這幾天處理我們 Nextjs 服務身分認證的流程，我們用到蠻多的 library，網路上很多教學，不過都沒有一次把全部都串再一起，當中遇到一些問題，也做了一些嘗試，做個簡單的紀錄。

我們使用了下面這些 library：React、Nextjs、koa、koa-router、koa-jwt、ldapjs。下面列一些網路上找到的範例：

- 在 Nextjs 中使用 koa-custom-server
- jwt-token 結合 koa 做身份認證