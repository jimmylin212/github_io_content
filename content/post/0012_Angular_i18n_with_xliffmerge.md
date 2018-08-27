---
title: "使用 xliffmerge 讓 Angular i18n 更強大"
date: 2018-08-27T10:40:47+08:00
draft: false
author: "Jimmy Lin"
isCJKLanguage: true
tags: ["Frontend", "Angular"]
categories: ["Web App Development", "Software Development", "Front-end Development"]
---

最近這幾個月使用 Angular + Firebase 開發 side-project，想要利用 Angular 中的 i18n 的功能來完成多國語言支援。雖然不一定用的到，不過可以把語言的相關資料都存放在同一個地方也比較方便管理。Angular 中提供非常方便的 i18n 的方法，[官方的教學](https://angular.io/guide/i18n) 也寫得非常清楚，按照教學做，就可以成功地讓網站支援多國語言。不過我使用的過程當中發現兩個問題

1. 執行 `angular xi18n`，是重新產生一份新的 xlf 檔案，並不會把舊有的合併進來，所以之前的翻譯都會被刪除。
2. 無法只翻譯部分的句子。

網路上找到 [ngx-i18nsupport](https://www.npmjs.com/package/ngx-i18nsupport?activeTab=readme) 可以解決上述兩個問題，看看要怎麼使用吧。

{{< figure src="/images/cover/0012.png" width="" height="" >}}

## 專案架構
一般來說，專案會有預設語言，假設是英文，Angular i18n 會幫我們產生中文的按鈕

{{< figure src="/images/0012/btn_en_to-zh.png" width="" height="" >}}

這樣設計的好處是在開發的時候就可以看到英文版的介面了，不過當專案越來越大，共用的單字越來越多，反而變得不好修改。想像我們要一次改掉所有的 Edit 變成 Editable ，就要考慮會不會有漏改的字，反而不好維護，因此我們專案的架構改成利用一組 raw string，再利用 Angular i18n 幫我轉換成其他語言，以便日後維護。

{{< figure src="/images/0012/btn_raw_to_en_zh.png" width="" height="" >}}

Angular 當中提供了一個內建的 HTML Attribute `i18n` 來指名這個 HTML TAG 的字需要被翻譯，另外提供很多種方法讓翻譯人員更方便

- 在 i18n attribute 寫下描述

```html
<h1 i18n="An introduction header for this sample">Hello i18n!</h1>
```

- 在 i18n attribute 寫下描述 + 意思

```html
<h1 i18n="site header|An introduction header for this sample">Hello i18n!</h1>
```

- 在 i18n attribute 寫下特定的 ID

```html
<h1 i18n="@@introductionHeader">Hello i18n!</h1>
```

- 在 i18n attribute 寫下 ID + 描述

```html
<h1 i18n="An introduction header for this sample@@introductionHeader">Hello i18n!</h1>
```

- 在 i18n attribute 寫下 ID + 描述 +意思

```html
<h1 i18n="site header|An introduction header for this sample@@introductionHeader">Hello i18n!</h1>
```

我們採取第三種作法，因為翻譯人員就是我們自己，所以很清楚知道每一個 raw string 代表的意思，另外我們會把 ID 和 raw string 設成一樣，比如上面編輯的按鈕如下

```html
<button i18n="@@EDIT_BTN">EDIT_BTN</button>
```

## 產生 msssage.xlf 檔
當我們為每一個要翻譯的字串都加上 i18n 的 attribute 之後，執行下面指令會在 `src/locale` 資料夾產生 `messages.xlf` 檔案

```shell
ng xi18n --output-path locale
```

打開 `message.xlf` 看一下，可以看到類似下面的內容

```xml
<trans-unit id="EDIT_BTN" datatype="html">
    <source>EDIT_BTN</source>
    <context-group purpose="location">
        <context context-type="sourcefile">PATH_TO_COMPONENT_FILE</context>
        <context context-type="linenumber">10</context>
    </context-group>
</trans-unit>
```

這份檔案寫明了要翻譯那些字串，這些字串在哪個檔案的哪一行。

當然我們也可以把剛剛的指令加進 `package.json` 中

```json
{
    ...
    "scripts": {
        ...
        "extract-i18n": "ng xi18n --output-path locale"
        ...
    }
    ...
}
```
如此一來，下面指令就可以產生一份新的 message.xlf 檔案，並且放在 `src/locale` 資料夾中了

```batch
npm run extract-i18n
```
***
## 使用 xliffmerge
到目前為止，我們的主角 xliffmerge 都還沒有出現，剛剛都是 Angular CLI 幫我們產生的檔案，要使用 xliffmerge，利用下列指令安裝

```node
npm install --save-dev ngx-i18nsupport
```

先在 root 資料夾產生一個檔案 `xliffmerge.json` 內容如下:
```json
{
    "xliffmergeOptions": {
      "srcDir": "src/locale",
      "genDir": "src/locale"
    }
}
```

另外把剛剛 `extract-i18n` 的指令改成
```json
{
    ...
    "scripts": {
        ...
        "extract-i18n": "ng xi18n --output-path locale && xliffmerge --profile xliffmerge.json raw zh"
        ...
    }
    ...
}
```

執行
```barch
npm run extract-i18n
```

這時候應該會看到警告訊息
```
WARNING: please translate file "src/locale/messages.zh.xlf" to target-language="raw
```

再看一次 locale 發現多了兩個檔案

- `message.raw.xlf`: 存放 raw string 的檔案
- `message.zh.xlf`: 這就是我們準備要放中文翻譯的地方檔案了

簡單比較一下這兩個檔案

```xml
<!-- message.raw.xlf -->
<trans-unit id="EDIT_MEMBER_DATA" datatype="html">
  <source>EDIT_BTN</source><target state="final">EDIT_BTN</target>
  <context-group purpose="location">
    <context context-type="sourcefile">PATH_TO_COMPONENT_FILE</context>
    <context context-type="linenumber">10</context>
  </context-group>
</trans-unit>
```

```xml
<!-- message.zh.xlf -->
<trans-unit id="EDIT_MEMBER_DATA" datatype="html">
  <source>EDIT_BTN</source><target state="new">EDIT_BTN</target>
  <context-group purpose="location">
    <context context-type="sourcefile">PATH_TO_COMPONENT_FILE</context>
    <context context-type="linenumber">10</context>
  </context-group>
</trans-unit>
```

兩者只有一個些微的差異 HTML tag `<target>` state 的值不一樣，在 `message.raw.xlf` 是 `state="final"` 而在 `message.zh.xlf` 是 `state="new"`
***
## 開始翻譯
翻譯的工作很簡單，打開 `message.zh.xlf` 修改 `<target>` 裡面的值，翻譯完成之後修改 state 成 `"tranlsated"` 即可

```xml
<!-- message.zh.xlf 翻譯完成 -->
<trans-unit id="EDIT_MEMBER_DATA" datatype="html">
  <source>EDIT_BTN</source><target state="translated">編輯</target>
  <context-group purpose="location">
    <context context-type="sourcefile">PATH_TO_COMPONENT_FILE</context>
    <context context-type="linenumber">10</context>
  </context-group>
</trans-unit>
```

## 執行翻譯過後的 App
在 `package.json` 加上另外一個指令
```json
{
    ...
    "scripts": {
        ...
        "extract-i18n": "ng xi18n --output-path locale && xliffmerge --profile xliffmerge.json raw zh",
        "start-zh": "ng serve --configuration=zh-Hant"
        ...
    }
    ...
}
```

另外修改 `angular.json` 
```json
{
    ...
    "build": {
        "configurations": {
            ...
            "zh-Hant": {
                "aot": true,
                "outputPath": "dist/PROJECT_NAME-zh",
                "i18nFile": "src/locale/messages.zh.xlf",
                "i18nFormat": "xlf",
                "i18nLocale": "zh-Hant"
            }
        }
    },
    "serve": {
        ...
        "configurations": {
            ...
            "en": {
              "browserTarget": "PROJECT_NAME:build:zh-Hant"
            }
        }
    }
    ...
}
```

執行
```batch
npm run start-zh
```

就可以看到翻譯過後的 App 了。

## 新增一筆翻譯
要新增一筆翻譯非常簡單，只要執行

```batch
npm run extract-i18n
```

就會看到有新的 raw string 被加進 `message.xlf`、`message.raw.xlf` 以及 `message.zh.xlf` 中，照著上面的方法翻譯，再執行程式就可以看到翻譯已經被套用上去了。