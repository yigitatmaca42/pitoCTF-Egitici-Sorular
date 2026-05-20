# 🎯 Amca - Misc Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Misc-Challenge-darkblue?style=for-the-badge" alt="Misc">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Password Break + Steganography-purple?style=for-the-badge" alt="Steganography">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Misc  
**Seviye:** Orta  
**Puan:** 750

**Challenge Açıklaması:** Cihan hocanın Türkçe wordlisti varmış. İlk adım için iş görür belki?

**Challenge Dosyası:** [📥 Google Drive - RockyoudenemeGercekten.zip](https://drive.google.com/file/d/1CL3b3tXrSKqByFNAPEz0XnP-jN9cT5Ie/view?usp=drivesdk)

**İpucu:** `iki bilinen varsa 3.sü bulunabilecek bir şifreleme algoritması`

---

## 🔍 Analiz

**Hedef:** Şifreli ZIP → RAR → JPEG zincirini kırarak flag'i bulmak

**Strateji:** Wordlist attack → CyberChef görsel analizi → XOR key tespiti → EXIF metadata analizi

**İpuçları:**
- 🎯 ZIP şifresi Türkçe wordlist ile kırılabilir
- 🔍 JPG dosyası bir CyberChef ekran görüntüsü — sol panel kırpılmış
- 💡 XOR'da 2 bilinen varsa 3. bulunabilir
- 🔑 EXIF metadata'sında Samsung sticker gizli veri içerebilir

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Dosyaları Keşfet**

```bash
ls -la
```

**Çıktı:**
```
total 192
drwxrwxr-x  3 kali kali   4096 Mar  8 17:11 .
drwxr-xr-x 11 kali kali   4096 Mar  8 17:11 ..
-rw-rw-r--  1 kali kali  59823 Mar  8 17:08 RockyoudenemeGercekten.zip
```

**Analiz:**
- ✅ Elimizde şifreli bir ZIP dosyası var
- 💡 ZIP adı "Rockyou**deneme**Gercekten" → Rockyou işe yaramaz mesajı veriyor 😄
- 🎯 Türkçe wordlist ile kırmayı deneyeceğiz

---

### 🔑 **Adım 2: Türkçe Wordlist'i İndir**

```bash
git clone https://github.com/cihangungor/turkce-wordlist
ls turkce-wordlist/
```

**Çıktı:**
```
analizler  CONTRIBUTING.md  count.txt  wordlist.txt
arama.py   corpus.txt       README.md
```

**Analiz:**
- ✅ Wordlist başarıyla indirildi
- 💡 `count.txt` ve `wordlist.txt` olmak üzere iki ana liste var
- 🎯 Önce `count.txt` ile deneriz, olmadı `wordlist.txt` ile devam ederiz

---

### 🔓 **Adım 3: ZIP Şifresini Kır**

```bash
zip2john RockyoudenemeGercekten.zip > hash.txt
john hash.txt --wordlist=turkce-wordlist/count.txt
```

**Çıktı:**
```
Using default input encoding: UTF-8
Loaded 2 password hashes with 2 different salts (ZIP, WinZip [PBKDF2-SHA1 256/256 AVX2 8x])
Session completed.
```

**count.txt bulamadı! wordlist.txt ile deneyelim:**

```bash
john hash.txt --wordlist=turkce-wordlist/wordlist.txt
```

**Çıktı:**
```
Using default input encoding: UTF-8
Loaded 2 password hashes with 2 different salts (ZIP, WinZip [PBKDF2-SHA1 256/256 AVX2 8x])
"pass            (RockyoudenemeGercekten.zip/Keyibul/KeyVeSifrelemeKirpildi.jpg)
"pass            (RockyoudenemeGercekten.zip/Keyibul/Amca.rar)
2g 0:00:00:00 DONE (2026-03-08 17:15) 11.11g/s 113777p/s 227555c/s 227555C/s
Session completed.
```

**Analiz:**
- 🎉 **ZIP Şifresi Bulundu: `"pass`**
- ✅ Başında çift tırnak var — standart bir kelime değil!
- 💡 `wordlist.txt` halletti, `count.txt` yetersiz kaldı

---

### 📂 **Adım 4: ZIP'i Aç**

```bash
7z x RockyoudenemeGercekten.zip -p'"pass'
ls -la Keyibul/
```

**Çıktı:**
```
Everything is Ok
Folders: 1
Files: 2
Size: 77285

total 88
drwxrwxr-x 2 kali kali  4096 Mar  6 06:44 .
drwxrwxr-x 4 kali kali  4096 Mar  8 17:15 ..
-rw-rw-r-- 1 kali kali 38971 Mar  6 06:42 Amca.rar
-rw-rw-r-- 1 kali kali 38314 Mar  6 06:34 KeyVeSifrelemeKirpildi.jpg
```

**Analiz:**
- ✅ ZIP başarıyla açıldı
- 💡 `Keyibul/` klasörü içinde 2 dosya çıktı
- 🎯 "KeyVeSifrelemeKirpildi" → isim bize ipucu veriyor: key ve şifreleme kırpılmış!

---

### 🖼️ **Adım 5: KeyVeSifrelemeKirpildi.jpg Analizi**

Görüntüyü açtığımızda bir **telefon ekran görüntüsü** — CyberChef arayüzünü gösteriyor:

- **Input (sağ üst):** `flagbudegilkeyibul`
- **Output (sağ alt):** `icnhmzkjhfcdjvfmzc`
- **Sol panel KIRPILMIŞ** → Operation adı ve key görünmüyor
- HEX ve "Null preserving" seçenekleri görünüyor → **XOR operasyonu**

"flagbudegilkeyibul" = *"flag bu değil, key'i bul!"* mesajı 😄

---

### 🔢 **Adım 6: XOR Key'i Hesapla**

Sol panel kırpılmış ama input ve output elimizde. XOR key'i şöyle hesaplayabiliriz:

```bash
python3 -c "
a = 'flagbudegilkeyibul'
b = 'icnhmzkjhfcdjvfmzc'
keys = [ord(a[i]) ^ ord(b[i]) for i in range(len(a))]
print('XOR sonuçları:', keys)
print('Hex:', [hex(k) for k in keys])
"
```

**Çıktı:**
```
XOR sonuçları: [15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15, 15]
Hex: ['0xf', '0xf', '0xf', ...]
```

**Analiz:**
- 🎉 **XOR Key = `0x0F` (15) — hepsi aynı!**
- 💡 Hex'te `0x0F` → son hane **`F`**
- 🎯 RAR şifresi büyük ihtimalle `F`!

---

### 📦 **Adım 7: RAR Şifresini Kır**

```bash
unrar x Keyibul/Amca.rar -pF
```

**Çıktı:**
```
UNRAR 7.20 beta 3 freeware      Copyright (c) 1993-2025 Alexander Roshal
Extracting from Keyibul/Amca.rar
Extracting  Amca.jpeg                                                    OK
All OK
```

**Analiz:**
- 🎉 **RAR Şifresi: `F`** ✅
- 💡 XOR key `0x0F`'nin hex gösterimindeki **"F"** harfi RAR şifresi olarak kullanılmış
- 🎯 `Amca.jpeg` başarıyla çıktı

---

### 🔍 **Adım 8: Amca.jpeg EXIF Analizi**

```bash
exiftool Amca.jpeg
```

**Kritik Çıktı:**
```
Sticker Edit Info : {"text_bitmap_width":439,"text_bitmap_height":59,
"arrays":"[{\"textStickerText\":\"xorda2kisininbildigisirdegildir\",...}]"}
```

**Analiz:**
- 🎉 **EXIF'te gizli sticker metni bulundu!**
- ✅ Fotoğrafa **görünmez metin sticker** eklenmiş
- 💡 Zaten resimi açtığımızda aslan akbeyin üzerinde `xorda2kisininbildigisirdegildir`
- 🎯 `xorda2kisininbildigisirdegildir` = *"XOR'da 2 kişinin bildiği sır değildir"*
- 📖 Türk atasözü versiyonu: *"İki kişinin bildiği sır değildir"*

---

### 🚩 **Adım 9: Flag'i Oluştur**

İpucu der ki: *"iki bilinen varsa 3.sü bulunabilecek bir şifreleme algoritması"* → **XOR**

XOR'un temel özelliği: `A XOR B = C` → `A XOR C = B` → `B XOR C = A`

- **Bilinen 1:** `flagbudegilkeyibul`
- **Bilinen 2:** `icnhmzkjhfcdjvfmzc`
- **3. = XOR Key:** `0x0F` → `F` → RAR şifresi → Amca.jpeg → **FLAG**

Sticker metni hem görselde hem EXIF'te teyit edildi → Bu flag'in ta kendisi!

**FLAG:**
```
Flag{xorda2kisininbildigisirdegildir}
```

---

## 🚩 **FLAG**
```
Flag{xorda2kisininbildigisirdegildir}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔓<br><b>john</b><br><sub>ZIP şifre kırma</sub></td>
<td align="center">📦<br><b>7z / unrar</b><br><sub>Arşiv açma</sub></td>
<td align="center">🐍<br><b>Python</b><br><sub>XOR hesaplama</sub></td>
<td align="center">🔍<br><b>exiftool</b><br><sub>EXIF metadata analizi</sub></td>
<td align="center">📝<br><b>CyberChef</b><br><sub>XOR şifreleme</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 Amca Challenge
       ↓
📂 Dosyayı incele
   → RockyoudenemeGercekten.zip
   → Rockyou işe yaramaz mesajı var 😄
       ↓
🔑 ZIP şifresini kır
   → zip2john + john + turkce-wordlist/wordlist.txt
   → Şifre: "pass (başında çift tırnak!)
       ↓
📂 ZIP'i aç
   → Keyibul/Amca.rar
   → Keyibul/KeyVeSifrelemeKirpildi.jpg
       ↓
🖼️ JPG'yi analiz et
   → Samsung ekran görüntüsü → CyberChef
   → Input: flagbudegilkeyibul
   → Output: icnhmzkjhfcdjvfmzc
   → Sol panel kırpılmış → XOR operasyonu
       ↓
🔢 XOR Key hesapla
   → python3 ile her karakteri XOR'la
   → Tüm sonuçlar: 0x0F
   → Key: 0x0F → hex'teki F harfi
       ↓
📦 RAR şifresini kır
   → unrar x Amca.rar -pF
   → Şifre: F ✅
   → Amca.jpeg çıktı
       ↓
🔍 EXIF metadata analizi
   → exiftool Amca.jpeg
   → Sticker Edit Info içinde gizli metin
   → "xorda2kisininbildigisirdegildir"
       ↓
🚩 FLAG oluştur
   → Flag{xorda2kisininbildigisirdegildir}
```
