---
layout: post
title: "Bilgisayar Nasıl Çalışır?"
date: 2026-04-15

category: Research

image: /assets/posts/introduction-to-cybersecurity/cover.png

excerpt: Bilgisayar Açıldığında Arka Planda Neler Oluyor?

---

## Bilgisayar Açıldığında Arka Planda Neler Oluyor?

İlk olarak bilgisayarın power tuşuna basıldığında, **PSU (Power Supply Unit)** elektriği bileşenlere dağıtır.

Sonra **CPU resetlenir.**

### Peki CPU nedir?

CPU (**Central Processing Unit**) bilgisayarın beynidir. Bilgisayarın talimatlarını gerçekleştirir, matematiksel ve mantıksal işlemler yapar, bellek ile donanımı yönetir.

### Nasıl çalışır?

Örneğin bir program açıldı ve **3+1** işlemi yapıldı.

1. Komut RAM'den gelir (**Fetch**).

> RAM'i büyük bir depo, CPU'yu ise küçük ama çok hızlı çalışan bir fabrika gibi düşünebiliriz.
> 
1. Gelen komut CPU içindeki **register**'a gelir. Register, CPU'nun içindeki mini bir not kağıdı gibidir.
2. Komut çözülür (**Decode**). Control Unit, "3+1" işleminin bir toplama işlemi olduğunu anlar ve bunu **ALU**'ya gönderir.

> ALU, bir nevi hesap makinesi gibidir.
> 
1. İşlem yapılır (**Execute**) → **3 + 1 = 4**
2. Sonuç (**4**) tekrar register'a yazılır.
3. Sonuç geri gönderilir (**RAM → Ekran → Program**).

CPU'nun içi milyarlarca minik **transistörden** oluşur.

**Transistör = Elektrikli aç/kapa anahtarı**

- Elektrik var → **1**
- Elektrik yok → **0**

### CPU'nun siber güvenlik açısından kritik noktaları

- Yanlış yere zıplatılırsa → **Buffer Overflow**
- Başka kod çalıştırılırsa → **Shellcode**
- Gizli veri sızdırılırsa → **Spectre**

---

## Peki neden CPU resetlenir?

Resetleme işlemi, işlemcinin her şeyi sıfırlayıp en başlangıç noktasından tekrar çalıştırılmasıdır.

Bilgisayar açılırken CPU resetlenir.

Bu sırada:

- O anki komutlar durur.
- Register'lar temizlenir.
- Program sayacı başlangıç noktasına döner.
- CPU yeniden boot sürecine girer.
- Resetten sonra CPU'nun baktığı ilk komut firmware'den okunur.

Bilgisayar kapatılırken resetleme olmaz. Çünkü elektrik doğrudan kesilir ve CPU'nun yukarıdaki işlemleri yapacak zamanı olmaz.

Kapatıldıktan sonra yeniden açıldığında ise reset türüne göre farklı davranışlar görülebilir.

- **Cold Reset** → Elektriğin tamamen kesilmesi.
- **Warm Reset** → Elektriğin kesilmemesi (Restart / Yeniden Başlat).

> **Not:** Bazı zararlılar RAM'de kalabilir.
> 

---

## Bilgisayarın Açılış Süreci

**Power Tuşu → PSU → CPU → BIOS → Disk (İşletim Sistemi) → Bootloader → Kernel**

### 1. Güç verilmesi

Power tuşuna basınca kasa üzerindeki buton, anakart üzerindeki güç yönetim devresine **"başla"** sinyali gönderir.

Bu devre, PSU'daki **PS_ON** hattını aktive eder ve bilgisayara elektrik vermeye başlar.

Elektrik stabil hâle gelince PSU, **PWR_OK** sinyalini göndererek "hazırım" der.


### 2. CPU'nun çalışmaya başlaması

Elektrik geldiğinde, yani CPU güç aldığında donanımsal olarak **reset durumuna** girer.

- Register'lar temel değerlerine sıfırlanır.
- Program Counter (PC) register'ına **Reset Vector** adresi yazılır.

Bu nedenle CPU otomatik olarak BIOS çipindeki ilk komutu çalıştırmaya başlar.


### 3. BIOS / UEFI ve POST

BIOS/UEFI açıldığında ilk yapılan işlem **POST (Power On Self Test)** prosedürüdür.

Bu aşamada bilgisayarın temel donanımlarının çalışıp çalışmadığı kontrol edilir.

İlk olarak RAM adresleme testi yapılır. Yani RAM'in gerçekten çalışıp çalışmadığı ve her adrese veri yazılıp yazılamadığı test edilir.

Sonra CPU mikro kodu kontrol edilir. Mikro kod bozuk mu, resetleme sırasında bir sorun oluşmuş mu diye bakılır.

Bunun yanında;

- Klavye denetleyicisi
- Ekran kartı
- Diğer temel donanımlar

kontrol edilir.

POST sırasında herhangi bir hata tespit edilirse BIOS **beep kodları** veya hata mesajları üretir.

Eğer herhangi bir sorun yoksa temel donanımların çalıştığı doğrulanmış olur.


### 4. Boot cihazının seçilmesi

POST tamamlandıktan sonra BIOS/UEFI boot sırasını kontrol eder.

- Disk
- USB
- Ağ üzerinden boot

gibi seçeneklere bakar.

İşletim sistemi bulunan diski seçer ve diskteki uygun **bootloader** dosyasını belleğe yükleyerek kontrolü ona teslim eder.


### 5. Bootloader

Bootloader'ın temel görevi işletim sisteminin **kernel** dosyasını diskten bulup RAM'e yüklemektir.

Bu sırada gerekli temel sürücüleri veya dosya sistemi sürücülerini geçici olarak kullanır.

Kernel tamamen yüklendiğinde kontrolü ona teslim eder.

Artık tüm kontrol işletim sistemindedir.


### 6. Kernel

Kernel çalışmaya başladığında ilk olarak bellek yönetimini başlatır.

RAM'de hangi bölgelerin kullanılabileceğini analiz eder ve **Memory Map** oluşturur.

Ardından donanım soyutlama katmanını kullanarak sistemde bulunan donanımları tespit eder.

Her donanım için gerekli sürücüler RAM'e yüklenir.

Sürücülerin yüklenmesinin amacı, işletim sisteminin donanımlarla iletişim kurabilmesini sağlamaktır.

Diskler bu sayede erişilebilir olur ve kök dosya sistemi (**C:** vb.) aktif hâle gelir.

Buna **dosya sistemini mount etmek** denir.


### 7. İşlem yöneticisi (Scheduler)

Kernel daha sonra işlem yöneticisini (**Scheduler**) başlatır.

CPU aynı anda yalnızca tek bir komutu çalıştırırken, scheduler sayesinde birçok işlem sırayla çalıştırılarak aynı anda çalışıyormuş gibi görünür.

Artık kullanıcı alanı (**User Space**) hazırdır ve giriş ekranı açılır.

---

## Terimler

- **PSU (Power Supply Unit):** Prizdeki elektriği bilgisayarın ihtiyaç duyduğu temiz ve düzenli elektriğe çeviren parça.
- **PWR_OK Sinyali:** Güç kaynağının "Her şey yolunda, çalışabilirsin." demesi.
- **Anakart (Motherboard):** Tüm parçaların bağlı olduğu ana devre kartı.
- **CPU (İşlemci):** Bilgisayarın beyni; tüm hesaplamaları yapar.
- **Reset State:** İşlemcinin fabrika ayarına dönmüş hâli.
- **Reset Vector:** İşlemcinin ilk çalıştıracağı komutun adresi.
- **BIOS / UEFI:** Bilgisayarın açılmasını sağlayan firmware.
- **BIOS (Basic Input Output System):** Eski sistemlerde açılışı yöneten yazılım.
- **UEFI (Unified Extensible Firmware Interface):** Yeni nesil, daha gelişmiş açılış sistemi.
- **Firmware:** Donanımların içinde gömülü bulunan küçük yazılımlar.
- **POST (Power On Self Test):** Açılışta yapılan donanım sağlık testi.
- **Disk:** Bilgisayardaki verilerin kalıcı olarak saklandığı yer.
- **Boot Order:** Hangi diskin önce kontrol edileceği.
- **Bootloader:** İşletim sistemini RAM'e yükleyen küçük program (ör. Windows Boot Manager veya GRUB).
- **SSD / HDD:** İşletim sisteminin depolandığı yer.
- **Kernel:** İşletim sisteminin beyni; donanım ile programlar arasında iletişim kurar.
- **GRUB:** Linux sistemlerde kullanılan bootloader.
- **Windows Boot Manager:** Windows'un bootloader'ı.
- **RAM:** Çalışan programların geçici olarak bulunduğu hızlı bellek.
- **Driver (Sürücü):** Donanımların işletim sistemiyle konuşmasını sağlayan yazılımlar.
- **File System (Dosya Sistemi):** Diskteki dosyaları düzenleyen yapı (NTFS, ext4).
- **Process:** İşletim sistemi üzerinde çalışan program.

---

## RAM

RAM, çalışan bilgisayarın tüm aktif verilerini tutar.

Her RAM hücresinin bir adresi vardır.

Örneğin:

```
Adres      → Veri
1000       → 0xAA
```

CPU RAM'e veri yazarken:

> "Şu adrese git, şu veriyi koy."
> 

şeklinde işlem yapar.

Her RAM hücresinin kendine ait bir adres numarası bulunur.

### Neden adresleme testi yapılır?

Çünkü RAM bozuksa;

- İşletim sistemi çöker.
- Uygulamalar hata verir.
- Boot işlemi tamamlanamaz.

Bu nedenle BIOS önce RAM'in temel adresleme mantığının doğru çalıştığını kontrol eder.

---

## İşletim Sistemi (Operating System)

İşletim sistemi, bilgisayarda gerçekleşen her şeyi koordine eden temel yazılımdır.

```
User
   ↓
Applications
   ↓
Operating System
   ↓
Hardware
```

Görevleri:

- **Süreç Yönetimi:** Çalışan programları oluşturur ve her işlemin ne kadar CPU süresi alacağına karar verir. Böylece çoklu görev yürütme sağlanır.
- **Bellek Yönetimi:** İşlemlere RAM tahsis eder ve işlemleri birbirinden izole eder.
- **Dosya Sistemi Yönetimi:** Dosyaları dizinler, yollar, izinler ve metadata ile düzenler.
- **Kullanıcı Yönetimi:** Kullanıcı hesaplarını, kimlik doğrulamayı ve yetkileri yöneterek kimin neye erişebileceğini belirler.
- **Aygıt Yönetimi:** Sürücüleri yükler ve uygulamaların donanımlarla ortak bir arayüz üzerinden iletişim kurmasını sağlar.

## İşletim sistemiyle iki şekilde iletişim kurabiliriz

1. **GUI (Graphical User Interface)**
2. **CLI (Command Line Interface)**