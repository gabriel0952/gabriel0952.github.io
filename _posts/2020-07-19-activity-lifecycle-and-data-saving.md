---
title: Activity 生命週期與資料保存
author: gabriel0952
date: 2020-07-19 11:33:00 +0800
categories: [Android]
tags: [Android, Lifecycle]
---

## 前言
相信不管是有學過，或是有打算學過 Android APP 開發的人一定都會知道的一件事是 Activity 的生命週期，詳見下圖：

![lifecycle](/assets/img/post/activity_lifecycle.png)

這是一張來自 Android Doc 的 [Activity Lifecycle](https://developer.android.com/guide/components/activities/activity-lifecycle)，其實他的核心概念很簡單，基本上就是 創建 → 開始 → 恢復 → (運作) → 暫停 → 停止 → 銷毀 由外向內兩兩對應，既然創建了就一定需要銷毀、可以開始就需要停止、能夠恢復就代表會被暫停，知道這樣後再去看左右兩側的流程基本就很清楚了。

## 正文
雖然說明這些環節不是我主要的目的，不過可以為了更清楚的說明後面，這邊還是簡單的帶過一下各個方法的功用。

- onCreate()：他會在 Activity 創建的時候呼叫，所以我們應該在這個方法中完成初始化的動作。
- onStart()：他會在 Activity 由不可鍵便為可見的時候被呼叫。
- onResume()：他會在 Activity 已經準備好要與用戶開始互動的時候被呼叫，此時 Activity 會處於運行的狀態。
- onPause()：他會在系統準備去啟動或是恢復另外一個 Activity 的時候呼叫，通常會在這個方法中去釋放一些消耗 CPU 資源的資源(功能)。
- onStop()：他會在 Activity “不可見“的時候的時候被呼叫。
- onDestroy()：他會在 Activity 被銷毀前的時候呼叫。
- onRestart()：他會在 Activity 由停止狀態變成運行狀態的時候呼叫，換句話說就是他被重新啟動了。

那因為一個 Activity 在 Stop 的狀態下是有可能會被回收的，也就是上途中左側的那條路徑。這個時候心中可能會有一個問題，那就是假設今天有一個 Activity 在運行中暫存了一些內容還要使用，但是因為在 Stop 的狀態下因為記憶體不足而被系統回收掉了，這時候如果他被又要被返回的時候會發生甚麼事情呢？

其實大致上不會有甚麼大問題，我們可以從途中看出來，他會變成從 onCreate() 來恢復，這代表著這個 Aactivity 會被重新創見一次。雖然這樣看起來好像一切正常，不過有沒有注意到我剛剛特別說了他原本有暫存一些內容還要使用，所以這時候問題就出現了，因為這個 Activity 是被重新創建的，也因此這些暫存的內容理所當然的也不在了。那這個時候我們該怎麼做才能在恢復的時候還能去使用我們想要的內容呢？

其實這件事情 android 也有替我們想到，所以我們可以看一下 [android doc](https://developer.android.com/guide/components/activities?hl=zh-tw#SavingActivityState)，會發現 activity 還有提供一個方法 onSaveInstanceState() ，這個方法會需要傳入一個 Bundle 類型的參數來保存數據。那這個 bundle 的使用方式很基本，要傳入兩個參數一個是 key value、一個是要保存的內容。

直接讓我們看一下要怎麼使用：

``` java
@Override
protected void onSaveInstanceState(@NonNull Bundle outState) {
    super.onSaveInstanceState(outState);
    String tempData = "我還會被使用喔 >.0";
    outState.putString("temp", tempData);
}
```

知道了怎麼傳入後，其實怎麼使用也是簡單的，我們稍微觀察一下在 onCreate() 中其實有傳入一個參數，再仔細看一下有沒有發現他跟我們剛剛寫的是一模一樣的東西啊，所以我們只要在 onCreate() 中去呼叫就可以了，是不是很快速方便阿。

``` java
if (savedInstanceState != null) {
	  String data = savedInstanceState.getString("temp");
	  Log.d(TAG, data);
}
```
