---
title: "Hugo 的文件管理 - 資料夾結構"
date: 2017-12-05T22:00:48+08:00
draft: false
tags: ["Hugo"]
categories: ["Web App Development"]
author: "Jimmy Lin"
isCJKLanguage: true
---

建立完個人網站之後，最重要的就是開始寫文章，或是讓充實網頁的內容，像是加上個人介紹頁面，或是為自己的網站建立一些基本部落格應該要有的功能，像是標籤，這些功能其實 Hugo 都已經有提供了，不過到底要怎麼用呢？那就要仔細閱讀一下 [Hugo Content Management][Hugo Content Management]，這邊我就針對一些特定的功能紀錄一下，以後也方便參照。

### 資料夾結構
每一個不同的 theme，會有不同的資料夾結構，不過大同小異，有些只會抓在 `content/post` 底的文章，有些只會抓 `content/posts` 下的文章，所以如果文章沒有出現，那要先去看一下 theme 的說明文件。

在 Hugo 當中，把頁面分成兩種: **List Page** 以及 **Single Page**，另外有一個特殊的頁面 `_index.md`。簡介如下：

- List Page: 列出所有在這個資料夾底下的所有 `*.md` 檔的頁面。
- Single Page: 某一個特定的 `*.md`的頁面。
- _index.md: 可用來控制以及增加一些特定的內容於 List Page 之頂端的頁面。

直接看官網所提供的例子
```
.
└── content
    └── about
    |   └── _index.md  // <- https://example.com/about
    ├── post  // <- https://example.com/post
    |   ├── _index.md 
    |   ├── a.md   // <- https://example.com/post/a
    |   ├── happy  // <- https://example.com/post/happy
    |   |   └── b.md  // <- https://example.com/post/happy/b
    |   └── c.md  // <- https://example.com/post/c
    └── quote     // <- https://example.com/quote
        └── d.md  // <- https://example.com/quote/d
```
按照上面的資料夾結構，跑`hugo server -D`，連到 `http://localhost:1313`，應該會看到四篇文章，分別是 A, B, C, D。因為 `hugo` 會把 `content` 資料夾下面的除了 `_index.md` 以外的所有 `*.md` 都列出來，這也就是我們上面所提到的 **List Page**。再來我們嘗試連到 `http://localhost:1313/post/` ，這時候應該只會看到三篇文章: A, B, C。同樣如果我們連到 `http://localhost:1313/quote/` 只會看到一篇文章: D。因為 **List Page** 會列出在這個資料夾下面除了 `_index.md` 以外所有的 `*.md` 檔。一起來看下面的截圖:

連線至 `http://localhost:1313`

![](/images/0002/home_directory.PNG)

連線至 `http://localhost:1313/post/`
![](/images/0002/post_directory.PNG)

連線至 `http://localhost:1313/quote/`
![](/images/0002/quote_directory.PNG)

值得注意的是 Hugo 建立 List Page 只會會抓第一層的資料夾，第二層不會抓，所以連到 `http://localhost:1313/post/happy` 是看不到東西的，這時候就要用手動的方式讓 Hugo 知道要來抓其他層的資料夾，怎麼做？我們可以用這個特殊的檔案 `_index.md` 來辦到。

連線至 `http://localhost:1313/post/happy`
![](/images/0002/post_happy_directory.PNG)

新增一篇 `_index.md` 再試試看，執行指令 `hugo new post/happy/_index.md`
再次連線到 `http://localhost:1313/post/happy`
![](/images/0002/post_happy_success.PNG)


這個特別的檔案功能不只如此，我們可以利用這個檔案來設定針對個別的資料夾下的文章設定專屬的 template 或是 標籤，也可以加上一些專屬的文字於所有文章或是頁面最上面，像下面的例子。

先在所有的資料夾下面都建立 `_index.md`:

`hugo new post/_index.md`

`hugo new quote/_index.md`

記得要修改 `_index.md` 裡面的 title，發現如果 title 都一樣，會無法正確執行。

編輯 `post/_index.md` 加入下面文字
```
---
title: "post_Index"
date: 2017-12-08T14:45:37+08:00
draft: true
---

下面這些文章都在 post 裡面
```
編輯 `quote/_index.md` 加入下面文字
```
---
title: "quote_index"
date: 2017-12-08T14:42:41+08:00
draft: true
---

下面文章都在 quote 裡面
```
編輯 `post/happy/_index.md` 加入下面文字
```
---
title: "post/happy_Index"
date: 2017-12-08T14:35:58+08:00
draft: true
---

下面這些文章都在 post/happy 裡面
```
再次執行 `hugo server -D`，看一下結果
連線至 `http://localhost:1313/post/`
![](/images/0002/post_index.PNG)

連線至 `http://localhost:1313/post/happy/`
![](/images/0002/post_happy_index.PNG)

連線至 `http://localhost:1313/quote/`
![](/images/0002/quote_index.PNG)

可以看到每一個 List Page 都被加上剛剛所編輯的文字，可以讓讀者更清楚現在的所在的位置，我相信 `_index.md` 應該有更多的用法，等我之後發現再來補。

原始連結: [Hugo 資料夾結構原文介紹][Hugo CM organization]

[Hugo Content Management]: https://gohugo.io/content-management/
[Hugo CM organization]: https://gohugo.io/content-management/organization/