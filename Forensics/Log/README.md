# 🔍 Log - Forensics Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Forensics-Challenge-darkblue?style=for-the-badge&logo=files" alt="Forensics">
  <img src="https://img.shields.io/badge/Difficulty-Easy-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Base64%20%2B%20Hex-purple?style=for-the-badge&logo=lock" alt="Encoding">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Forensics  
**Seviye:** Kolay  
**Puan:** 200  
**Açıklama:** Ekibimiz şüpheli ve şifrelenmiş bir log dosyası kaydetmiş. Çözebilir misin?

**Challenge Dosyası:** [📥 Google Drive - logs.txt](https://drive.google.com/file/d/1ldJ5DBQi3Do1p49HkGH4wyorIm3k2ZFv/view?usp=drivesdk)

**Verilen:**
- logs.txt (Şifrelenmiş log dosyası)

---

## 🔍 Analiz

### Base64 ve Hex Encoding Nedir?

**Base64**, binary verileri ASCII karakterlere dönüştüren bir encoding yöntemidir. **Hex (Hexadecimal)** ise 16'lık sayı sistemiyle veriyi temsil eder.

| Özellik | Açıklama |
|---------|----------|
| 🔤 **Base64** | Binary → ASCII text encoding |
| 🔢 **Hexadecimal** | 16'lık sayı sistemi (0-9, A-F) |
| 🖼️ **PNG** | Portable Network Graphics görüntü formatı |
| 🔐 **Double Encoding** | İç içe encoding katmanları |

**Örnek:** Base64 → Binary → PNG → Hex → Text

---

## ✅ Çözüm Adımları

### 📥 **1. Dosya İndirme ve İlk İnceleme**

Dosyayı indirip içeriğine bakalım:

```bash
head -n 5 logs.txt
```

**Çıktı Örneği:**
```
iVBORw0KGgoAAAANSUhEUgAAAlgAAAJYCAIAAAAxBA+gAAAACXBIWXMAAA7EAAAOxAGV...
...
...end'de "=" karakteri var
```

> **Gözlem:** Çıktının sonunda **"="** padding karakteri var - kesinlikle **Base64** encoding!

---

### 🔍 **2. Hex Dump İncelemesi**

Soru forensics kategorisinde ve **"şifrelenmiş bir log dosyası"** diyor. Dosyanın gerçek türünü öğrenmek için hex dump'a bakalım:

```bash
base64 -d logs.txt | xxd | head
```

**Çıktı:**
```
00000000: 8950 4e47 0d0a 1a0a 0000 000d 4948 4452  .PNG........IHDR
00000010: 0000 0258 0000 0258 0806 0000 00c4 103e  ...X...X.......>
00000020: 6800 0000 0970 4859 7300 000e c300 000e  h....pHYs.......
```

> **Önemli:** Hex dump başında **".PNG"** (hex: 89 50 4E 47) yazıyor - bu bir PNG dosyası!

---

### 🖼️ **3. Base64'ü PNG'ye Çevirme**

Base64 encoded veriyi PNG dosyasına dönüştürüyoruz:

```bash
base64 -d logs.txt > log.png
```

**Açıklama:**
- `base64 -d` = Base64 decode
- `logs.txt` = Input dosyası
- `> log.png` = Output PNG dosyası

---

### 👁️ **4. PNG Dosyasını İnceleme**

Oluşan PNG dosyasını açıyoruz:

```bash
open log.png
# veya
xdg-open log.png
```

**Gözlem:**
- Bir resim görüntüsü açılıyor
- Resmin **alt kısmında** şu metin yazıyor:

```
7069636F4354467B666F72656E736963735F616E616C797369735F69735F616D617A696E675F61633165333538347D
```

> **Dikkat:** Bu bir **hexadecimal** string!

---

### 🔓 **5. Hex String'i Decode Etme**

Python ile hex string'i decode ediyoruz:

```bash
python3 -c 'print(bytes.fromhex("7069636F4354467B666F72656E736963735F616E616C797369735F69735F616D617A696E675F61633165333538347D").decode())'
```

**Çıktı:**
```
picoCTF{forensics_analysis_is_amazing_ac1e3584}
```

> **Başarılı!** Flag elde edildi!

---

## 🚩 **FLAG**

```
picoCTF{forensics_analysis_is_amazing_ac1e3584}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔤<br><b>base64</b><br><sub>Base64 decode</sub></td>
<td align="center">🔍<br><b>xxd</b><br><sub>Hex dump</sub></td>
<td align="center">🐍<br><b>Python</b><br><sub>Hex decode</sub></td>
<td align="center">👁️<br><b>Image Viewer</b><br><sub>PNG görüntüleme</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**
- 🔤 **base64** - Base64 encoding/decoding
- 🔍 **xxd** - Hex dump oluşturma
- 📂 **head** - Dosya başını görüntüleme
- 🐍 **Python** - Hex string decode

---

## 💻 **Kullanılan Komutlar**

```bash
# Dosyanın ilk 5 satırını görüntüleme
head -n 5 logs.txt

# Base64 decode + hex dump
base64 -d logs.txt | xxd | head

# Base64'ü PNG'ye çevirme
base64 -d logs.txt > log.png

# PNG dosyasını açma
open log.png
# veya
xdg-open log.png

# Hex string'i decode etme
python3 -c 'print(bytes.fromhex("7069636F4354467B666F72656E736963735F616E616C797369735F69735F616D617A696E675F61633165333538347D").decode())'
```

---

## 💭 **Çözüm Akış Şeması**

```
🔍 Log Challenge: logs.txt verildi
                    ↓
        📥 Dosyayı İndir
                    ↓
        🔍 head -n 5 logs.txt → "=" var
                    ↓
        💡 Base64 Encoding Tespit Edildi
                    ↓
        🔍 base64 -d | xxd → .PNG header
                    ↓
        🖼️ Dosya PNG formatında!
                    ↓
        🔓 base64 -d logs.txt > log.png
                    ↓
        👁️ PNG Dosyasını Aç
                    ↓
        📝 Alt Kısımda Hex String Bulundu
                    ↓
        7069636F4354467B666F72656E73696373...
                    ↓
        🐍 Python ile Hex Decode
                    ↓
        🚩 FLAG: picoCTF{forensics_analysis_is_amazing_ac1e3584}
```
