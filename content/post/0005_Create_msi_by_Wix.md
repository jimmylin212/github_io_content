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

## 簡單指令介紹
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
## The Software Package
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
    <Product Name='PRODUCT_NAME' 
             Manufacturer='COMPANY_NAME' 
             Id='YOUR_GUID' 
             UpgradeCode='YOUR_GUID' 
             Language='1033' 
             Codepage='1252' 
             Version='1.0.0'>

        <Package Id='*' 
                 Keywords='Installer'  
                 Manufacturer='COMPANY_NAME'
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
## 把檔案放進去
這邊就是這個工具的重點了，我們要把 source file 放進 xml，讓 Wix 看得懂。直接看整份的 xml，再一塊一塊看裡面的內容：

### Directory
這個地方就是放要把 source file 安裝到電腦的哪邊，比如說安裝到 Program Files 還是安裝到 Program Files (x86)；如果想要新增一個捷徑在開始選單，那要安裝到哪邊；如果想要新增一個捷徑到桌面，那要安裝到那邊，全部都是在這一個 **Directory** 當中設定。上面的例子是安裝到 Program Files 以及新增一個捷徑到開始選單。

```xml
<Directory Id="TARGETDIR" Name="SourceDir">
    <Directory Id="ProgramFiles64Folder" Name="PFiles">
        <Directory Id="ProgramFilesFolderCompany" Name="COMPANY_NAME">
            <Directory Id="INSTALLDIR" Name="PRODUCT_NAME" />
        </Directory>
    </Directory>
    <Directory Id="ProgramMenuFolder">
        <Directory Id="ProgramMenuDirCompany" Name="COMPANY_NAME">
            <Directory Id="ProgramMenuDirProduct" Name="PRODUCT_NAME" />
        </Directory>
    </Directory>
</Directory>
```

可以看到這邊我們不用特別寫清楚到底 Program Files 的絕對路徑在哪裡，只要在 `Id` 寫上 `ProgramFilesFolder`，Wix 就會自動幫我們轉換成相對應的資料夾。像我們常會用到的保留字包括：`ProgramFilesFolder`、`ProgramMenuFolder` 和 `DesktopFolder`，[這邊](https://msdn.microsoft.com/en-us/library/aa372057.aspx) 可以查詢到其他的保留字。而名稱(Name) 就是你要資料夾顯示的名稱，蠻直覺的。這邊要特別注意的是在每一個 xml Element 中，都會有 Id 這個 Attribute，Wix 利用這個 Id 來辨認不同的 Element，所以 Id 一定要是唯一的。在我們的例子當中最後安裝的資料夾叫做 `INSTALLDIR`，之後也會在其他地方看到他。可以看到在開始選單的部分，大架構都一樣，就只有差在第一層的 Id 改成了 `ProgramMenuFolder`。

更詳細的內容可以參考：[Directory Element]

### DirectoryRef
在官網的教學上，並沒有特別提到這一個 Element，但是我東查西查之後發現利用 DirectoryRef 把內容跟資料夾分開，比較清楚，也更有益於了解整個架構。先看簡單的例子：

```xml
<DirectoryRef Id="INSTALLDIR">
    <Component Id="myapplication.exe" Guid="PUT-GUID-HERE">
        <File Id="myapplication.exe" 
              Source="MySourceFiles\MyApplication.exe" 
              KeyPath="yes" 
              Checksum="yes"/>
    </Component>
    <Component Id="documentation.html" Guid="PUT-GUID-HERE">
        <File Id="documentation.html" 
              Source="MySourceFiles\documentation.html" 
              KeyPath="yes"/>
    </Component>
</DirectoryRef>
```
看到在 DirectoryRef 中的 Id 的值指向了剛剛在 Directory 中我們最後希望安裝進去的資料夾的 `INSTALLDIR`，這就代表在這個 DirectoryRef 下面的所有的檔案，都要被安裝進 INSTALLDIR 中。接下來看到 Component，Component 當中可以包含多個檔案，可以想像在同一個 Component 中的所有檔案都是一起的，一個壞掉這個 Component 就壞掉了，雖然這樣說，但是官方強烈建議我們一個檔案一個 Component。Component 的 Id 必須是唯一的 (之後會用到)，另外還要多產生一組的 GUID 給每一個 Component。這時候就出現了兩個問題：

1. 要是我有上千個檔案甚至上萬個檔案，要自己產生這些 Component 和 File 嗎？
2. 如果我的資料夾結構很複雜，很多層，那不就要自己產生一堆 Directory 以及一堆的 DirectoryRef 嗎？

理論上是這樣沒錯，所以官方建議我們在開發的時候不應該把 Build msi 這件事當成開發完畢之後才做的事情，而應該把 Build msi 視為開發中的一部分，在 Build 專案的過程當中，也要考慮到 Build msi 這件事情，一併處理。

*這不是風涼話嗎？*

是！不過好在 Wix 還是提供了一個 [方法](https://www.firegiant.com/wix/tutorial/com-expression-syntax-miscellanea/components-of-a-different-color/) 可以自動幫我們產生資料夾的結構。這邊要使用另外一個內建的工具 `heat.exe`。使用指令如下：

```bash
heat dir SOURCE_FILE_DIRECTORY -t HeatTransform.xslt -cg HarvestedComponentId -out heat_harvested_results.wxs -dr INSTALLDIR -var var.Dist -scom -frag -srd -sreg -gg
```

這邊需要把 SOURCE_FILE_DIRECTORY 改成你放那些 source file 的資料夾

- -cg 的意思是要把 ComponentGroup 叫什麼名字 (在 example.wxs 中會用到)，這邊我們先命名為 **HarvestedComponentId**
- -o 指定輸出檔案名稱，假設為 `heat_harvested_results.wxs`
- -dr 可以指定資料夾結構從哪邊開始參照，因為有可能檔案階層多層，heat 也會自動幫我們產生 DirectoryRef
- -var 可以在之後 Build 的時候用較方便的方式把檔案的路徑都指定到正確的路徑
- -t 使用 template 來修改產生之後的 wxs。因為 heat 內建的設定有限，利用 template 更方便，比如說要為所有的檔案增加 `win64='yes'` 的 attribute。 

使用了 heat 這個工具，可以讓 DirectoryRef 更加簡化，看一下我們的真實的 DirectoryRef 吧

```xml
<DirectoryRef Id="INSTALLDIR">
    <Component Id="componentINSTALLDIR" Guid="SOME_GUID" Win64="yes">
        <RemoveFolder Directory="INSTALLDIR" 
                      Id="componentINSTALLDIR" 
                      On="uninstall" />
    </Component>
</DirectoryRef>
<DirectoryRef Id="ProgramMenuDirHpProduct">
    <Component Id="componentProgramMenuDirHpProduct" 
               Guid="SOME_GUID" 
               Win64="yes">
        <Shortcut Name="PRODUCT_NAME" 
                  Id="PRODUCTSHORTCUT" 
                  Target="[INSTALLDIR]PRODUCT_NAME.exe" 
                  WorkingDirectory="INSTALLDIR" />
        <RemoveFolder Directory="ProgramMenuDirCompany" 
                      Id="removeProgramMenuDirCompany" 
                      On="uninstall" />
        <RemoveFolder Directory="ProgramMenuDirProduct" 
                      Id="removeProgramMenuDirProduct" 
                      On="uninstall" />
        <RegistryValue Value="1" 
                       Name="installed" 
                       Root="HKCU" 
                       KeyPath="yes" 
                       Type="integer" 
                       Key="Software\COMPANY_NAME\PRODUCT_NAME" />
    </Component>
</DirectoryRef>
```
這邊我們定義了兩個 DirectoryRef。第一個我們給了 `Id="INSTALLDIR"`，讓 Wix 知道這個 DirectoryRef 對應到 Directory 中 Id 為 `INSTALLDIR` 的那一個資料夾；在這個 DirectoryRef 中，只有一個 Component，代表我們要包含哪些東西，在這個 Component 中只有一個 RemoveFolder 的 Element，意思說如果今天要移除這個軟體，我們也要同時移除這個資料夾。再來看第二個比較複雜的 DirectoryRef，這個 DirectoryRef 指向 Id 為 `ProgramMenuDirHpProduct` 的資料夾，就是開始功能捷徑的資料夾，一樣是一個 Component，不一樣的是在這個 Component 下面有四個 Element，分別是 Shortcut、RemoveFolder 以及 RegistryValue。都蠻直覺的，Shortcut 代表這個捷徑的目標執行檔是哪一個，在哪一個資料夾中執行：第二個 Remove 之前說過了就不多作介紹，最後一個 RegistryValue，表示在安裝的同時要新增一筆資料到 Registry 中，這樣就完成了我們的 DirectoryRef 了。前面有提到 Component 的 Id 必須要是唯一的，在下面的 Feature 當中會使用到這些 Id。

那我們的檔案呢？因為前面我們使用了 `heat` 來產生檔案相關的 wxs 了，在 DirectoryRef 裡面就不用重複寫上，只需要在下面的 Feature 加上即可。


更詳細的內容可以參考：[DirectoryRef Element], [Component Element], [File Element], [Heat Tool]

### Feature
再來介紹一下 Feature 這個 Element，我們前面定義了這麼多東西，倒底安裝的時候全部都要進去，還是我只要安裝某幾個 Component 就好了，這些設定都放在 Feature 裡面。所以看一下下面的 xml 也很直覺，只要簡單得把我們要安裝的 Component Id 寫下，並且放在 ComponentRef Element 中，在安裝的時候就會把這個 Component 一起安裝進去。這邊要特別注意就是 ComponentGroupRef Element，`Id="HarvestedComponentId"`，這邊的 Id 就是前面使用 `heat` 指令時 `-cg` 的 attribute，利用這個 Element，把 heat 產生落落長的檔案全部都包含進來，一來可以讓原本的 xml 乾淨，可讀性又高，二來方便管理，如果新增或刪除了某些檔案，不用一個一個編輯 xml，方便許多。

```xml
<Feature Id="FEATUREPRODUCTNAME" 
         AllowAdvertise="no" 
         Display="expand" 
         Level="1" 
         InstallDefault="local"
         ConfigurableDirectory="INSTALLDIR">
    <ComponentRef Id="componentINSTALLDIR" />
    <ComponentRef Id="componentProgramMenuDirHpProduct" />
    <ComponentGroupRef Id="HarvestedComponentId" />
</Feature>
```

到這邊其實就已經告一段落了，產生出來的 msi 已經可以成功安裝，可惜的沒有漂亮的介面，使用者的體驗不是很好。Wix 也提供了一些方法，可以讓我們方便又快速的新增安裝的介面，提升使用者體驗，繼續看下去吧。

更詳細的內容可以參考：[Feature Element]

### UI
我對 UI 比較沒有深入研究，在 Wix 的教學中看到，利用 Wix 可以製造出很漂亮的 UI，排版之類的也都可以很動態的設置，另外 Wix 還提供了[懶人介面設計](http://wixtoolset.org/documentation/manual/v3/wixui/wixui_dialog_library.html)，利用內建的幾個樣板，可以製造出看起來還不錯的介面，我用了 `WixUI_InstallDir` 的樣板，可以幫我打造出兩頁，一頁歡迎頁，一頁讓使用者可以選擇要安裝在哪個資料夾，如果想要看更詳細的介面設計，可以看[這邊](https://www.firegiant.com/wix/tutorial/user-interface-revisited/)

```xml
<Property Id="WIXUI_INSTALLDIR" Value="INSTALLDIR" />
<UIRef Id="WixUI_InstallDir" />
<UI>
    <Publish Control="Next" 
             Dialog="WelcomeDlg" 
             Value="InstallDirDlg" 
             Event="NewDialog">1</Publish>
    <Publish Control="Back" 
             Dialog="InstallDirDlg" 
             Value="WelcomeDlg" 
             Event="NewDialog" 
             Order="2">2</Publish>
</UI>
```

更詳細的內容可以參考：[UI Element], [Publish Element]

***
## 產生 msi
好了，說了這麼多，也做了一堆事情，是應該把所有東西結合再一起產生 .msi 了。使用下面的指令就可以四行產生 msi

```
## Using heat to generate the harvested file from SOURCE_FILE_DIRECTORY
heat dir SOURCE_FILE_DIRECTORY -t HeatTransform.xslt -cg HarvestedComponentId -out heat_harvested_results.wxs -dr INSTALLDIR -var var.Dist -scom -frag -srd -sreg -gg

## Generate the wixobj file of heat result
candle.exe heat_harvested_results.wxs -dDist="SOURCE_FILE_DIRECTORY"

## Generate the wixjob file of the xml that you create, change the example.wxs to your file name
candle.exe example.wxs

## Generate the msi by the 2 wixobj file above
light.exe heat_harvested_results.wixobj example.wixobj -o "MSI_OUTPUT_FILENAME" -ext WixUIExtension
```

***
## 自動產生 wxs 的 script
因為擔心以後還會用到，我自己做了一個 powershell script，用來產生最基本可用的 `.wxs`，包含上面說的功能，另外也包含了 heat 要用到的 template。如果要使用的話，請自己修改每一個 Function 中定義的部分，就可以根據這個 script 產生對應的 `.wxs` 了，script 在[這邊](https://gist.github.com/jimmylin212/83c71fffd4820b69db0e3c3959ecd3ae)。


[Wix 上的教學]: https://www.firegiant.com/wix/tutorial/
[Wix How-To]: http://wixtoolset.org/documentation/manual/v3/howtos/
[Getting Start]: https://www.firegiant.com/wix/tutorial/getting-started/
[Wiki GUID 介紹]: https://en.wikipedia.org/wiki/Universally_unique_identifier
[Product Elements]: http://wixtoolset.org/documentation/manual/v3/xsd/wix/product.html
[Packge Elements]: http://wixtoolset.org/documentation/manual/v3/xsd/wix/package.html
[The Software Package]: https://www.firegiant.com/wix/tutorial/getting-started/the-software-package/
[Directory Element]: http://wixtoolset.org/documentation/manual/v3/xsd/wix/directory.html
[DirectoryRef Element]: http://wixtoolset.org/documentation/manual/v3/xsd/wix/directoryref.html
[Component Element]: http://wixtoolset.org/documentation/manual/v3/xsd/wix/component.html
[File Element]: http://wixtoolset.org/documentation/manual/v3/xsd/wix/file.html
[Feature Element]: http://wixtoolset.org/documentation/manual/v3/xsd/wix/feature.html
[Heat Tool]: http://wixtoolset.org/documentation/manual/v3/overview/heat.html
[UI Element]: http://wixtoolset.org/documentation/manual/v3/xsd/wix/ui.html
[Publish Element]: http://wixtoolset.org/documentation/manual/v3/xsd/wix/publish.html