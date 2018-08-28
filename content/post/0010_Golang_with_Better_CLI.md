---
title: "為 Golang CLI 打造漂亮的 Help Message"
date: 2018-06-28T13:28:30+08:00
draft: false
author: "Jimmy Lin"
isCJKLanguage: true
tags: ["Golang", "Backend"]
categories: ["Back-end Development"]
image: "/images/cover/0010.png"
---

Golang 提供了簡單的方法產生 Help Message 以及支援使用者輸入，不過如果你的 CLI Tool 功能較多，或是有階層性的，那要怎麼辦？我們可以利用 [Cobra](https://github.com/spf13/cobra) 以及 [Viper](https://github.com/spf13/viper)，這兩個其實是完全不相關的 module，可以單獨使用，不過我覺得結合再一起真的是非常不錯。Cobra 是用來產生漂亮的 CLI 介面以及支援處理使用者的輸入；Viper 則是負責處理 config 檔。想像一下，一個常見的 CLI，程式輸入不外乎就是從這兩個入口進來，所以一起來看一下吧！

{{< figure src="/images/cover/0010.png" width="" height="" >}}

## Cobra
雖然 Cobra 的 ReadMe 提供了一些教學，不過有些東西並沒有寫得非常清楚，而且這些教學最終的結果都只是印出一些無用的訊息，對於要建構一個大型一點的或複雜一點的 CLI 幫助不大，另外有些基本用法可能太基本，所以並沒有提到，蠻可惜的，這邊就補充一下。

本來 Golang 的程式入口都在 `main()`，不過利用了 Cobra，程式的入口變成一個一個的 command，再由 command 去呼叫他所需要的 function，所以本來的 `main()` 就單純許多。要開始使用 Cobra，可能有兩種情況：

1. 程式的功能都已經完善，只是想要加上 CLI 的輸入
2. 程式都還沒開始開發，想要從 CLI 輸入開始

### 初始化
Cobra 提供了很方便的 `root.go` 把基本設定都寫好了，不過一定要透過 `cobra init [AppName]` 才有辦法取得這個檔案，所以如果你是屬於第一種狀況的人，就利用 cobra init 一個暫時的專案，再把 `root.go` 複製到自己專案的資料夾中；如果是第二種狀況的人，可以直接使用 cobra 初始化你的專案。

```go
// 第一種狀況，利用 cobra 初始化暫時專案，再複製到你的專案中
cobra init TempProject

// 第二種狀況，直接利用 cobra 初始化你的專案
cobra init MyProject
```

看一下 `root.go` 可以了解到整個 Cobra 的架構，資料結構也都會沿用到其他的 command 中

```go
package cmd
// 定義 rootCmd 的 command、簡短的描以及詳細的描述
var rootCmd = &cobra.Command{
	Use:   "myproject",
	Short: "myproject's short description",
	Long:  "myproject's loooooooooong description",
}

func init() {
    // Cobra 初始化要呼叫的函式，可以自行定義，initMyConfig 就是我自己定義的 init 函式
	cobra.OnInitialize(initConfig, initMyConfig)

    // 這邊開始我們定義 flags，Cobra 支援全域設定，寫在這邊的都會是全域，所有的 command 都可以使用
    // 參數解釋
    // 第一個 &cfgFile 為這個 command binding 的變數名稱
    // 第二個 config 為指令名稱
    // 第三個 "" 為參數為預設值，這邊設為空字串
    // 第四個為指令描述
	rootCmd.PersistentFlags().StringVar(&cfgFile, "config", "", "config file (default is $HOME/config.toml)")
    // 標記 config 為必填欄位
	rootCmd.MarkFlagRequired("config")
}

// 初始化 config 設定，這邊完全不用動，會自動把 config 裡的 key 都存下來，存在 viper 裡面
func initConfig() {
	if cfgFile != "" {
		// Use config file from the flag.
		viper.SetConfigFile(cfgFile)
	} else {
		// Find home directory.
		home, err := homedir.Dir()
		if err != nil {
			fmt.Println(err)
			os.Exit(1)
		}

		// Search config in home directory with name "config" (without extension).
		viper.AddConfigPath(home)
		viper.SetConfigName("config")
	}

	viper.AutomaticEnv() // read in environment variables that match

	// If a config file is found, read it in.
	if err := viper.ReadInConfig(); err != nil {
		fmt.Println("Reading config file", viper.ConfigFileUsed(), "failed.")
	}
}
// 自定義的初始化函式，我把程式中的全域變數存在 viper 中，方便之後不同的 package 都可以直接利用 viper 呼叫
func initMyConfig() {
    viper.Set("baseDomain", "http://google.com/")
}
```
***
### 建立自定義 command
看完 `root.go` 之後我們來新增一個 command，叫 concat，執行完下面指令之後 cobra 會幫你新增一個資料夾 _(cmd)_，並且在裡面新增一個 `concat.go` 的檔案

```go
// 新增 command
cobra add concat
```

來看一下 `concat.go`，可以發現到這邊邏輯和 `root.go` 走的是同一個套路，只是 `rootCmd` 變成了 `concatCmd`，設定方式幾乎都一樣，我們看一下前面沒有看到的 PreRun 和 Run 即可。
```go
// package 為 cmd，和 root.go 的相同
package cmd
// 要 import 的 package
import (
    "PROJECT_PATH/mypath"

    "github.com/spf13/cobra"  
)
// 記得把要對應的變數放在最外面
var path string

// 定義 squareCmd 的設定
var concatCmd = &cobra.Command{
	Use:   "concat",
	Short: "Concat the path with base domain",
    Long:  "Concat the path with base domain, this is a longer description",
    // PreRun 會在 Run 之前執行，這邊我們可以針對這個 command 做檢查，或是把 authentication 放在這邊
	PreRun: func(cmd *cobra.Command, args []string) {
        // 假設長度小於 5 的 path 不處理
        if len(path) < 5 {
            log.Fatal("Path is too short")
        }
    },
    // 重頭戲，主要的執行邏輯
	Run: func(cmd *cobra.Command, args []string) {
        // 呼叫其他的 package
		concatURL, err := mypath.Concat(path)
		if err != nil {
			log.Fatal(err)
		}
		fmt.Println(concatURL)
	},
}

func init() {
	rootCmd.AddCommand(concatCmd)

	concatCmd.Flags().StringVarP(&path, "path", "p", "", "the original sentence")
	concatCmd.MarkFlagRequired("path")
}
```
***
### 呼叫外部 package，並使用全域變數
最後來看一下我們的 package mypath ，看到有些做法是把所有的程式都放到 package cmd 下面，不過官方建議參考 [Hugo repo](https://github.com/gohugoio/hugo) 的作法，Hugo 根據不同的功能，又分成不同的 package，這樣管理起來也挺便利的。

```go
package mypath

import (
    "log"
    "net/url"

    "github.com/spf13/viper"
)

// 如果這個 function 要 export 出去，首字必須大寫
func Concat(path string) (concatURL string) {
    // 讀取剛剛存在 viper 裡面的全域變數
    baseDomain := viper.GetString("baseDomain")

    // Parse base domain
    base, err := url.Parse(baseDomain)
	if err != nil {
		log.Fatal(err)
    }
    // Parse path 
    parsedPath, err := url.Parse(path)
	if err != nil {
		log.Fatal(err)
    }
    // concat base and path 
    concatURL = base.ResolveReference(parsedPath).String()
    return concatURL
}
```
***
這樣就簡單的完成了 golang 的 help message 了，來看一下結果

`main.go` 的 help message，接受兩個 command：concat 和 help，以及一個全域 flag `--config`
```
C:\Users\USERNAME\go\src\myproject>go run main.go --help
myproject's loooooooooong description

Usage:
  myproject [command]

Available Commands:
  concat      The short description of concat command
  help        Help about any command

Flags:
      --config string   config file (default is $HOME/config.toml)
  -h, --help            help for myproject

Use "myproject [command] --help" for more information about a command.
```

concat 指令的 help message，接受一個自定義 flag：--path，以及全域 flag `--config`
```
C:\Users\USERNAME\go\src\myproject>go run main.go concat --help
The loooooooooooooooooooooooooong description of concat command

Usage:
  myproject concat [flags]

Flags:
  -h, --help          help for concat
  -p, --path string   the original sentence

Global Flags:
      --config string   config file (default is $HOME/config.toml)
```

執行結果，把 path 和 base domain concat 起來並印出

```
C:\Users\USERNAME\go\src\myproject>go run main.go concat -p /a123b456c789
http://google.com/a123b456c789

C:\Users\USERNAME\go\src\myproject>go run main.go concat --path /a123b456c789
http://google.com/a123b456c789

```

這個方法真得是簡單直覺好用，只是有些地方因為不熟悉花了些時間研究，但是整體下來我覺得甚至比 python 的 parsearg 都好用許多，程式也相對乾淨。Cobra 的官網說 docker 以及 git 都是用 cobra 來做 CLI 的介面，用這個 module 也讓簡單的 CLI 看起來更專業了。有問題在發問，看到都會回。