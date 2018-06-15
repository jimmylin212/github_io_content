---
title: "利用 Golang 與 GraphQL Server 做檔案操作"
date: 2018-06-14T00:16:02+08:00
draft: true
author: "Jimmy Lin"
isCJKLanguage: true
tags: ["Golang", "GraphQL", "Backend"]
categories: ["Web App Development", "Back-end Development"]
---

我們目前做的網站提供上傳以及下載的功能，另外想要提供一個 offline CLI Tool 可以讓使用者更方便的上傳或下載檔案，也有可能可以整合這個 CLI tool 到他們自己的 CI\CD 流程當中。因為是獨立的 tool，想要用 Golang 來完成，順便可以了解一下這個語言的特性。

先簡單說一下我們 server 的環境配置：

- 利用 Json Web Token (JWT) 來作為認證的方式
- GraphQL 來當作 api 的唯一接口

因為功能不多，所以這篇文章就專注在如何使用 Golang 來發送 GraphQL 的 query 以及 mutation 指令到指定的 server。這個簡單的小工具，流程是這樣的：

1. 檢查 `config.toml` 當中有沒有 JWT 的欄位，有的話就讀出來，沒有的話回報錯誤
2. 從使用者輸入查看 action 是 import 還是 export，以及指令的 ProjectID，執行不同的函式
3. 發送 GraphQL 指令
4. 整理 GraphQL response

## 利用 Golang 讀取 config 檔
每個語言或是作業系統都有不同對應的設定檔格式，像 Windows 鍾愛 `.ini`，有些語言又可以支援多種的格式，像 Python 支援 `.ini` 以及 `.json`。在 Golang 當中官方推薦的格式是 `.toml`，這個格式很像 `.ini`，不過又更彈性。在 Golang 當中要讀取 toml 的檔案相當簡單，直接看下面程式碼：

```go
type Config struct {
	Token string
}

func readConfig() (config Config) {
    // 讀取執行檔所在路徑
	exeFile, err := os.Executable()
	if err != nil {
		log.Fatal(err)
	}
    // 把資料夾路徑和 config.toml Join 在一起，方便之後讀檔
	configFilePath := filepath.Join(filepath.Dir(exeFile), "config.toml")
	_, err = os.Stat(configFilePath)
	if err != nil {
		log.Fatal("Config file is missing: ", configFilePath)
	}

    // 利用這行就可以直接把 config.toml 檔案的內容直接轉換到上面定義好的 Config Struct 中
	_, err = toml.DecodeFile(configFilePath, &config)
	if err != nil {
		log.Fatal(err)
    }
    // 回傳一個 config Config 的結構
	return config
}
```
要簡單的利用 toml 的格式，我們必須安裝另外一個第三方套件 [https://github.com/BurntSushi/toml](https://github.com/BurntSushi/toml)，利用下面指令可以直接安裝

```bash
go get github.com/BurntSushi/toml

# 如果被 Proxy 擋住了
https_proxy=PROXY_URL:PORT go get github.com/BurntSushi/toml
```
***
## 利用 Golang 建立一個簡單的 Help Message並支持使用者輸入
我覺得 Golang 在這塊沒有像 Python 這麼的完善，介面相對簡單許多，也沒有提供 required 的功能，需要我們自己來實踐，直接看程式碼：

```go
// 定義變數
var action string
var projectID string
var dirFilePath string

// StringVar 參數解釋: 對應變數, 參數名稱, 預設值, 說明
flag.StringVar(&action, "action", "", "Action should be import / export")
flag.StringVar(&projectID, "pid", "", "The target project ID")
flag.StringVar(&dirFilePath, "path", "", "The directory or file path. Always using / on Windows")
flag.Parse()

// 簡單做一下 required，如果有欄位缺失就跳錯誤
if action == "" || projectID == "" || dirFilePath == "" {
	log.Fatal(errors.New("Input is not complete, please check again"))
}
```
上面這段編譯之後執行 `./PROJECT.EXE -h` 可以看到下面的 help 訊息

```bash
Usage of C:\\Users\\USER_NAME\\go\\src\\PROJECT\\PROJECT.exe:
  -action string
        Action should be import / export
  -path string
        The directory or file path. Always using / on Windows
  -pid string
        The target project ID
```

使用者藉由輸入
```bash
./PROJECT.EXE -action import -path PATH_TO_DIR_FILE -pid PROJECT_ID 
```

即可進入程式

***
## 利用 Golang 發送 GraphQL Query 以及 Mutation 指令
發現 Golang 當中相當推薦使用 Struct 的方式來處理複雜的變數，用一用也覺得蠻不錯用的，所以結果都會轉換成 Struct 再回傳。這邊是定義的回傳 Struct。

```go
type Import struct {
	originRows  float64
	currentRows float64
	errorRows   float64
}

type Export struct {
	filePath string
	fileName string
}
```

### 共用函式 
因為最後的 request 都會走到同一個步驟，所以把發送 request 的部分抽出來，這樣才不會有大量重複的程式碼。

```go
func graphqlQuery(request *http.Request) (statusCode string, response map[string]map[string]map[string]interface{}, err error) {
    // 增加其他 headers
	request.Header.Add("Authorization", fmt.Sprintf("Bearer %s", gJwtToken))
	request.Header.Add("Cache-Control", "no-cache")

    // 發送 request
	resp, err := http.DefaultClient.Do(request)
	if err != nil {
		return statusCode, response, err
	}

	defer resp.Body.Close()

    statusCode = resp.Status
    // 讀取 response.body 
    body, _ := ioutil.ReadAll(resp.Body)
    // 轉換成 json 格式
	err = json.Unmarshal(body, &response)

	return statusCode, response, err
}
```
### Query / Mutation
這邊雖然說是 Query，不過還是發送 mutation，因為對 Graphql 來說根本沒有差，query 也可以按照下面寫法。

```go
func graphqlExport(projectID string) (finalRes Export, prettyRes string, err error) {
	query := `mutation { 
		export(projectId:"%s") {
			filepath 
			filename
		} 
	}`

    // 把 query 變數轉成 Graphql 格式
    querySrting := `{"query":` + strconv.QuoteToASCII(fmt.Sprintf(query, projectID)) + `}`
    
    // 建立新的 request，以及特定的 header
	request, _ := http.NewRequest("POST", gGraphqlURL, strings.NewReader(querySrting))
	request.Header.Add("Content-Type", "application/json")

    // 呼叫 共同函式發 query
	_, response, err := graphqlQuery(request)
	if err != nil {
		return finalRes, prettyRes, err
	}

    // 轉換 response 成 struct 格式
	finalRes.fileName = response["data"]["export"]["filename"].(string)
	finalRes.filePath = response["data"]["export"]["filepath"].(string)

    // Pretty print response 格式
	tmpPrettyRes, err := json.MarshalIndent(response, "", "  ")
	prettyRes = string(tmpPrettyRes)

	return finalRes, prettyRes, nil
}
```
### Mutation Form-data Multipart
這邊比較特別一點，因為我們要上傳檔案至 server，利用了 form-data 的功能，所以這邊其實是把所有的 key-value 都包成 payload，再一次傳出去。總共有三個 key：

- operations: Graphql 的指令
- map: 檔案對應的變數
- file: 檔案的相關資訊

這三個的格式是因為 server 上的 [Apollo-server-upload](https://github.com/jaydenseric/apollo-upload-server) module 的[教學](https://github.com/jaydenseric/graphql-multipart-request-spec)文件這樣寫，我們就這樣採用了，並沒有特殊原因。

```go
func graphqlImport(filePath, projectID string) (finalRes Import, prettyRes string, err error) {
    // 開啟要上傳的檔案
	file, err := os.Open(filePath)
	if err != nil {
		return finalRes, prettyRes, errors.New("Could not open the file, file format incorrect")
	}
	defer file.Close()

    queryBody := &bytes.Buffer{}
    // 建立 multipart writer instance
    writer := multipart.NewWriter(queryBody)
	boundary := writer.Boundary()

	query := `mutation ($projectId:ID! $file:Upload!) { 
		import(projectId:$projectId file:$file) {
			originRows 
			currentRows 
			errorRows
		} 
	}`
	variables := `{	
		"projectId" : "%s",	
		"file" : null 
	}`

    // 把 query 變數以及 variables 變數串一起
	operationsValue := `{"query":` + strconv.QuoteToASCII(query) + `,"variables":` + fmt.Sprintf(variables, projectID) + `}`
    mapValue := `{"file" : ["variables.file"]}`
    
    // 寫入 multipart 中
	_ = writer.WriteField("operations", operationsValue)
    _ = writer.WriteField("map", mapValue)
    
    // 讀取檔案相關資訊
	part, err := writer.CreateFormFile("file", filepath.Base(filePath))
	if err != nil {
		return finalRes, prettyRes, errors.New("Could not open the file, file format incorrect")
	}

    // 寫入 multipart 中
	_, err = io.Copy(part, file)

	err = writer.Close()
	if err != nil {
		return finalRes, prettyRes, err
	}

    // 建立新的 request，以及特定的 header 
	request, _ := http.NewRequest("POST", gGraphqlURL, queryBody)
	request.Header.Add("Content-Type", fmt.Sprintf("multipart/form-data; boundary=%s", boundary))

    // 呼叫 共同函式發 query
	_, response, err := graphqlQuery(request)
	if err != nil {
		return finalRes, prettyRes, err
	}

    // 轉換 response 成 struct 格式
	finalRes.currentRows = response["data"]["import"]["currentRows"].(float64)
	finalRes.originRows = response["data"]["import"]["originRows"].(float64)
	finalRes.errorRows = response["data"]["import"]["errorRows"].(float64)

    // Pretty print response 格式
	tmpPrettyRes, err := json.MarshalIndent(response, "", "  ")
	prettyRes = string(tmpPrettyRes)

	return finalRes, prettyRes, nil
}
```
***
## 利用 Golang 下載某個 URL 的檔案
```go
func downloadFromURL(dstDirPath string, finalRes Export) error {

	base, err := url.Parse(gBaseDomain)
	if err != nil {
		return err
	}

	downloadPath, err := url.Parse(finalRes.filePath)
	if err != nil {
		return err
	}

	downloadURL := base.ResolveReference(downloadPath).String()
	dstFilePath := filepath.Join(dstDirPath, finalRes.fileName)

	log.Printf("Download %v -> %v", downloadURL, dstFilePath)

	file, err := os.Create(dstFilePath)
	if err != nil {
		return err
	}
	defer file.Close()

	resp, err := http.Get(downloadURL)
	if err != nil {
		return err
	}
	defer resp.Body.Close()

	_, err = io.Copy(file, resp.Body)
	if err != nil {
		return err
	}

	return nil
}
```
***
## 利用 Golang 檢查檔案是否存在或是輸入是否為資料夾
Golang 裡面這種對於資料夾的驗證比較麻煩一點，或是並沒有提供，查一查發現許多人都會自己寫並且包成一個 function 方便未來使用，如下：

```go
// 檢查檔案是否存在
func fileExists(path string) error {
	_, err := os.Stat(path)
	if err == nil {
		return nil
	}
	if os.IsNotExist(err) {
		return errors.New("File is not exists")
	}
	return err
}
// 檢查路徑是否為資料夾
func isDirectory(path string) error {
	pathStat, err := os.Stat(path)
	if err != nil {
		return errors.New("Path is not exists")
	}

	if pathStat.IsDir() {
		return nil
	}

	return errors.New("Path is not directory")
}
```
***
