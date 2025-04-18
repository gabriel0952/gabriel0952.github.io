---
title: Android 多媒體檔案權限 - 部分存取
author: gabriel0952
date: 2024-05-01 11:33:00 +0800
categories: [Android]
tags: [Android, Kotlin, Permission, Media, Image, Video]
---

## 概述
小小前情提要一下，近幾年 Android 版本更新時總是會去調整各種的權限取用限制，其中更令人煩躁的就是檔案存取的權限。

還記得 Android 10 開始因為導入了 Scoped Storage 的機制，APP 開發時被禁止使用絕對路徑去存取公共儲存空間的資料，讓用戶的隱私有更好的保護，避免被惡意的任意存取裝置上各種檔案。

從那之後我們要寫入或存取相片、影片、音樂等等共用資源的檔案時，通常都改用 MediaStore 來進行操作。到這個時候我們在寫入檔案時是不用進行權限索取的，不過要讀取檔案的話則需要索取 `READ_EXTERNAL_STORAGE` 權限才能夠取用。

還記得那個時候有大量的網路文章、問題都是在詢問或是幫忙解決這個問題，而當時 Google 可能也知道這個改動會影響到大量的應用，而許多應用應該也沒辦法這麼快的完成調整以符合這個規範，因此他們有額外提供一個臨時的屬性 `requestLegacyExternalStorage`。只要將這個屬性設定為 true 那麼 APP 就還是可以透過絕對路徑來存取公共資源的檔案。

後來到了 Android 11 之後 `requestLegacyExternalStorage` 這個屬性也開始不再有用。到了 Android 13 又陸續做了調整，針對 `READ_EXTERNAL_STORAGE` 不能讓使用者細部的去授權各類型檔案進行管控的問題，廢除了這個權限新增了分別針對照片、影片、錄音的 `READ_MEDIA_IMAGES`、`READ_MEDIA_VIDEO`、`READ_MEDIA_AUDIO` 運行時權限提供開發者讓使用者授權。

最後來到這篇的重點，Android 14 希望讓使用者可以更好的根據自己的需求提供 APP 存取特定照片與影片的的功能。

## 多媒體檔案權限 - 部分存取
這個新增的權限是：
``` xml
android.permission.READ_MEDIA_VISUAL_USER_SELECTED
```

我們只需要把他在 AndroidManifest.xml 中進行宣告就可以讓使用者進行授權。
不過要是我們希望可以讓 APP 向下相容到 Android 13、12 甚至夠久以前的版本，勢必我們需要在多加一些宣告以符合各版本的規範：

``` xml
<manifest>
    <!-- Android 12 (API level 32) 以下的版本 -->
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" android:maxSdkVersion="32" />

    <!-- Android 13 (API level 33) 以上的版本 -->
    <uses-permission android:name="android.permission.READ_MEDIA_IMAGES" />
    <uses-permission android:name="android.permission.READ_MEDIA_VIDEO" />
    <!-- 若是有存取聲音檔案的需求可以再加上這個授權 -->
    <!-- <uses-permission android:name="android.permission.READ_MEDIA_AUDIO" /> -->

    <!-- Android 14 (API level 34) 的版本 -->
    <uses-permission android:name="android.permission.READ_MEDIA_VISUAL_USER_SELECTED" />
    ...
</manifest>
```

基本上到這邊就可以符合各系統版本的照片、影片存取存限授權。

接著，相信有索取過 Android 權限的人一定知道我們需要做的事情還不到一半QQ
我們還需要在主程式裡面去寫一些程式碼來向使用者索取相關授權。下面簡單示範一段向使用者索取讀取權限的例子提供參考：

``` kotlin
private val permissionLauncher =
    registerForActivityResult(ActivityResultContracts.RequestMultiplePermissions()) { _ ->
        checkPermissions()
        loadImages()
}

private fun requestPermissions() {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.UPSIDE_DOWN_CAKE) {
        // Android 14 (API level 34)
        permissionLauncher.launch(
            arrayOf(
                READ_MEDIA_IMAGES,
                READ_MEDIA_VIDEO,
                READ_MEDIA_VISUAL_USER_SELECTED
            )
        )
    } else if (Build.VERSION.SDK_INT == Build.VERSION_CODES.TIRAMISU) {
        // Android 13 (API level 33)
        permissionLauncher.launch(
            arrayOf(
                READ_MEDIA_IMAGES,
                READ_MEDIA_VIDEO
            )
        )
    } else {
        // Android 12 (API level 32) 以下
        permissionLauncher.launch(
            arrayOf(
                READ_EXTERNAL_STORAGE
            )
        )
    }
}
```

最後，在向使用者請求完本地的存取權限後，理論上我們還需要去檢查我們對權限請求的結果。
這邊也很粗略的附上一個範例提供參考：

``` kotlin
private fun checkPermissions(): Boolean {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU
        && (ContextCompat.checkSelfPermission(this, READ_MEDIA_IMAGES) == PERMISSION_GRANTED
                || ContextCompat.checkSelfPermission(this, READ_MEDIA_VIDEO) == PERMISSION_GRANTED)) {
        // Android 13 (API level 33) 以上的本地照片和影片讀寫存取權限
        // ... 相應要做的動作
    } else if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.UPSIDE_DOWN_CAKE
        && ContextCompat.checkSelfPermission(this, READ_MEDIA_VISUAL_USER_SELECTED) == PERMISSION_GRANTED) {
        // Android 14 (API level 34) 以上的本地照片和影片讀寫存取權限（部分存取）
        // ... 相應要做的動作
    } else if (ContextCompat.checkSelfPermission(this, READ_EXTERNAL_STORAGE) == PERMISSION_GRANTED) {
        // Android 12 (API level 32) 以下的完整本地檔案讀寫存取權限
        // ... 相應要做的動作
    } else {
        // 未取得本地檔案讀寫存取權限
        // ... 相應要做的動作
    }
    return true
}
```

以上的內容基本上可以參考：
* [官方的介紹文件](https://developer.android.com/about/versions/14/changes/partial-photo-video-access)，裡面有提供更詳細的說明以及一些示例。
* 個人相當推薦的開發者 - 郭霖也有一篇相關的文章 [Android 14新特性，选择性照片和视频访问授权](https://mp.weixin.qq.com/s?__biz=MzA5MzI3NjE2MA==&mid=2650283218&idx=1&sn=6c2cf695f19f3d3ea8d563dd76a56a84&chksm=886cedbdbf1b64ab4541b8925b6e30e7c95e7b085819209d928956f35408b5b95414efc68bf0&scene=178&cur_album_id=1455589563531214850#rd)

程式範例有放在 GitHub 上有需要可供參考：[MediaAccessDemo](https://github.com/gabriel0952/MediaAccessDemo/)
