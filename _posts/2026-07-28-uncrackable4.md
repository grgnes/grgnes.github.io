---
layout: post
title: "Owasp Uncrackable Level 4 R2pay  APK Walkthrough"
date: 2026-07-28

category: APK

image: /assets/posts/uncrackable1/owasp_mas_header.png

excerpt: Owasp Uncrackable Level 4 R2pay APK Çözümü

tags: 
- Mobile Security
- APK

---
Apkyı cihaza indirdiğimizde direkt crash olup kapandığını gözlemliyoruz.

<img src="/assets/posts/uncrackable4/image.png" width="700">

Frida ile çalıştırdığımda bir hata çıktısı ile karşılaşıyoruz. Bu hata çıktısı,

- Hata Mesajı,
- Sistem Bilgileri,
- Signal Bilgisi,
- CPU Registerları (Çökme Anındaki Değerler),
- Backtrace Bilgisini içeriyor.

Bu içerikleri tek tek incelediğimizde,

### - Hata Mesajı

```powershell
[Android Emulator 5554::re.pwnme ]-> Process crashed: Bad access due to invalid address
```

uygulamanın çalışırken segmentation fault yani geçersiz bir bellek adresine erişmeye çalışırken çöktüğünü gösteriyor.

### - Sistem Bilgileri

```powershell
*** *** *** *** *** *** *** *** *** *** *** *** *** *** *** ***
Build fingerprint: 'google/sdk_gphone_x86_64/generic_x86_64:9/PSR1.180720.122/6736742:userdebug/dev-keys'
Revision: '0'
ABI: 'x86_64'
pid: 3949, tid: 4060, name: re.pwnme  >>> android.process.media <<<
```

- **Cihaz**: Android Emulator 5554 (x86_64 mimarisi)
- **Android Sürümü**: Android 9 (API 28)
- **Mimari**: x86_64
- **PID**: 3949 (Process ID)
- **TID**: 4060 (Thread ID)
- **Process name**: re.pwnme  >>> android.process.media <<<

### - Signal Bilgisi

```powershell
signal 11 (SIGSEGV), -> Segmentation fault - geçersiz bellek erişimi
code 1 (SEGV_MAPERR), -> Haritalanmamış (mapped olmayan) bir bellek adresine erişim denemesi
fault addr 0xfa929095 -> Erişilmeye çalışılan geçersiz adres
```

Hatalı adres çok spesifik ve random görünüyor.
Normal bir NULL + offset: `0x00000000 + 0xfa929095`  

### - CPU Registerları

```powershell
rax 0000000000000000  rbx 00000000ffffffff  rcx 00000000000035b2  rdx 00000000fa929095
r8  ffffffffffffffb0  r9  000075b810d8a000  r10 0000000000000000  r11 0000000000000206
r12 55348a26b14153ff  r13 dd898eb03fc0e0fe  r14 00000000ffffffff  r15 f83d5132fb225700
rdi 000000000000002b  rsi 000000000000d6c8
rbp 000075b810d8a4b0  rsp 000075b810d83fa0  rip 000075b810e16529

```

Çökme anındaki değerleri incelediğimizde,

- rax 0000000000000000  → NULL pointer
- rdx 00000000fa929095  → Hatalı adres burada
- rip 000075b810e16529  → Çöktüğü instruction pointer yani kodun çöktüğü yeri gösteriyor.

### - Backtrace

```powershell
backtrace:
    #00 pc 000000000008b529  /data/app/re.pwnme-8PIKZDvmqRR84sJMKhxw2g==/lib/x86_64/libnative-lib.so
***
```

Backtrace de ise hata, native library(libnative-lib.so) içindeki 0x8b529 offsetinde oluşmuş.

## Anti-Debugging Mekanizması Tespiti

Bu hata çıktısını inceledikten sonra, crash'in normal bir bug değil, kasıtlı bir anti-debugging mekanizması olduğuna karar verdim. İşte bu sonuca varmamı sağlayan kanıtlar:

### 1- Process Name Manipülasyonu

```powershell
name: re.pwnme  >>> android.process.media <
```

Normal bir Android uygulamasında process name ile package name aynı olur:

```powershell
name: com.example.app  >>> re.pwnme <
```

Ancak burada process name kasıtlı olarak `android.process.media` ile değiştirilmiş. Bu, klasik bir anti-debugging tekniğidir:

```powershell
<!-- AndroidManifest.xml -->
<application android:process="android.process.media">
```

Buradaki amaç debuggerların ve analiz araçlarının uygulamayı bulmasını zorlaştırmaktır.

### 2- Şüpheli Register Pattern’i

```powershell
rax 0000000000000000  ← NULL pointer
rdx 00000000fa929095  ← Çok spesifik bir offset
```

Bu kombinasyon kasıtlı bir crash pattern'i gösteriyor:

Normal bir bug'da göreceğimiz:

- Rastgele register değerleri
- Stack corruption (rbp, rsp'de garip değerler)
- Return address'in bozulması

Bizim durumumuzda:

- RAX temiz bir şekilde NULL (0x00000000)
- RDX tam olarak `0xfa929095` gibi hardcoded bir değer
- Stack tamamen sağlam (`rbp` ve `rsp` normal görünüyor)

Daha anlaşılır olması için tahmini kod şu şekilde olabilir:

```c
void anti_tamper_check() {
    void *trap = NULL;  // rax = 0x00000000
    uint64_t offset = 0xfa929095;  // rdx = kasıtlı değer
    *((char*)(trap + offset)) = 0;  // Kasıtlı SIGSEGV
}
```

### 3- Garip Register Değerleri (r12-r15)

```powershell
r12 55348a26b14153ff
r13 dd898eb03fc0e0fe
r15 f83d5132fb225700
```

Bu değerler integrity check değerleri olabilir.

Integrity check değerleri→ Bir dosyanın bütünlüğünü doğrulamak için hesaplanan kontrol sayıları.

### 4- Stack Frame’in Çok Temiz Olması

```powershell
rbp 000075b810d8a4b0  ← Normal stack base pointer
rsp 000075b810d83fa0  ← Normal stack pointer
```

Normal bir memory corruption'da (buffer overflow gibi), stack'teki
veriler bozulur ve RBP/RSP geçersiz değerler gösterir.

Ancak bizim durumumuzda:

- RBP: 0x75b810d8a4b0 (normal stack adresi)
- RSP: 0x75b810d83fa0 (normal stack adresi)

Stack'in bu kadar temiz olması, crash'in kontrollü ve kasıtlı olduğunu gösteriyor.

## **Anti-Debugging Bypass: Exception Handler İmplementasyonu**

Anti-debugging mekanizmasını tespit ettikten sonra, crash'i bypass etmek için bir Frida script'i yazdım. Bu script, crash anında devreye girip sanki fonksiyon normal şekilde return etmiş gibi davranarak uygulamanın çalışmaya devam etmesini sağlıyor.

Frida scripti çalışma mantığı:

```jsx
var base = null;              -> Native library'nin base adresi
var crashCount = 0;           -> Kaç kez exception yakalandığı

Process.setExceptionHandler(function(details) {
     // Exception'ı yakala (SIGSEGV)
     // Crash'in libnative-lib.so içinde olup olmadığını kontrol et
     // Stack'i restore et (sanki fonksiyon normal bitti)
     // 0 döndür ve çalışmaya devam et
});
```

Genel kod yapısına baktığımızda:

```jsx
var base = null;
var crashCount = 0;
```

**base**: Native library'nin bellekteki başlangıç adresi

- Android'de **ASLR** (Address Space Layout Randomization) aktif
- Her çalıştırmada farklı bir adrese yüklenir

**crashCount**: Debug amaçlı sayaç

- Kaç kez exception yakalandığını takip eder

```jsx
Process.setExceptionHandler(function(details) {
    crashCount++;
```

**Process.setExceptionHandler**: Frida'nın exception yakalama API'si

**details objesi** şunları içerir:

- details.type: Exception türü ("access-violation", "abort", vb.)
- details.address: Crash oluşan bellek adresi
- details.context: CPU registerları (rax, rbx, rip, rsp, rbp, vb.)

```jsx
if (base === null) {
    base = Module.findBaseAddress("libnative-lib.so");
    console.log("base adress:" + base);
}

if (base === null) {
    return false;
}
```

İlk exception anında modül henüz yüklenmemiş olabilir. Bu yüzden:

1. İlk crash'te Module.findBaseAddress() çağırılır
2. Bulunamazsa exception'ı işleme alınır (return false)

**Module.findBaseAddress()**: Verilen modülün base adresini döndürür

ASLR Nedeniyle:

1. Çalıştırma: 0x75b810d8b000
2. Çalıştırma: 0x7a2340a1c000 (farklı)

```jsx
try {
    var offset = details.address.sub(base);
    ..
    
} catch(e) {
     return false;
  }   
```

Offset değerini bulmak için details.address.sub() kullanılır. Eğer offset aralığının dışında ise return false döner.

> Base Address:     0x75b810d8b000
Crash Address:    0x75b810e16529
Offset:           0x75b810e16529 - 0x75b810d8b000 = 0x8b529
> 

**Offset neden önemli?**

- Her çalıştırmada base değişir ama offset sabit kalır.
- Offset ile crash'in hangi fonksiyonda olduğunu biliriz.

```jsx
..
if (offset.compare(0) > 0 && offset.compare(0x500000) < 0) {
```

Crash , libnative-lib.so içinde 0x0 ile 0x500000 (5MB) arasında mı kontrol edilir.

if koşulu sağlanıyorsa, stack frame restore işlemi yapılır.

```jsx
try {
       var savedRbp = details.context.rbp;
       console.log("savedRbp" + savedRbp);
       var returnAddr = savedRbp.add(8).readPointer();
       console.log("returnAddr" + returnAddr);
       ..
       
       
} catch (e) {
      return false;
  }
```

<img src="/assets/posts/uncrackable4/image (1).png" width="700">

Peki bu kodta ne yapıyor:

- **savedRbp**: Mevcut stack frame'in base pointer'ını al
- **savedRbp.add(8)**: 8 byte yukarı çık (64-bit sistemde pointer = 8 byte)
- **readPointer()**: O adresdeki **return address'i** oku

ve daha sonra return adress güvenlik kontrolü yapılır:

```jsx
..
if (returnAddr.compare(0) > 0) {
```

**Return address'in NULL olup olmadığını kontrol edilir:**

- **0x0 (NULL)** → Stack bozuk, restore etmeye çalışma
- 0x7ffabcd123 gibi bir değer → OK, devam et

Devamında ise:

```jsx
if (returnAddr.compare(0) > 0) {
                    details.context.rax = ptr(0);
                    details.context.rsp = savedRbp.add(16);
                    details.context.rbp = savedRbp.readPointer();
                    details.context.pc = returnAddr;
                    return true;
                }
```

Stack store işlemi başlar:

- **details.context.rax = ptr(0); →** return değerini 0 yapar yani fonksiyom 0 döndürmüş gibi davranır.

```jsx
RAX = 0
```

- **details.context.rsp = savedRbp.add(16); →** stack pointer’ı 16 byte yukarı taşır yani stack’i temizler, üst frame’e geçirir.

```jsx
savedRbp = 0x75b810d8a4b0
savedRbp.add(16) = 0x75b810d8a4b0 + 0x10 = 0x75b810d8a4c0
```

- **details.context.rbp =** savedRbp.readPointer(); → Önceki frame’in RBP’sini geri yükler.
- **details.context.pc = returnAddr; →** Çağırılan fonksiyona geri döner.

```jsx
PC (Program Counter) = returnAddr
RIP = 0x75b810e14a20
```

- **return true; →** Fridaya “Expection’ı hallettim” der.

Return adresi okunmazsa veya stack bozuksa fallback mekanizması devreye girer.

```jsx
details.context.rax = ptr(0);       // rax = 0 -> 0 döndür
details.context.pc = base.add(0);   // pc = base -> başlangıç adresine atla
return true;
```

Buradaki amaç en azından crash’i engellemesi, çökmemeye devam etmesi.

Bu kodu çalıştırdığımda:

<img src="/assets/posts/uncrackable4/image (2).png" width="700">

## Java Anti-Tampering Bypass : Java Exception **İmplementasyonu**

Native crash'i başarıyla bypass ettikten sonra, uygulama yeni bir hatayla karşılaştım:

```jsx
Process crashed: java.lang.ArithmeticException: divide by zero
```

Java kaynak kodunu incelediğimde, `MainActivity.onCreate()` içinde kasıtlı bir crash mekanizması buldum:

```java
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    
    this.f508 = (byte) -16;  
    
    C0282 rb = new C0282(getApplicationContext());
    
    // Anti-tampering kontrolü
    if (rb.m1162() || (rb.m1155() && rb.m1150())) {
        int r1 = 1337 / 0;  
        this.f508 = (byte) (this.f508 | 15);
    }
    
    m316();  
}
```

C0282 sınıfı, anti-debugging/anti-tampering kontrolleri yapıyor:

- m1162(): Frida detection
- m1155(): Root detection
- m1150(): Xposed/Magisk detection

Eğer bu kontrollerden herhangi biri başarılı olursa, kasıtlı olarak sıfıra bölme işlemi yapılıyor.

Bu mekanizmayı bypass etmek için iki yaklaşım mevcut:

### **Yaklaşım 1: Detection Metodlarını Hook'lamak**

Her bir detection metodunu ayrı ayrı hook'layıp false döndürmek.

İlk olarak dlopen patching yaparak, uygulamanın root detection mekanizmalarını native seviyede devre dışı bırakıyoruz.

```jsx
var dlopen = Module.findExportByName(null, "android_dlopen_ext");
if(dlopen) {
    Interceptor.attach(dlopen, {
        onEnter: function(args) {
            var lib = args[0].readCString();
            console.log("lib: "+lib);
            if(lib && lib.includes("libnative-lib.so")){
                this.hook = true;
            } 
        },
        onLeave: function(retval) {
            if(this.hook) {
                patchAll();
                setTimeout(patchAll, 100);
            }
        }
    });
}

function patchAll() {
    base = Module.findBaseAddress("libnative-lib.so");
    console.log("patchAll Base: "+base);
        if (base === null) {
        return false;
    }
    
    [0x8b529, 0xcf150, 0xcf1b7].forEach(function(offset) {
        try {
            var addr = base.add(offset);
            console.log("addr: "+ addr);
            Memory.patchCode(addr, 3, function(code) {
                var writer = new X86Writer(code, { pc: addr });
                writer.putXorRegReg('eax', 'eax');
                writer.putRet();
                writer.flush();
            });
        } catch(e) {}
    });
}
```

- `android_dlopen_ext` kütüphanesini yakalar → Android'de dinamik kütüphane yükleme fonksiyonudur.
- `libnative-lib.so` yüklendiğini tespit eder → Bu, uygulamanın root kontrolü yapan native (C/C++) kodudur
- Kütüphane yüklendiği anda `patchAll()` fonksiyonunu çalıştırır → Belirlenen offset'lerdeki root detection kodlarını patch eder.

Root detection kütüphanesi yüklendiği anda hemen tespit edilip kontrol kodlarının çalıştırılmadan `false` (başarısız) döndürülmesi sağlanır. Bu şekilde uygulama "cihazda root yok" zanneder ve devam eder.

<img src="/assets/posts/uncrackable4/image (3).png" width="700">

dlopen patching ile native kodu değiştirdikten sonra, sistem seviyesinde root araçlarının varlığını gizlemek için dosya erişim fonksiyonlarını yakalarız.

```jsx
["open", "access"].forEach(function(funcName) {
    var ptr = Module.findExportByName("libc.so", funcName);
    if(ptr) {
        Interceptor.attach(ptr, {
            onEnter: function(args) {
                var path = args[0].readCString();
                console.log("path: "+path);
                this.block = path && (path.includes("/su") || path.includes("magisk"));
            },
            onLeave: function(retval) {
                if(this.block) retval.replace(-1);
            }
        });
    }
});
```

- İşletim sisteminin temel kütüphanesinde `open` ve `access` fonksiyonlarını bulur - bunlar dosya açmak ve dosya varlığını kontrol etmek için kullanılır.
- Uygulama herhangi bir dosyaya erişmeye çalıştığında, dosya yolunu kontrol eder. Eğer `/su` (SuperUser binary'si) veya `magisk` (root aracı) içeriyorsa işaretler.
- Eğer root aracı dosyasına erişim denenirse, sistem fonksiyonunun dönüş değerini -1 (hata) olarak değiştirir. Böylece uygulama "bu dosya bulunamadı" sonucuna varır.

<img src="/assets/posts/uncrackable4/image (4).png" width="700">

Son olarak Java seviyesinde root detection kütüphanelerini ve sistem API'lerini hook ederek, uygulamanın tüm root kontrol mekanizmalarını devre dışı bırakırız.

**Ana Sınıf - C0282**

**Java:**

```java
public class C0282 {
    public boolean m1162() {
        return m1160() || m1163() || m1156("su") || m1152() || ...;
    }

    public boolean m1161() {
        String buildTags = Build.TAGS;
        return buildTags != null && buildTags.contains("test-keys");
    }

    public boolean m1160() { /* PackageManager checks */ }
    public boolean m1163() { /* Dangerous apps */ }
    public boolean m1156(String filename) { /* File.exists */ }
    public boolean m1152() { /* System properties */ }
    public boolean m1150() { /* RootBeerNative */ }
}

```

**Frida JS:**

```jsx
if(className.startsWith("p000.")) {
    var clazz = Java.use(className);  // C0282 sınıfını yakala
    var methods = clazz.class.getDeclaredMethods();

    methods.forEach(function(method) {
        var methodName = method.getName();      // m1162, m1161, m1160 vb
        var returnType = method.getReturnType().toString();

        // Boolean dönen m* metodları hook et
        if(methodName.match(/^m\d+$/) && returnType.includes("boolean")) {
            clazz[methodName].implementation = function() {
                return false;  // Orijinal metod çalışmaz
            };
        }
    });
}

```

---

**m1161() - Build.TAGS Kontrolü**

**Java:**

```java
public boolean m1161() {
    String buildTags = Build.TAGS;
    return buildTags != null && buildTags.contains("test-keys");
}

```

Eğer TAGS "test-keys" içerirse → root var

**Frida JS:**

```jsx
var Build = Java.use("android.os.Build");
Build.TAGS.value = "release-keys";  // Değeri değiştir

```

Direkt `Build.TAGS` değerini spoof ederek, m1161() kontrol yapamaz.

---

**m1160() - PackageManager Kontrolü**

**Java:**

```java
public boolean m1160() {
    return m1153(null);  // Root management apps
}

public boolean m1153(String[] additionalRootManagementApps) {
    ArrayList<String> packages = new ArrayList<>(Arrays.asList(C0272.f1009));
    return m1157(packages);
}

public final boolean m1157(List<String> packages) {
    PackageManager pm = this.f1026.getPackageManager();
    for (String packageName : packages) {
        try {
            pm.getPackageInfo(packageName, 0);  // Paket var mı?
            result = true;  // Bulundu!
        } catch (PackageManager.NameNotFoundException e) {
        }
    }
    return result;
}

```

Magisk, SuperSU gibi paketleri arar.

**Frida JS:**

```jsx
var PM = Java.use("android.content.pm.PackageManager");
PM.getPackageInfo.overload('java.lang.String', 'int').implementation = function(pkg, flags) {
    var blockedApps = [
        "magisk", "supersu", "chainfire", "koushik", "topjohnwu",
        "luckypatcher", "substrat", "xposed"
    ];

    for(var i = 0; i < blockedApps.length; i++) {
        if(pkg.toLowerCase().includes(blockedApps[i])) {
            var NameNotFound = Java.use("android.content.pm.PackageManager$NameNotFoundException");
            throw NameNotFound.$new();  // Paket bulunamadı hatası
        }
    }
    return this.getPackageInfo.overload('java.lang.String', 'int').call(this, pkg, flags);
};

```

Exception fırlatarak m1157() kontrol başarısız olur.

---

**m1156() - File.exists Kontrolü**

**Java:**

```java
public boolean m1156(String filename) {  // "su" veya "magisk"
    String[] pathsArray = C0272.m1135();
    for (String path : pathsArray) {
        File f = new File(path, filename);
        boolean fileExists = f.exists();  // Dosya var mı?
        if (fileExists) {
            result = true;
        }
    }
    return result;
}

```

Tüm olası path'lerde su binary'si arar.

**Frida JS:**

```jsx
var File = Java.use("java.io.File");
File.exists.implementation = function() {
    var path = this.getAbsolutePath();
    if(path.includes("/su") || path.includes("magisk") || path.includes("supersu")) {
        return false;  // Dosya yok demesini sağla
    }
    return this.exists.call(this);  // Diğer dosyalar normal
};

```

`File.exists()` hook edilerek su dosyaları "yok" kabul edilir.

---

**m1152() - System Properties Kontrolü**

**Java:**

```java
public boolean m1152() {
    Map<String, String> dangerousProps = new HashMap<>();
    dangerousProps.put("ro.debuggable", "1");
    dangerousProps.put("ro.secure", "0");

    String[] lines = m1154();  // getprop çalıştır
    for (String line : lines) {
        for (String key : dangerousProps.keySet()) {
            if (line.contains(key)) {
                if (line.contains(badValue)) {
                    result = true;  // Root property bulundu
                }
            }
        }
    }
    return result;
}

public final String[] m1154() {
    InputStream inputstream = Runtime.getRuntime().exec("getprop").getInputStream();
}

```

Tehlikeli prop'ları kontrol eder.

**Frida JS:**

```jsx
var System = Java.use("java.lang.System");
System.getProperty.overload('java.lang.String').implementation = function(key) {
    if(key.includes("ro.debuggable")) {
        return "0";  // Güvenli değer
    }
    if(key.includes("ro.secure")) {
        return "1";  // Güvenli değer
    }
    return this.getProperty.overload('java.lang.String').call(this, key);
};

```

Prop'ları direkt spoof ederek m1152() başarısız olur.

---

**m1148() - Runtime.exec("which su")**

**Java:**

```java
public boolean m1148() {
    Process process = Runtime.getRuntime().exec(new String[]{"which", "su"});
    BufferedReader in = new BufferedReader(new InputStreamReader(process.getInputStream()));
    boolean z = in.readLine() != null;  // Çıktı var mı?
    return z;  // su bulunduysa true
}

```

`which su` komutu çalıştırır.

**Frida JS:**

```jsx
var Runtime = Java.use("java.lang.Runtime");
Runtime.exec.overload('[Ljava.lang.String;').implementation = function(cmd) {
    var command = cmd[0];
    if(command.includes("su") || command.includes("which") ||
       command.includes("getprop") || command.includes("mount")) {
        var IOException = Java.use("java.io.IOException");
        throw IOException.$new("Permission denied");
    }
    return this.exec.overload('[Ljava.lang.String;').call(this, cmd);
};

```

Komut çalıştırılmadan Exception fırlatılır.

---

**m1150() - RootBeerNative Kontrolü**

**Java:**

```java
public boolean m1150() {
    if (!m1155()) {  // Native kütüphane yüklendi mi?
        return false;
    }
    String[] checkPaths = new String[paths.length];
    for (int i = 0; i < checkPaths.length; i++) {
        checkPaths[i] = paths[i] + "su";
    }
    RootBeerNative rootBeerNative = new RootBeerNative();
    return rootBeerNative.checkForRoot(checkPaths) > 0;
}

public boolean m1155() {
    return new RootBeerNative().m306();
}

```

Native kütüphane üzerinden kontrol.

**Frida JS:**

```jsx
if(className.includes("RootBeerNative")) {
    var RBN = Java.use(className);

    RBN.checkForRoot.implementation = function(paths) {
        console.log("RootBeerNative.checkForRoot() -> 0");
        return 0;  // Root yok
    };

    RBN.m306.implementation = function() {
        console.log("RootBeerNative.m306() -> false");
        return false;  // Başarısız
    };
}

```

<img src="/assets/posts/uncrackable4/image (5).png" width="700">

### **Yaklaşım 2: Exception Handling**

onCreate() metodunu hook'layıp exception'ları yakalamak.

```jsx
Java.perform(function() {
    var MainActivity = Java.use("re.pwnme.MainActivity");
    
    MainActivity.onCreate.overload("android.os.Bundle").implementation = function(bundle) {
        console.log("MainActivity.onCreate() called");
        
        try {
            this.onCreate(bundle);
        } catch(e) {
            // Exception'ı yut, uygulama kapanmasın
        }
    };
    
    console.log("Java Exception Handler passed :)");
});
```

Koda baktığımızda:

1. Önce frida’nın java kodlarına erişebilmesi için Java.perform() kullanıyoruz.
2. Java.use() ile kullanacağımız main class’ı bir değişkene atıyoruz.
3. MainActivity’nın onCreate metodunun Bundle parametreli versiyonunu bulup, orijinal kodun yerine çalışacak try/catch kodunu yazıyoruz.
4. Burada try kodunda orijinal onCreate() metodunu çağırır.
5. Catch’te Exception yakalandığı için uygulama crash olmaz ve açılır.


<img src="/assets/posts/uncrackable4/image (6).png" width="700">





