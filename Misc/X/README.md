# 🎯 X - Misc Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Misc-Challenge-darkblue?style=for-the-badge" alt="Misc">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Forensics-purple?style=for-the-badge" alt="Forensics">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Misc  
**Seviye:** Kolay  
**Puan:** 500

**Challenge Dosyası:** [📥 Google Drive - X](https://drive.google.com/file/d/1Y9sMeWRZ3CSUioSL4B0P38x7d1gHidNk/view?usp=drivesdk)

---

## 🔍 Analiz

**Hedef:** Hexdump formatındaki dosyayı çözümleyip içindeki şifreli ZIP arşivini kırarak flag'e ulaşmak

**Strateji:** Hexdump → Binary dönüşümü → ZIP şifre kırma → Flag okuma

**İpuçları:**
- 🎯 Dosya **ASCII text** formatında ama aslında bir **hexdump**
- 🔍 `PK` magic bytes → ZIP arşivi işareti
- 💡 İçinde `xmen/flag.txt` şifreli dosya var
- 🕵️ Rockyou wordlist ile brute force yapılabilir
- 🐺 X-Men temalı (Wolverine referansı)

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Dosya Tipini Kontrol Et**
```bash
file x
```

**Çıktı:**
```
x: ASCII text
```

**Analiz:**
- ✅ ASCII text ama içerik hexdump formatında
- 💡 İlk satıra bakınca `504b` → "PK" → ZIP magic bytes

---

### 📄 **Adım 2: Dosya İçeriğini İncele**
```bash
cat x
```

**Çıktı:**
```
00000000: 504b 0304 0a00 0000 0000 9996 b350 0000  PK...........P..
00000010: 0000 0000 0000 0000 0000 0500 1c00 786d  ..............xm
00000020: 656e 2f55 5409 0003 4263 c45e 5263 c45e  en/UT...Bc.^Rc.^
...
00000060: 6e2f 666c 6167 2e74 7874 5554 0900 0342  n/flag.txtUT...B
...
```

**Analiz:**
- ✅ Hexdump formatı (`xxd` çıktısına benziyor)
- 🚨 `PK` başlığı → ZIP arşivi
- 💡 `xmen/flag.txt` dosya adı görünüyor
- 🔑 Arşiv muhtemelen şifreli

---

### 🔨 **Adım 3: Hexdump'ı Binary ZIP'e Çevir**
```bash
xxd -r x > archive.zip
```

**Analiz:**
- ✅ `xxd -r` → reverse hexdump (hex → binary)
- 💡 `archive.zip` olarak kaydedildi

---

### 📦 **Adım 4: ZIP İçeriğini Listele**
```bash
unzip -l archive.zip
```

**Çıktı:**
```
Archive:  archive.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
        0  2020-05-19 18:52   xmen/
       27  2020-05-19 18:52   xmen/flag.txt
---------                     -------
       27                     2 files
```

**Analiz:**
- ✅ `xmen/` klasörü + `xmen/flag.txt` dosyası
- 💡 Flag dosyası sadece **27 byte** → kısa flag
- 🔒 Şifreli olduğu için direkt açılamaz

---

### 🔓 **Adım 5: Şifreyi Kır (fcrackzip)**
```bash
fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt archive.zip
```

**Çıktı:**
```
PASSWORD FOUND!!!!: pw == password
```

**Analiz:**
- ✅ Şifre bulundu: **`password`**
- 💡 `-u` → unzip test (gerçek şifre kontrolü)
- 💡 `-D` → dictionary attack
- 💡 `-p` → wordlist path (rockyou.txt)

**Alternatif: John the Ripper**
```bash
zip2john archive.zip > hash.txt
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

**Çıktı:**
```
password         (archive.zip/xmen/flag.txt)
```

---

### 🎯 **Adım 6: Arşivi Aç**
```bash
unzip archive.zip
```

**Problem:** `unzip` ile "incorrect password" hatası!

**Çözüm: 7z kullan**
```bash
7z x archive.zip -ppassword
```

**Çıktı:**
```
7-Zip 25.01 (x64) : Copyright (c) 1999-2025 Igor Pavlov : 2025-08-03

Extracting archive: archive.zip
--
Path = archive.zip
Type = zip
Physical Size = 369

Everything is Ok

Folders: 1
Files: 1
Size:       27
Compressed: 369
```

**Analiz:**
- ✅ 7z başarıyla açtı (`unzip` bazen eski ZIP encryption'larla sorun yaşar)
- 💡 `xmen/` klasörü oluşturuldu
- 🎯 `flag.txt` çıkarıldı

---

### 🚩 **Adım 7: Flag'i Oku**
```bash
cat xmen/flag.txt
```

**Çıktı:**
```
flag{w0lv3rin3_hey_it5_m3}
```

**Analiz:**
- ✅ Flag bulundu! 🎉
- 🐺 Wolverine temalı flag (X-Men referansı)
- 💡 Leet speak kullanılmış: `w0lv3rin3`, `it5`, `m3`

---

## 🚩 **FLAG**
```
flag{w0lv3rin3_hey_it5_m3}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔧<br><b>xxd</b><br><sub>Hexdump reverse</sub></td>
<td align="center">🗜️<br><b>unzip</b><br><sub>ZIP extraction</sub></td>
<td align="center">🔓<br><b>fcrackzip</b><br><sub>ZIP password cracking</sub></td>
<td align="center">🔨<br><b>7z</b><br><sub>7-Zip archiver</sub></td>
<td align="center">🕵️<br><b>file</b><br><sub>File type detection</sub></td>
</tr>
</table>

---


## 💭 **Çözüm Akış Şeması**
```
🎯 X Challenge
       ↓
🔎 Dosya tipini kontrol et
   → file x → ASCII text
   → cat x → Hexdump formatı görünüyor
       ↓
📄 İçeriği incele
   → 504b (PK) → ZIP magic bytes
   → xmen/flag.txt → Şifreli dosya
       ↓
🔨 Hexdump'ı binary'ye çevir
   → xxd -r x > archive.zip
       ↓
📦 ZIP içeriğini listele
   → unzip -l archive.zip
   → xmen/flag.txt (27 byte)
       ↓
🔓 Şifreyi kır
   → fcrackzip -u -D -p rockyou.txt
   → Şifre bulundu: "password"
       ↓
🎯 Arşivi aç
   → unzip → incorrect password hatası!
   → 7z x archive.zip -ppassword → Başarılı!
       ↓
🚩 Flag'i oku
   → cat xmen/flag.txt
   → flag{w0lv3rin3_hey_it5_m3}
```
