---
title: "在 Angular 中用 Directive 實作密碼確認表單"
date: 2019-07-11T17:28:45+08:00
draft: false
author: "Jimmy Lin"
isCJKLanguage: true
tags: ["Frontend", "Angular"]
categories: ["Web App Development", "Software Development", "Front-end Development"]
image: "/images/cover/0014.png"
---

這篇文章主要敘述如何使用 Directive 來做到密碼確認表單。在製作網站的時候，常常需要使用者輸入密碼，常看到的設計是會多一個輸入欄位，讓使用者確認密碼，這件事情在 Angular 裡面有簡單的做法，像是利用 ngModel 直接在 typescript 裡面比對兩個值是否相等，不相等就在畫面上顯示錯誤訊息；同樣的作法也可以用在 Reactive Form 上。來看看怎麼做吧。

## 基本的 Component
首先我們要有一個最基礎的 Component，才有辦法使用等等寫出來的 Directive，這邊是最基本的 Component 的內容。可以看到就只有簡單的新增一個 Reactive Form: exampleForm，然後初始化他。

```typescript
import { Component, OnInit } from '@angular/core';
import { FormBuilder, FormGroup } from '@angular/forms';

@Component({
  template: `
  <form [formGroup]="exampleForm">
    <input formControlName="password">
    <input formControlName="passwordConfirm">
  </form>
  `
})

class TestMockComponent implements OnInit {
  exampleForm: FormGroup;

  constructor(
    private formBuilder: FormBuilder,
  ) {}

  ngOnInit() {
    exampleForm = this.formBuilder.group({
      password: null,
      passwordConfirm: null
    });
  }
}
```

## 新增 Directive
新增的指令這邊就不贅述了。這個 Directive 要做到四件事情：

1. 擴充原生的 validators，使用 NG_VALIDATORS
2. 讀取 Directive 的值
3. 拿到寫上這個 Directive 的 Form Control
4. 比較兩個欄位是否相等，如果不相等就把 Form Control 設 Error

直接看整段程式碼，我把做的事情用註解寫起來比較清楚

```typescript
import { Directive, Attribute } from '@angular/core';
import { Validator, NG_VALIDATORS, AbstractControl } from '@angular/forms';

@Directive({
  selector: '[equalsTo]',
  providers: [
    // 擴充原生的 validators，使用 NG_VALIDATORS
    { provide: NG_VALIDATORS, useExisting: EqualstoDirective, multi: true },
  ],
})

export class EqualstoDirective implements Validator {

  constructor(
    // 讀取 equalsTo 帶的值，這個值就是被比較的 form control 名稱
    @Attribute('equalsTo') public equalsTo: string,
  ) {}

  // 利用 AbstractControl 得到使用這個 directive 的 form control
  validate(control: AbstractControl) {
    const controlValue = control.value;
    const compareToCtrl = control.parent.get(this.equalsTo); // 讀取被比較對象的值
    const errorObj = { equalsTo: { valid: false } }; // 定義錯誤格式

    // 當有被比較對象，而且兩者都已經被使用者 interact 過了才開始比較值是否相等
    if (compareToCtrl && control.dirty && compareToCtrl.dirty && controlValue !== compareToCtrl.value) {
      control.setErrors(errorObj); // 如果值不相等，則設定 Error 給目前的 form control
      return errorObj; // 回傳錯誤可以接到 validator，吐出錯誤訊息，或是針對 DOM 操作
    } else if (compareToCtrl && control.dirty && compareToCtrl.dirty && controlValue === compareToCtrl.value) {
      // 如果相等，把 equalsTo 從 Error 拿掉
      if (control.errors && 'equalsTo' in control.errors) {
        delete control.errors.equalsTo;
        // 檢查是否還有其他錯誤。有時候我們在新增 form 時會寫其他的 validator。
        // 如果沒有其他錯誤，設 Error 為 null
        if (Object.keys(control.errors).length === 0) {
          control.setErrors(null);
        }
      }
    }
    return null;
  }
}
```

## 使用 Directive
使用方法很直覺，直接在比較對象加上 `equalsTo` 即可，如下面範例就是希望 `passwordConfirm` 和 `password` 相等，如果不相等，會標記 `passwordConfirm` 為錯誤，整個 form 會變成 invalid。

```HTML
<form [formGroup]="exampleForm">
  <input formControlName="password">
  <input formControlName="passwordConfirm" equalsTo="password">
</form>
```

## 現成懶人包
為了要順便學習下如何打包一個 npm package 並且 publish 到 npm 上，我用了這個範例來當作練習，看到有一些人下載也是頗是欣慰。

- npm package: https://www.npmjs.com/package/ngx-equalsto
- Github repo: https://github.com/jimmylin212/ngx-equalsto

這只是個很簡單的 Directive，不過很常用到，如果有一些建議也歡迎留言。


