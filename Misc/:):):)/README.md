# 🎯 :) :) :) - Misc Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Misc-Challenge-darkblue?style=for-the-badge" alt="Misc">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Steganography-purple?style=for-the-badge" alt="Steganography">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Misc  
**Seviye:** Kolay  
**Puan:** 500

**Challenge Dosyası:** [📥 Google Drive - Emoji.png](https://drive.google.com/file/d/1Rt1os3vRoLw18Jsu21dGCTwHrRZYO5de/view?usp=drivesdk)

---

## 🔍 Analiz

**Hedef:** PNG dosyasının sonuna eklenmiş (appended) gizli ELF binary'yi çıkarıp çalıştırarak flag'i almak

**Strateji:** Metadata analizi → Binwalk ile gizli dosya tespiti → ELF extraction → Password ile flag alma

**İpuçları:**
- 🎯 PNG görünümlü ama sonunda **extra data** var
- 🔍 ExifTool metadata'da **"This_Is_Not_the_Flag_but_Useful"** comment'i
- 💡 Binwalk ile **ELF 64-bit** binary tespit edilir
- 🔑 Comment aslında ELF binary'nin şifresi
- 🚨 PNG IEND chunk'ından sonra **8448 byte** gizli veri

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Dosya Tipini Kontrol Et**
```bash
file Emoji.png
```

**Çıktı:**
```
Emoji.png: PNG image data, 480 x 480, 8-bit/color RGBA, non-interlaced
```

**Analiz:**
- ✅ Normal PNG görünüyor
- 💡 Ama steganography olabilir

---

### 📄 **Adım 2: Metadata Analizi**
```bash
exiftool Emoji.png
```

**Çıktı:**
```
ExifTool Version Number         : 13.44
File Name                       : Emoji.png
File Size                       : 88 kB
...
Comment                         : This_Is_Not_the_Flag_but_Useful
Warning                         : [minor] Trailer data after PNG IEND chunk
```

**Analiz:**
- 🚨 **Comment:** "This_Is_Not_the_Flag_but_Useful" → Önemli ipucu!
- 🔍 **Warning:** Trailer data → PNG sonunda extra veri var
- 💡 Bu comment muhtemelen bir şifre olabilir

---

### 🔨 **Adım 3: Binwalk ile Gizli Dosya Tespiti**
```bash
binwalk Emoji.png
```

**Çıktı:**
```
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 480 x 480, 8-bit/color RGBA, non-interlaced
154           0x9A            Zlib compressed data, best compression
79757         0x1378D         ELF, 64-bit LSB shared object, AMD x86-64, version 1 (SYSV)
```

**Analiz:**
- ✅ **Offset 79757:** ELF binary bulundu!
- 🎯 PNG'nin sonuna gizli bir executable eklenmiş
- 💡 File appending tekniği kullanılmış

---

### 🔓 **Adım 4: Zsteg ile Detaylı Analiz**
```bash
zsteg -a Emoji.png
```

**Çıktı:**
```
[?] 8448 bytes of extra data after image end (IEND), offset = 0x1378d
extradata:0         .. file: ELF 64-bit LSB pie executable, x86-64
meta Comment        .. text: "This_Is_Not_the_Flag_but_Useful"
```

**Analiz:**
- ✅ **8448 byte** extra data doğrulandı
- 🔍 ELF binary'nin boyutu: 8448 byte
- 💡 Comment tekrar görünüyor → şifre olma ihtimali yüksek

---

### 📦 **Adım 5: ELF Dosyasını Çıkar**
```bash
dd if=Emoji.png of=hidden_elf bs=1 skip=79757
```

**Çıktı:**
```
8448+0 records in
8448+0 records out
8448 bytes (8.4 kB, 8.2 KiB) copied, 0.00495815 s, 1.7 MB/s
```

**Analiz:**
- ✅ `dd` komutu ile offset 79757'den itibaren veri çıkarıldı
- 💡 `bs=1` → byte by byte okuma
- 💡 `skip=79757` → PNG kısmını atla, sadece ELF'i al

**Alternatif: Binwalk extraction**
```bash
binwalk -e Emoji.png
cd _Emoji.png.extracted/
```

---

### 🎯 **Adım 6: ELF Dosyasını Kontrol Et**
```bash
file hidden_elf
```

**Çıktı:**
```
hidden_elf: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), 
dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, 
for GNU/Linux 3.2.0, BuildID[sha1]=b69c1cf0ad3c9bc837a7e23139f321fc81238045, 
not stripped
```

**Analiz:**
- ✅ Çalıştırılabilir ELF binary
- 💡 `not stripped` → debug semboller var
- 🔍 Linux executable olarak çalıştırılabilir

---

### 🔑 **Adım 7: Binary'yi Çalıştır**
```bash
chmod +x hidden_elf
./hidden_elf
```

**Çıktı:**
```
Say the password:
```

**Analiz:**
- 🚨 Şifre istiyor!
- 💡 ExifTool'daki Comment hatırla: **"This_Is_Not_the_Flag_but_Useful"**

---

### 🚩 **Adım 8: Şifreyi Gir ve Flag Al**
```bash
./hidden_elf
```

**Input:**
```
This_Is_Not_the_Flag_but_Useful
```

**Çıktı:**
```
FLAG{APPENEDING_FILES_REALLY!!}
```

**Analiz:**
- ✅ Flag bulundu! 🎉
- 💡 Metadata'daki comment gerçekten şifreymiş
- 🎯 "APPENEDING" (typo: APPENDING) → File appending tekniği referansı

---

## 🚩 **FLAG**
```
FLAG{APPENEDING_FILES_REALLY!!}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔧<br><b>file</b><br><sub>File type detection</sub></td>
<td align="center">📸<br><b>exiftool</b><br><sub>Metadata extraction</sub></td>
<td align="center">🔍<br><b>binwalk</b><br><sub>Hidden file detection</sub></td>
<td align="center">🎨<br><b>zsteg</b><br><sub>PNG steganography</sub></td>
<td align="center">💾<br><b>dd</b><br><sub>Binary extraction</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**
```
🎯 :) :) :) Challenge
       ↓
🔎 Dosya tipini kontrol et
   → file Emoji.png → PNG image
       ↓
📄 Metadata analizi
   → exiftool → Comment: "This_Is_Not_the_Flag_but_Useful"
   → Warning: Trailer data after PNG IEND chunk
       ↓
🔨 Gizli dosya tespiti
   → binwalk → Offset 79757: ELF 64-bit binary!
   → zsteg → 8448 bytes extra data
       ↓
📦 ELF dosyasını çıkar
   → dd if=Emoji.png of=hidden_elf bs=1 skip=79757
   → 8448 bytes extracted
       ↓
🎯 ELF kontrolü ve çalıştırma
   → file hidden_elf → ELF executable
   → chmod +x hidden_elf
   → ./hidden_elf → "Say the password:"
       ↓
🔑 Metadata'daki şifreyi kullan
   → Input: This_Is_Not_the_Flag_but_Useful
       ↓
🚩 Flag alındı
   → FLAG{APPENEDING_FILES_REALLY!!}
```
