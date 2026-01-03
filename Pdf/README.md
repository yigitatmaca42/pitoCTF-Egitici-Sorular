# 📄 Pdf - Forensics Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Forensics-Challenge-darkblue?style=for-the-badge&logo=files" alt="Forensics">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Metadata Analysis-purple?style=for-the-badge&logo=database" alt="Metadata">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Forensics  
**Seviye:** Kolay  
**Soru No:** 48  
**Açıklama:** Flag pdfte. Çok kolay. Bakman yeter.

**Challenge Dosyası:** [📥 Google Drive - Flagnerede.pdf](https://drive.google.com/file/d/15HASjQD_Z_9miIdbe4dMNbF60AZ17j_d/view?usp=drivesdk)

---

## 🔍 Analiz

Dosyayı indirdik ve **"Flagnerede.pdf"** adında bazı yerleri şifrelenmiş mesaj geldi. Fotoğrafın **meta verilerine** bakacağız.

---

## ✅ Çözüm Adımları

### 🔍 **1. EXIF Verilerini İnceleme**

İlk başta PDF'in EXIF meta verilerine bakıyoruz:

**Komut:**
```bash
exiftool "Flagnerede.pdf"
```

**Çıktı:**
Aşağıda bir **base64** şifresi var, onu çözeceğiz.

---

### 🧩 **2. Base64 Şifresini Birleştirme**

Şifre **ikiye ayrılmış**, bunları birleştiriyoruz.

> **Not:** `=` işareti neredeyse orası son kısımdır.

**Parçalar:**

| Kısım | Değer |
|-------|-------|
| **1. Kısım (Sağ)** | `cGljb0NURntwdXp6bDNkX20zdGFkYXRhX2Yw` |
| **2. Kısım (Sol)** | `dW5kIV80MjQ0MGM3ZH0=` |

**Birleştirilmiş:**
```
cGljb0NURntwdXp6bDNkX20zdGFkYXRhX2YwdW5kIV80MjQ0MGM3ZH0=
```

---

### 🔓 **3. Base64 Decode**

Base64 şifresini decode ediyoruz:

**Komut:**
```bash
echo "cGljb0NURntwdXp6bDNkX20zdGFkYXRhX2YwdW5kIV80MjQ0MGM3ZH0=" | base64 -d
```

**Çıktı:**
```
picoCTF{puzzl3d_m3tadata_f0und!_42440c7d}
```

---

## 🚩 **FLAG**

```
picoCTF{puzzl3d_m3tadata_f0und!_42440c7d}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>exiftool</b><br><sub>Metadata extraction</sub></td>
<td align="center">🔐<br><b>base64</b><br><sub>Decoding</sub></td>
<td align="center">📄<br><b>PDF Reader</b><br><sub>Dosya inceleme</sub></td>
</tr>
</table>

**Terminal Komutları:**
```bash
# Metadata görüntüleme
exiftool "Flagnerede.pdf"

# Base64 decode
echo "cGljb0NURntwdXp6bDNkX20zdGFkYXRhX2YwdW5kIV80MjQ0MGM3ZH0=" | base64 -d
```

---

## 💭 **Çözüm Akış Şeması**

```
📥 Dosya İndirme: Flagnerede.pdf
            ↓
📄 PDF Açma: Şifrelenmiş içerik
            ↓
🔍 Meta Veri Kontrolü: exiftool
            ↓
🔎 Base64 Tespiti: İki parçaya ayrılmış
            ↓
🧩 Parça 1: cGljb0NURntwdXp6bDNkX20zdGFkYXRhX2Yw
            ↓
🧩 Parça 2: dW5kIV80MjQ0MGM3ZH0=
            ↓
🔗 Birleştirme: = işareti son kısım
            ↓
🔓 Base64 Decode: echo | base64 -d
            ↓
🚩 FLAG: picoCTF{puzzl3d_m3tadata_f0und!_42440c7d}
```

---

## 🔐 **Base64 Cheat Sheet**

```bash
# Encode
echo "Hello" | base64
# SGVsbG8K

# Decode
echo "SGVsbG8K" | base64 -d
# Hello

# Dosyadan encode
base64 file.txt > encoded.txt

# Dosyadan decode
base64 -d encoded.txt > decoded.txt
```
