---
layout: post
title: "Mobil Uygulama Güvenliği Statik Analiz Araçları"
date: 2026-07-28

category: Research

image: /assets/posts/mobiluyg-temelleri/mobilsec.jpeg

excerpt: Bu yazıda mobil uygulama güvenliğinde kullanılan statik analiz araçlarını inceledim. APKTool, Dex2Jar, CFR, JADX ve MobSF araçlarının ne işe yaradığını, kullanım alanlarını ve temel kurulum adımlarını örnek komutlarla ele aldım.
tags: 
- Mobile Security
- Application Security
- Static Analysis Tools

---

## Statik Analiz Araçları

Statik analiz, uygulamayı çalıştırmadan kaynak kodu ve dosyalarını inceleyerek uygulama hakkında bilgi edinme aşamasıdır.

### APKTool

APKTool, APK dosyalarını **Smali seviyesinde decompile** eder ve tekrar derlemeye olanak sağlar.

**Kullanım Alanları**

- `AndroidManifest.xml` dosyası düzenlenebilir.
- Smali kodları üzerinde değişiklik yapılabilir.
- Değiştirilen uygulama yeniden derlenerek test edilebilir.

**Kurulum ve Kullanım**

1) APKTool sitesinden `apktool.jar` dosyasını indirin.
2) APK dosyasını decompile edin.

```bash
java -jar apktool.jar d uygulama.apk
```

veya

```bash
apktool d uygulama.apk -o cikis_klasoru
```

3) APK'yı yeniden derleyin.

```bash
apktool b cikis_klasoru -o yeni.apk
```

---

### Dex2Jar

Dex2Jar, `.dex` dosyalarını `.jar` formatına çeviren bir araçtır.

**Kullanım Alanları**

- Java decompiler araçları ile daha rahat analiz yapılmasını sağlar.
- Kodun daha okunabilir hale gelmesine yardımcı olur.

**Kurulum ve Kullanım**

1) Dex2Jar'ın son sürümünü indirin.
2) APK dosyasını açarak `.dex` dosyalarını çıkarın.

```bash
unzip uygulama.apk -d uygulama_dosyalar
```

> `uygulama.apk` dosyası açılır ve `classes.dex` dosyaları `uygulama_dosyalar` klasörüne çıkarılır.

3) `.dex` dosyasını `.jar` formatına çevirin.

```bash
d2j-dex2jar.bat uygulama_dosyalar\classes.dex
```

> `classes-dex2jar.jar` dosyası oluşturulur.

4) Oluşan JAR dosyasını açın.

```bash
jd-gui classes-dex2jar.jar
```

5) Analize `AndroidManifest.xml` dosyasından başlayın.

---

### CFR

CFR, Java `.class` ve `.jar` dosyalarını orijinal kaynak koduna en yakın şekilde decompile etmek için kullanılan bir Java decompiler aracıdır.

Android analizinde özellikle Dex2Jar ile oluşturulan `.jar` dosyalarını Java kaynak koduna dönüştürmek için kullanılır.

**Kurulum ve Kullanım**

1) CFR'ı indirin.

```bash
wget https://github.com/leibnitz27/cfr/releases/download/0.152/cfr-0.152.jar
```

2) JAR dosyasını Java kaynak koduna çevirin.

```bash
java -jar cfr-0.152.jar classes-dex2jar.jar --outputdir kaynak_kodu
```

3) Tek bir `.class` dosyasını Java koduna dönüştürün.

```bash
java -jar cfr-0.152.jar MainActivity.class
```

---

### JADX

JADX, APK içerisindeki `.dex` dosyalarını Java kaynak koduna geri dönüştüren bir decompiler aracıdır.

**Kullanım Alanları**

- Java kaynak kodu incelenebilir.
- APK içerisindeki sınıflar ve paketler kolayca analiz edilebilir.

**Kurulum ve Kullanım**

1) JADX'yı yükleyin.

```bash
winget install jadx
```

2) APK'yı Java koduna dönüştürün.

```bash
jadx -d output_folder uygulama.apk
```

3) Grafik arayüzünü kullanın.

```bash
jadx-gui uygulama.apk
```

---

### MobSF

MobSF (Mobile Security Framework), mobil uygulamalar için otomatik güvenlik analizi yapan bir framework'tür.

**Kullanım Alanları**

- Güvenlik açıklarını tespit eder.
- Native ve Smali kodlarını analiz eder.
- Sertifikaları, API anahtarlarını ve şüpheli bağlantıları bulur.
- İzinleri, kullanılan kütüphaneleri ve güvenlik kontrollerini analiz eder.

**Kurulum ve Kullanım**

1)  Depoyu klonlayın.

```bash
git clone https://github.com/MobSF/Mobile-Security-Framework-MobSF.git
```

2) Proje dizinine girin.

```bash
cd Mobile-Security-Framework-MobSF
```

3) Gerekli paketleri yükleyin.

```bash
pip install -r requirements.txt
```

4) Sunucuyu başlatın.

```bash
python manage.py runserver
```

5) Tarayıcıdan aşağıdaki adrese gidin.

```text
http://127.0.0.1:8000
```

6) APK dosyanızı yükleyerek otomatik taramayı başlatın.
7) Oluşturulan raporu analiz edin.