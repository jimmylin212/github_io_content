---
title: "利用 Wix 建立 Windows .msi 安裝檔"
date: 2018-03-14T22:33:39+08:00
draft: true
author: "Jimmy Lin"
isCJKLanguage: true
tags: ["Build"]
categories: [Software Development]
---
最近兩週花了蠻多時間研究該如何使用 Wix (Windows Installer XML) 來打包 source file 成一個 `.msi` 檔，因為沒有打包成 `.exe` 的經驗，無法比較兩者的差異。中文資料真的少之又少，有很多又很舊了，雖然我覺得以後自己也不太會用到，但還是寫一下希望可以幫助到需要的人。這篇文章是以 [Wix 上的教學] 為基礎，再加上許多自一些的經驗，雖然只是略懂皮毛，不過至少可以成功建立 `.msi` 檔。

### 簡單指令介紹
Wix 是一套利用 XML 來編寫一些設定，並且利用內建的一些工具來建立完整的 `.msi`。我們寫的 XML 需要用副檔名 `.wxs`，並且使用內建的工具 `candle.exe` 來產生一個編譯過後的檔案，假設我們已經有一個 `example.wxs`，使用下面指令可以產生編譯過後的檔案。

```bash
candle.exe example.wxs
```
如果沒有特別指名輸出的檔名，檔名會以 xml 的檔名命名，副檔名為 `.wixobj`，之後在使用一次內建的工具 `light.exe` 產生 `.msi`，可以使用下面指令。

```bash
light.exe example.wixobj
```

更詳細內容可以參考： [Getting Start]
***
### The Software Package
Windows Installer 利用 GUID 來追蹤我們的程式，GUID 是一組保證為一個十六進位的字串，原理可以看一下 [Wiki GUID 介紹]，因為每一台電腦每一次所產生的 GUID 都不一樣，所以下面的所有程式碼，如果有遇到需要寫 GUID，我都會用 **YOUR_GUID**，請自己換成另外一組你自己的 GUID。使用下面指令可以在 powershell 中產生，根據不同的 powershell 版本，產生方式可能有所不同。

```bash
## Win 9x
[guid]::NewGuid().GUID.ToUpper()

## Win 2008 or higher
New-Guid.GUID.ToUpper()
```

產生完之後就來建立我們的 xml 吧，先假設我們的檔名叫做 `example.wxs`，下面為第一階段的檔案內容，可以看到有兩個 `YOUR_GUID`，用上面的指令產生兩組，並且填進去即可。

```xml
<?xml version='1.0' encoding='windows-1252'?>
<Wix xmlns='http://schemas.microsoft.com/wix/2006/wi'>
    <Product Name='Foobar 1.0' 
             Manufacturer='Acme Ltd.' 
             Id='YOUR_GUID' 
             UpgradeCode='YOUR_GUID' 
             Language='1033' 
             Codepage='1252' 
             Version='1.0.0'>

        <Package Id='*' 
                 Keywords='Installer' 
                 Description="Acme's Foobar 1.0 Installer"
                 Comments='Foobar is a registered trademark of Acme Ltd.' 
                 Manufacturer='Acme Ltd.'
                 InstallerVersion='100' 
                 Languages='1033' 
                 Compressed='yes' 
                 SummaryCodepage='1252' />
    </Product>
</Wix>
```
我覺得這段蠻直覺的，就是填上一些 Product 以及 Package 的資訊，並且把語言都設定。這邊比較要注意的是 **Package 裡面的 Compressed 一定要設為 yes**，如果沒有這個 Attribute，最後產生的 `.msi` 不會把 source files 都包進去，就只有殼，這種情況下，如果把 msi 檔案換到其他資料夾，就無法安裝了。至於 Product 和 Package 當中有哪些特徵可以用，可以參考 [Product Elements] 以及 [Packge Elements]。

更詳細的內容可以參考：[The Software Package]
***
### 把檔案放進去
這邊就是這個工具的重點了，我們要把 source file 放進 xml，讓 Wix 看得懂。

### 我的 Gist
因為擔心以後還會用到，我自己做了一個 powershell script，用來產生最基本可用的 `.wxs`，包含上面說的功能。如果要使用的話，請自己修改每一個 Function 中定義的部分，就可以根據這個 script 產生對應的 `.wxs` 了，script 在[這邊](https://gist.github.com/jimmylin212/83c71fffd4820b69db0e3c3959ecd3ae) 或是直接看內容：

{{< gist jimmylin212 83c71fffd4820b69db0e3c3959ecd3ae >}} 


[Wix 上的教學]: https://www.firegiant.com/wix/tutorial/
[Wix How-To]: http://wixtoolset.org/documentation/manual/v3/howtos/
[Getting Start]: https://www.firegiant.com/wix/tutorial/getting-started/
[Wiki GUID 介紹]: https://en.wikipedia.org/wiki/Universally_unique_identifier
[Product Elements]: http://wixtoolset.org/documentation/manual/v3/xsd/wix/product.html
[Packge Elements]: http://wixtoolset.org/documentation/manual/v3/xsd/wix/package.html
[The Software Package]: https://www.firegiant.com/wix/tutorial/getting-started/the-software-package/