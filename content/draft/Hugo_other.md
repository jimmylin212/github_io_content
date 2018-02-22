---
title: "Hugo 其他使用方式以及 Cheat Sheet"
date: 2018-01-22T23:18:37+08:00
draft: true
isCJKLanguage: true
---
Hugo 上有很多方便的功能，有些蠻簡單易用的，就直接都整理在這邊。最後面附上常用到的指令，方便未來查詢。

1. 文章標頭設定
2. 插入標籤
3. 文章標頭樣板
4. 類似文章
5. 選單系統
6. 使用內建樣板
7. 使用靜態檔案

## 文章標頭設定
在我們建立完文章之後，可以看到有一個最基本的文章標頭，Hugo 裡面支援三種格式：`YAML`、`JSON` 和 `TOML`，格式有些不同。

#### YAML 
使用 `----` 當作開頭以及結尾，範例如下：
```yaml
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

#### JSON
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

#### TOML
使用 `++++` 當作開頭以及結尾，範例如下：
```toml
+++
title = "spf13-vim 3.0 release and new website"
description = "spf13-vim is a cross platform distribution of vim plugins and resources for Vim."
tags = [ ".vimrc", "plugins", "spf13-vim", "vim" ]
date = "2012-04-06"
categories = ["Development","VIM"]
+++
```

標頭裡面有些 attribute 可以使用，介紹一些常用的 attribute 如下：
#### date
這篇文章被創造出來的時間，這個時間是自動建立的。

#### description
簡單的描述這篇文章。

#### title
這篇文章的標題。

#### tags
可參考 Taxonomies 部分，如何在 Hugo 中使用 tags

#### categories
可參考 Taxonomies 部分，如何在 Hugo 中使用 tags

#### draft
這篇文章是否為稿草稿，若為草稿，則執行 `hugo` 指令時並不會產生這篇文章，除非用了 `--buildDrafts` 參數

#### isCJKLanguage
這篇文章當中是否有中文、日文或是韓文，如果有的話在這邊標明，會讓 Summary 以及 WordCount 較正確。

#### weight
這篇文章的權重，權重會影響我們開啟一個文章列表時文章的的排序。

更詳細的 attribute 請看官方文件：[Front Matter]

## 插入標籤
## 文章標頭樣板
## 類似文章
## 選單系統
## 使用內建樣板
## 使用靜態檔案

[Front Matter]: https://gohugo.io/content-management/front-matter/
[Taxonomies]: https://gohugo.io/content-management/taxonomies/
[Archetypes]: https://gohugo.io/content-management/archetypes/
[RelatedContent]: https://gohugo.io/content-management/related/
[Menus]: https://gohugo.io/content-management/menus/
[Shortcodes]: https://gohugo.io/content-management/shortcodes/
[StaticFile]: https://gohugo.io/content-management/static-files/