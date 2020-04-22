---
title: 透過 ViewContainerRef 了解 Angular DOM 修改機制
date: 2018-12-20T03:17:31.000+00:00
author: Jimmy Lin
isCJKLanguage: true
tags:
- Frontend
- Angular
categories:
- Web App Development
- Software Development
- Front-end Development
image: "/images/cover/0013.png"
draft: true

---
最近因為工作需要，要自己寫一個 Angular 專用的 UI System，這樣的好處是可以自己維護，也可以適用到公司內部其他的 Angular 專案。在開發一些簡單的 Component 像是 Button、Checkbox 都蠻簡單的，只要了解如何在 Host 以及 Component 溝通就可以了，不過開發到 Modal(Dialog) 的時候碰到了一些問題，網路上找到一些解決方法，也看到一篇很厲害的文章，這篇文章主要就是翻譯那篇神文，原文在：[Exploring Angular DOM manipulation techniques using ViewContainerRef](https://blog.angularindepth.com/exploring-angular-dom-abstractions-80b3ebcfc02)，如果有錯的話也請留言互相切磋一下。

有別於 AngularJS，Angular2 跑在許多不同的平台上，在瀏覽器、手機或是在 Web Worker 裡面。因為這樣，需要一個中間層 (abstractions) 在平台的 API 以及 framework 的 Interface 間溝通。在 Angular 的世界裡，這些中間層就是我們常看到的 ElementRef、TemplateRef、ViewRef、ComponentRef 以及 ViewContainerRef。在這篇文章裡我們會解釋每一個不同的中間層，並且了解他們是如何操作 DOM。

## @ViewChild
在了解一切操作之前，我們先來看一下從 Component 或是 Directive 中如何 access 到這些中間層的。Angular 提供了一種機制叫做 DOM Queries，即是使用 `@ViewChild` 或是 `@ViewChildren` 裝飾器。這兩個的行為是一樣的，差別只在回傳的資料不同。後者回傳的是一個 [QueryList](https://angular.io/docs/ts/latest/api/core/index/QueryList-class.html) 物件，在接下來的文章中，作者都使用 `@ViewChild`。

這兩個裝飾器要搭配 [template reference variable](https://angular.io/docs/ts/latest/guide/template-syntax.html#!#ref-vars) 使用。Template reference variable 是一個可以在 Template 為某一段 DOM 命名的方法。有點類似原生 `html` 中的 `id`。也就是說我們可以在 html 標記某一段 DOM 為某個變數，並且在 component 裡面拿到這段 DOM 並且操作他。這邊寫個簡單的例子：

```typescript
@Component({
    selector: 'sample',
    template: `
        <span #tref>I am span</span>
    `
})
export class SampleComponent implements AfterViewInit {
    @ViewChild("tref", {read: ElementRef}) tref: ElementRef;

    ngAfterViewInit(): void {
        // outputs `I am span`
        console.log(this.tref.nativeElement.textContent);
    }
}
```

基本的 `@ViewChild` 語法為：

```typescript
@ViewChild([reference from template], {read: [reference type]});
```

在上面的程式碼可以看到我們使用 `tref` 在 `HTML` 裡面標了某一段 DOM，在 Component 中我們利用 `@ViewChild` 拿到了一個型別為 `ElementRef` 的資料關連到所標記的 DOM。`@ViewChild` 中的變數 `read` 並非必要，因為 Angular 會根據所標記的 DOM 推斷是哪種類別。舉例來說，若所標記的是一段簡單的 HTML 像範例中的 `<Span>`，Angular 回傳 `ElementRef`；若標記的是 `template`，則 Angular 回傳 `TemplateRef`。不過有些較複雜的像是 `ViewContainerRef` 就無法被 Angular 正確的猜到，因此要在 `read` 裡面特別註明。另外 `ViewRef` 無法從 DOM 回傳，因此要手動建置。

## ElementRef
這是最基礎的中間層，如果你細看一下回傳的資料結構，可以看到他關聯到原生的 element，好用的地方在於我們可以很方便的取得原生地 DOM element，例如下面這段：
```typescript
// output `I am span`
console.log(this.tref.nativeElement.textContent);
```
不過根據 Angular 官方的文件，他們[不建議](https://angular.io/docs/ts/latest/api/core/index/ElementRef-class.html)這樣使用，不單單是有安全上的疑慮，另外也讓你的應用程式和 render layer 緊密連結，更困難跑在多平台上。

使用 @ViewChild 裝飾器可以回傳任何的 DOM 元件，並且給定型別 `ElementRef`。不過因為所有的 Component 都 host 在 custom DOM 內，所有的 Directive 也都寫在 DOM 中，我們可以利用 DI 方法來取得 host element。

```typescript
@Component({
    selector: 'sample',
})
export class SampleComponent{
    constructor(private hostElement: ElementRef) {
        //outputs <sample>...</sample>
        console.log(this.hostElement.nativeElement.outerHTML);
    }
}
```
因為 componet 可以藉由 DI 取得 host element，`@ViewChild` 裝飾器最常使用在拿到 DOM element 在 view 中的 reference。同樣的，在 directive 沒有 view，因此可以直接使用相對應的 element。

## TemplateRef
template 的概念在網頁開發中屢見不鮮，template 就是 DOM 的組合，而且可以被不同的 app 重複使用。HTML5 原生支援了 [`template`](https://developer.mozilla.org/en/docs/Web/HTML/Element/template) tag，Angular 開發了 `TemplateRef` 並且可以和 template tag 一起使用。看一下怎麼使用：

```typescript
@Component({
    selector: 'sample',
    template: `
        <ng-template #tpl>
            <span>I am span in template</span>
        </ng-template>
    `
})
export class SampleComponent implements AfterViewInit {
    @ViewChild("tpl") tpl: TemplateRef<any>;

    ngAfterViewInit() {
        let elementRef = this.tpl.elementRef;
        // outputs `` (empty)
        console.log(elementRef.nativeElement.textContent);
    }
}
```
Angular 從 DOM 移除了 `template` element，並且新增了一段註解進去，render 的結果如下：

```html
<sample>
    <!-- -->
</sample>
```
`TemplateRef` 是一個簡單的 class，他存著與其 host element 的關聯在 elementRef 內，而且有一個 method `createEmbededView`，用這個 method 可以新增 view，並且回傳 `ViewRef`。

## ViewRef
這個中間層表示了 Angular 的 view。在 Angular 的世界中，View 是一個應用程式 UI 的基本組成。他是最小的 element 組成單位，在同一個 view 中的 element 會同時被新增或同時被摧毀 (destroyed)。Angular 建議開發者把 UI 視為 Views 的組合，而不是 HTML Tag 樹狀結構的一部分。Angular 支援兩種不同類型的 View：

1. **Embeded Views** 連結到 Template
2. **Host Views** 連結到 Component

### 新增 Embeded View
View 可以透過 createEmbededView 從 template 初始化。

```typescript
ngAfterViewInit() {
    let view = this.tpl.createEmbeddedView(null);
}
```

### 新增 Host View
Host view 在 component 初始化時同時被產生，利用 ComponentFactoryResolver 可以動態產生 component。
```typescript
constructor(private injector: Injector, private r: ComponentFactoryResolver) {
    let factory = this.r.resolveComponentFactory(ColorComponent);
    let componentRef = factory.create(injector);
    let view = componentRef.hostView;
}
```
在 Angular 中，每一個 component 都與特定的 injector 綁定，因此當新增一個 component 時，可以把目前的 injector instance 傳進去。最重要的一點是，當 component 是動態產生的時候，一定要把這個 component 家道 [EntryComponents](https://angular.io/docs/ts/latest/cookbook/ngmodule-faq.html#!#q-entry-component-defined) 裡面。


這樣就清楚要如何動態的產生 view 了，一旦產生 view，這個 view 就可以使用 ViewContainer 被加入到 DOM 裡面去。

## ViewContainerRef
ViewContainer 代表一個 Container 可以 attach 一到多個 view。首先要先知道任何的 DOM 都可以被用來當作 view container。有趣的是 Angular 並**沒有**在 element 中插入(not insert)  view，而是附加(but append) 到目前綁定的 ViewContainer 上。這與 `router-outlet` 的做法類似。通常我們用 `ng-container` 讓 ViewContainer 知道哪些地方可以插入 view。先看例子吧：

```typescript
@Component({
    selector: 'sample',
    template: `
        <span>I am first span</span>
        <ng-container #vc></ng-container>
        <span>I am last span</span>
    `
})
export class SampleComponent implements AfterViewInit {
    @ViewChild("vc", {read: ViewContainerRef}) vc: ViewContainerRef;

    ngAfterViewInit(): void {
        // outputs ``
        console.log(this.vc.element.nativeElement.textContent);
    }
}
```
例子中可以看到 vc (ViewContainerRef) 與 ng-container 綁定，而且在 render 的時候是 render 註解，因此不會產生多餘的 HTML。

### 操作 views
ViewContainerRef 提供許多方便的 API 讓開發者可以操作 DOM：

```typescript
class ViewContainerRef {
    ...
    clear() : void
    insert(viewRef: ViewRef, index?: number) : ViewRef
    get(index: number) : ViewRef
    indexOf(viewRef: ViewRef) : number
    detach(index?: number) : ViewRef
    move(viewRef: ViewRef, currentIndex: number) : ViewRef
}
```
前面提了兩種方法可以手動從 template 或 component 新增 view。一旦我們有 view，可以用 `insert` 新增到 DOM 內。下面的例子展現了如何從 template 新增一個 embeded view 並且新增到 ng-container element 中：

```typescript
@Component({
    selector: 'sample',
    template: `
        <span>I am first span</span>
        <ng-container #vc></ng-container>
        <span>I am last span</span>
        <ng-template #tpl>
            <span>I am span in template</span>
        </ng-template>
    `
})
export class SampleComponent implements AfterViewInit {
    @ViewChild("vc", {read: ViewContainerRef}) vc: ViewContainerRef;
    @ViewChild("tpl") tpl: TemplateRef<any>;

    ngAfterViewInit() {
        let view = this.tpl.createEmbeddedView(null);
        this.vc.insert(view);
    }
}
```
產生出來的 html 如下：

```html
<sample>
    <span>I am first span</span>
    <!-- -->
    <span>I am span in template</span>

    <span>I am last span</span>
    <!-- -->
</sample>
```

### 新增 views
ViewContainer 也有提供 API 來新增 view：

```typescript
class ViewContainerRef {
    element: ElementRef
    length: number

    createComponent(componentFactory...): ComponentRef<C>
    createEmbeddedView(templateRef...): EmbeddedViewRef<C>
    ...
}
```
以上就是簡單的範例可以從 template 或 component 產生 view，並且新增到 html 中特定的位置。

## ngTemplateOutlet 和 ngComponentOutlet
身為工程師，總是希望找到更簡單的方法，所以 Angular 也很貼心的提供了兩個語法糖更易用，利用這兩個語法糖，可以少打上面那些指令。

### ngTemplateOutlet
這一個語法糖會把 DOM element 直接轉成 `ViewContainer` 接著插入一個 embeded view 到 template，這些行為完全不用寫任何的 code 在 component class 中。也就是說上面那段新增 view，然後插入到 `#vc` DOM element 的範例可以改寫成：

```typescript
@Component({
    selector: 'sample',
    template: `
        <span>I am first span</span>
        <ng-container [ngTemplateOutlet]="tpl"></ng-container>
        <span>I am last span</span>
        <ng-template #tpl>
            <span>I am span in template</span>
        </ng-template>
    `
})
export class SampleComponent {}
```
由上面的例子可以看到不用再 component 寫任何程式碼也可以把你要的 template 插入特定的 `ng-container` 中。

### ngComponentOutlet
這個語法糖和上一個類似，不過這個是新增一個 host view (component)，而**不是** embeded view，用法如下：

```typescript
<ng-container *ngComponentOutlet="ColorComponent"></ng-container>
```

## 最後
其實看完原文之後就很清楚要怎麼動態產生 DOM 了，也大概可以猜到 *ngIf 底層是怎麼做的，這樣也可以在撰寫 UI system 時候有更多的掌控權，不用依賴第三方的套件。為了避免以後忘記，寫了一個範例放在 Stackblitz 上：[https://stackblitz.com/edit/explore-dom](https://stackblitz.com/edit/explore-dom)，會更清楚喔！