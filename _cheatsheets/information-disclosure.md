---
title: Information Disclosure
order: 5
---

## TIPS

- Parametreye geçersiz bir değer verilerek hata tetiklenebilir. Uygulamanın döndürdüğü stack trace incelenerek uygulamaya ait hassas teknik bilgiler elde edilebilir.

```bash
productId=1 → productId=example
```

- HTML yorumları veya Site Map üzerinden gizli debug endpoint'i keşfedilebilir. `/cgi-bin/phpinfo.php` gibi debug sayfaları incelendiğinde hassas bilgiler açığa çıkabilir.

```
Debug endpoint → phpinfo() → SECRET_KEY
```

- `/robots.txt` incelenerek gizli dizinler keşfedilebilir. Bu dizinler incelendiğinde kaynak kod ve içerisinde hard-coded veritabanı parolası gibi hassas bilgiler açığa çıkabilir.
- Login request'i Repeater'a gönderilip `/admin` endpoint'i test edilir. `TRACE` isteği ile `X-Custom-IP-Authorization` header'ı keşfi ile değer `127.0.0.1` olarak değiştirilerek admin paneline erişim sağlanabilir.
- `/.git` dizini erişilebilir olduğunda Git commit geçmişi incelenebilir. Eski commitlerde kaldırılmış hassas bilgiler `git log -p` ile tekrar elde edilebilir.