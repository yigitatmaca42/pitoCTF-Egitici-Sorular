# 🎯 Pide - Stego Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Stego-Challenge-darkblue?style=for-the-badge" alt="Stego">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Image Steganography-purple?style=for-the-badge" alt="Steganography">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Stego  
**Seviye:** Kolay  
**Puan:** 200

**Challenge Dosyası:** [📥 Google Drive - Pide](https://drive.google.com/file/d/1yC51URjrJpBWivDkfHgouw0qIio2xjtB/view?usp=drivesdk)

---

## 🔍 Analiz

**Hedef:** JPEG dosyasının içine gizlenmiş flag'i bulmak

**Strateji:** String extraction → Metadata analysis → ZIP extraction → Flag

**İpuçları:**
- 🎯 `.jpg` dosyası → Steganografi
- 🔍 Dosya içinde gizli veri olabilir
- 💡 JPEG metadata'sında bilgi saklanabilir
- 🔑 200 puan → Orta zorluk

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Strings ile İçeriği İncele**

```bash
strings Pide.jpg
```

**Çıktı (ilgili kısım):**
```
12345678934
JFIF
...
flag.txtUT
flag.txtUT
```

**KRİTİK BULGULAR:**
- ✅ En başta `12345678934` sayısı görülüyor
- ✅ En sonda `flag.txt` ismi görülüyor
- 💡 Dosyanın içinde bir ZIP arşivi gizlenmiş olabilir!
- 🎯 `flag.txt` adında bir dosya var

---

### 📦 **Adım 2: ZIP Arşivini Tespit Et**

```bash
unzip Pide.jpg
```

**Çıktı:**
```
Archive:  Pide.jpg
warning [Pide.jpg]:  282416 extra bytes at beginning or within zipfile
  (attempting to process anyway)
[Pide.jpg] flag.txt password:
   skipping: flag.txt                incorrect password
```

**ANALİZ:**
- ✅ JPEG dosyasının içine ZIP arşivi gizlenmiş! (Polyglot dosya)
- ✅ ZIP içinde `flag.txt` mevcut
- ❌ Ama şifreli! → Şifreyi bulmamız gerekiyor

---

### 🔍 **Adım 3: Metadata'yı İncele**

```bash
exiftool Pide.jpg
```

**Çıktı:**
```
ExifTool Version Number         : 13.44
File Name                       : Pide.jpg
File Type                       : JPEG
MIME Type                       : image/jpeg
Comment                         : 12345678934
JFIF Version                    : 1.01
Image Width                     : 864
Image Height                    : 486
...
```

**KRİTİK BULGULAR:**
- ✅ **JPEG Comment alanında `12345678934` gizlenmiş!**
- 💡 Bu sayı strings çıktısında da en başta görünmüştü
- 🎯 Bu ZIP şifresi olabilir!

---

### 🔓 **Adım 4: ZIP'i Şifre ile Aç**

```bash
unzip -P 12345678934 Pide.jpg
```

**Çıktı:**
```
Archive:  Pide.jpg
warning [Pide.jpg]:  282416 extra bytes at beginning or within zipfile
  (attempting to process anyway)
 extracting: flag.txt
```

```bash
cat flag.txt
```

**Çıktı:**
```
Flag{pidelersicakolsunfirinciabi}
```

**ANALİZ:**
- ✅ **FLAG BULUNDU!** 🎉🎉🎉
- 💡 JPEG Comment alanındaki `12345678934` ZIP şifresiydi
- 🚩 **Flag:** `Flag{pidelersicakolsunfirinciabi}`

---

## 🚩 **FLAG**
```
Flag{pidelersicakolsunfirinciabi}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>strings</b><br><sub>String extraction</sub></td>
<td align="center">📦<br><b>unzip</b><br><sub>ZIP extraction</sub></td>
<td align="center">📷<br><b>exiftool</b><br><sub>Metadata analysis</sub></td>
<td align="center">🐚<br><b>bash</b><br><sub>Command line</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 Pide Challenge
       ↓
🔍 strings Pide.jpg
   → "12345678934" görüldü
   → "flag.txt" görüldü
   → ZIP arşivi gizlenmiş!
       ↓
📦 unzip Pide.jpg
   → ZIP içinde flag.txt var ✅
   → Ama şifreli! ❌
       ↓
📷 exiftool Pide.jpg
   → Comment: 12345678934
   → Bu ZIP şifresi!
       ↓
🔓 unzip -P 12345678934 Pide.jpg
   → flag.txt çıkarıldı ✅
       ↓
📄 cat flag.txt
   → Flag{pidelersicakolsunfirinciabi}
       ↓
✅ FLAG: Flag{pidelersicakolsunfirinciabi}
```
