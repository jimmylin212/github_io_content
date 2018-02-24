---
title: "Hugo 其他使用方式以及 Cheat Sheet"
date: 2018-01-22T23:18:37+08:00
draft: false
isCJKLanguage: true
tags: ["Hugo"]
categories: ["Web App Development"]
author: "Jimmy Lin"
---
Hugo 上有很多方便的功能，有些蠻簡單易用的，就直接都整理在這邊。最後面附上常用到的指令，方便未來查詢。

1. 文章標頭設定
2. 插入標籤
3. 文章標頭樣板
4. 使用內建樣板
5. Cheat Sheet

***
# 文章標頭設定
在我們建立完文章之後，可以看到有一個最基本的文章標頭，Hugo 裡面支援三種格式：`YAML`、`JSON` 和 `TOML`，格式有些不同。

### YAML 
使用 `----` 當作開頭以及結尾，範例如下：
```
---
title: "spf13-vim 3.0 release and new website"
description: "spf13-vim is a cross platform distribution of vim plugins and resources for Vim."
tags: [ ".vimrc", "plugins", "spf13-vim", "vim" ]
lastmod: 2015-12-23
date: "2012-04-06"
categories:
  - "Development"
  - "VIM"
---
```

### JSON
使用 `{` 作開頭，`}`作為結尾，範例如下：
```json
{
    "title": "spf13-vim 3.0 release and new website",
    "description": "spf13-vim is a cross platform distribution of vim plugins and resources for Vim.",
    "tags": [ ".vimrc", "plugins", "spf13-vim", "vim" ],
    "date": "2012-04-06",
    "categories": [
        "Development",
        "VIM"
    ]
}
```

### TOML
使用 `++++` 當作開頭以及結尾，範例如下：
```
+++
title = "spf13-vim 3.0 release and new website"
description = "spf13-vim is a cross platform distribution of vim plugins and resources for Vim."
tags = [ ".vimrc", "plugins", "spf13-vim", "vim" ]
date = "2012-04-06"
categories = ["Development","VIM"]
+++
```

標頭裡面有些 attribute 可以使用，介紹一些常用的 attribute 如下：
### date
這篇文章被創造出來的時間，這個時間是自動建立的。

### description
簡單的描述這篇文章。

### tags
可參考 Taxonomies 部分，如何在 Hugo 中使用 tags

### categories
可參考 Taxonomies 部分，如何在 Hugo 中使用 tags

### draft
這篇文章是否為稿草稿，若為草稿，則執行 `hugo` 指令時並不會產生這篇文章，除非用了 `--buildDrafts` 參數

### isCJKLanguage
這篇文章當中是否有中文、日文或是韓文，如果有的話在這邊標明，會讓 Summary 以及 WordCount 較正確。

### weight
這篇文章的權重，權重會影響我們開啟一個文章列表時文章的的排序。

更詳細的文章標頭設定，請看官方文件：[Front Matter]
***
# 插入標籤
標籤在部落格當中也是一個非常重要的功能，當然 Hugo 也提供這個功能，用法也相當簡單，直接在文章標頭加上 `tags` 或 `categories` 即可簡單的設定文章的類別以及標籤。但是這邊可以多介紹一下 Hugo 的標籤系統。

想要使用 Hugo 的標籤，需要在 `config` 中加入 `[taxonomies]` 的設定，根據不同的 config 格式，設定如下：

### YAML
```
taxonomies:
  tag: "tags"
  category: "categories"
```
### TOML
```
[taxonomies]
  tag = "tags"
  category = "categories"
```

在 `config` 中設定完之後，就可以在每一篇文章的文章標頭設定這篇文章的 `tags` 和 `categories` 了，設定方法也因不同的檔案格式有所不同：
### YAML
```
+++
title = "Hugo: A fast and flexible static site generator"
tags = [ "Hugo" ]
categories = [ "Development" ]
+++
```
### TOML
```
---
title: "Hugo: A fast and flexible static site generator"
tags: ["Hugo" ]
categories: ["Development"]
---
```
### JSON
```
{
    "title": "Hugo: A fast and flexible static site generator",
    "tags": [
        "Hugo"
    ],
    "categories" : [
        "Development"
    ]
}
```
當我們用以上的方法建立文章的 `tags` 或是 `categories` 之後，Hugo 會聰明的自動幫我們產生相對應的文章列表。就拿上面的例子來說，文章的 categories 是 Development，Hugo 就會產生兩個頁面

1. `your_website_domain/categories/` 在這個頁面當中列出了所有的 categories 的文章
2. `your_website_domain/categories/developments/` 在這個頁面當中列出了所有 categories 是 development 的文章

另外因為 tag 是 Hugo，所以也會自動產生兩個 tags 相關的頁面，分別是

1. `your_website_domain/tags/` 在這個頁面當中列出了所有的 tag 的文章
2. `your_website_domain/tags/hugo/` 在這個頁面當中列出了所有 tags 是 hugo 的文章 

如果發現沒有上述的連結，有可能是因為 theme 並沒有支援，如果需要這些功能，可以換一個有支援的 theme。

更詳細的標籤資訊請看官方文件：[Taxonomies]
***
# 文章標頭樣板
從文章標頭裡面我們可以設定很多東西，不過如果每一次發文章都需要編輯一次類似的東西，比如說每次都要加上 tag 以及 categories，並且把 draft 改成 true，如果每次都要手動做，一定會有旺季時候，這時我們就可用這邊要介紹的 **文章標頭樣板** 來解決了。

先打開資料夾 `archetypes/default.md`，應該可以看到類似下面內容：
```
---
title: "{{ replace .TranslationBaseName "-" " " | title }}"
date: {{ .Date }}
draft: true
---
```
這個意思是說，當我們產生一篇新的文章，這篇新的文章會自動產生這些文章標頭，title, date 和 draft。如果我們希望在每一篇文章都加上作者的資訊，可以修改這個 `default.md` 檔案變成：
```
---
title: "{{ replace .TranslationBaseName "-" " " | title }}"
date: {{ .Date }}
draft: true
author: "Jimmy Lin"
---
```
如此一來，所有利用 `hugo new post/the_title_of_this_post.md` 所產生的文章都會在文章標頭被加上 `author: "Jimmy Lin"`。

假設我們在 content 資料夾當中有多個不同的子資料夾，又希望為了每一個子資料夾設定不同的文章標頭樣板，可以嗎？當然可以。在資料夾 `archetypes` 下多產生一個 `movies.md` 檔案，編輯文章內容如下：
```
---
title: "{{ replace .TranslationBaseName "-" " " | title }}"
date: {{ .Date }}
draft: true
tags: []
categories: ["movie"]
author: "Movie Watcher"
---
```

接著利用 `hugo new movies/the_title_of_this_post.md`，就會看到這篇在 movies 子資料夾下面的文章的文章標頭上多了 tags, categories 和 author = "Movie Watcher" 的設定。Hugo 的邏輯是：當產生一篇新文章時，會去檢查有沒有同樣子資料夾名稱的 archetypes 設定，如果有，就套用這個設定，如果沒有，就使用 default.md。我現在使用的 archetypes 如下：
```
---
title: "{{ replace .TranslationBaseName "-" " " | title }}"
date: {{ .Date }}
draft: true
tags: []
categories: []
author: "Jimmy Lin"
isCJKLanguage: true
---
```
可以避免我新增文章忘記加上 tags 和 categories。

更詳細的內容請看官方文件：[Archetypes]
***
# 使用內建樣板
當初選擇使用 Hugo 就是看上他是使用 Markdown 編輯，方便又快速，不過有的時候 Markdown 並不是非常方便，像是插入圖片，又要設定這個圖片的長寬，再加上標題，這件事情在 Markdown 裡面做不到；或是想要新增一段 Gist，用 Markdown 也做不到。因此 Hugo 很貼心的設計了這個 **Shortcodes** 功能，方便我們插入一些特定的內容。

目前 Hugo 支援下面這幾種內容：圖片、Gist、Instagram、Youtube、Speakerdeck (線上投影片)、Tweet、Vimeo、Ref/Relref (關聯其他網頁)，我覺得已經涵蓋了大部分的 use case。

要使用 Shortcodes，插入以下格式 `{{\% shortcodename parameters \%}}` 或 `{{\< shortcodename parameters \>}}`(記得把反斜拿掉)。shortcodename 填上 Hugo 定義好的保留字，比如圖片就是 figure，Gist 就是 gist；parameters 填上可以使用的參數，分隔符號請用空白，若要指定值得話，用雙引號包住，直接看下面例子。

### 插入圖片

插入圖片並指定大小，加上 `width` 和 `height`，如下：

`{{\< figure src="/images/0004/IMG_01.jpg" width="" height="" \>}}`

{{< figure src="/images/0004/IMG_01.jpg" width="50" height="50" >}} 
{{< figure src="/images/0004/IMG_01.jpg" width="250" height="250" >}}

### 插入關聯網頁

這個功能會去找尋檔名一樣的 `.md` 檔，並且回傳他們的 `ref` 或 `relref`，範例如下：

`[使用 CircleCI 部署 Hugo 靜態網站]({{\< ref "post/0003_Deploy_Hugo_Using_CircleCI.md" \>}})`

[使用 CircleCI 部署 Hugo 靜態網站]({{< ref "post/0003_Deploy_Hugo_Using_CircleCI.md" >}})

這樣就可以達到文章內相互參照的功能了。Hugo Shortcodes 功能確實非常方便，不用再自己寫一堆程式碼，也對開發 theme 的開發者來說省了很多事。

更詳細的 Short Codes 請看官方文件：[Shortcodes]
***
# Cheat Sheet
最後整理一下我常用的指令表，以免以後找半天，會持續編輯這部分。
```
# 建立新文章
hugo new draft/POST_NAME.md

# 開啟一個 Hugo service，一邊編輯一邊看結果
hugo server -D
```


[Front Matter]: https://gohugo.io/content-management/front-matter/
[Taxonomies]: https://gohugo.io/content-management/taxonomies/
[Archetypes]: https://gohugo.io/content-management/archetypes/
[Shortcodes]: https://gohugo.io/content-management/shortcodes/