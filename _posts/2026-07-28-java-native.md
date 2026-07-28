---
layout: post
title: "Java'da Native Kod Kullanımı ve JNI Temelleri"
date: 2026-07-28

category: Research

image: /assets/posts/java101/coverj.png

excerpt: Java'da native kod kullanımını, JNI (Java Native Interface) yapısını ve Java ile C/C++ kodlarının nasıl haberleştiğini örneklerle inceledim.

tags: 
  - Java
  - JNI
  - Android
  - Native
  - Mobile Security
  - Reverse Engineering
---


## 1-Java kodu yazmak

```java
package com.example.karslan;

public class deneme {
    // JNI üzerinden C++/C fonksiyonunu çağıracağız
    public native String stringFromJNI();

    static {
        System.loadLibrary("native-lib"); // native-lib.so kütüphanesini yükle
    }
}
```

- `public native String stringFromJNI();` — Bu, Java'dan C/C++ tarafındaki fonksiyona bağlanmak için *native* fonksiyon bildirimi.
- `System.loadLibrary("native-lib");` — Android cihazda ya da emülatörde `libnative-lib.so` dosyasını yükler (native kütüphanemiz).

## 2- Java kodunu derleyip JNI header dosyasını oluşturmak

```java
javac -h app/src/main/cpp app/src/main/java/com/example/karslan/deneme.java
```

Bu komut hem java dosyasını derler → .class ı oluşturur,

hemde JNI için “com_example_karslan_deneme.h” header dosyasını cpp klasörünün içine oluşturur.

neden header dosyası oluşturur ? 

çünkü header dosyası C/C++ tarafında java fonksiyonun imzasını bildirir yani,

```java
public native String stringFromJNI();
```

burdaki “native” keyini kullanıma açar.

`native`, bu fonksiyonun gövdesinin Java'da değil, C veya C++ gibi dış (native) bir dilde yazılacağını söyler.

C/C++ dosyan, Java’daki `stringFromJNI()` fonksiyonuna tam olarak nasıl bağlanacağını bu header dosyasından öğrenir.

## 3- C/C++ native kodu yazmak

```java
#include <jni.h>
#include "com_example_karslan_deneme.h"

JNIEXPORT jstring JNICALL Java_com_example_karslan_deneme_stringFromJNI(JNIEnv* env, jobject obj) {
    return env->NewStringUTF("Merhaba JNI'den!");
}
```

- `Java_com_example_karslan_deneme_stringFromJNI` fonksiyonu, Java tarafındaki native metodun imzasına uygun isimlendirilir.
- `env->NewStringUTF` fonksiyonu, C stringini Java String'ine dönüştürür.

## 4- Native kütüphaneyi derleme(ndk)

- CMakeList.txt ve build.gradle uygun şekilde ayarlanır
- android studio doğrudan build yapınca libnative-lib.so oluşturulur ve lib klasörüne gömülür.

## 5- Java tarafında native fonksiyonu çağırmak

```java
package com.example.karslan;

import android.os.Bundle;
import android.widget.TextView;
import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        TextView tv = new TextView(this);
        deneme d = new deneme();
        tv.setText(d.stringFromJNI()); // C++ fonksiyonu çağrılır
        setContentView(tv);
    }
}
```

mainactivityde deneme classından d nesnesi oluşturarak stringFromJNI() fonksiyonu çağrılır.

bu fonksiyon java da tanımlanmış ama gövdesi c kodundan çağrılıyor