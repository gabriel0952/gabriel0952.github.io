---
title: Android 中使用 OpenSSL
author: gabriel0952
date: 2023-10-13 12:33:00 +0800
categories: [Android]
tags: [Android, Java, JNI, OpenSSL]
---

## 概述

在正式開始把 OpenSSL 導入 Android 專案前先簡單介紹一下 OpenSSL 還有我們大概會需要做些甚麼。

## OpenSSL

他是一個開源的加密與安全套件，提供了許多功能例如：加密、解密、簽章等等功能，並且支援多種加密協定和多種加密演算法。

因為功能完整，常被運用到許多不同的應用之中，包含網路通信、資料庫加密、數位簽章。

簡單列出常見的使用情境：

1. 加密和解密：OpenSSL 有提供對稱式加密（AES、DES）和非對稱式加密（RSA、ECC），可以用來對數據進行加密保護。
2. 證書管理：OpenSSL 支援證書的生成、簽名、驗證以及存儲。
3. SSL/TLS 實現：OpenSSL 是許多網路應用中實現安全 Socket Layer (SSL) 和 Transport Layer Security (TLS) 協定的重要庫。這些協定用於安全網路通信，例如 HTTPS。
4. SSL/TLS 伺服器和客戶端應用：可以使用 OpenSSL 開發 SSL/TLS 伺服器和客戶端應用程式，以建立安全的通信連接。

## JNI 是甚麼

在說明導入步驟前還需要簡介一個東西，那就是 JNI（Java Native Interface） 在 WIKI 的說明中可以看到：

簡而言之就是當我們需要引用、呼叫一些外部的 library，但他們是 C/C++ 或更底層的組語時，我們就可以透過 JNI 來呼叫他們。

### 導入到 Android 專案中的步驟
1. 匯入 OpenSSL Library 到 project
2. 撰寫 JNI 程式碼，實現 OpenSSL 功能
3. 創建 JNI 接口
4. 呼叫 JNI 接口

## 開始導入

### 1. 匯入 OpenSSL Library 到 project

第一步，我們先將 OpenSSL 放到我們的專案裡面，那我們這邊就先用別人編好的 library 就好了，如果有需要自己編自己的 library 可以再到 [OpenSSL 官網](https://www.openssl.org/) 了解。

我們可以從 [openssl_for_ios_and_android](https://github.com/leenjewel/openssl_for_ios_and_android) 這邊取得編譯好的資源。

然後將所需要的 ABI 對應版本放到 Project 的路徑下，我是放在這邊： `src/main/cpp/${ANDROID_ABI}/`

資料夾結構大概是這樣：

```
- src
    +--- main
    |    +--- cpp
    |    |    +--- arm64-v8a
    |    |    +--- armeabi-v7a
    |    |    +--- x86
    |    |    +--- x86_64
    |    +--- java
    |    +--- res
    +--- test
    +--- androidTest
```

### 2. 撰寫 JNI 程式碼，實現 OpenSSL 功能

接著，我們在 cpp 的目錄下再新增幾個檔案，第一個是 openssl-wrapper.cpp：

``` cpp
#include <string.h>
#include <jni.h>

extern "C" {
    JNIEXPORT jstring JNICALL Java_com_example_openssldemoapp_OpenSSLHelp_stringFromJNI(JNIEnv *env, jclass clazz) {
        return env->NewStringUTF("Hello JNI");
    }
}
```

這邊有幾個地方要稍微注意一下：

1. `extern "C"` 如果原始碼是用 C++ 寫的那就一定要加上
2. `Java_com_example_openssldemoapp_OpenSSLHelp_stringFromJNI` 裡面的 `com_example_openssldemoapp` 是 package，`OpenSSLHelp` 是 class name，最後式函式名。後續會定義一個 native 來實現功能，例如：`public native String stringFromJNI();`

再來，我們需要設定 CMakeLists.txt。因為 Android Studio 是透過 CMake 編譯，我們這邊就把設定檔 CMakeLists 放到 cpp 的目錄下就可以了。

CMakeLists.txt：
``` gradle
cmake_minimum_required(VERSION 3.4.1)

add_library(
    openssl-wrapper SHARED
    openssl-wrapper.cpp
)

target_link_libraries(
    openssl-wrapper
)
```

這邊也簡單說明一下：

1. 第二段 add_library 會將 .cpp/.c/.cc 的文件生成 .a 的靜態 library
2. 第三段 target_link_libraries 會對 add_library 生成的文件進行鏈結

到這邊其實我們已經快要完成了，接著我們來修改一下 build.gradle 的內容，讓編譯 CMake 的行為會被 Android Studio 運行。在 android 的區域加入下面內容：

``` gradle
android {
    ...

    externalNativeBuild {
        cmake {
            path 'src/main/cpp/CMakeLists.txt'
        }
    }
}
```

### 3. 創建 JNI 接口
接著我們就只要在專案主程式的地方新增一個 java 把前面寫的 `Java_com_example_openssldemoapp_OpenSSLHelp_stringFromJNI` C++ 接進來就可以了，所以新增一個 `OpenSSLHelp.java` 然後對應的將 class、function name 進行命名即可。

``` java
package com.example.openssldemoapp;

public class OpenSSLHelp {
   static {
      System.loadLibrary("openssl-wrapper");
   }

   public static native String stringFromJNI();
}
```

### 4. 呼叫 JNI 接口
到這邊我們就可以實際的來呼叫 OpenSSL 的 library 了。我們簡單地回到 MainActivity 呼叫看看效果如何

``` java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    Log.d(TAG, "onCreate: " + OpenSSLHelp.stringFromJNI());
}
```

最後附上範例程式提供參考 [GitHub Link](https://gabriel0952.github.io/gabriel.github.io/)
其中有再多加了一個接口的 stringFromMD5 提供參考
