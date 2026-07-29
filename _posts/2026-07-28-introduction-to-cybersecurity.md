---
layout: post
title: "Siber Güvenliğe Giriş Notlarım"
date: 2024-12-18

category: Research

image: /assets/posts/introduction-to-cybersecurity/cover.png

excerpt: Siber güvenliğe ilk başladığımda zamanlarda çalıştığım siber güvenlik 101 notlarım

tags:
- Siber Güvenlik 
---

## Siber Güvenliğe Giriş

Hack → bir sisteme erişim izni olmadan o sisteme erişebilmek

Hacklemek terimi yasal da olabilir yasa dışı da olabilir. burada white hat ve black hat kavramları devreye giriyor

Black hat → yasa dışı bir sistemde açık arayan ve o sisteme erişen kişilerdir

White hat → yasal olarak, bir şirkette çalışan sistemde açık arayan ve o açığı kapatan daha güvenli hale getirmek için uğraşan kişilerdir. Bu kişilere etik hacker veya siber güvenlik uzmanları da denilir. 

Tüm bu terimleri içerisinde bulunduran alan ise siber güvenliktir.

Siber Güvenlik ise içerisinde 2 ye ayrılır:

Defensive tarafı → Bir şirketin tüm güvenliğini sağlayan, daha iyi nasıl güvenlik sağlanır, ağlar nasıl olucak, şirket içindeki insanlara siber güvenlik bilinci nasıl aşılanacak gibi sorular ile ilgilenen uzmanlardır. koruyucu tarafta denilebilir 

Ofansive tarafı → sistemde açık var mı, varsa nasıl erişim sağlanabilir, erişim sağlandıktan sonra sistemde neler yapılabilir gibi soruları anlamaya çalışan uzmanlardır. saldırı tarafı da denilebilir. bu tarz senaryoları öngörebilmek için yapılan saldırı simülasyonlarına da sızma testi/penetrasyon testi nedir. 

### Sızma Testleri ne içerir: Hacker Metodolojisi

1. Information Gathering → bilgi toplamak (aktif ve pasif tarama)
2. Threat Modeling → Tehditleri modellemek
3. Vulnerability Analysis → açık var mı onu analiz etmek
4. Exploitation → bulduğumuz zafiyeti kullanarak sömürülebiliniyor mu
5. Post Exploitation → erişim sağladıktan sonra neler yapılabiliyor
6. Reporting → bulguları raporlama

### Cyber Kill Chain (mitre attack)

Saldıraları daha iyi anlamamıza yarayan 7 aşamalı bir frameworktür. 

1. Keşif
2. Silahlandırma
3. İletme
4. Sömürme
5. Yükleme
6. Komuta kontrol
7. Eylemler

### Zararlı Yazılımlar (malware)

- Virüs → malware in spesifik bir türüdür, kendini diğer yazılımlara entegre ederek zarar veren yazılımlardır, kullanıcı etkileşimi ile çalışırlar yani kullanıcı bir resmi açar ve o virüs yayılır
- trojan → güvenli duran(resim,pdf gibi) zararlı yazılımlardır
- solucan (warm) → kullanıcı etkileşimine ihtiyaç duymayan ağ aracılığı ile kendini kopyalayarak yayılan zararlı yazılım. kopyalayarak çoğaldığı için sistem kaynaklarını çabuk tüketir bu da performans düşüklüğüne sebep olur
- spyware → internetten indirdiğimiz uygulamaların içerisine konulan zararlı yazılımlardır
- ronsomware → fidye yazılımları, bilgisayara indirdiğimizde bilgisayarı kitleyen para karşılığında verilen şifre ile kurtarılabilen zararlı yazılımlardır
- cryptojacking → kripto para madenciliği yapmak için kullanılan zararlı yazılımlardır, bizim adımıza kripto para üretmeye çalışır ordan kazanılan parayı da hacker a yollar
- scareware → korkutma yazılımları, sahte virüs kaldırma programları

### Tehditler

- DoS (denial of service) → kişilerin sunucuya erişimini engelleme saldırısı
- DDoS → birçok farklı yönden saldırarak, koordineli çalışarak ölü bilgisayarlar kullanarak kişilerin sunucuya erişimini engellemek
- Spoofing → kandırmak, belli bir kaynaktan istek geldiğini göstererek kandırmaktır
- Sosyal Mühendislik → psikolojik manipülasyon
- Man in the middle →  saldırganın birbiri ile doğrudan iletişim kuran iki taraf arasındaki iletişimi gizlice ilettiği veya değiştirdiği saldırı türüdür
- pasif tarama → hedef sistem ile herhangi bir şekilde etkileşime girmeden, herhangi bir iz bırakmadan yani tespit edilemeyecek şekilde bilgi toplamak.
- aktif tarama → hedef sisteme erişim sağlayarak, güvenlik sistemleri tarafından izlenebilir ayak izleri bırakarak bilgi toplamak

## Sanallaştırma

Tek bir donanım kullanarak birden fazla işletim sistemi çalıştırmak

Sanal Makine → gerçek bir bilgisayarın içinde çalışan, tamamen yazılımla oluşturulmuş başka bir bilgisayardır.

Linux 

→ Açık kaynaklı, ücretsiz bir işletim sistemi 

→ linus torvalds

→ linux distribution(dağılım) → ubuntu, debian, kali, fedora

kali linux

→ penetration testing distribution

→ sızma testinde kullanılacak tüm toolların bulunduğu işletim sistemi

→ parrot os, black arch os

---


## Network Temellerine Giriş

### Ağlara Giriş

**Ağ (Network):** Birbirleri ile iletişim kurabilen cihazların oluşturduğu yapıdır.

Bilgisayardaki ağ bilgilerini görmek için:

```
ifconfig
```

veya

```
ipconfig
```

komutları kullanılabilir.

### Ağ Arayüzleri

**eth0** → Kablolu ağ arayüzüdür.

- **inet** → IPv4 adresi (Örn: `192.168.0.1`)
- **inet6** → IPv6 adresi
- **ether** → MAC adresi

**lo (localhost)** → Bilgisayarın kendisini temsil eden sanal ağ arayüzüdür.


### NIC (Network Interface Card / Ethernet Kartı)

NIC, cihazların internete veya bir ağa bağlanmasını sağlayan donanımdır.

IP adresi ve MAC adresi gibi ağ bilgileri bu arayüz üzerinden kullanılır.


### Switch

Birden fazla cihazın aynı ağ içerisinde birbiriyle iletişim kurmasını sağlar.

Switch, cihazlar arasında yönlendirme yaparken **IP adreslerini değil, MAC adreslerini** kullanır.


### Router

Bir veya birden fazla switch'i birbirine bağlar.

Farklı ağlar (**networkler**) arasında **IP adreslerine göre yönlendirme (routing)** yapar.


### Firewall

Gelen ve giden ağ paketlerini inceler.

Belirlenen kurallara göre paketlere izin verir veya engeller.


### LAN (Local Area Network)

Yerel ağdır.

Örneğin;

- Ev ağı
- Okul ağı
- Ofis ağı


### WAN (Wide Area Network)

Birden fazla LAN'ın birleşmesiyle oluşan geniş alan ağıdır.

En büyük örneği **İnternet**'tir.


### Subnet Mask (Alt Ağ Maskesi)

IP adresini **Network ID** ve **Host ID** olarak ayırmayı sağlayan yapıdır.

Hangi kısmın ağı (**Network ID**), hangi kısmın cihazı (**Host ID**) temsil ettiğini belirler.


### Network ID

Aynı ağdaki cihazlarda ortak olan IP adresi kısmıdır.

Örneğin:

```
192.168.32.x
```

Burada:

**Network ID → 192.168.32**

Aynı ağdaki tüm cihazlarda aynıdır.

### Host ID

Ağ içerisindeki cihazı temsil eden bölümdür.

Örneğin:

```
192.168.32.170
```

Burada:

**Host ID → 170**

Aynı ağdaki her cihazın **Host ID'si farklı olmak zorundadır.**

--- 

## OSI Modeli

<img src="/assets/posts/introduction-to-cybersecurity/image.png" width="700" >

### 1. Physical (Fiziksel Katman)

Donanım katmanıdır.

- Fiziksel cihazlar
- Kablolar
- Bitlerin iletimi

### 2. Data Link (Veri Bağlantı Katmanı)

Veri transferini sağlayan katmandır.

- MAC adresi kullanılır.
- Aynı ağ içerisindeki cihazlar arasında iletişim sağlar.

### 3. Network (Ağ Katmanı)

Verilerin farklı ağlar arasında iletilmesini sağlar.

- IP adresleri kullanılır.
- Yönlendirme (Routing) işlemleri burada yapılır.

### 4. Transport (Ulaşım Katmanı)

Verinin uygulamalara güvenli şekilde ulaştırılmasını sağlar.

- TCP
- UDP

### 5. Session (Oturum Katmanı)

Cihazlar arasındaki bağlantıları (oturumları) oluşturur, yönetir ve sonlandırır.

Örneğin WAN bağlantıları.

### 6. Presentation (Sunum Katmanı)

Verilerin uygulamaların kabul edeceği hâle dönüştürülmesini sağlar.

Örneğin:

- JPG
- PNG

### 7. Application (Uygulama Katmanı)

Kullanıcının doğrudan etkileşim kurduğu katmandır.

---

## Ağ Protokolü

Ağ protokolleri, cihazların birbiriyle nasıl iletişim kuracağını belirleyen **kurallar ve kural setleridir.**

### TCP

TCP daha yavaştır.

Çünkü gönderdiği verinin karşı tarafa ulaşıp ulaşmadığını kontrol eder.

Bağlantı kurulurken:

- SYN
- SYN-ACK
- ACK

paketleri kullanılır.

### UDP

UDP daha hızlıdır.

Ancak verinin ulaşıp ulaşmadığını kontrol etmediği için TCP kadar güvenilir değildir.

Genellikle;

- Ses iletimi
- Video iletimi

gibi gecikmenin önemli olduğu alanlarda kullanılır.

### Portlar

Portlar, uygulamaların giriş kapıları gibidir.

TCP ve UDP bağlantılarının doğru uygulamaya ulaştırılması için port numaraları kullanılır.

---

## MAC / IP

### ARP

ARP, IP adresini MAC adresine çeviren protokoldür.

IP ve MAC bilgilerini görmek için:

```
arp
```

Aynı ağdaki cihazları keşfetmek için:

```
netdiscover-i eth0-r10.0.2.0/24-c100
```

Bu komut ARP istek paketleri göndererek aynı ağdaki cihazların IP ve MAC adreslerini keşfetmemizi sağlar.

### DNS

**DNS (Domain Name System)**

Domain isimleri ile IP adreslerini eşleştiren sistemdir.

Bir sitenin IP adresini görmek için:

```
ping google.com
```

gibi bir komut kullanılabilir.

## DHCP

DHCP, istemcilere otomatik IP adresi dağıtan sunucudur.

Normalde IP adresleri otomatik dağıtılır.

Ancak elle dağıtılması istenirse DHCP yapılandırılabilir.

## NAT

**NAT (Network Address Translation)**

Ağ adresi dönüştürme mekanizmasıdır.

İki tür IP adresi vardır.

- Private IP → Yerel ağlarda kullanılır.
- Public IP → İnternette görünen IP adresidir.

NAT, private ve public IP adresleri arasında dönüşüm yaparak iletişimi sağlar.

Modemde ISS tarafından atanmış bir **public IP** bulunur.

NAT, istemcinin private IP adresini modemin public IP adresi üzerinden hedefe göndererek internet erişimini sağlar.

## VPN

**VPN (Virtual Private Network)**

İki temel VPN türü vardır.

### Client-to-Site

Tek bir kullanıcının uzaktan kurumsal ağa bağlanması.

Örneğin uzaktan çalışanlar.

### Site-to-Site

İki fiziksel ağın birbirine bağlanması.

Örneğin şube ve merkez ofis bağlantıları.

VPN'e bağlanmak için örnek:

```
sudo openvpn abc.ovpn
```

### VPN bağlandıktan sonra çalışmıyorsa

İlk olarak kullanılan DNS sunucularına bakılabilir.

```
cat /etc/resolv.conf
```

DNS sunucularını değiştirmek için:

```
nano /etc/resolv.conf
```

VPN bağlandıktan sonra internet çalışmıyorsa bunun nedeni çoğu zaman **domain isimlerinin çözümlenememesidir.**

Örneğin DNS sunucusu **8.8.8.8** olarak ayarlandığında bilgisayar, **google.com** gibi alan adlarının IP adresini doğrudan Google'ın açık DNS sunucusundan öğrenmeye başlar.

İsim çözümlemesi tekrar çalışınca internet erişimi de düzelir.

Yeni IP adresi:

- whatismyip

DNS sızıntısı kontrolü:

- dnsleaktest

## Ağ İçi Saldırılar

Kullanılan bazı araçlar:

- ARP
- Nmap
- Netdiscover

## ARP Zehirlenmesi (ARP Spoofing)

Saldırgan, hem kurbana hem de router'a sahte ARP cevapları gönderir.

```
Saldırgan → Kurban:
"192.168.1.1'in MAC adresi AA:BB:CC:DD:EE:FF"

Saldırgan → Router:
"192.168.1.10'un MAC adresi AA:BB:CC:DD:EE:FF"
```

Sonuç:

- Kurban paketi router'a göndermek ister.
- Ancak router'ın MAC adresi olarak saldırganın MAC adresini bilir.
- Paket fiziksel olarak saldırgana gider.

## arpspoof

Windows (kurban) için:

```
arpspoof-i eth0-t192.168.1.1192.168.1.10
```

Router için:

```
arpspoof-i eth0-t192.168.1.10192.168.1.1
```

Hem kurbanın hem de router'ın ARP tabloları saldırganın MAC adresi ile güncellenirse **MITM (Man-in-the-Middle)** gerçekleşmiş olur.

## Wireshark

Ağ üzerinden gelen ve giden paketleri analiz etmek için kullanılır.

## IP Forwarding

IP iletiminin açık olup olmadığını kontrol etmek için:

```
cat /proc/sys/net/ipv4/ip_forward
```

Sonuç:

- **0** → Etkin değil. Bilgisayar router gibi davranmaz.
- **1** → Etkin. Bilgisayar router gibi paket iletebilir.

Bu değer **nano** ile düzenlenebilir.

Amaç, saldırganın sistemi paket iletimi yapan bir router gibi davranmasını sağlamaktır.

## Bettercap

Bettercap, ağdaki cihazları keşfetmek, dinlemek ve test etmek için kullanılan bir araçtır.

Başlatmak için:

```
bettercap-iface eth0
```

Kullanılan temel komutlar:

```
net.probe on
```

Ağdaki cihazları keşfetmeyi etkinleştirir.

```
net.show
```

Ağdaki cihazları listeler.

```
set arp.spoof.fullduplextrue
```

Saldırganın hem kurbanı hem de router'ı kandırmasını sağlar.

```
set arp.spoof.targets192.168.1.1
```

Saldırı hedefini belirler.

```
arp.spoof on
```

ARP Spoofing saldırısını başlatır (MITM).

```
net.sniff on
```

Ağ trafiğini dinlemeye başlar.

Wireshark benzeri şekilde paketleri yakalayabilir.

```
caplets.show
```

Hazır Bettercap scriptlerini (caplet) listeler.