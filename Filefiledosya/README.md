# 📁 Filefiledosya - Forensics Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Forensics-Challenge-darkblue?style=for-the-badge&logo=files" alt="Forensics">
  <img src="https://img.shields.io/badge/Difficulty-Easy-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-File%20Repair-purple?style=for-the-badge&logo=wrench" alt="File Repair">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Forensics  
**Seviye:** Easy  
**Soru No:** 49  
**Açıklama:** Bu dosyayı ben açamadım sen açabilir misin?

**Challenge Dosyası:** [📥 Google Drive](https://drive.google.com/file/d/1EkPCflGHdSYQ5fQsb5P1Jl2ADkjHaTX7/view?usp=drivesdk)

---

## 🔍 Analiz

### **1. Dosya Türü Tespiti**

İndirdiğimiz dosyanın türünün ne olduğunu öğreniyoruz:

**Komut:**
```bash
file filefiledosya
```

**Çıktı:**
```
filefiledosya: data
```

❌ Dosya bozuk çünkü türü vermedi.

---

### **2. Hex Analizi**

Dosya bozuk olduğu için hex verisinden bakıyoruz:

**Komut:**
```bash
xxd filefiledosya | head
```

**Çıktı:**
Hex dump'ta **"JFIF"** yazıyor!

✅ Bu **JPG** dosyasının başlık kısmı bozuk.

---

## ✅ Çözüm Adımları

### 🔧 **1. JPG Magic Bytes Bulma**

JPG'nin başlık kısmı bozuk ve orayı düzeltiyoruz. Önce dosyada doğru magic bytes'ı buluyoruz:

**Komut:**
```bash
grep -oba $'\xff\xe0' filefiledosya
```

Bu komut JPG'nin **JFIF marker**'ını (`FF E0`) dosyada bulur.

---

### 🛠️ **2. Dosyayı Onarma**

Düzelttiğimiz dosyayı tekrar **JPG** yapıyoruz:

**Komut 1: JPG header ekleme**
```bash
printf "\xff\xd8" > repaired.jpg
```

**Komut 2: Doğru başlangıçtan itibaren veriyi kopyalama**
```bash
tail -c +$(( $(grep -oba $'\xff\xe0' filefiledosya | cut -d: -f1) + 1 )) filefiledosya >> repaired.jpg
```

**Açıklama:**
- `\xff\xd8` → JPG magic bytes (Start of Image)
- `grep -oba` → JFIF marker'ın pozisyonunu bul
- `tail -c +N` → N. byte'tan itibaren dosyayı kopyala

---

### 🖼️ **3. Onarılmış Resmi Açma**

Düzeltilen resmi açıyoruz:

**Komut:**
```bash
xdg-open repaired.jpg
```

**Resimde Yazan:**
```
picoCTF{r3st0r1ng_th3_by73s_b67c1588}
```

---

## 🚩 **FLAG**

```
picoCTF{r3st0r1ng_th3_by73s_b67c1588}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">📄<br><b>file</b><br><sub>Dosya türü tespiti</sub></td>
<td align="center">🔍<br><b>xxd</b><br><sub>Hex dump</sub></td>
<td align="center">🔎<br><b>grep</b><br><sub>Magic bytes arama</sub></td>
<td align="center">🛠️<br><b>printf/tail</b><br><sub>Dosya onarma</sub></td>
</tr>
</table>

**Terminal Komutları:**
```bash
# Dosya türü kontrolü
file filefiledosya

# Hex dump analizi
xxd filefiledosya | head

# Magic bytes bulma
grep -oba $'\xff\xe0' filefiledosya

# JPG onarma
printf "\xff\xd8" > repaired.jpg
tail -c +$(( $(grep -oba $'\xff\xe0' filefiledosya | cut -d: -f1) + 1 )) filefiledosya >> repaired.jpg

# Resmi açma
xdg-open repaired.jpg
```

---

## 💭 **Çözüm Akış Şeması**

```
📥 Dosya İndirme: filefiledosya
            ↓
📄 Dosya Türü Kontrolü: file → "data" (bozuk)
            ↓
🔍 Hex Analizi: xxd | head → "JFIF" bulundu
            ↓
✅ Dosya Türü Tespiti: JPG (bozuk header)
            ↓
🔎 Magic Bytes Arama: grep -oba '\xff\xe0'
            ↓
🛠️ JPG Header Ekleme: printf "\xff\xd8"
            ↓
📋 Veri Kopyalama: tail -c +N
            ↓
🖼️ Onarılmış Dosya: repaired.jpg
            ↓
👁️ Resmi Açma: xdg-open
            ↓
🚩 FLAG: picoCTF{r3st0r1ng_th3_by73s_b67c1588}
```

---

## 📚 **Öğrenilenler**

### **File Format Forensics**

1. **Magic Bytes (File Signatures):**
   - Her dosya formatının kendine özgü başlık byte'ları var
   - JPG: `FF D8 FF E0` (JFIF) veya `FF D8 FF E1` (EXIF)
   - PNG: `89 50 4E 47`
   - PDF: `25 50 44 46`

2. **File Command:**
   - `file` komutu magic bytes'a bakarak dosya türünü tespit eder
   - "data" çıktısı → bozuk veya bilinmeyen format

3. **Hex Analysis:**
   - `xxd` ile binary dosyaları hex formatında inceleyebiliriz
   - JFIF/EXIF marker'ları JPG içeriğini gösterir

### **JPG File Structure**

| Segment | Magic Bytes | Açıklama |
|---------|-------------|----------|
| **SOI** | `FF D8` | Start of Image |
| **APP0** | `FF E0` | JFIF marker |
| **APP1** | `FF E1` | EXIF marker |
| **EOI** | `FF D9` | End of Image |
