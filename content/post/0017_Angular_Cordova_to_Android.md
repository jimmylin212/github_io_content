---
title: "Anfular + Firebase + Cordova 完成 Andoird 程式"
date: 2021-01-20T23:10:28+08:00
draft: false
author: "Jimmy Lin"
isCJKLanguage: true
tags: ["Frontend", "Angular", "Firebase", "Cordova", "Android"]
categories: ["Web App Development", "Software Development", "Front-end Development", "Mobile App Development"]
image: "/images/cover/0017.png"
---

一直對同一份程式碼可以在不同的環境下執行很著迷，也覺得這是未來發展方向，因此當初做了 Angular + PWA 以為這樣就夠了，沒想到客戶一定要手機版的 APP ，PWA 這種假裝是 APP 的產物客戶無法接受，因此開始研究了如何把 Angular 包裝成 Android App，至於 iOS，有機會再寫一篇吧 (上架費也太貴了吧!!!)。先簡單說一下我的環境，專案使用 Angular，Host 在 Firebase，因此這篇文章還會簡單講一下 Firebase 相關的設定。

## 打包 Angular App
相信網路上教學已經很多了，重點就是要改 `base-href` 、設定好 `deviceready` 和加上 `cordova.js`，寫在 `app.component.ts` 的 `ngOnInit()` 裡面。

```typescript
// app.component.ts
  ngOnInit() {
    // code for app version
    document.addEventListener("deviceready", function() { 
      alert(this.device.platform); 
    }, false); 
  }
```

```html
<!-- index.html -->
<script type=”text/javascript” src=”cordova.js”></script>
```

另外在 package.json 裡面新增一個指令專門 build mobile app

```json
{
    "build-app": "ng build --prod --base-href=. --output-path=dist/mobile",
}
```

## 新增 Cordova 專案
因為網路上教學頗多的，就直接把指令列下來了。需要注意的是如果你跟我把網站放在 Firebase，ID 需要和 Firebase 上的 Android Package Name 相同，不然會找不到。

```bash
npm install -g cordova
# The "com.mobile.app" should be the same with what you create in Firebase, or it report can't find package issue.
cordova create CordovaMobileApp com.mobile.app "CordovaMobileApp"
cd CordovaMobileApp
cordova platform add android
# 在本機使用需安裝 cordova-plugin-device
cordova plugin add cordova-plugin-device
```

接著把剛剛執行 `build-app` 後在 `dist` 資料夾下的所有檔案都複製到 `CordovaMobildApp/www` 路徑下。一般的教學就會到這邊，不過因為我的後端串了 Firebase，所以還需要再把 Firebase 的一些資料填過來，才有辦法讓 Andoird App 連到 Firebase。在 Firebase 新增一個 Android App，然後跟著教學把該填的資料都填一填，主要會

1. 新增 google-service.json 至 project 中
2. 修改 Project-level 的 build.gradle (<project>/build.gradle) 以及 App-level build.gradle (<project>/<app-module>/build.gradle)

接下來輸入 

```bach
cordova build android
```

然後就會開始出錯....

## 遇到的錯誤們

### AndroidX dependencies
遇到的第一個錯誤

> This project uses AndroidX dependencies, but the 'android.useAndroidX' property is not enabled. Set this property to true in the gradle.properties file and retry.

蠻直覺的，直接修改 `grade.properties` 內的

```
android.useAndroidX=true
android.enableJetifier=true
```

結果發現錯誤一樣出現，回去看 `grade.properties` 發現剛剛改的值又被改回來了。查了一下找到 Stackoverflow 上的一個解答 (https://stackoverflow.com/a/63842227/743158) ，說即使環境用的是 Android@9.0.0 還是發生同樣的錯誤，有可能是因為你安裝的 cordova plugin 用了舊的環境，所以你的設定被改寫了。嘗試安裝 plugin `cordova plugin add cordova-plugin-androidx-adapter` 後再 build 一次就成功了，立刻打開 Android Studio 看看結果。

一片空白!!!!!

### Service Worker
搜尋了一下發現有可能是 service-worker 在搞鬼，試圖拿掉 service worker 又加回來，過程中偶然發現一個錯誤訊息

> Service worker registration failed with: TypeError: Failed to register a ServiceWorker: The URL protocol of the current origin ('file://') is not supported.

寫得很明顯了，不能用 file:// 當 portocol，搜尋一下又在 Stackoverflow 找到這篇文章 (https://stackoverflow.com/a/53616776/743158)，原來可以安裝另外一個 plugin 來 enable service worker。立刻試試看~

```bash
cordova plugin add cordova-plugin-ionic-webview
```

並且按照 Github repo (https://github.com/ionic-team/cordova-plugin-ionic-webview) 的 Readme 在 `config.xml` 加上

```xml
<platform name="android">
  <allow-intent href="market:*" />
  <preference name="Scheme" value="https" /> <!-- 加上這行 -->
</platform>
```

重 build 一次，再打開，終於看到畫面了，試試看登入的流程，結果發現明明在程式裡面已經使用 `signInWithRedirect()` 但是卻還是開了瀏覽器，登入之後也無法導回 App，流程有問題。

### 登入失敗的種種原因
查了一下發現原來 Firebase 的文件有寫到，如果要使用 Cordova 登入，需要做額外的設定 (https://firebase.google.com/docs/auth/web/cordova)，按照教學做了一遍，結果 build 的後出現錯誤

> Cannot read property 'manifest' of undefined

Google 了一下找到 Github 這個 issue [Cannot read property 'manifest' of undefined](https://github.com/nordnet/cordova-universal-links-plugin/issues/133)，其中有條 comment 寫到他上了 PR 不過沒人理他，建議大家可以直接安裝他的 repo (https://github.com/nordnet/cordova-universal-links-plugin/issues/133#issuecomment-369260863)，死馬當活馬醫，裝裝看。

```bash
cordova plugin rm cordova-universal-links-plugin
cordova plugin add https://github.com/walteram/cordova-universal-links-plugin
```

再 build 一次，出現另外一個錯誤

> Manifest merger failed : uses-sdk:minSdkVersion 16 cannot be smaller than version 22 declared in library [:CordovaLib] C:\PATH\platforms\android\CordovaLib\build\intermediates\library_manifest\debug\AndroidManifest.xml as the library might be using APIs not available in 16

又找到另外一個 Github issue [Manifest merger failed : uses-sdk:minSdkVersion 19 cannot be smaller than version 22 declared in library](https://github.com/apache/cordova-android/issues/1070)，有條 comment (https://github.com/apache/cordova-android/issues/1070#issuecomment-714923226) 說要在 `config.xml` 加上

```xml
<edit-config file="AndroidManifest.xml" mode="merge" target="/manifest">
    <manifest xmlns:tools="http://schemas.android.com/tools" />
</edit-config>
<config-file parent="/manifest" target="AndroidManifest.xml">
    <uses-sdk tools:overrideLibrary="org.apache.cordova" />
</config-file>
```

再 build 一次，又出錯

> A failure occurred while executing com.android.build.gradle.internal.tasks.Workers$ActionFacade

> Android resource compilation failed
  C:\PATH\platforms\android\app\src\main\res\xml\config.xml:57: AAPT: error: unbound prefix.
  C:\PATH\platforms\android\app\src\main\res\xml\config.xml: AAPT: error: file failed to compile.

再找到另外一個 Github issue [Error parsing XML: unbound prefix when doing cordova build android](https://github.com/dpa99c/cordova-custom-config/issues/24) 中提到要在 `config.xml` 加上 namespace。

```xml
<!-- 加上 xmlns:tools="http://schemas.android.com/tools" -->
<widget id="com.divingseafu" version="1.0.0" xmlns="http://www.w3.org/ns/widgets" xmlns:cdv="http://cordova.apache.org/ns/1.0" xmlns:tools="http://schemas.android.com/tools">
</widgt>
```

再 build 一次，還是一樣，不知道為什麼還是開了 chrome 要執行登入

找到另外一個 Github issue [signInWithRedirect does not redirect back to app](https://github.com/angular/angularfire/issues/1860)，有條 comment (https://github.com/angular/angularfire/issues/1860#issuecomment-481641153) 說要在 config.xml 中加上

```xml
<allow-navigation href="*"/>
```

再 build 一次，發現不會導到 chrome 了，不過還是出現錯誤

> 403: disallowed_useragent 

簡單搜尋一下 Stackoverflow 這篇 (https://stackoverflow.com/a/57035086/743158) 提到，需要在 `config.xml` 再加上

```xml
<preference name="OverrideUserAgent" value="Mozilla/5.0 Google" />
```

終於可以順利登入看到畫面了。

### 上一頁破圖
在使用的過程當中發現按下手機上的返回鍵，畫面會卡在中間，簡單搜尋一下找到 Github issue [Bug router angular v5.2.4+ with cordova (BrowserAnimationsModule bug)](https://github.com/angular/angular/issues/22509) 發現是 `BrowserAnimationsModule` 的問題，結果有條 comment (https://github.com/angular/angular/issues/22509#issuecomment-382297711) 提供了 workaround 的作法，在 `index.html` 中，引入 `cordova.js` 的前面加上：

```html
<script>
  window.addEventListener = function () {
    EventTarget.prototype.addEventListener.apply(this, arguments);
  };
  window.removeEventListener = function () {
    EventTarget.prototype.removeEventListener.apply(this, arguments);
  };
  document.addEventListener = function () {
    EventTarget.prototype.addEventListener.apply(this, arguments);
  };
  document.removeEventListener = function () {
    EventTarget.prototype.removeEventListener.apply(this, arguments);
  };
</script>
<script src="cordova.js"></script>
```

## 打包
打包 Android 也是很重要的一環，不過這部分網路上有蠻多教學，我就只列出會用到的指令

```bash
## Build release version
cordova build android --release
## Generate keystore and sign the apk with this file
keytool -genkey -v -keystore release-key.keystore -alias cordova-demo -keyalg RSA -keysize 2048 -validity 10000
## Sign the apk 
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore release-key.keystore android-apk/android-release-unsigned.apk cordova-demo
## compress the apk file
zipalign -v 4 android-apk/android-release-unsigned.apk android-apk/cordova-demo.apk
```

也可以把這些指令都包進 cordova 的 `build.json` 中

```json
{
	"android": {
		"release": {
			"keystore": "release-key.keystore",
			"alias": "ALIAS", // 自行取代
			"storePassword": "STOREPASSWORD", // 自行取代
			"password": "PASSWORD" // 自行取代
		}
	}
}
```

以後只要執行

```bash
cordova build android --release
```

就可以產生 sign 好的 apk 檔囉。

## Reference
其他參考的網頁：
- https://dotblogs.com.tw/leo_codespace/2018/08/21/110458
- https://codertw.com/android-%E9%96%8B%E7%99%BC/28014/
- https://medium.com/@EliaPalme/how-to-wrap-an-angular-app-with-apache-cordova-909024a25d79 