---
layout: post
title: "BugBazaar APK Walkthrough"
date: 2026-07-28

category: APK

image: /assets/posts/bugbazaar-apk/bugbazaarcover.png

excerpt: BugBazaar APK Çözümü

tags: 
- Mobile Security
- APK

---


# BugBaazar

## 1.Genel Bilgiler

- **Uygulama Adı**: BugBaazar
- **Paket Adı**: “com.bugbaazar”
- **Versiyon**: “1.0"
- **Hash Bilgileri**:
    - MD5: B0151366A3211610FDBDFFAA2061E8AC
    - SHA1: B6C6E2A1749C1AB0F56086CC0B9055B341385039
    - SHA256: C307C7856B270DA40357006EAA763C06BCDC2913D0D0B0BAB7E0B2ED564F5F4C
- **Test Ortamı**:
    - Cihaz/Emülatör: Pixel 3a XL
    - Android Sürümü: 9.0
    - Kullanılan Araçlar: jadx, apktool, frida


## Uygulama İşleyişi

<img src="/assets/posts/bugbazaar-apk/image.png" width="400">

<img src="/assets/posts/bugbazaar-apk/image (1).png" width="400">


- **Giriş / Kayıt:**
    
    Uygulama açıldığında kullanıcıyı bir **Splash Screen** karşılıyor. Ardından kullanıcı **kullanıcı adı / şifre** ile giriş yapabiliyor (`Signin` activity).
    
    Alternatif olarak `ExternalAuthLogin` activity üzerinden harici yetkilendirme (Google, Facebook vb.) destekleniyor olabilir.
    
- **Ana Ekran / Dashboard:**
    
    Giriş yaptıktan sonra kullanıcı ürünleri görüntüleyebiliyor, kategoriler arasında gezinebiliyor ve sepete ürün ekleyebiliyor (`Deeplink` activity’de “cart/add” parametreleri mevcut).
    
- **Kullanıcı Profili:**
    
    `MyProfile` activity ile kullanıcı bilgileri ve geçmiş işlemleri gösteriliyor.
    
    Kullanıcı bilgileri `UserProfileProvider` üzerinden erişilebiliyor.
    
- **Arkadaşına Öner / İletişim:**
    
    Uygulama içerisinde `ReferUs` ve `Contact_us` activity’leri mevcut. Kullanıcı arkadaşına davet gönderebiliyor veya destek ekibine ulaşabiliyor.
    
- **Adres ve Ödeme:**
    
    `AddressContentProvider` ile kullanıcı adresleri yönetiliyor.
    
    Ödeme altyapısı için **Razorpay** entegrasyonu mevcut (`RzpTokenReceiver`).
    
- **Bildirimler ve Arka Plan Servisleri:**
    - Firebase push bildirimleri destekleniyor (`FirebaseMessagingService`).
    - Uygulama açılmasa bile arka planda servisler (`SMSService`, `WorkManager`, `Evernote JobService`) çalışabiliyor.
    - `RECEIVE_BOOT_COMPLETED` izni sayesinde cihaz yeniden başlatıldığında servisler otomatik başlatılıyor.
- **Ekstra İşlevler:**
    - SMS tabanlı doğrulama ve OTP yakalama mekanizmaları (`SMSService`).
    - Kullanıcı rehberi erişimi (`MyContactsProvider`).
    - Dosya erişimi ve paylaşımı (`MANAGE_EXTERNAL_STORAGE`).

## 3. Manifest Analizi

- **İzinler**

| İzin | Değerlendirme |
| --- | --- |
| `INTERNET` | Normal – uygulamanın ağ üzerinden iletişim kurması için gerekli |
| `READ_CONTACTS` | Kritik – kullanıcı rehberine erişim  |
| `READ_PHONE_STATE` | Kritik – cihaz bilgilerine erişim |
| `SEND_SMS`, `RECEIVE_SMS` | Kritik – SMS üzerinden işlem yapılabilir |
| `MANAGE_EXTERNAL_STORAGE` | Kritik – tüm dosya sistemine erişim |
| `REQUEST_INSTALL_PACKAGES` | Kritik – uygulama dışarıdan APK yükleyebilir |
| `RECEIVE_BOOT_COMPLETED` | Normal/Kritik – cihaz açıldığında başlatılır |
| `POST_NOTIFICATIONS` | Normal – bildirim göndermek için |
- **Exported Activity / Service / Receiver**

| Bileşen | Tür | Exported | Risk |
| --- | --- | --- | --- |
| `com.BugBazaar.ExternalAuthLogin` | Activity | true | Yetkisiz erişimle oturum açma akışı tetiklenebilir |
| `com.BugBazaar.ui.Deeplink` | Activity | true | Deeplink üzerinden kötü amaçlı veri gönderilebilir |
| `com.BugBazaar.provider.UserProfileProvider` | ContentProvider | true | Kullanıcı profili dışarıya sızdırılabilir |
| `com.BugBazaar.ui.addresses.AddressContentProvider` | ContentProvider | true | Adres bilgileri dışarı açılabilir |
| `com.BugBazaar.ui.components.SMSService` | Service | true | Yetkisiz SMS işlevleri tetiklenebilir |
| `com.BugBazaar.ui.components.CustomBR` | BroadcastReceiver | true | Broadcast Injection riski |
| `com.razorpay.RzpTokenReceiver` | BroadcastReceiver | true | Harici uygulamalar tarafından tetiklenebilir |
- **Debuggable Flag**

`android:debuggable="true"` → Riskli (production uygulamada olmamalı).

- **Backup Flag**

`android:allowBackup="true"` → Kullanıcı verileri ADB yedeği ile alınabilir.

## 4.Bulgular

- **Insecure Logging**

<img src="/assets/posts/bugbazaar-apk/image (2).png" width="700">

- **Insecure Storage**

<img src="/assets/posts/bugbazaar-apk/image (3).png" width="700">

- **Improper Input Validation**

<img src="/assets/posts/bugbazaar-apk/image (4).png" width="400">

<img src="/assets/posts/bugbazaar-apk/image (5).png" width="400">

Buradaki zafiyet, email kısmına @ gerekmeden input değerini kabul etmesidir.

- **WebView Ayarları**

<img src="/assets/posts/bugbazaar-apk/image (6).png" width="700">

```java
webView.getSettings().setJavaScriptEnabled(true); -> xss saldırılarına açık
webView.getSettings().setAllowFileAccess(true); -> file:// ile lokal dosyaları okumaya açık
webView.getSettings().setAllowUniversalAccessFromFileURLs(true); -> file:// ile açılan sayfaya request atmaya açık
```

- **Arbitrary File Read (path traversal)**

<img src="/assets/posts/bugbazaar-apk/image (7).png" width="700">

- Kullanıcı `http://…/local_cache/XYZ` gibi bir şey çağırırsa:
    - Uygulama `getCacheDir()` içine `XYZ` adında bir dosya bakıyor.
    - Dosya varsa → `FileInputStream` ile okunup WebView’e dönüyor.
    - Dosya yoksa → `url.toString().replace("/local_cache", "")` ile dış URL’den indiriliyor, cache’e kaydediliyor ve tekrar okunuyor.

Buradaki sıkıntı: path kontrolü çok zayıf.

- Sadece `lastPathSegment` kullanılıyor.
- Yani `"../../../../data/data/com.BugBazaar/shared_prefs/UserAuthSave.xml"` gibi manipülasyonlar yapılıp uygulamanın local storage’ındaki dosyalar okunabilir.

- **WebView**

```jsx
if (getIntent().getExtras() != null) {
    if (getIntent().hasExtra(AppConstants.KEY_WEBVIEW_URL)) {
        String string = getIntent().getExtras().getString(AppConstants.KEY_WEBVIEW_URL);
        this.webViewUrl = string;
        startWebView(string);
        return;
    }
}
startdefaultwebview(AppConstants.Terms_Conditions_URL);
```

intent olarak bir web url alıyor. Almışsa startWebView() ile yükleniyor. 

```powershell
adb shell am start -n com.BugBazaar/.ui.TermsAndConditionsActivity --es webViewUrl "https://google.com"
```

<img src="/assets/posts/bugbazaar-apk/image (8).png" width="400">

- **XSS**

Aynı şekilde intent üzerinden xss açığı da oluşturulabilir.

```powershell
adb shell am start -n com.BugBazaar/.ui.TermsAndConditionsActivity --es webViewUrl "javascript:alert%28%27Hello%27%29"
```
<img src="/assets/posts/bugbazaar-apk/image (9).png" width="400">


- **SQL injection**

<img src="/assets/posts/bugbazaar-apk/image (10).png" width="700">

input kısmına ‘ or ‘1’=’1’ girince home page de ki product kısmı geliyor ekrana.

<img src="/assets/posts/bugbazaar-apk/image (11).png" width="400">

<img src="/assets/posts/bugbazaar-apk/image (12).png" width="400">


- **Content Provider SQL Injection in Address**

<img src="/assets/posts/bugbazaar-apk/image (13).png" width="700">

exported= true olduğu için uygulamanın telefonun contacts’ine erişim izni veriyor.

<img src="/assets/posts/bugbazaar-apk/image (14).png" width="700">

Bugbazaar uygulamasından contacts’e erişim sağlanabiliyor.

<img src="/assets/posts/bugbazaar-apk/image (15).png" width="700">

Bu kodta ise, query çağırıldığında cihazın rehberine gidiyor.

SQL Injection tetiklenebilir:

```java
@Override
public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder) {
    return getContext().getContentResolver().query(
        ContactsContract.CommonDataKinds.Phone.CONTENT_URI,
        projection,
        selection,
        selectionArgs,
        sortOrder
    );
}
```

- Burada `selection` ve `sortOrder` dışarıdan geliyor.
- Ama hiç filtrelenmeden direkt olarak **ContactsContract sorgusuna** gidiyor.
- `ContactsContract` aslında altında SQLite kullanıyor.

```java
Cursor c = getContentResolver().query(
    Uri.parse("content://com.bugbazaar.mycontacts/contacts"),
    null,
    "1=1 OR 1=1",   
    null,
    null
);

```

- **Data insertion via insecure Content Provider in Address**

```java
private static final String AUTHORITY = "com.BugBazaar.UserProfile";
private static final String BASE_PATH = "user_profiles";
public static final Uri CONTENT_URI = Uri.parse("content://com.BugBazaar.UserProfile/user_profiles");
```

Bu satırlar, ContentProvider’ın adresini belirliyor.

```java
private static final String DATABASE_NAME = "UserProfileDatabase";
private static final String TABLE_NAME = "user_profiles";
```

Provider, cihazda `UserProfileDatabase` adlı bir SQLite veritabanı açıyor:

<img src="/assets/posts/bugbazaar-apk/image (16).png" width="900">

<img src="/assets/posts/bugbazaar-apk/image (17).png" width="900">

Bu method veri okumak için kullanılıyor. Yani başka bir uygulamanın içerisinde,

```java
getContentResolver().query(
   Uri.parse("content://com.BugBazaar.UserProfile/user_profiles"),
   null, null, null, null
);
```

bu kod çağırılırsa tüm kullanıcı bilgileri çekilebilir.

<img src="/assets/posts/bugbazaar-apk/image (18).png" width="700">

Bu methodta yeni kullanıcı eklemek için kullanılır. Herhangi bir izin/onay alınmadan database’e yeni kullanıcı eklenebilir.

<img src="/assets/posts/bugbazaar-apk/image (19).png" width="700">

Bu method ise mevcut kayıtları değiştirmek için kullanılabilir.

- **Unrestricted file upload**

<img src="/assets/posts/bugbazaar-apk/image (20).png" width="400">

<img src="/assets/posts/bugbazaar-apk/image (21).png" width="400">

Buradaki zafiyet, profil fotoğrafı olarak sadece jpeg,png olan uzantılar değilde tüm uzantılı dosyaların yüklenebilmesidir.

- **Misconfigured firebase's firestore**

<img src="/assets/posts/bugbazaar-apk/image (22).png" width="700">

<img src="/assets/posts/bugbazaar-apk/image (23).png" width="700">

<img src="/assets/posts/bugbazaar-apk/image (24).png" width="700">


[https://firebasestorage.googleapis.com/v0/b/[PROJECT ID].appspot.com/o/](https://firebasestorage.googleapis.com/v0/b/bugbazaar-cb1a1.appspot.com/o/)

<img src="/assets/posts/bugbazaar-apk/image (25).png" width="700">

https://firebasestorage.googleapis.com/v0/b/bugbazaar-cb1a1.appspot.com/o/poc.txt?alt=media

<img src="/assets/posts/bugbazaar-apk/image (26).png" width="700">

Buradaki zafiyet firebasede storage okunabilirliği.  Normalde Firebase Storage’daki dosyalar, auth (kimlik doğrulama) ile korunur.  Herhangi bir kullanıcı login olmadan link ile dosyayı görebiliyorsa, bu yanlış yapılandırılmış izinler demektir.

### Easy Level

- **Root Detection Bypass**

<img src="/assets/posts/bugbazaar-apk/image (27).png" width="400">


```jsx
Java.perform(function () {
    var rootd = Java.use("com.scottyab.rootbeer.RootBeer");

    rootd.isRooted.implementation = function () {
        return false;
    };
});
```

### Intermittent Level

chechdetect() methodunun çalışması engellendi.

- Emulator Detection Bypass
- ADB Detection Bypass
- Frida Detection Bypass

<img src="/assets/posts/bugbazaar-apk/image (28).png" width="400">


```jsx
Java.perform(function() {
    var EmulatorCheck = Java.use("com.bug.hook.emultorcheck");

    // Parametresiz overload
    EmulatorCheck.isEmulator.overload().implementation = function() {
        return false;
    };

    // Context parametreli overload
    EmulatorCheck.isEmulator.overload('android.content.Context').implementation = function(ctx) {
        return false;
    };

    // AdbEnabled1(context) override
    var AdbEnabled = Java.use("com.bug.hook.AdbEnabled");
    AdbEnabled.AdbEnabled1.implementation = function(ctx) {
        return false;
    };

    // checkfrida() override
    var CheckDetect = Java.use("com.bug.hook.checkdetect");
    CheckDetect.checkfrida.implementation = function() {
        return false;
    };

    // Alert dialog boşalt
    var AlertDialogManager = Java.use("com.BugBazaar.utils.AlertDialogManager");
    AlertDialogManager.showRootedDeviceAlert.implementation = function(ctx, msg) {
        return;
    };
});

```

Kaynak: [BugBazaar GitHub Repository](https://github.com/payatu/BugBazaar)