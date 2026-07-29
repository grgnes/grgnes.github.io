---
layout: post
title: "Owasp Uncrackable Level 2 APK Walkthrough"
date: 2025-07-20

category: APK

image: /assets/posts/uncrackable1/owasp_mas_header.png

excerpt: Owasp Uncrackable Level 2 APK Çözümü

tags: 
- Mobile Security
- APK

---

Cihaza Kurulum:

<img src="/assets/posts/uncrackable2/image.png" width="700">

Uygulamayı statik olarak analiz ettiğimde,

<img src="/assets/posts/uncrackable2/image (1).png" width="400">

Cihazım rootlu olduğu için, uygulamayı sonlandıracağına dair bir uyarı mesajı alıyorum.

Uygulamayı statik olarak analiz etmek ve kaynak koda bakmak için önce apk’yı JADX’te inceliyorum.

<img src="/assets/posts/uncrackable2/image (2).png" width="700">

AndroidManifest.xml koduna baktığımda, 1 adet activity sınıfı (MainActivity) var.

MainActivity class’ını incelemeye başladığımda,

<img src="/assets/posts/uncrackable2/image (3).png" width="700">

kodun başında Native C/C++ kütüphanesi yükleniyor.

<img src="/assets/posts/uncrackable2/image (4).png" width="700">

Devamında güvenlik ihlali tespit edildiğinde "This is unacceptable. The app is now going to exit.” mesajını gösterip uygulamayı kapatıyor.

<img src="/assets/posts/uncrackable2/image (5).png" width="700">

Daha sonra bir Native Init methodu ile libfoo.so kütüphanesinin içindeki C fonksiyonunu çağırır.

<img src="/assets/posts/uncrackable2/image (6).png" width="700">

onCreate() methodu ile uygulamayı başlatılır ve sırayla;

```java
protected void onCreate(Bundle bundle) {
```

- init() fonksiyonu ile native kodunu çağrılır. Muhtemelen şifreleme anahtarları hazırlanır,

```java
init();
```

- if kontrolü ile 3 farklı root kontrolü yapılır,

```java
if (b.a() || b.b() || b.c()) {
    a("Root detected!");
}
```

herhangi biri true dönerse, "Root detected!" uyarısı gösterilir ve uygulama kapanır.

b class’ına bakıcak olursam 3 farklı tespit yönteminin detaylarını görüyorum,

```java
public class b {
    public static boolean a() {
        for (String str : System.getenv("PATH").split(":")) {
            if (new File(str, "su").exists()) {
                return true;
            }
        }
        return false;
    }

    public static boolean b() {
        String str = Build.TAGS;
        return str != null && str.contains("test-keys");
    }

    public static boolean c() {
        for (String str : new String[]{"/system/app/Superuser.apk", "/system/xbin/daemonsu", "/system/etc/init.d/99SuperSUDaemon", "/system/bin/.ext/.su", "/system/etc/.has_su_daemon", "/system/etc/.installed_su_daemon", "/dev/com.koushikdutta.superuser.daemon/"}) {
            if (new File(str).exists()) {
                return true;
            }
        }
        return false;
    }
}
```

1. a() methodunda Path kontrolü yapıyor,

```java
public static boolean a() {
    for (String str : System.getenv("PATH").split(":")) {
        if (new File(str, "su").exists()) {
            return true;
        }
    }
    return false;
}
```

Path environment variable’ı alınır, her dizininde su binary’si aranır. Eğer bulunuyorsa true döner(root var), bulunmuyorsa false döner.

1. b() methodunda ise Build Tags kontrolü yapıyor,

```java
public static boolean b() {
    String str = Build.TAGS;
    return str != null && str.contains("test-keys");
}
```

önce Build.TAGS sistem özelliklerini kontrol eder, eğer test-keys içeriyorsa true, içermiyorsa false döner.

1. c() methodunda ise root dosyası veya dizin kontrolü yapıyor,

```java
public static boolean c() {
    for (String str : new String[]{
        "/system/app/Superuser.apk",
        "/system/xbin/daemonsu",
        "/system/etc/init.d/99SuperSUDaemon",
        "/system/bin/.ext/.su",
        "/system/etc/.has_su_daemon",
        "/system/etc/.installed_su_daemon",
        "/dev/com.koushikdutta.superuser.daemon/"
    }) {
        if (new File(str).exists()) {
            return true;
        }
    }
    return false;
}
```

SuperSu veya superuser gibi root yönetim uygulamalarının dosyalarını arar, herhangi biri bulunuyorsa true, bulunmuyorsa false döner.

- Debug modu kontrolü yapılır,

```java
if (a.a(getApplicationContext())) {
    a("App is debuggable!");
}
```

APK'nın debuggable modda build edilip edilmediğini kontrol eder. Debug modundaysa uygulama kapanır.

- AsyncTask kodu ile debugger algılama işlemi başlatılır,

```java
new AsyncTask<Void, String, String>() {
    public String doInBackground(Void... voidArr) {
        while (!Debug.isDebuggerConnected()) {
            SystemClock.sleep(100L);
        }
        return null;
    }
    
    public void onPostExecute(String str) {
        MainActivity.this.a("Debugger detected!");
    }
}.execute(null, null, null);
```

**Nasıl çalışır:**

- Arka planda sürekli çalışan bir thread başlatılır
- Her 100ms'de bir `Debug.isDebuggerConnected()` kontrol edilir
- Debugger bağlanırsa döngü kırılır ve `onPostExecute()` çalışır
- Uygulama "Debugger detected!" mesajı ile kapanır

En sonda da şifre kontrol nesnesi oluşturulur ve kullanıcı arayüzü gösterilir.

```java
this.m = new CodeCheck();
super.onCreate(bundle);
setContentView(R.layout.activity_main);
```

Bu kısımı görüntüleyebilmek için önce root kontrollerini frida ile bypass etmem gerekiyor. 

Öncellikle Java kodunda yapılan bir root kontrolü olduğu için Java.perform() kullanarak kodu başlatıyorum:

```java
Java.perform(function() {
```

Sonra sırasıyla önce b class’ının kontrollerini false yaparak kaldırıyorum:

```java
    var b = Java.use("sg.vantagepoint.a.b");
    b['a'].implementation = function () {
        return false;
    }
    b['b'].implementation = function () {
        return false;
    }
    b['c'].implementation = function () {
        return false;
    }
```

Sonra debug modu kontrolünü kaldırıyorum:

```java
    var a = Java.use("sg.vantagepoint.a.a");
    a['a'].implementation = function (context) {
        return false;
    }
```

En sonda da AsyncTask kodu ile debugger algılama işlemi kaldırıyorum:

```java
    var Debug = Java.use("android.os.Debug");
    Debug.isDebuggerConnected.implementation = function () {
        return false;
    }
```

Yazdığım kodu frida ile enjekte edebilmek için önce fridayı çalıştırıyorum:

<img src="/assets/posts/uncrackable2/image (7).png" width="700">

Fridayı başlattıktan sonra yazdığım scripti çalıştırıyorum:

<img src="/assets/posts/uncrackable2/image (8).png" width="700">

Ve tüm kontrolleri kaldırdıktan sonra uygulama açılıyor.

<img src="/assets/posts/uncrackable2/image (9).png" width="400">

Benden istenilen Secret String’i bulmak için MainActivity kodunun devamını incelemeye devam ediyorum.

```java
    public void verify(View view) {
        String str;
        String obj = ((EditText) findViewById(R.id.edit_text)).getText().toString();
        AlertDialog create = new AlertDialog.Builder(this).create();
        if (this.m.a(obj)) {
            create.setTitle("Success!");
            str = "This is the correct secret.";
        } else {
            create.setTitle("Nope...");
            str = "That's not it. Try again.";
        }
        create.setMessage(str);
        create.setButton(-3, "OK", new DialogInterface.OnClickListener() { // from class: sg.vantagepoint.uncrackable2.MainActivity.3
            @Override // android.content.DialogInterface.OnClickListener
            public void onClick(DialogInterface dialogInterface, int r2) {
                dialogInterface.dismiss();
            }
        });
        create.show();
    }
}
```

Verify() methodunda bir şifre kontrolü yapıldığını ve bu şifre kontrolünün CodeCheck sınıfının a() methodunda yapıldığı anlaşılıyor.

CodeCheck sınıfında baktığımda,

```java
public class CodeCheck {
    private native boolean bar(byte[] bArr);

    public boolean a(String str) {
        return bar(str.getBytes());
    }
}
```

a() methodunda şifre kontrolü native kod (libfoo.so) içinde yapılıyor. 

Apktool ile apk’yı decompile edip .so dosyasını erişiyorum:

```java
> java -jar .\apktool_2.11.1.jar d .\UnCrackable-Level2.apk
I: Using Apktool 2.11.1 on UnCrackable-Level2.apk with 8 threads
I: Baksmaling classes.dex...
I: Loading resource table...
I: Decoding file-resources...
I: Loading resource table from file: C:\Users\AppData\Local\apktool\framework\1.apk
I: Decoding values */* XMLs...
I: Decoding AndroidManifest.xml with resources...
I: Regular manifest package...
I: Copying original files...
I: Copying lib...
I: Copying unknown files...
```

lib dosyasının içindeki .so dosyalarından kendi cihazımın mimarisine uygun olan .so dosyasını Ghidra ile açılıyorum.

<img src="/assets/posts/uncrackable2/image (10).png" width="700">

<img src="/assets/posts/uncrackable2/image (11).png" width="600">

CodeCheck isimli fonksiyonu bulup içine bakıyorum.

<img src="/assets/posts/uncrackable2/image (12).png" width="600">

Ghidra çıktısından secret key'i buldum.

<img src="/assets/posts/uncrackable2/image (13).png" width="700">

```java
builtin_strncpy(local_38,"Thanks for all the fish",0x18);
```

Native kod, girilen stringi **"Thanks for all the fish"** ile karşılaştırıyor.

<img src="/assets/posts/uncrackable2/image (14).png" width="400">


Script: [Yazdığım Script](https://github.com/grgnes/Owasp-Uncrackable-Levels)

Kaynak: [Owasp Uncrackable 2 GitHub Repository](https://github.com/OWASP/mastg/tree/master/Crackmes/Android/Level_02)
