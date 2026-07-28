---
layout: post
title: "Root Detection"
date: 2026-07-28

category: Research

image: /assets/posts/root-detection/rootd.png

excerpt: Bu yazıda Android uygulamalarında root detection mekanizmalarının nasıl çalıştığını, yaygın tespit yöntemlerini ve güvenlik açısından neden kullanıldığını inceledim.

tags: 
- Mobile Security
- Application Security
- Root Detection

---

## Rooting nedir?

Rooting yani rootlama, kök erişimi veya süper kullanıcı erişimi olarak adlandırılan bir cihazın yönetici ayrıcalıkları elde etme işlemidir. Cihazdaki normal kullanıcı sınırlı erişimi aşıp, işletim sistemi seviyesine erişmeyi sağlar.

## Android Root Detection Nedir?

Mobil uygulamaların root'lu cihazları tespit ederek bazı işlemleri engelleme yöntemidir. Genellikle finans, bankacılık, e-devlet gibi uygulamalarda kullanılır.

### Neden Root Detection?

Root erişimi, saldırganın uygulama içindeki verileri okumasına / değiştirmesine olanak tanır.

- Root cihazlarda:
    - Uygulama dosyaları dışarı taşınabilir
    - `Frida`, `Xposed`, `Magisk` gibi araçlar rahatlıkla çalışabilir

Bu yüzden uygulamalar, root’lu cihazlarda çalışmayı bilerek engelleyebilir.

## **Magisk Nedir?**

Magisk, Android cihazlarda root erişimi sağlamanın ve sistemi modifiye etmenin modern ve gizli yoludur. İşlevi sistemin en alt katmanına (boot seviyesine) root erişimi enjekte etmektir.

### Magisk Ne İşe Yarar?

- Root erişimi sağlar
- Root detection bypass edebilir
- Modül desteği vardır (frida, xposed, sslunpinning)

### Magisk Nasıl Çalışır?

Magisk, boot image üzerinden çalışır. (**boot image**, Android cihazın açılması için gereken "ilk çalışan dosyadır)

Root erişimi sistem bölümlerini modifiye etmeden verilir.

Bu sayede:

- Google SafetyNet testlerinden geçilebilir
- Uygulamalar root'u algılayamaz (gizlersen)

### Ne Zaman Gerekli Olur?

- Gerçek cihazda root detection’ı bypass etmek istiyorsak,
- SafetyNet geçilsin istiyorsak (**SafetyNet**, cihazın "güvenilir" olup olmadığını uygulamalara bildirir.)

### Nasıl İndirilir?

- https://github.com/newbit1/rootAVD?tab=readme-ov-file dan indirilir.
- emülatör çalıştırılır.
- indirilen dosyanın dizininde .\rootAVD.bat ListAllAVDs komutu çalıştırılır.
- api’ya göre seçilir ve indirilir.

→ rootAVD.bat system-images\android-29\google_apis\x86\ramdisk.img

- cihazda indikten sonra uygulamadan magisk install’a basılır.
- direct install seçeneğini seçerek, let’s go butonuna basılır.
- reboot butonu ile cihaz rebootlanır.

## Xposed

Xposed, telefona sistemin içine gizli eklentiler yüklemeye yarayan bir sistemdir. Bu eklentiler sayesinde uygulamaları kandırabilir, rootu gizleyebilir, davranışları değiştirebilirsin.

### Xposed Ne İşe Yarar?

- Uygulama rootlu mu diye bakıyorsa kandırmak
- Reklamları kaldırmak
- Gizli mesajları göstermek
- Uygulamanın davranışını değiştirmek
- GPS konumunu sahte göstermek

### Modül Örneği :

1. RootCloak → root kontrolü yapan uygulamaları kaldırır
2. JustTrustMe → Uygulamanın SSL pinning kontrolünü kalıdırır
- Bazı uygulamalar, HTTPS bağlantısını sadece kendi tanıdığı sertifikayla kurar (SSL Pinning). Bu da **burpsuite, mitmproxy gibi araçlarla veri dinlemeni engeller. JustTrustMe**, uygulamanın bu sertifika kontrolünü iptal eder. Böylece güvenli sandığı bağlantı üzerinden veri okuyabilirsin.
1. FakeGApps → Google servislerini taklit eder
2. XPrivacyLua → Uygulamalara sahte bilgiler verir

## Native Root Detection Nedir?

> Android uygulamaları root kontrolünü yalnızca Java değil, C/C++ ile yazılmış native katmanda da yapabilir. Bu yöntemler, doğrudan sistem dosyalarını kontrol ederek daha derin ve zor tespit edilebilen analizler gerçekleştirir.
> 

| Fonksiyon | Amaç |
| --- | --- |
| `access()` | Dosya var mı? |
| `system()` | Komut çalıştırmak |
| `stat()` | Dosya özellikleri |
| `fopen()` | Dosya açılıp okunabilir mi? |

Native Root Detection Kodu (C dilinde örnek)

```jsx
#include <unistd.h>

int is_rooted() {
    if (access("/system/xbin/su", F_OK) == 0) {
        return 1; // root
    }
    return 0; // değil
}
```

## Root Detection İçin Yaygın Yöntemler

| Kategori | Yöntem | Açıklama |
| --- | --- | --- |
| **1. Binary Kontrolü** | `/system/xbin/su` var mı? | `su` komutunun varlığı kontrol edilir. Root erişimi sağlar. |
|  | `which su` komutu | `su` komutunun yolu terminalde aranır. |
| **2. Dosya Kontrolü** | Magisk dosyaları (`.magisk`) | `/sbin/.magisk`, `/data/adb/magisk` gibi dizinler kontrol edilir. |
|  | Xposed dosyaları | `/data/data/de.robv.android.xposed.installer/` gibi yollar kontrol edilir. |
|  | BusyBox binary’si | `/system/xbin/busybox` varsa rootlu olabilir. |
| **3. Build.prop Kontrolü** | `ro.debuggable=1`, `ro.secure=0` | Sistem debug moddaysa ya da `secure=0` ise root işareti olabilir. |
| **4. Paket Kontrolü** | `com.noshufou.android.su` gibi | SuperSU veya Magisk gibi root yöneticisi uygulamaların varlığına bakılır. |
| **5. Komut Çalıştırma** | `id`, `whoami`, `mount` komutları | Komut çıktılarından root olup olmadığı anlaşılır. |
| **6. System partition yazılabilir mi?** | `mount` ile `rw` kontrolü | `/system` bölümünün yazılabilir olması root göstergesidir. |
| **7. SELinux kontrolü** | `getenforce` çıktısı `Permissive` mi? | Normalde Enforcing olur. Permissive root ipucu olabilir. |
| **8. Frida/Xposed Detection** | `frida`, `xposed` kelimeleri bellekte var mı? | `/proc/self/maps` içinde Frida/Xposed izleri aranır. |
| **9. Native API Kullanımı** | `access()`, `fopen()`, `stat()` | C/C++ tarafında dosya varlığı veya erişim kontrol edilir. |


## Root gizleme araçları nedir?

Root gizleme araçları, Android işletim sisteminde root (köklendirme) erişiminin varlığını maskelemek veya gizlemek için tasarlanmış yazılımlardır. Bu araçlar, root erişimini tespit eden uygulamaları "yanıltarak" cihazın root'lu olmadığı izlenimini verir.

**Popüler Root Gizleme Araçları:**

1. **Magisk Hide** (Magisk içinde yer alır)
2. **Shamiko** (Magisk modülü)
3. **KernelSU**
4. **TaiChi**
5. **XPrivacy**


### Bu araçlar rootu nasıl gizler?

1) Süreç ile Gizleme

- **Ne yapar?** Uygulamaların process listesinde su, magisk gibi root ile ilgili süreçleri görmesini engeller
- **Nasıl?** Sistem çağrılarını intercept ederek (yakalayarak) listelerden root süreçlerini çıkarır

<br>

2) Dosya Sistemi ile Gizleme

- **Ne yapar?** Klasik root dosya yollarını (/system/bin/su, /system/xbin/su gibi) gizler
- **Nasıl?** Dosya sistemi çağrılarını yönlendirerek:
    - Uygulama root dosyasına erişmek istediğinde "dosya bulunamadı" hatası alır
    - Aslında dosya oradadır, ama sadece o uygulama göremez
<br>

3) Zygote Enjeksiyonu (Magisk'te Zygisk) ile Gizleme

```
Uygulama Başlatma Süreci:
Uygulama → Zygote (Ana proses) → Yeni uygulama prosesi
                    ↓
         Zygisk enjeksiyonu yapılır
                    ↓
    Uygulama root'u göremez hale gelir
```

- **Zygote:** Android'de tüm uygulamaların doğduğu ana proses
- **Zygisk:** Magisk'in Zygote'a kendini enjekte ederek her uygulamada root'u gizleyebilmesi

<br>

4) API Seviyesinde Gizleme

- **SafetyNet/Play Integrity:** Google'ın güvenlik API'lerine sahte "güvenli" yanıtlar döndürür
- Safetynet ve play integrity, cihazın güvenli olup olmadığını kontrol eden apilardır. Safetynet eski sürüm, play integrity daha güncel ve daha sıkı kontrollerin yapıldığı sürüm

<br>

5) Sistem Çağrılarına Müdahale ile Gizleme

- `open(), stat(), access()` gibi sistem fonksiyonlarını değiştirir
- Uygulama root dosyasını kontrol etmek istediğinde, fonksiyon "dosya yok" der

Örneğin, uygulamada 

```jsx
// Root'lu mu diye kontrol eden örnek kod
if (access("/system/bin/su", F_OK) == 0) {
    // Dosya varsa → ROOT'LU!
    exit(); // Uygulamayı kapat
}
```

şeklinde bir kontrol varsa.

<br>
**open() Fonksiyonu Nedir?**

- **open()**, Unix/Linux sistemlerinde dosya açmak için kullanılan sistem çağrısı
- **Prototip:** `int open(const char *pathname, int flags);`
- **args[0]:** Dosya yolunu içeren string pointer (`pathname`)
- **args[1]:** Dosya açma flag'leri (`O_RDONLY`, `O_WRONLY`, vb.)
- Uygulama her dosya açmaya çalıştığında bu fonksiyon tetikleniyor
- Hangi dosyaların açılmaya çalışıldığını logluyor
- Root detection için kritik dosyaları (/system/bin/su, /proc/self/status, vb.) tespit etmeye yarıyor

**snprintf() Fonksiyonu Nedir?**

- **snprintf()**, string formatlama ve buffer'a güvenli yazma fonksiyonu
- **Prototip:** `int snprintf(char *str, size_t size, const char *format, ...);`
- **args[0]:** Yazılacak buffer (`str`), **args[1]:** Buffer boyutu (`size`), **args[2]:** Format string'i (`format`), **...:** Değişken sayıda argüman
- **onEnter:** Buffer pointer'ını saklıyor
- **onLeave:** Buffer'daki string'i okuyor, manipüle ediyor ve logluyor