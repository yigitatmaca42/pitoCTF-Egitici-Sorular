# 🎯 Selman - Stego Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Stego-Challenge-darkblue?style=for-the-badge" alt="Stego">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-LSB Steganography-purple?style=for-the-badge" alt="LSB Stego">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Stego  
**Seviye:** Kolay  
**Puan:** 500

**Challenge Dosyası:** [📥 Google Drive - selman.png](https://drive.google.com/file/d/1aT3Nm7xtBsWInDvbs-HL_BllVmmxCnFB/view?usp=drive_link)

**Beklenen Flag Formatı:** `Flag{...}`

---

## 🔍 Analiz

**Hedef:** PNG görüntüsünün bit plane'lerinde gizlenmiş mesajı bulmak

**Strateji:** File analysis → LSB (Least Significant Bit) steganography → Bit plane browsing → Flag

**İpuçları:**
- 🎯 PNG dosyası → LSB steganography kullanılabilir
- 🔍 Bit plane analysis → Her renk kanalının her biti ayrı ayrı incelenir
- 💡 RGB kanalları: Red, Green, Blue (her biri 0-7 bit)
- 🎨 LSB (Least Significant Bit) → En düşük değerlikli bit (0. bit)
- 🌐 StegOnline web aracı → Online bit plane viewer

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Dosya Tipini Kontrol Et**
```bash
file selman.png
```

**Çıktı:**
```
selman.png: PNG image data, 343 x 450, 8-bit/color RGB, non-interlaced
```

**Analiz:**
- ✅ Geçerli bir PNG dosyası
- 💡 Boyut: 343x450 piksel
- 🎨 8-bit/color RGB → Her piksel 3 byte (Red, Green, Blue)
- 📊 Non-interlaced → Standart PNG formatı
- 🎯 Steganography için uygun format

---

### 📸 **Adım 2: Metadata Kontrolü**
```bash
exiftool selman.png
```

**Çıktı:**
```
ExifTool Version Number         : 13.44
File Name                       : selman.png
File Type                       : PNG
MIME Type                       : image/png
Image Width                     : 343
Image Height                    : 450
Bit Depth                       : 8
Color Type                      : RGB
```

**Analiz:**
- ✅ Normal PNG metadata
- 🚨 Gizli comment veya text chunk yok
- 💡 Başka yöntemler denemeli

---

### 🔨 **Adım 3: Binwalk ile Embedded Dosya Kontrolü**
```bash
binwalk selman.png
```

**Çıktı:**
```
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 343 x 450, 8-bit/color RGB, non-interlaced
```

**Analiz:**
- ✅ Sadece PNG dosyası bulundu
- 🚨 Embedded ZIP, RAR veya başka dosya yok
- 💡 Steganography bit plane'lerde olabilir

---

### 🌐 **Adım 4: StegOnline ile Bit Plane Analizi**

**1. StegOnline'a Git:**
```
URL: https://georgeom.net/StegOnline/upload
```

**2. Dosyayı Yükle:**
- 📤 "Browse" butonuna tıkla
- 📁 `selman.png` dosyasını seç
- ⬆️ "Go!" butonuna tıkla

**3. Bit Plane Browser'ı Aç:**
- 🎨 Sol menüden **"Browse Bit Planes"** seçeneğini tıkla
- 💡 Bu araç, her renk kanalının her bitini ayrı ayrı gösterir

**Analiz:**
- ✅ StegOnline web tabanlı LSB analiz aracı
- 💡 Bit plane viewing → Her bit katmanını görselleştirir
- 🎯 Gizli mesajlar genelde düşük bit'lerde saklanır (LSB)

---

### 🎨 **Adım 5: Bit Plane'leri Tara**

StegOnline'da bit plane'leri tek tek kontrol ediyoruz:

**Red Channel:**
```
Red 0 → Normal 
Red 1 → Normal 
Red 2 → Normal 
...
Red 7 → Normal 
```

**Green Channel:**
```
Green 0 → Normal 
Green 1 → Normal 
Green 2 → Normal 
...
Green 7 → Normal 
```

**Blue Channel:**
```
Blue 0 → 🚩 FLAG BULUNDU!
```

**Görsel:**
```
Blue 0 bit plane'de:
┌─────────────────────────────────────┐
│                                     │
│  Flag{HappyBirthdayMultiT00LS4LM4N} │
│                                     │
└─────────────────────────────────────┘
```

**Analiz:**
- ✅ **Blue channel'ın 0. bit'inde** (LSB) flag bulundu!
- 💡 LSB (Least Significant Bit) steganography tekniği
- 🎯 Blue 0 → Mavi renk kanalının en düşük değerlikli biti
- 🎉 "Happy Birthday MultiT00LS4LM4N" → Doğum günü mesajı!

---

### 🚩 **Adım 6: Flag'i Oku**

StegOnline'da Blue 0 bit plane'de açıkça görünen flag:

```
Flag{HappyBirthdayMultiT00LS4LM4N}
```

---

## 🚩 **FLAG**
```
Flag{HappyBirthdayMultiT00LS4LM4N}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>file</b><br><sub>File type detection</sub></td>
<td align="center">📸<br><b>exiftool</b><br><sub>Metadata extraction</sub></td>
<td align="center">🔨<br><b>binwalk</b><br><sub>Embedded file analysis</sub></td>
<td align="center">🌐<br><b>StegOnline</b><br><sub>Bit plane viewer</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**
```
🎯 Selman Challenge
       ↓
🔍 Dosya tipini kontrol et
   → file selman.png
   → PNG image data, 343 x 450, RGB
       ↓
📸 Metadata analizi
   → exiftool selman.png
   → Normal PNG metadata
   → Gizli bilgi yok
       ↓
🔨 Embedded dosya kontrolü
   → binwalk selman.png
   → Sadece PNG, başka dosya yok
       ↓
🌐 StegOnline'a yükle
   → https://georgeom.net/StegOnline/upload
   → Dosyayı upload et
       ↓
🎨 Bit Plane Browser
   → "Browse Bit Planes" seçeneğini aç
   → Red 0-7 → Normal
   → Green 0-7 → Normal
   → Blue 0 → 🚩 FLAG BULUNDU! ✅
       ↓
🚩 Flag'i oku
   → Blue 0 bit plane'de açıkça görünüyor
   → Flag{HappyBirthdayMultiT00LS4LM4N}
```
