---
title: "Angular + Firebase 以角色為基礎的權限管理"
date: 2020-04-22T21:32:07+08:00
draft: true
author: "Jimmy Lin"
isCJKLanguage: true
tags: ["Frontend", "Angular", "Firebase"]
categories: ["Web App Development", "Software Development", "Front-end Development"]
image: "/images/cover/0016.png"
---

這篇文章要來介紹一下我如何結合 Angular 和 Firebase 做到以角色為基礎的權限管理，有些英文的文章可以參考，不過還沒有看到太多中文的，分享一下。這篇文章會跳過 Firebase 上的基礎設定，以及基本的登入，這些網路上都有相關文章。

## 使用情境及流程
這次的故事是這樣，一個內部網站，只能接受有限度的人登入，登入之後會給予該使用者應該有的權限。假設有三種人：管理者，老師以及學生。另外有一個 `canUse` 的標籤來表示是否可以使用系統。

流程如下：

- 如果該使用者是第一次使用，檢查是否有其權限在資料庫中，若有，將權限寫入 Firebase Auth；若無，則不給登入。
- 如果該使用者並非第一次使用，則從 Firebase Auth 讀出資料，並且寫到 UserDetail 中。在 Angular 中用 UserDetail 來決定是否可執行某些操作。
- 改動使用者權限，檢查該使用者是否在 Firebase Auth 中，若有，則寫入權限至 Firebase Auth 以便下次登入使用；若無，僅將權限寫回資料庫。

我們必須使用 Firebase Function 來幫我們做一些額外的事情。

## 新使用者登入
針對新使用者，我們利用 Firebase Functions 來塞權限。程式碼如下：

```typescript
// Firebase Function code
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';

admin.initializeApp();

// 當有使用者被新增時，執行這段程式碼
export const newUser = functions.auth.user().onCreate(async(user) => {
  const setClaimResult = await setNewUserDefaultPriv(user);
});

async function setNewUserDefaultPriv(user) {
  const uid = user.uid;
  const email = user.email;
  // 檢查是否在資料庫
  const memberRes =
    await admin.firestore().collection(`PATH-TO-MEMBER`).where('email', '==', email).limit(1).get();

  if (memberRes.empty) {
    return false;
  }

  let memberData;
  memberRes.forEach((data) => {
    memberData = data.data();
  });
  // 讀書使用者在資料庫的權限，寫到 claims 
  const claims = {
    isAdmin: memberData.isAdmin ? memberData.isAdmin : false,
    isTeacher: memberData.isTeacher ? memberData.isTeacher : false,
    isStudent: memberData.isStudent ? memberData.isStudent : false,
    canUse: memberData.canUse ? memberData.canUse : false,
  };
  // 將 claims 寫回該使用者的帳號中
  await admin.auth().setCustomUserClaims(uid, claims);
  return true;
}
```

## 舊使用者登入
當使用者按下了登入按鈕，被我們寫到 Firebase Auth 中之後，就變成舊使用者了。這邊我們直接看 AuthService 的程式碼：

```typescript
// auth.service.ts should be declare in constructor within auth-guard.ts in Angular
import { AngularFireAuth } from '@angular/fire/auth';
import * as firebase from 'firebase/app';

@Injectable({
  providedIn: 'root'
})
export class AuthService {
  firebaseUserDetail: firebase.User = null;
  userDetail = {
    email: '',
    canUse: false,
    isAdmin: false,
    isTeacher: false,
    isStudent: false,
  };
  private isNewUser: boolean; // 為了給不同的錯誤訊息

  constructor(private afAuth: AngularFireAuth,) {
    this.processUserInfo().subscribe((verified) => {
      if (verified) {

      } else {
        if (this.isNewUser) {
          // 可能是合法使用者，請稍後登入
        } else {
          // 不合法使用者
        }
      }
    }
  }

  private processUserInfo() {
    return this.afAuth.authState.pipe(
      mergeMap(async(firebaseUser) => {
        if (firebaseUser) {
          this.firebaseUserDetail = firebaseUser;
          return firebaseUser.getIdTokenResult();
        } else {
          return of(null);
        }
      }),
      map((idTokenResult) => {
        // 記得在新使用者加入的時候我們塞的 claims 嗎？
        // claims 會存在 Firebase User 的 idTokenResult
        // 拿出來之後存回 userDetail 以便之後使用了
        if (idTokenResult && idTokenResult.hasOwnProperty('claims')) {
          this.userDetail.email = this.firebaseUserDetail.email;
            this.userDetail.canUse = idTokenResult['claims'].canUse;
            this.userDetail.isAdmin = idTokenResult['claims'].isAdmin;
            this.userDetail.isTeacher = idTokenResult['claims'].isTeacher;
            this.userDetail.isStudent = idTokenResult['claims'].isStudent;
            return true;
        } else {
          return false;
        }
      }),
    );
  }
}
```

## 改動使用者權限

這個做法很可惜的地方就是在 Firebase 上沒有辦法改動之前設定的 claims，也看不到。變得反而不好管理，因此在程式碼中我們還是把權限寫進資料庫，並且讓資料庫跟 Firebase Auth 連動，改動資料庫就順便改動 Firebase User 的資料，下次登入也可以拿到最新的權限。一樣利用 Firebase Function 做到這件事情：

```typescript
// Firebase Function code

// 只要有改動某個使用者的資料，就會執行這個函式
export const changeUserPrivOnUpdate = functions.firestore.document(`DOCUMENT-TO-THE-USER`).onUpdate(
  async (change, context) => {
    await changeUserPriv(change.after.data());
  }
);

async function changeUserPriv(memberData)) {
  try {
    // 從使用者資料的 email 找到該使用者的 Firebase User 
    user = await admin.auth().getUserByEmail(memberData.email);
    // 寫進目前資料庫的權限至 Firebase user 中
    await admin.auth().setCustomUserClaims(user.uid, {
      isAdmin: employeeData.isAdmin,
      isTeacher: employeeData.isTeacher,
      isStudent: employeeData.isStudent,
      canUse: employeeData.canUse,
    });
    return true;
  } catch {
    return false;
  }
}
```

## 結束
上述程式碼結合 Angular 和 Firebase 做到以角色為基礎的權限管理，我只有遇到有一個問題

- 第一次登入的使用者因為要設定權限，大約要等個 5~10 秒等待 Function 執行完畢，重新整理頁面之後才可以登入成功。

不過因為是內部網頁，而且也有提示訊息，所以使用者還算清楚。整理來說我覺得還可以接受，另外也是使用原生地 Firebase 方式，不用自己維護使用者登入資料，簡單很多。

如果有問題或建議請留言，看到都會回。