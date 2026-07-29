---
title: SSRF
order: 1
---

## WAF Bypass

```text
127.0.0.1
127.1
2130706433
017700000001
0x7f000001
```

## Tips

- API URL'sini içeren isteği bul.
- `http://localhost/` dene.
- `api=http://192.168.x.x` varsa IP'nin son oktetini ve dizinleri brute force ile dene.
- `Referer` üzerinden istek yönlendirmesi dene (`evil.com`) → Blind SSRF.
- Bazı IP'ler ve kelimeler blacklist'te olabilir. Sayı ve harf değişikliklerini dene (`aDmin`, `127.0.0.2`).

Blacklist backend örneği:

```python
url = request.form["stockApi"]

if "127.0.0.1" in url:
    reject()

elif "localhost" in url:
    reject()

elif "/admin" in url:
    reject()

else:
    requests.get(url)
```

- Whitelist filtrelerinde host doğrulamasını atlatmak için `@`, `#`, subdomain ve URL encoding kaynaklı URL parsing farklılıklarını test et.

Örnek:

```text
expected-host.evil.com
```

Whitelist backend örneği:

```python
if url.startswith("https://stock.example.com"):
    allow()
else:
    reject()
```

- **Open Redirect + SSRF = Whitelist Bypass**

İzin verilen domainde Open Redirect varsa önce whitelist'i geçip daha sonra iç ağa yönlenebilirsin.

```text
http://weliketoshop.net/product/nextProduct?currentProductId=6&path=http://192.168.0.68/admin
```

- Relative path (`/`) kabul ediliyor mu kontrol et.

```text
stockApi=/
```

```http
GET /product/nextProduct?currentProductId=3&path=https://google.com
```

- Whitelist bypass için `username@host`, URL encoding (`%23`, `%40`), double encoding ve parser farklılıklarını test et.
- **Blind SSRF:** URL parametrelerinin yanında `Referer`, `Host`, `X-Forwarded-*` gibi header'ları da test et.
- **Shellshock:** İç sunucuda komut çalıştırmayı doğrulamak için kullanılabilir.
- **Blind SSRF + Shellshock:** `Referer` ile SSRF yaptırıp `User-Agent` içine Shellshock payload'ı yerleştir; sonucu Burp Collaborator/OAST ile doğrula.