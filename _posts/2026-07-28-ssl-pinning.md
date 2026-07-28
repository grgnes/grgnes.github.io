---
layout: post
title: "SSL Pinning Nedir? Frida ile Bypass Örnekleri"
date: 2026-07-28

category: Research

image: /assets/posts/frida/frida.png

excerpt: Bu yazıda, Android cihazlardaki SSL pinning mekanizmasının ne olduğunu ve bu mekanizmanın Frida aracı kullanılarak nasıl hook'lanıp bypass edilebileceğini detaylı bir şekilde ele aldım.
tags: 
- Mobile Security
- Application Security
- SSL Pinning
- Frida

---

## SSL Pinning Nedir? Frida ile Bypass Örnekleri

Android uygulamalarımız, arka planda iletişim kurduğu API sunucularıyla veri alışverişi yaparken güvenliği sağlamak için HTTPS protokolünü kullanmaktadır. Öncelikle HTTPS protokolünün çalışma mantığına bakacak olursak,

<img src="/assets/posts/ssl-pinning/ssl.png" width="500" >

HTTPS, HTTP protokolünün SSL/TLS ile şifrelenmiş halidir. İlk olarak TCP üçlü el sıkışması (3-way handshake) ile bağlantı kurulur, ardından SSL/TLS katmanı devreye girerek iletişim güvenliğini sağlar. 

Kısaca,

> HTTPS = HTTP+ SSL/TLS
> 

Peki SSL/TLS nedir?

Internet üzerindeki iletişimi şifreleyerek güvenliği hale getiren protokoldür. SSL ilk olarak Netscape Communications tarafından geliştirilmiş, sonra TLS olarak güncellenmiştir. Günümüzde ise SSL/TLS olarak ifade edilmektedir. SSL/TLS, client (android uygulama) ile server (API) arasında gerçekleşen iletişimi şifreler.

Nasıl Çalışır?

1. Client server’e bağlanmak ister ve bir istek gönderir.
2. Server, kendi dijital sertifikasını ve public key’inin gönderir.
3. Client, gönderilen dijital sertifikanın doğruluğun kontrol eder. Bir session key üretir ve bunu server'ın public key’i ile asimetrik olarak şifreler, sonra server’a gönderir.
4. Server, kendi private key’iyle şifreli session key’i çözer ve artık client ile aynı session key’e sahip olur.
5. İki tarafta aynı session key’e sahiptir. Tüm iletişim session key ile (yani simetrik şifreleme) üzerinden devam eder.

SSL/TLS ‘i anladığımıza göre şimdi gelelim SSL Pinning’e.

<img src="/assets/posts/ssl-pinning/SSL-Pinning.png" width="500" >

Client, serverdan gelen ve geçerliliğini kontrol ettiği her sertifikaya güvenir. SSL Pinning bu riski ortadan kaldırmak için kullanılır.

SSL Pinning, belirli bir sertifika veya public key’in istemci (client) uygulamasına önceden sabitlenmesi (pinlenmesi) işlemidir. Bağlantı kurulduğunda gelen sunucu sertifikası, istemcide sabitlenmiş olan ile karşılaştırılır. Eğer eşleşme varsa iletişim devam eder, değilse bağlantı reddedilir. Böylece, istemci sadece belirli bir sunucuya güvenmiş olur ve Man-in-the-Middle (MitM) gibi saldırılar engellenmiş olur.

SSL Pinning olduğunu nasıl anlarım?

1)  Kodda özel sertifika veya public key var mı?
Uygulamada `res/raw/`, `assets/` veya `raw` klasöründe `.crt`, `.pem`, `.cer` gibi **sertifika dosyaları** varsa bunların içerisinde sertifika eklenmiş olabilir. → app/src/main/res/raw/my_cert.crt

2) Kodda sertifika doğrulaması yapılıyor mu?
Kod içerisinde bazı sınıflar veya metotlar varsa örneğin, X509TrustManager gibi. 

```java
// bu örnekte raw klasöründen sertifika alınıyor.

CertificateFactory cf = CertificateFactory.getInstance("X.509");
InputStream caInput = context.getResources().openRawResource(R.raw.my_cert);
Certificate ca = cf.generateCertificate(caInput);
```

- X.509 formatında sertifika işlemek için bir `CertificateFactory` oluşturuyoruz.
- Uygulamanın kaynaklarından `my_cert` adlı sertifika dosyasını okuyoruz.
- Okuduğumuz dosyadan gerçek bir `Certificate` nesnesi oluşturuyoruz.

3) OkHttp kullanıyor mu? 
    
    Uygulama OkHttp kullanıyorsa ve CertificatePinner nesnesi varsa public key pinning yapılıyor demektir.
    
    OkHttpClient: Android'de internete bağlanmak için kullanılan bir HTTP istemcisidir. 
    
    CertificatePinner : OkHttp içinde gelen bir özellik, **sertifika pinleme** yapmamızı sağlar.
    
    ```java
    //Bu örnekte public key pinning yapılıyor.
    
    CertificatePinner certificatePinner = new CertificatePinner.Builder()
        .add("example.com", "sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=")
        .build();
    
    OkHttpClient client = new OkHttpClient.Builder()
        .certificatePinner(certificatePinner)
        .build();
    ```
    

4) Network Security Config var mı?
    
    res/xml/network_security_config.xml dosyasında özel CA belirtilmiş olabilir.
    
    ```java
    // xml dosyası ile pinning yapılmış.
    
    <network-security-config> //Android’e özel bir güvenlik yapılandırmasının başladığını söyler.
        <domain-config> //  Belirli bir domain (örneğin example.com) için ayar yapacağım demek.
            <domain includeSubdomains="true">example.com</domain> 
            <pin-set expiration="2025-01-01">
                <pin digest="SHA-256">abc123456789=</pin>
            </pin-set>
        </domain-config>
    </network-security-config>
    ```
    
    ```xml
    <application
        android:networkSecurityConfig="@xml/network_security_config"
        android:usesCleartextTraffic="false" //HTTP gibi şifrelenmemiş bağlantıları da engeller
        ... >
    
    ```
    

## Frida ile SSL Pinning Bypass Genel Yapısı Örnekleri

1. Uygulamanın TLS bağlantısının yaptığı sertifika doğrulamasını tamamen etkisiz hale getirmek ve sahte sertifikayı geçerliymiş gibi göstermek:

```java
Java.perform(function () {
    // TrustManager sınıfına erişiliyor
    var X509TrustManager = Java.use("javax.net.ssl.X509TrustManager");

    // Uygulamanın sertifika doğrulama görevini devralacak kendi sınıfımızı oluşturuluyor
    var FakeTrustManager = Java.registerClass({
        name: "com.example.FakeTrustManager", // sahte sınıfa verilen isim
        implements: [X509TrustManager], // X509TrustManager arayüzünü uyguluyor
        methods: { // TrustManager da gerekli metotlar override ediliyor
            checkClientTrusted: function (chain, authType) {}, // client tarafının sertifika kontrolünü yapar.
            checkServerTrusted: function (chain, authType) { // Sunucudan gelen sertifikayı doğrulayan metodun içini boşaltılıyor
                console.log("[+] checkServerTrusted bypass edildi");
            },
            
            // Uygulamanın hangi sertifika otoritelerine (CA) güveneceğini belirten metot
            getAcceptedIssuers: function () {
                return [];
            }
        }
    });
});

```

2) SSL bağlantısını yapılandırmak için TrustManager atamak ve pinning mekanizmasını aktifleştirmek

```java
Java.perform(function () {
    // SSLContext sınıfı çağırılır ( TLS/SSL bağlantısını başlatmak için kullanılır)
    var SSLContext = Java.use("javax.net.ssl.SSLContext");

    // init metodu ile diziler override edilir çünkü hangisine müdahale edeceğimizi bilmezsek hata verir
    SSLContext.init.overload(
        "[Ljavax.net.ssl.KeyManager;", // keyManager dizisi
        "[Ljavax.net.ssl.TrustManager;", // TrustManager dizisi
        "java.security.SecureRandom"  // Rastgelelik sağlayan yapı
    )
    // init() metodu override edilir
    .implementation = function (km, tm, sr) { 
        console.log("[+] SSLContext.init() override edildi");
        //tm yani TrustManager parametresi olarak uygulamanın orijinal TrustManager'ını değil, bizim sahte (Fake) TrustManager'ımızı verilir
        this.init(km, [Java.use("com.example.FakeTrustManager").$new()], sr); 
    };
});

```

- `km`: key manager (şifreleme anahtarları)
- `tm`: uygulamanın sertifika doğrulayıcıları (pinning buradadır!)
- `sr`: rastgelelik sağlayan sınıf

3) OkHttp kullanan uygulamalarda SSL Pinning’i devre dışı bırakmak, CertificatePinner.check() fonksiyonunu devre dışı bırakmak

Bu yapıda try-catch kullanıldı çünkü bazı uygulamalar OkHttp kullanmayabilir bu durumda hata alınabilir bu yüzden script kırılsın istenmez.

```java
Java.perform(function () {
    try {
        // OkHttp kütüphanesinin SSL pinning yapan sınıf alınır
        var CertificatePinner = Java.use("okhttp3.CertificatePinner");
        //CertificatePinner sınıfındaki check(domain, certificateList) fonksiyonunu boş hale getiriliyor yani uygulama sertifikayı kontrol edilmiyor
        CertificatePinner.check.overload("java.lang.String", "java.util.List").implementation = function (hostname, peerCertificates) {
            console.log("[+] OkHttp CertificatePinner bypass edildi: " + hostname);
            return;
        };
    } catch (err) {
        console.log("[-] CertificatePinner sınıfı bulunamadı");
    }
});

```

4) Sunucu sertifikasındaki hostname ile bağlanılmak istenen domain’in eşleşip eşleşmediğini kontrol ermek

```java
Java.perform(function () {
    // SSL/TLS bağlantılarda hostname doğrulaması yapan arayüzünü çağırır
    var HostnameVerifier = Java.use("javax.net.ssl.HostnameVerifier");

    //Bu kısımda bağlanılmak istenen hostname ile sertifikadaki CN eşleniyor mu diye kontrol ediliyor
    HostnameVerifier.verify.implementation = function (hostname, session) {
        console.log("[+] HostnameVerifier bypass edildi: " + hostname);
        return true; // her zaman onaylanır
    };
});

```