# 🎯 İz - Misc Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Misc-Challenge-darkblue?style=for-the-badge" alt="Misc">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Steganography-purple?style=for-the-badge" alt="Steganography">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Misc  
**Seviye:** Orta  
**Puan:** 500

**Challenge Dosyası:** [📥 Google Drive - İzPesinde.rar](https://drive.google.com/file/d/10DBeeLMyoobzYBjWrjDqLLCG75dlqnsE/view?usp=drivesdk)

---

## 🔍 Analiz

**Hedef:** Çok katmanlı steganografi challenge - RAR arşivindeki resim dosyalarının birinde gizlenmiş base64 encoded veriyi bulup decode ederek ikinci bir RAR'a ulaşmak, ardından şifreyi kırarak flag'i almak

**Strateji:** RAR extraction → Multi-layer steganography → Base64 decoding → Password cracking → Flag

**İpuçları:**
- 🎯 Challenge adı **"İz"** (след/trace) → dosyaların izini takip et
- 🔍 7 resim dosyası: **1.jpg, 2.jpg, 3.jpg, 5.jpg, 6.jpg, 77.jpg, 44.png**
- 💡 Eksik numaralar: **4** ve **7** yok, ama **44** ve **77** var
- 🕵️ 3.jpg dosyasının **sonunda** (FFD9'dan sonra) gizli veri
- 🔑 İç içe encoding: Base64 → Base64 → Hex → RAR
- 🐺 RAR şifresi wordlist'te var

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: RAR Arşivini Extract Et**
```bash
unrar x İzPesinde.rar
cd İzPesinde/
ls -la
```

**Çıktı:**
```
-rw-rw-r-- 1 kali kali 220406 Aug 12  2020 1.jpg
-rw-rw-r-- 1 kali kali  81070 Aug 12  2020 2.jpg
-rw-rw-r-- 1 kali kali 133558 Aug 12  2020 3.jpg
-rw-rw-r-- 1 kali kali 904801 Aug 12  2020 44.png
-rw-rw-r-- 1 kali kali 436949 Aug 12  2020 5.jpg
-rw-rw-r-- 1 kali kali 166364 Aug 12  2020 6.jpg
-rw-rw-r-- 1 kali kali 406072 Aug 12  2020 77.jpg
```

**Analiz:**
- ✅ 7 resim dosyası çıkarıldı
- 💡 Dosya isimleri: 1, 2, 3, 5, 6, 77, 44
- 🚨 **4** ve **7** eksik ama **44** ve **77** var → İlginç pattern!

---

### 📄 **Adım 2: Steghide ile Analiz Denemeleri**
```bash
# Her dosyada steghide verisi var mı kontrol et
steghide info 1.jpg
steghide info 2.jpg
steghide info 77.jpg
```

**Çıktı:**
```
"1.jpg":
  format: jpeg
  capacity: 11.1 KB
Try to get information about embedded data ? (y/n) y
Enter passphrase: 
steghide: could not extract any data with that passphrase!
```

**Analiz:**
- ✅ Tüm JPG'lerde steghide capacity var
- 🚨 Ama boş şifre ile çıkmıyor
- 💡 Farklı bir yöntem denemeliyiz

---

### 🔨 **Adım 3: zsteg ile PNG Analizi**
```bash
zsteg 44.png | grep text
```

**Çıktı:**
```
b3,abgr,msb,xy      .. text: "}EWtEWtEw}"
```

**Analiz:**
- 🔍 PNG'de ilginç bir pattern bulundu
- 💡 Ama bu flag değil, bir ipucu olabilir
- 🎯 Başka yerlere de bakmamız lazım

---

### 📦 **Adım 4: JPEG Dosya Sonlarını Kontrol Et**

JPEG dosyaları **FFD9** ile bitmeli. Bundan sonra veri varsa appended data demektir.

```bash
# Her JPEG'in sonundaki extra byte'ları kontrol et
for f in *.jpg; do
    echo "=== $f - data after FFD9 ==="
    offset=$(grep -abo $'\xff\xd9' "$f" | tail -1 | cut -d: -f1)
    total_size=$(stat -c%s "$f")
    extra_bytes=$((total_size - offset - 2))
    echo "Extra bytes: $extra_bytes"
done
```

**Çıktı:**
```
=== 1.jpg - data after FFD9 ===
Extra bytes: 0
=== 2.jpg - data after FFD9 ===
Extra bytes: 0
=== 3.jpg - data after FFD9 ===
Extra bytes: 747
=== 5.jpg - data after FFD9 ===
Extra bytes: 0
...
```

**Analiz:**
- ✅ **3.jpg** dosyasında **747 byte** extra data bulundu!
- 🎯 Bu challenge'ın anahtarı bu olabilir
- 💡 Diğer dosyaların hepsi normal

---

### 🔓 **Adım 5: 3.jpg'den Gizli Veriyi Çıkar**
```bash
# FFD9'dan sonraki tüm veriyi çıkar
offset=$(grep -abo $'\xff\xd9' 3.jpg | tail -1 | cut -d: -f1)
dd if=3.jpg of=full_encoded.txt bs=1 skip=$((offset + 2)) 2>/dev/null

cat full_encoded.txt
```

**Çıktı:**
```
.....NTI2MTcyMjExQTA3MDEwMDg1QTg2MkI0MjEwNDAwMDAwMTBGQkRDN0NEMTc3Q0RFQjAwN0VGOTA0
OEI2NUQwNDE4OEQzRTRDQkU0MTJGOTNFMTkzNzg5QkIyQTBBMTIxQkEwREIwQUQzRDE5ODJDM0NC
RkIzNEUzQUNDRjE5RkE1MTAwNDY3Qjk0RTMxNkQyN0Y2RUM5OTJCQzUxMkYwMDM3RkFFRTI5RUVG
...
```

**Analiz:**
- ✅ Base64 encoded data!
- 🚨 Başında **5 nokta** var (padding/junk)
- 💡 Bu veriyi decode etmeliyiz

---

### 🎯 **Adım 6: İlk Base64 Decode**
```bash
# İlk 5 karakteri (noktalar) atla
tail -c +6 full_encoded.txt > clean_base64.txt

# Satır sonlarını temizle ve decode et
cat clean_base64.txt | tr -d '\r\n' | base64 -d > decoded_step1.bin

# İçeriğe bak
xxd decoded_step1.bin | head -20
```

**Çıktı:**
```
00000000: 3532 3631 3732 3231 3141 3037 3031 3030  526172211A070100
00000010: 3835 4138 3632 4234 3231 3034 3030 3030  85A862B421040000
00000020: 3031 3046 4244 4337 4344 3137 3743 4445  010FBDC7CD177CDE
...
```

**Analiz:**
- ✅ ASCII hex string çıktı!
- 🔍 `526172211A070100` → "Rar!.." → **RAR magic bytes!**
- 💡 Bu da hex encoded bir RAR dosyası

---

### 🔨 **Adım 7: Hex String'i Binary RAR'a Çevir**
```bash
xxd -r -p decoded_step1.bin > hidden_full.rar

# Kontrol et
file hidden_full.rar
ls -lh hidden_full.rar
```

**Çıktı:**
```
hidden_full.rar: RAR archive data, v5
-rw-rw-r-- 1 kali kali 270 Feb 13 10:09 hidden_full.rar
```

**Analiz:**
- ✅ Başarıyla RAR dosyası oluşturuldu!
- 💡 Boyutu sadece 270 byte
- 🔑 Muhtemelen şifreli

---

### 🔓 **Adım 8: RAR Şifresini Kır**
```bash
# John the Ripper ile hash oluştur
rar2john hidden_full.rar > rar.hash

# Rockyou wordlist ile kır
john rar.hash --wordlist=/usr/share/wordlists/rockyou.txt
```

**Çıktı:**
```
Using default input encoding: UTF-8
Loaded 1 password hash (RAR5 [PBKDF2-SHA256 256/256 AVX2 8x])
Cost 1 (iteration count) is 32768 for all loaded hashes
Will run 10 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
komputer         (hidden_full.rar)     
1g 0:00:00:01 DONE (2026-02-13 10:10) 0.5434g/s 3478p/s 3478c/s 3478C/s 332211..shopping1
```

**Analiz:**
- ✅ Şifre bulundu: **`komputer`**
- 💡 Rockyou wordlist'te varmış
- 🎯 İsmi "computer" kelimesinin yanlış yazılışı (typo)

---

### 📦 **Adım 9: RAR'ı Aç**
```bash
unrar x -p"komputer" hidden_full.rar
```

**Çıktı:**
```
UNRAR 7.20 beta 3 freeware      Copyright (c) 1993-2025 Alexander Roshal

Extracting from hidden_full.rar

Extracting  txt.txt                                                      OK 
All OK
```

**Analiz:**
- ✅ RAR başarıyla açıldı
- 🎯 `txt.txt` dosyası çıkarıldı
- 💡 Flag bu dosyada olmalı

---

### 🚩 **Adım 10: Flag'i Oku**
```bash
cat txt.txt
```

**Çıktı:**
```
ASCWG{F0Ren$ics_I$_FUn_;)}
```

**Analiz:**
- ✅ Flag bulundu! 🎉
- 💡 "Forensics is fun" mesajı leet speak'le yazılmış
- 🎯 Emoji `;)` ile bitiyor - güzel bir dokunuş

---

## 🚩 **FLAG**
```
ASCWG{F0Ren$ics_I$_FUn_;)}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🗜️<br><b>unrar</b><br><sub>RAR extraction</sub></td>
<td align="center">🔍<br><b>steghide</b><br><sub>Steganography analysis</sub></td>
<td align="center">🎨<br><b>zsteg</b><br><sub>PNG steganography</sub></td>
<td align="center">💾<br><b>dd</b><br><sub>Binary extraction</sub></td>
<td align="center">🔧<br><b>xxd</b><br><sub>Hex manipulation</sub></td>
</tr>
<tr>
<td align="center">📦<br><b>base64</b><br><sub>Base64 decode</sub></td>
<td align="center">🔓<br><b>john</b><br><sub>Password cracking</sub></td>
<td align="center">🕵️<br><b>file</b><br><sub>File type detection</sub></td>
<td align="center">📸<br><b>exiftool</b><br><sub>Metadata extraction</sub></td>
<td align="center">🔎<br><b>grep</b><br><sub>Pattern matching</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**
```
🎯 İz Challenge
       ↓
🗜️ RAR arşivini extract et
   → 7 resim dosyası: 1,2,3,5,6,77,44
   → Eksik: 4 ve 7
       ↓
🔍 Steghide analizi
   → Tüm JPG'lerde capacity var
   → Ama şifre bulunamadı
       ↓
🎨 zsteg ile PNG kontrolü
   → 44.png'de "}EWtEWtEw}" pattern'i
   → İpucu olabilir ama flag değil
       ↓
📦 JPEG sonlarını kontrol et
   → 1.jpg → 0 extra byte
   → 2.jpg → 0 extra byte
   → 3.jpg → 747 extra byte! ✅
       ↓
💾 3.jpg'den veri çıkar
   → dd ile FFD9 sonrası 747 byte
   → Base64 encoded data
       ↓
🔓 Decode chain
   → İlk 5 karakter atla (nokta padding)
   → Base64 decode #1 → Hex string
   → "526172..." → RAR magic bytes!
   → xxd -r -p → Binary RAR
       ↓
🔑 RAR şifresini kır
   → rar2john → hash oluştur
   → john + rockyou.txt
   → Şifre: "komputer"
       ↓
📂 RAR'ı aç
   → unrar x -p"komputer"
   → txt.txt çıkarıldı
       ↓
🚩 Flag'i oku
   → cat txt.txt
   → ASCWG{F0Ren$ics_I$_FUn_;)}
```
