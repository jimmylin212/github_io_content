---
title: "使用 Hugo 建立靜態網站，並部署在 Github Page"
date: 2017-11-27T21:53:29+08:00
draft: false
tags: ["Hugo", "GitHub"]
categories: ["Web App Development"]
author: "Jimmy Lin"
isCJKLanguage: true
---

工作幾年了，下班之後會找些事情來做，寫一些小小的 side project，發現有時候要在網路上面找到中文的教學還挺難的。想要了解一些最新的技術也都只有英文的文件，不然就是簡體中文的，想一想既然都花時間讀了，也花時間搞懂，甚至實做出來了，那就順便寫個簡單的步驟或是翻譯。一方面自己可以更熟，另外一方面下次要找也可以直接從自己的文章當中找尋，比較省時間，所以，就來寫部落格吧。

因為工作的關係，寫 Markdown 寫得很習慣了，搜尋了一下有沒有用可以 Markdown 編輯的部落格，發現有一些可以轉換，不過還是不夠直接，那乾脆就架一個吧。找過蠻多靜態網站產生器的，像是 Hexo、Jekyll 和 Hugo。Jekyll 是 Github 認證，不過因為對 Windows 環境不友善，又因為我家裡的電腦是 Windows，所以作罷；Hexo 蠻不錯的，不過看到有些文章說如果網站內文章過多，產生靜態網站的速度就慢很多；Hugo 是用 Golang 寫的，速度很快，操作方法還蠻直覺的，社區也很大，所以最後選擇使用 Hugo 來當我的部落格的 Framework。

不過要架在哪裡？發現其實這些靜態網站都可以很聰明地部署到 Github Page，有自己的專屬域名，無限流量，自動備份，好移轉，就算之後想要換 Framework 或是移到其他部落格供應商，其實都有方法。所以這篇文章大概寫一下安裝以及架設的方法，之後應該會深入研究一下 Hugo 的一些功能，也會補上。

## 1. 安裝 Hugo
這邊只寫在 Windows 要怎麼安裝，要安裝 Hugo，首先要先安裝一個 Package manager 叫做 [chocolatey](https://chocolatey.org/)。

### 1.1. 安裝 Package manager - Chocolatey
要安裝 Chocolatey 可以用 `cmd.exe` 或是 `PowerShell.exe`，在 Chocolatey 網站上也有教學：

使用 `cmd.exe`
```bash
@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
```

使用 `Powershell.exe`
```bash
Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```

### 1.2. 安裝 Hugo
```bash
choco install hugo -confirm
```
當然還有一些需要的套件像是 [Git](https://git-scm.com/) 也要記得一起安裝上去。

可參考官網：https://gohugo.io/getting-started/installing/

## 2. 初始化網站
再來就是直接使用 Hugo 建立網站了

### 2.1. 新增網站
這個指令會幫你產生一些必要的檔案以及資料夾結構，這些都是日後需要的東西。
```bash
hugo new site my_first_site  ## 建立網站 my_first_site
```
建立之後可以進入資料夾看一下，可以發現有個檔案叫做 `config.toml`，這個就是 Hugo 主要的設定檔，其他資料夾應該會在後面其它文章介紹到。

### 2.2. 設定佈景主題
找一個你喜歡的佈景主題來用，[這個網站](https://themes.gohugo.io/) 有很多，而且也有依不同類別分類。
```bash
cd my_first_site ## 進入網站資料夾
git init ## 在這個資料夾初始化 Git
git submodule add git@github.com:digitalcraftsman/hugo-cactus-theme.git themes/cactus ## 把佈景主題加到這個 project 的 submodule
```
官網建議直接把 `theme="cactus"` 加入到 `config.toml` 中；不過我發現幾乎所有的主題都會有自己的 `config.toml` 檔案，所以可以先瀏覽一下這個主題的資料夾，看一下有沒有 `exampleSite`，如果有找到主題的 `config.toml`，複製並且取代本來的檔案即可。

### 2.3. 在本機端讓網站跑起來
```bash
hugo server -t cactus ## 記得加上 theme
```
跑起來之後可以看到在 stdout 看到可以連到 http://localhost:1313 看看網站長啥樣子，是不是符合你的期望。

### 2.4 上傳到 Github 先
如果希望可以用到 Github 的備份功能，建議這邊可以先離開 Hugo，到 Github 開一個新的專案，假設叫做 `hugo_blog`，可以使用下面的指令上傳這些配置以及資料結構到專案當中。如何使用 Github 以及如何新增專案，這邊就不多說拉，網路上中文介紹蠻多的。
```bash
git status
git add -A
git commit -m "my first commit for hugo blog"
git remote add origin git@github.com:hugo_blog/hugo_blog.github.io.git
git push -u origin master 
```

可參考官網：https://gohugo.io/getting-started/quick-start/

## 3. 部署到 Github Page
因為 Github 支援使用每一個帳號有一個專屬的 Github Page，這也是我們今天的重點，這樣就可以完整使用到 Github 全部的功能。首先我們要先到 Github 開另外一個新的專案，取名叫做 `<your-account>.github.io`。這邊要注意一下，剛剛執行 `hugo server` 指令，並沒有真正產生一個靜態網站，如果要用我們所擁有的設定，文章產生靜態網站，指令是 `hugo`，當執行 `hugo` 之後，會在同樣的目錄產生一個新的資料夾叫 `public`，這個資料夾才是我們真的要上傳到 `<your-account>.github.io` 專案的東西。說這麼多，那我們要怎麼關聯呢？建議的做法是使用 submodule，這樣也可以方便日後管理。

### 3.1 建立 submodule
```bash
git submodule add git@github.com:<your-account>/<your-account>.github.io.git public
```

### 3.2 生成靜態網站
```bash
hugo -t cactus ## 記得加上 theme
```

這樣應該可以看到在同目錄下新增了一個資料夾 `public`，裡面就是我們的網站拉，有 `index.html`，Github Page，就是認這個 `index.html`來生成網頁。如果上傳之後發現 CSS 或是 Javascript 讀不到，那就是在 `config.toml` 中的 `baseurl` 要改成 `<your-account>.github.io.git`，才可以確保路徑是對的。

### 3.3 上傳 submodule 到專案中
```bash
cd public
git status ## 正確的話，這邊應該要出現以 public 為 root 的修改內容，如果還是看到前一層的，那就是有地方錯了
git commit -m "first commit for whole site"
git remote add origin git@github.com:<your-account>/<your-account>.github.io.git
git push -u origin master
```

### 3.4 看成果
等待一會兒之後可以連到 `https://<your-account>.github.io` 看看成果了。

可參考官網：https://gohugo.io/hosting-and-deployment/hosting-on-github/#host-github-user-or-organization-pages