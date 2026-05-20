# 🖼️ Thumbcache - Forensics Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Forensics-Challenge-darkblue?style=for-the-badge&logo=microscope" alt="Forensics">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Windows%20Thumbcache-purple?style=for-the-badge&logo=windows" alt="Thumbcache">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Forensics  
**Seviye:** Kolay  
**Puan:** 200

**Challenge Dosyası:** [📥 Google Drive - thumbcache.db](https://drive.google.com/file/d/1gXcNzkZWssJlSn605c70bxOh1B2CqIUh/view?usp=drivesdk)


---

## 🔍 Analiz

### Challenge Özeti

**Hedef:** Windows thumbcache veritabanından gömülü resimleri çıkarıp flag'i bulmak

**Thumbcache Nedir?**
- Windows işletim sisteminin küçük resim önbellek dosyası
- Kullanıcıların görüntülediği resimlerin thumbnail'lerini saklar
- Silinen resimlerin izlerini içerebilir
- Forensics analizinde kritik kanıt kaynağı

**Strateji:** File analysis → Strings search → Image extraction → Flag discovery

---

## ✅ Çözüm Adımları

### 📥 **1. Dosyayı İndirme**

Google Drive'dan `thumbcache.db` dosyasını indiriyoruz ve Desktop'a kaydediyoruz.

**Terminal:**
```bash
cd ~/Desktop
ls -lh thumbcache.db
```

**Çıktı:**
```
-rw-rw-r-- 1 kali kali 1.0M Jan  7 00:12 thumbcache.db
```

> 💾 **Dosya Boyutu:** 1 MB

---

### 🔎 **2. İlk Analiz - Dosya Tespiti**

**Dosya Tipini Kontrol:**
```bash
file thumbcache.db
```

**Çıktı:**
```
thumbcache.db: data
```

> 💡 **Not:** Generic "data" olarak gösteriyor, daha detaylı bakmamız gerekiyor.

---

### 🔍 **3. Hex Dump Analizi**

**Hex İmzasını İnceleyelim:**
```bash
xxd thumbcache.db | head -n 20
hexdump -C thumbcache.db | head -n 5
```

**Çıktı:**
```
00000000: 434d 4d4d 2000 0000 0400 0000 0000 0000  CMMM ...........
00000010: 1800 0000 3851 0100 434d 4d4d 5800 0000  ....8Q..CMMMX...
00000020: 9232 2456 e5e5 8fb9 2000 0000 0000 0000  .2$V.... .......
00000030: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00000040: 0000 0000 0000 0000 5af2 d92d 5553 538a  ........Z..-USS.
```

> 🎯 **Kritik Bulgu:** `CMMM` signature → Bu kesinlikle **Windows Thumbcache** dosyası!

---

### 📝 **4. Strings Analizi - Unicode Tarama**

**Unicode String'leri Çıkaralım:**
```bash
# Unicode strings (Windows dosyaları için -e l parametresi önemli)
strings -e l thumbcache.db > unicode_strings.txt

# Normal ASCII strings
strings thumbcache.db > ascii_strings.txt

# Flag benzeri pattern ara
cat unicode_strings.txt | grep -i "flag\|first\|ctf"
cat ascii_strings.txt | grep -i "flag\|first\|ctf"
```

**Sonuç:** String'lerde direkt flag bulamadık, ancak hex değerler ve GUID'ler var.

---

### 🔐 **5. GUID ve Hex String Analizi**

**Unicode String'leri İnceleyelim:**
```bash
cat unicode_strings.txt | grep "{"
```

**Çıktı:**
```
::{645FF040-5081-101B-9F08-00AA002F954E}
```

**Tüm Hex String'leri:**
```bash
cat unicode_strings.txt | head -20
```

**Bulduklarımız:**
```
b98fe5e556243292
::{645FF040-5081-101B-9F08-00AA002F954E}
9fbc3a1ea0b5b4ca
4f19a6644df93971
...
```

> 💡 **Analiz:** GUID Windows klasör tanımlayıcısı ama flag değil. Hex string'ler de yeterli bilgi vermiyor.

---

### 🖼️ **6. PNG Header Tespiti**

**ASCII String'lerde PNG İmzası:**
```bash
cat ascii_strings.txt | head -20
```

**Çıktı:**
```
CMMM 
CMMMX
...
IHDR
sRGB
gAMA
```

> 🎨 **BINGO!** `IHDR`, `sRGB`, `gAMA` → **PNG dosyası header'ları!**

Bu thumbcache içinde **gömülü PNG resimleri** var!

---

### 🔧 **7. Binwalk ile Dosya Tespiti**

**Gömülü Dosyaları Tespit Edelim:**
```bash
binwalk thumbcache.db
```

**Çıktı:**
```
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
1306          0x51A           PNG image, 256 x 256, 8-bit/color RGBA, non-interlaced
1376          0x560           Zlib compressed data, compressed
26084         0x65E4          PNG image, 256 x 256, 8-bit/color RGBA, non-interlaced
26154         0x662A          Zlib compressed data, compressed
37474         0x9262          PNG image, 256 x 256, 8-bit/color RGBA, non-interlaced
37544         0x92A8          Zlib compressed data, compressed
55356         0xD83C          PNG image, 256 x 256, 8-bit/color RGBA, non-interlaced
55426         0xD882          Zlib compressed data, compressed
80032         0x138A0         JPEG image data, JFIF standard 1.01
```

> 🎯 **Tespit:** 4 PNG + 1 JPEG resim bulundu!

---

### 📂 **8. Foremost ile Dosyaları Çıkarma**

**Dosya Kurtarma İşlemi:**
```bash
# Klasör oluştur
mkdir extracted

# Foremost ile resimleri çıkar
foremost -i thumbcache.db -o extracted/

# Çıkarılan dosyalara bak
ls -la extracted/
```

**Çıktı:**
```
total 20
drwxrwxr-x  4 kali kali 4096 Jan  7 00:18 .
drwxr-xr-x 13 kali kali 4096 Jan  7 00:18 ..
-rw-rw-r--  1 kali kali  932 Jan  7 00:18 audit.txt
drwxrwxr--  2 kali kali 4096 Jan  7 00:18 jpg
drwxrwxr--  2 kali kali 4096 Jan  7 00:18 png
```

---

### 🖼️ **9. Audit Dosyasını İnceleme**

**Foremost Raporu:**
```bash
cat extracted/audit.txt
```

**Rapor İçeriği:**
```
Foremost version 1.5.7
File: thumbcache.db
Length: 1024 KB (1048576 bytes)
 
Num	 Name (bs=512)	       Size	 File Offset	 Comment 

0:	00000156.jpg 	       5 KB 	      80032 	 
1:	00000002.png 	      24 KB 	       1306 	  (256 x 256)
2:	00000050.png 	      11 KB 	      26084 	  (256 x 256)
3:	00000073.png 	      17 KB 	      37474 	  (256 x 256)
4:	00000108.png 	      24 KB 	      55356 	  (256 x 256)

5 FILES EXTRACTED
	
jpg:= 1
png:= 4
```

> ✅ **Başarılı:** 5 dosya çıkarıldı!

---

### 🏆 **10. Flag Resmini Bulma**

**PNG Klasörünü İnceleyelim:**
```bash
cd extracted/png
ls -lh
```

**Dosyalar:**
```
00000002.png  (256x256) - Windows klasör ikonları
00000050.png  (256x256) - Windows klasör ikonları
00000073.png  (256x256) - Windows klasör ikonları
00000108.png  (256x256) - Windows klasör ikonları
```

> 🤔 PNG dosyaları sadece Windows klasör ikonları.

**JPG Klasörünü İnceleyelim:**
```bash
cd ../jpg
ls -lh
```

**Dosya:**
```
00000156.jpg (5 KB)
```

**Resmi Açalım:**
```bash
xdg-open 00000156.jpg
# veya
eog 00000156.jpg
```

---

### 🚩 **11. FLAG BULUNDU!**

**Resim İçeriği:**

```
┌─────────────────────────────┐
│                             │
│   flag{human_after_all}     │
│                             │
└─────────────────────────────┘
```

> 🎉 **BAŞARILI!** JPG dosyasında flag var!

---

## 🚩 **FLAG**

```
flag{human_after_all}
```

> 🎵 **Easter Egg:** "Human After All" - Daft Punk'ın 2005 albümü!

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>file</b><br><sub>Dosya tipi tespiti</sub></td>
<td align="center">📝<br><b>strings</b><br><sub>String extraction</sub></td>
<td align="center">🔢<br><b>xxd / hexdump</b><br><sub>Hex analizi</sub></td>
</tr>
<tr>
<td align="center">🔧<br><b>binwalk</b><br><sub>Gömülü dosya tespiti</sub></td>
<td align="center">📂<br><b>foremost</b><br><sub>Dosya kurtarma</sub></td>
<td align="center">🖼️<br><b>eog / xdg-open</b><br><sub>Resim görüntüleme</sub></td>
</tr>
</table>

**Kullanılan Teknikler:**
- 🔍 **File Signature Analysis:** CMMM header tespiti
- 📝 **String Analysis:** Unicode ve ASCII string extraction
- 🔢 **Hex Analysis:** Binary dosya incelemesi
- 🔧 **Carving:** Gömülü dosya çıkarma
- 🖼️ **Image Forensics:** PNG ve JPEG kurtarma

---

## 💻 **Komut Özeti**

```bash
# 1. Dosya analizi
file thumbcache.db
ls -lh thumbcache.db

# 2. Hex dump
xxd thumbcache.db | head -n 20
hexdump -C thumbcache.db | head -n 5

# 3. String extraction
strings -e l thumbcache.db > unicode_strings.txt
strings thumbcache.db > ascii_strings.txt

# 4. Pattern arama
cat unicode_strings.txt | grep "{"
cat ascii_strings.txt | grep -i "IHDR\|PNG"

# 5. Binwalk tarama
binwalk thumbcache.db

# 6. Foremost ile extraction
mkdir extracted
foremost -i thumbcache.db -o extracted/

# 7. Çıkarılan dosyaları listeleme
ls -la extracted/
cat extracted/audit.txt

# 8. Resimleri görüntüleme
cd extracted/jpg
xdg-open 00000156.jpg
```

---

## 💭 **Çözüm Akış Şeması**

```
🖼️ Forensics Challenge: "Thumbcache"
                    ↓
        📥 thumbcache.db dosyasını indir
                    ↓
        🔍 file komutu → "data" tipinde
                    ↓
        🔢 xxd/hexdump → "CMMM" signature bulundu
                    ↓
        💡 Windows Thumbcache olduğu anlaşıldı
                    ↓
        📝 strings -e l → Unicode string'ler
                    ↓
        🔐 GUID bulundu: {645FF040-5081-101B-9F08-00AA002F954E}
                    ↓
        🎨 PNG header'ları tespit edildi (IHDR, sRGB)
                    ↓
        🔧 binwalk thumbcache.db
                    ↓
        📊 4 PNG + 1 JPEG tespit edildi
                    ↓
        📂 foremost -i thumbcache.db -o extracted/
                    ↓
        ✅ 5 dosya başarıyla çıkarıldı
                    ↓
        📁 extracted/png/ → Windows klasör ikonları
                    ↓
        📁 extracted/jpg/ → 00000156.jpg
                    ↓
        🖼️ Resmi aç → Flag görüldü!
                    ↓
        🚩 flag{human_after_all}
```
