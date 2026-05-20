# 🎯 Paint - Stego Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Stego-Challenge-darkblue?style=for-the-badge" alt="Stego">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-PNG Steganography-purple?style=for-the-badge" alt="PNG">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Stego  
**Seviye:** Orta  
**Puan:** 400

**Challenge Dosyası:** [📥 Google Drive - PaintAdam.png](https://drive.google.com/file/d/1B70dg38fLuYQfe9FTiTnSLauQmfxQ3G3/view?usp=drivesdk)

---

## 🔍 Çözüm

### Adım 1: Dosya Tipini Kontrol Etme

```bash
# Dosya tipini kontrol et
file PaintAdam.png
```

**Çıktı:**
```
PaintAdam.png: PNG image data, 1200 x 800, 8-bit/color RGB, non-interlaced
```

---

### Adım 2: Strings Analizi

```bash
# Dosyadaki string'leri incele
strings PaintAdam.png
```

**Gözlemler:**
- İki tane `IHDR` header var
- İki tane `IEND` chunk var
- Normal bir PNG'de sadece birer tane olmalı

**Analiz:** Dosyada birden fazla PNG image gizlenmiş olabilir

---

### Adım 3: Binwalk ile Hidden Files Tespiti

```bash
# Gizli dosyaları tespit et
binwalk PaintAdam.png
```

**Çıktı:**
```
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 1200 x 800, 8-bit/color RGB, non-interlaced
41            0x29            Zlib compressed data, default compression
13676         0x356C          PNG image, 800 x 400, 8-bit/color RGB, non-interlaced
13717         0x3595          Zlib compressed data, default compression
```

**Analiz:** İki PNG image tespit edildi:
- **0x0:** 1200x800 (ana resim)
- **0x356C:** 800x400 (gizli resim)

---

### Adım 4: Exiftool ile Metadata İncelemesi

```bash
# Metadata kontrol et
exiftool PaintAdam.png
```

**Önemli Çıktı:**
```
Warning                         : [minor] Trailer data after PNG IEND chunk
Image Size                      : 1200x800
```

**Gözlem:** IEND chunk'ından sonra ekstra veri var - bu gizli PNG'nin bulunduğu yer

---

### Adım 5: Pngcheck ile Doğrulama

```bash
# PNG integrity check
pngcheck -v PaintAdam.png
```

**Çıktı:**
```
File: PaintAdam.png (19017 bytes)
  chunk IHDR at offset 0x0000c, length 13
    1200 x 800 image, 24-bit RGB, non-interlaced
  chunk IDAT at offset 0x00025, length 13619
    zlib: deflated, 32K window, default compression
  chunk IEND at offset 0x03564, length 0
  additional data after IEND chunk
ERRORS DETECTED in PaintAdam.png
```

**Analiz:** IEND chunk'ından sonra ek veri tespit edildi

---

### Adım 6: Foremost ile Gizli Dosyayı Çıkarma

```bash
# Gizli dosyaları extract et
foremost PaintAdam.png -o output
```

**Çıktı:**
```
Processing: PaintAdam.png
|*|
```

**Dizin yapısı:**
```bash
cd output/png
ls
# 00000000.png  (ana resim)
# 00000026.png  (gizli resim)
```

---

### Adım 7: Gizli PNG'yi Görüntüleme

```bash
# Gizli resmi aç
cd output/png
xdg-open 00000026.png
```

**Resimde görünen:**
```
Flag{stegozormuyaliuzunzamanoldu}
```

---

## 🚩 Flag

```
Flag{stegozormuyaliuzunzamanoldu}
```

---

## 🛠️ Kullanılan Araçlar

- **file** - Dosya tipi tespiti
- **strings** - String extraction
- **binwalk** - Hidden file detection
- **foremost** - File carving
- **exiftool** - Metadata analysis
- **pngcheck** - PNG integrity check

---

## 📊 Saldırı Akış Şeması

```
🎯 Paint Challenge
       ↓
📦 PNG dosyasını aç
   → 1200x800, 8-bit RGB
       ↓
🔍 Strings analizi
   → 2x IHDR, 2x IEND bulundu
       ↓
🔎 Binwalk taraması
   → 0x0: PNG 1200x800
   → 0x356C: PNG 800x400
       ↓
📋 Exiftool kontrolü
   → Warning: Trailer data after IEND
       ↓
✅ Pngcheck validation
   → Additional data detected
       ↓
🔓 Foremost extraction
   → output/png/00000000.png (ana)
   → output/png/00000026.png (gizli)
       ↓
🖼️ Gizli PNG'yi aç
   → Flag görünür
       ↓
🚩 Flag{stegozormuyaliuzunzamanoldu}
```
