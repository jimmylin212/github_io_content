---
title: "建立一個可用帳號密碼登入的 image"
date: 2018-05-20T21:06:13+08:00
draft: false
author: "Jimmy Lin"
isCJKLanguage: true
tags: ["Build", "Docker", "DevOps"]
categories: ["DevOps"]
---

最近因為工作需要了解了一下 Jenkins 以及 Docker 這兩個工具，雖然這兩個工具在 DevOps 中已經被使用多年，不過因為這次的經驗才第一次從零開始接觸，當中遇到很多問題，不過也學到蠻多東西的，希望可以把這些東西記錄下來，未來如果有人看到或是自己需要再 reference 的話，至少有個管道。

我們就從最基礎開始吧，如何建立一個 Docker Image。不過在先了解我們為什麼要建立之前，可以先了解一下 Docker 的概念，官網的[介紹](https://docs.docker.com/get-started/)其實蠻詳細的。另外我的環境是在 Windows 上面使用 [Docker for Windows](https://docs.docker.com/docker-for-windows/)，我覺得使用上沒有不一致，使用起來也相當得心應手，如果你的開發環境不全然是 Linux，是可以用 Docker for Windows 的。另外其實 Docker 的教學也寫得非常好，可以讓你真正從零開始打造一個自己的 Image，所以有很多類似的就不贅述了。

我們使用的情境是這樣，每一次 Jenkins build 的時候，會做下面三件事情

1. 會起一個暫時的 Container，這個 Container 裡面包含了設定好的環境以及安裝了一些必要的軟體
2. Jenkins 會使用我們設定好的帳號密碼登入 Container，並且執行 build 的指令
3. 當 build 完之後，會把 build 出來的結果，打包成一包 tar 檔，存放在 Jenkins 上面供其他的 Projects 使用。

那我們要怎麼做這個暫時的 Container 的 Image 呢？我們使用 Dockerfile，要製作一個客製化的 Image，有下面兩種方式：

1. 把你要的 base image 抓下來，登入或是利用 `docker exec CONTAINER COMMAND` 安裝必要軟體以及設定環境，之後再用 `commit` 指令建立新的 image。
2. 利用 Dockerfile 把你要做的事情一次寫完，之後只要下 `docker build -t IMAGENAME:TAG .` 就可以無痛的建立一個 image 了。

那我們先看這個 Dockerfile 的內容，我是拿最新的 Ubuntu 當作我的 base image
{{< gist jimmylin212 b53d72885b0f55c869901bdb85a8a56f >}}

總共做了下面幾件事情：

1. 安裝一些 Ubuntu 上需要的軟體。
2. 新增一組要登入的帳號以及密碼，並且給予 root 權限。
3. 切換到新增的帳號，安裝 nvm，主要是希望之後的同事可以把這份當作我們部門的 base image，只要指定好需要的 node 版本就好了。
4. 切換回 root，打開 22 port，讓外面可以透過 ssh 的方式連入

使用方法很簡單：

```
#把所有有開的 port 都給定一個對應的 port
docker run -P IMAGENAME:TAG 

#把 port 2222 指向 image 的 port 22，可用 port 22222 連入
docker run -p 22222:22 IMAGENAME:TAG 
```

啟動之後可以輸入

```bash
docker container ls
# or
docker ps 
```

可以看到你的 image 已經跑起來了，之後就可以透過 shell 連入 `127.0.0.1:22` 輸入我們設定好的帳號密碼，就可以登入囉。

列一下常用的 Docker 指令
```bash
docker image ls
docker image rm
docker container ls 
docker container ls --all
docker stop CONTAINER
docker rm CONTAINER
docker exec CONTAINER COMMAND
docker pull ACCOUNT/IMAGE:TAG
docker tag IMAGE:TAG ACCOUNT/IMAGE:TAG
docker push ACCOUNT/IMAGE:TAG
```