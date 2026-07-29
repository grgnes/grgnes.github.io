---
layout: post
title: "Mobil Uygulama Güvenlik Temelleri"
date: 2025-05-09

category: Research

image: /assets/posts/mobiluyg-temelleri/mobilsec.jpeg

excerpt: Mobil uygulama güvenliğine giriş yapmak isteyenler için hazırlanan bu yazıda Android işletim sistemi mimarisi, Linux Kernel, Android Runtime (ART), uygulama katmanları ve bir uygulamanın APK'ya dönüşme süreci temel seviyede açıklanmaktadır.
tags: 
- Mobile Security
- Application Security
- 101

---

## **Mobil Uygulama Güvenlik Temelleri**

**ANDROİD İŞLETİM SİSTEMİ**

```
Google yama yayınlar→ işlemci üreticisi qualcomm gönderir, kendi yamasını oluşturur→ telefon üreticisine gönderir → telekom şirketleri(operator) kendi yazılımlarını ekler → telefonumuz
```

## ÇEKİRDEK
    
**LİNUX KERNEL**
    
- İşletim sisteminin kalbidir.
- Güvenlik, hafıza yönetimi, süreç yönetimi, ağ yığınları ve sürücü modellerini içerir
- C diliyle oluşturulmuş bir çekirdek
- Donanım kaynaklarını ( CPU, bellek(ram), disk(HDD/SSD), ağ) yönetir ve kullanıcı programlarının donanımla doğrudan iletişim kurması sağlar
- Kullanıcı tabanlı izin modeli : bir uygulama indirdiğimizde onun başka uygulamalara erişmemesini sağlar.


<img src="/assets/posts/mobiluyg-temelleri/kernel.png" width="500" >


### DONANIM KATMANI
    
Yerleşik donanım birimleriyle etkileşimi sağlar. Örnek cihaz üzerindeki mikrofon, hoparlör, kamera gibi donanımlar ile iletişimi sağlar.


### KÜTÜPHANELER
    
- Android çekirdeği ile uygulamalar arasında köprü görevi görür.  
- Android uygulamalarının çekirdek hizmetlerine erişmesini sağlar. 
- İşletim sisteminin çekirdeğiyle ve Android Runtime (ART) ile etkileşime giren, uygulamalara temel işlevsellik sağlayan yazılım bileşenleridir    
- Bu kütüphaneler, donanım ve yazılım arasındaki köprü görevi görür.
- **Webkit :** Web tarayıcı motorlarının çalışması için kullanılır.
- **SQLite** : Veri yapıları kontrolü için kullanılır.
- **OpenGl**  : Grafik işlemleri için kullanılır.
- **Surface Manager** : Görüntüleme kontrolü yapan kütüphanedir.
- **libc (C Kütüphanesi):** Temel sistem işlevlerini sağlar.
- **SSL/TLS:** Güvenli ağ iletişimi (şifreleme) için kullanılan kütüphanelerdir.


### ANDROİD RUNTİME

- Dalvik VM ve ART VM 
- Android 5.0 sonrasında adı Dalvik kaldırıldı yerine ART geldi.
- Dalvik vm ve android runtime farkları 
- Garbage collection -> Pointer kullandığınızda ve sildiğinizde artık yer kaplamıyor java onu kendisi siliyor.
- Dalvik, android 2.2 sürümünden beri **“tam zamanında”** derleme **(Just-in-time ( JIT compilation))** kullanarak kodu derliyor bu da demek oluyor ki uygulamayı yazıp cihaza yüklediğimizde kod belli bir oranda derleniyor ve esas derleme uygulama çalışmaya başladığında yapılıyor. bu işlem uygulama her çalıştığında yapılıyor. bu da uygulama her çalıştığında fazladan yük getiriyor ve daha az verimli çalışılmasına sebep oluyor
- ART, bu durumu ortadan kaldırmak için **“zamanın ötesinde derleme” (Ahead-of-time (AOT compilation)** denilen ****bir işlemle bytecode derlemesini uygulama cihaza kurulurken yapıyor ve bytecode’u makine diline çeviriyor. böylece her uygulama açılırken yeni bir sanal makine başlatma, konu yeniden derleme durumu olmuyor.


| Özellik | **Dalvik** | **Android Runtime (ART)** |
| --- | --- | --- |
| **Derleme Yöntemi** | Just-In-Time (JIT) Derleme | Ahead-of-Time (AOT) Derleme |
| **Performans** | Düşük performans, çünkü her seferinde derleme yapılır | Daha hızlı, çünkü uygulamalar önceden derlenir |
| **Bellek Yönetimi** | Daha fazla bellek tüketimi | Daha az bellek tüketimi |
| **Uygulama Başlangıcı** | Daha yavaş başlar | Daha hızlı başlar |
| **Desteklenen Uygulama Türü** | Java bytecode'u çalıştırır | Java bytecode'u çalıştırır (AOT ile optimize edilmiş) |
| **Ne Zaman Derlenir?** | Uygulama her çalıştığında | Uygulama yüklenirken |



#### Android Sanal Makine nasıl çalışır?

1. uygulama, java veya kotlin ile yazılır
2. kod önce java bytecode’a, sonra da .dex formatına çevirilir
3. bu .dex formatı diske kaydedilir
4. ART bu dosyaları cihaza özel(işlemci) makine koduna (native code) çevirir ve önceden derleme (AOT) yapar
5. uygulama açıldığında ART önceden derlenmiş kodu doğrudan çalıştırır
6. bu sayede CPU ve RAM daha az yük altına girer ve uygulama hızlı açılır
7. ART garbage collecter kullanarak gereksiz ram kullanımını azaltır, sistem belleği dolarsa kullanılmayan uygulamalar ram den temizlenir
8. uygulama kapatılınca da çalışan işlemler sonlandırılır, ancak bazı veriler cache(önbellek) de tutabilir, diske kayıtlı olan AOT derlenmiş kod, bir sonraki çalıştırmada tekrar kullanılır

! Android runtime(ART), uygulamaları donanımdan bağımsız hale getirerek her cihazda çalışmasını sağlar.

```bash
apk → .class (javabytecode) → .dex → dalvik (JIT)
```

! Yeniden başlattığımızda bytecode dan itibaren derlenir.

<img src="/assets/posts/mobiluyg-temelleri/dalvik.png" width="400" >

```bash
apk → .class (javabytecode) → .dex → art (machine code) (AOT)
```
→ Yeniden başlattığımızda uygulamayı makine kodundan itibaren derlenir.

<img src="/assets/posts/mobiluyg-temelleri/3.png" width="400" >

- **Platform Bağımsızlık (Java Bytecode ve .class):**
    - Java, platform bağımsız çalışabilen bir dil olduğu için, **.class** dosyasındaki bytecode, farklı cihazlarda çalışabilmek için ilk adımı oluşturur. JVM, bytecode'u çalıştırmak için cihazın işlemci mimarisine göre makine koduna dönüştürür.
- **Android İçin Optimize Edilmiş Format (.dex):**
    - Android, **.class** dosyalarını doğrudan çalıştırmaz. Bunun yerine, **.dex** formatına dönüştürür, çünkü Android cihazları sınırlı bellek ve işlemci gücüne sahiptir. **Dex**, bu özelliklere uygun ve optimize edilmiş bir formattır.
- **Performans (Makine Kodu):**
    - **Makine kodu**, doğrudan işlemci tarafından çalıştırılabilen en hızlı formattır. **ART**, uygulamayı çalıştırmadan önce **.dex** dosyasını makine koduna dönüştürür, böylece uygulama hızlı başlar ve daha verimli çalışır.


### UYGULAMA ÇATISI
    
Android işletim sisteminin orta katmanıdır ve uygulama geliştirme için gerekli olan temel bileşenleri ve API'leri sağlar.

- **Kullanıcı arayüzü yönetimi** : `Activity`, `Fragment`, `ViewV`
- **Veri yönetimi** : `content provider`  , `SQLite`
- **Arkaplan işlemleri** : `Services` , `Broadcast Receivers`
- **Uygulama iletişimi** : `Intent`

**API'ler**: Veritabanı işlemleri, ağ bağlantıları, kullanıcı arayüzü bileşenleri gibi işlevleri kolayca kullanmanızı sağlayan araçlar sunar.

<img src="/assets/posts/mobiluyg-temelleri/uygkatmanı.png" width="400" >

### UYGULAMA KATMANI
    
Android sisteminin kullanıcıya yönelik en üst katmanıdır. Uygulama kodu, kullanıcı arayüzü ve işlevler burada yer alır ve kullanıcıların cihazla doğrudan etkileşimde olduğu bölümdür.

### **Özet Olarak:** Fotoğraf Çekme Süreci

1. **Uygulama Katmanı:** Kullanıcı fotoğraf çekme butonuna basar.
2. **Uygulama Çatısı:** Bu isteği **Camera Manager** aracılığıyla işler ve kamerayı başlatır.
3. **Android Runtime:** Fotoğraf çekme kodlarını çalıştırır.
4. **Native Libraries:** Görüntü işleme işlemleri burada optimize edilir.
5. **Donanım Soyutlama Katmanı:** Kamerayı başlatır ve görüntüyü alır.
6. **Linux Kernel:** Kamera sürücüsünü yönetir ve görüntü verisini işler.

Son olarak, bu görüntü **Uygulama Katmanı**na geri döner ve kullanıcıya gösterilir. Eğer kullanıcı fotoğrafı kaydetmek isterse, dosya sistemine yazılır ve **Gallery** uygulamasında görünür hale gelir.

<img src="/assets/posts/mobiluyg-temelleri/01.png" width="400" >

### Kodun APK'ya Dönüşme Süreci

#### 1. Java/Kotlin Kodlarının Yazılması

- Uygulamanın kaynak kodları (`MainActivity.java`, `MainActivity.kt` vb.) yazılır.
- Arayüz dosyaları (`activity_main.xml` gibi) hazırlanır.

#### 2. Java Kodlarının Derlenmesi (Compilation)

- `javac` derleyicisi, `.java` dosyalarını `.class` (Java Bytecode) dosyalarına dönüştürür.

#### 3. DEX Formatına Dönüştürme (Dexing)

- Oluşturulan `.class` dosyaları, **D8** veya **R8** kullanılarak `.dex` (Dalvik Executable) formatına çevrilir.

#### 4. Kaynak Dosyalarının Derlenmesi

- **AAPT (Android Asset Packaging Tool)**; XML dosyalarını, resimleri ve diğer kaynakları derleyerek Android'in kullanabileceği binary formata dönüştürür.

#### 5. APK'nın Paketlenmesi (Packaging)

- `AndroidManifest.xml`, `.dex` dosyaları ve derlenmiş kaynaklar bir araya getirilerek APK oluşturulur.

#### 6. APK'nın İmzalanması (Signing)

- APK'nın güvenilirliğini sağlamak için `apksigner` veya `jarsigner` ile dijital olarak imzalanır.

#### 7. APK'nın Cihaza Yüklenmesi

- APK, aşağıdaki komut ile Android cihaza yüklenir:

```bash
adb install myapp.apk
```

- Uygulama yüklendikten sonra Android Runtime (ART) tarafından çalıştırılır.

<img src="/assets/posts/mobiluyg-temelleri/process.png" width="400" >