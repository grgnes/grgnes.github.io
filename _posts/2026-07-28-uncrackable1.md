---
layout: post
title: "Owasp Uncrackable Level 1 APK Walkthrough"
date: 2026-07-28

category: APK

image: /assets/posts/uncrackable1/owasp_mas_header.png

excerpt: Owasp Uncrackable Level 1 APK Çözümü

tags: 
- Mobile Security
- APK

---

Cihaza kurulum:

<img src="/assets/posts/uncrackable1/image.png" width="700">

Cihaza kurduktan sonra uygulamayı açtığımda:

<img src="/assets/posts/uncrackable1/image (1).png" width="400">

Dinamik analiz sırasında uygulamayı çalıştırınca aşağıdaki uyarı ile karşılaştım:

**“Root detected! This is unacceptable. The app is now going to exit.”**

Bu, uygulamanın runtime ortamında root kontrolü yaptığını ve root bulunduğunda uygulamayı sonlandırdığını gösterir.

Root tespiti, tersine mühendislik ve runtime hook’lama gibi analizleri engellemek için yaygın bir önlemdir. Bu uyarıyı gördükten sonra Java kaynak kodunu (JADX çıktısını) açıp root kontrolünü yapan fonksiyonu ve tetiklenme koşullarını aramaya başladım.

Apk’yı JADX ile açıp kodlara baktım.

<img src="/assets/posts/uncrackable1/image (2).png" width="700">

Tek activity ”sg.vantagepoint.uncrackable1.MainActivity”.

MainActivity kodunu incelemeye başladığımda,

<img src="/assets/posts/uncrackable1/image (3).png" width="700">

onCreate ve verify methodları var:

onCreate methodu 3 farklı root kontrolü (`c.a()`, `c.b()`, `c.c()`)  ve 1 debug (`b.a()`)  kontrolü yapıyor.

<img src="/assets/posts/uncrackable1/image (4).png" width="700">

Herhangi biri “true” dönerse uygulama kapanıyor.

c.java() sınıfına baktığımızda:

<img src="/assets/posts/uncrackable1/image (5).png" width="700">

c.a() - Su Binary’yi arıyor:

```java
for (String str : System.getenv("PATH").split(":")) {
    if (new File(str, "su").exists()) {
        return true;
    }
}
```

Path’teki tüm dizinlerde “su” dosyası var mı diye bakıyor. Root'lu cihazlarda genelde `/system/xbin/su` gibi yerlerde bu dosya bulunur.

c.b() - Build tag kontrolü yapıyor:

```java
return Build.TAGS != null && Build.TAGS.contains("test-keys");
```

Custom ROM'lar genelde "test-keys" ile imzalanır, bunu kontrol ediyor.

c.c() - Bilinen root dosyalarını arıyor:

```java
public static boolean c() {
        for (String str : new String[]{
        "/system/app/Superuser.apk", 
        "/system/xbin/daemonsu", 
        "/system/etc/init.d/99SuperSUDaemon", 
        "/system/bin/.ext/.su", 
        "/system/etc/.has_su_daemon", 
        "/system/etc/.installed_su_daemon", 
        "/dev/com.koushikdutta.superuser.daemon/"}) {
            if (new File(str).exists()) {
                return true;
            }
        }
```

SuperSU, Magisk gibi popüler root araçlarının bıraktığı izleri arıyor. Eğer bulursa “true” döndürüp, uygulamayı kapatıyor.

Frida ile root detection mekanizmasını kaldırmamız gerekiyor. Öncelikle fridayı başlatıyoruz:

<img src="/assets/posts/uncrackable1/image (5).png" width="700">

Fridayı başlattıktan sonra scirpti yazmaya başlıyoruz. 

```java
Java.perform(function () {

    var c = Java.use("sg.vantagepoint.a.c");
    ..
});
```

Java kodunu ve kullandığımız class’ı çağırıyoruz. Sonra sıra sıra “return true;” dönen tüm fonskiyonları “false” olucak şekilde ayarlıyoruz. Çünkü eğer return değeri true olarak dönerse, cihazın rootlu olduğunu anlayıp uygulamayı sonlandırıyor. Biz javascript scripti kullanarak frida ile araya girdiğimizde bu return değerini false olarak ayarlıyoruz ki root detection mekanizmasını manipüle edebilelim.

```java
Java.perform(function () {

    var c = Java.use("sg.vantagepoint.a.c");

    c.a.implementation = function () { // su binary kontrolü yok
        return false;
    };

    c.b.implementation = function () { // build tag kontrolü yok
        return false;
    };

     c.c.implementation = function () { // root dosya kontrolü yok
        return false;
    };
});
```

Scriptimizi yazdıktan sonra çalıştırıyoruz:

```java
frida -U -f owasp.mstg.uncrackable1 -l .\bypass.js
```
<img src="/assets/posts/uncrackable1/image (6).png" width="700">

Çalıştırdıktan sonra root detection mekanizması kalkıyor:

<img src="/assets/posts/uncrackable1/image (7).png" width="700">

Root detection mekanizması kalktıktan sonra bir secret string girmemiz gerekiyor.

MainActivitydeki diğer method olan verify methoduna baktığımızda, kullanıcının input alanına yazdığı stringi alıp `a.a()` metoduyla doğruluyor ve sonuca göre başarılı veya başarısız mesajı içeren bir dialog gösteriyor:

<img src="/assets/posts/uncrackable1/image (8).png" width="400">

a.java() sınıfına baktığımızda:

<img src="/assets/posts/uncrackable1/image (9).png" width="400">

a.a(String str) :

```java
    public static boolean a(String str) {
        byte[] bArr;
        byte[] bArr2 = new byte[0];
        try {
            bArr = sg.vantagepoint.a.a.a(b("8d127684cbc37c17616d806cf50473cc"), Base64.decode("5UJiFctbmgbDoLXmpL12mkno8HT4Lv8dlat8FxR2GOc=", 0));
        } catch (Exception e) {
            Log.d("CodeCheck", "AES error:" + e.getMessage());
            bArr = bArr2;
        }
        return str.equals(new String(bArr));
    }
```

Kullanıcının girdiği stringi, AES ile decrypt edilmiş gizli string ile karşılaştırır ve eşitse `true` döner.

b(String str) :

```java

    public static byte[] b(String str) {
        int length = str.length();
        byte[] bArr = new byte[length / 2];
        for (int r2 = 0; r2 < length; r2 += 2) {
            bArr[r2 / 2] = (byte) ((Character.digit(str.charAt(r2), 16) << 4) + Character.digit(str.charAt(r2 + 1), 16));
        }
        return bArr;
    }
```

Hex string'i byte array'e çevirir (örneğin "8d12" → byte array).

yani özetle secret, Base64 + AES ile şifrelenmiş olarak APK'da saklanıyor. Kullanıcı doğru secret'i girerse `true` döner ve "Success!" mesajı görünür.

Frida ile decrypt edilmiş halini yakalayarak bulabiliriz:

```java
    var a = Java.use("sg.vantagepoint.uncrackable1.a");
    
    a.a.implementation = function(str) {
        console.log("Girilen input: " + str);
        var result = this.a(str);
    
        var AESDecrypt = Java.use("sg.vantagepoint.a.a");
        var b = this.b("8d127684cbc37c17616d806cf50473cc");
        var Base64 = Java.use("android.util.Base64");
        var encrypted = Base64.decode("5UJiFctbmgbDoLXmpL12mkno8HT4Lv8dlat8FxR2GOc=", 0);
        
        var decrypted = AESDecrypt.a(b, encrypted);
        var secret = "";
        for(var i = 0; i < decrypted.length; i++) {
            secret += String.fromCharCode(decrypted[i]);
        }
        
        console.log("Secret: " + secret);
        return result;
    };
```

```java
  var a = Java.use("sg.vantagepoint.uncrackable1.a");
```

- `sg.vantagepoint.uncrackable1.a` adlı Java sınıfını Frida üzerinden erişilebilir hale getiriyoruz bu sınıfın methodlarına müdahale edebilmek için.

```java
a.a.implementation = function(str) {
        console.log("Girilen input: " + str);
        var result = this.a(str);
```

- a sınıfındaki a() methodunu hooklayıp yeniden tanımlıyoruz. Buradaki str parametresi, uygulamanın kullanıcıdan aldığı input değerini temsil ediyor. Bu kullanıcının girdiği input değerini ekrana bastırıp, daha sonra orijinal fonkiyonu(a.(str)) yeniden çalıştırıyoruz ve sonusun değerini result değişkenine atıyoruz. Buradaki sonuç değeri: true/false

```java
var AESDecrypt = Java.use("sg.vantagepoint.a.a");
var b = this.b("8d127684cbc37c17616d806cf50473cc");
var Base64 = Java.use("android.util.Base64");
var encrypted = Base64.decode("5UJiFctbmgbDoLXmpL12mkno8HT4Lv8dlat8FxR2GOc=", 0);
```

- `var AESDecrypt = Java.use("sg.vantagepoint.a.a");` — AES çözücü sınıfını Frida üzerinden erişilebilir hale getiriyor
- `var b = this.b("8d127684cbc37c17616d806cf50473cc");` — verdiğin hex stringi çağırılan b(String) metoduyla byte[] (AES anahtarı) haline çeviriyor.
- `var Base64 = Java.use("android.util.Base64");` — Android’in Base64 yardımcı sınıfını kullanıma açıyor.
- `var encrypted = Base64.decode("5UJiFctbmgbDoLXmpL12mkno8HT4Lv8dlat8FxR2GOc=", 0);` — Base64 kodlu diziyi byte[] olarak çözüyor.

```java
var decrypted = AESDecrypt.a(b, encrypted);
        var secret = "";
        for(var i = 0; i < decrypted.length; i++) {
            secret += String.fromCharCode(decrypted[i]);
        }
        console.log("Secret: " + secret);
```

- `AESDecrypt.a(b, encrypted)` → `b` anahtarıyla `encrypted` verisini çözüyor.
- `for` döngüsünde, çıkan baytları tek tek karaktere çeviriyor.
- Log olarak ekrana bastırıyor.

```java
 return result;
```

- En sonda da result değerimiz false döndüğü için "That's not it. Try again." mesajını alıyoruz.

Sonuç olarak, konsolda hem yanlış verdiğimiz input hem de secret ortaya çıkmış oluyor.

<img src="/assets/posts/uncrackable1/image (10).png" width="700">

<img src="/assets/posts/uncrackable1/image (11).png" width="700">

<img src="/assets/posts/uncrackable1/image (12).png" width="700">

Ya da bir başka yöntemde, verilen her input’u true olarak döndürebiliriz:

```java
    var a = Java.use("sg.vantagepoint.uncrackable1.a");
    
    a.a.implementation = function(str) {
        console.log("Input: " + str);
        return true;  
    }; 
```

<img src="/assets/posts/uncrackable1/image (13).png" width="400">


Script: [Yazdığım Script](https://github.com/grgnes/Owasp-Uncrackable-Levels)

Kaynak: [Owasp Uncrackable 2 GitHub Repository](https://github.com/OWASP/mastg/tree/master/Crackmes/Android/Level_01)
