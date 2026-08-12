---
title: Path Traversal
order: 4
---

## TIPS

- Content-Disposition → Burada sunucuya hangi form alanının gönderildiğini ve dosyanın adının ne olduğunu söylüyor.
- Content-Type → Burada gönderilen dosyanın MIME türünü söylüyor
- Normal bir resim yükle → resmin nereye kaydedildiğini bul (/uploads/image.jpg) → url’den resim açılıyor mu dene (https://test.com/uploads/image.jpg) → eğer açılıyorsa dosya web üzerinden ulaşılabilir demektir → shell.php oluştur
- **File type kontrolü yoksa** → Zararlı dosya yüklenir→ RCE alınır
- **Filename kontrolü yoksa** → Mevcut dosyanın üzerine yazılabilir
- **Path kontrolü yoksa** → Directory Traversal → İstenmeyen yere veya bir önceki dosyaya yazılabilir (../shell.php)

```bash
Content-Disposition: form-data; name="avatar"; filename="%2e%2e%2fshell.php"
Content-Type: application/x-php

<?php echo file_get_contents('/etc/passwd'); ?>
```

- **File size kontrolü yoksa** → Çok büyük dosya yükle→ DoS oluşturulabilir
- **Content-Type kontrolüne güveniliyorsa** → `Content-Type` değeri Burp ile değiştirilebilir → Dosya türü doğrulaması bypass edilebilir
- **Blacklist kullanılıyorsa** → Sadece bilinen tehlikeli uzantılar engellenir → Alternatif executable uzantı (`.php5`, `.shtml` vb.) kullanılarak **validation bypass** edilebilir
- **Server configuration override** → `.htaccess` / `web.config` gibi configuration dosyaları yüklenebilirse → Yeni bir uzantı executable MIME type ile eşleştirilebilir → **RCE**
- Extension blacklist bypass

```
php
php2
php3
php4
php5
php6
php7
phps
pht
phtm
phtml
pgif
shtml
htaccess
phar
inc
hphp
ctp
module
php
php4
php5
phtml
module
inc
hphp
ctp
```

### Obfuscation

- **Case manipulation** → `.php` yerine `.pHp` gibi farklı büyük/küçük harf kullanma
- **Multiple extensions** → `shell.php.jpg` gibi birden fazla uzantı kullanma
- **Trailing characters** → `shell.php.` gibi sona nokta/boşluk ekleme
- **URL encoding** → `shell%2Ephp` gibi karakterleri encode etme
- **Null byte / semicolon** → `shell.php%00.jpg` veya `shell.php;.jpg`
- **Unicode encoding** → Unicode karakterlerinin decode/normalize edilmesiyle `.` gibi karakterlere dönüşmesi
- **Extension stripping** → Uygulama `.php` ifadesini dosya adından siliyorsa → `exploit.p.phphp` gibi bir isimde `.php` silindikten sonra `.php` kalabilir