# 🎯 Sessiz - Stego Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Stego-Challenge-darkblue?style=for-the-badge" alt="Stego">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Audio Steganography-purple?style=for-the-badge" alt="Audio Stego">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Stego  
**Seviye:** Orta  
**Puan:** 500

**Challenge Dosyası:** [📥 Google Drive - SessizlikLutfen.jpg](https://drive.google.com/file/d/1aXyaxido9co1YeNyxbKW1chSOXDUloF6/view?usp=drivesdk)

**Beklenen Flag Formatı:** `Flag{gizliAnahtar}`

---

## 🔍 Analiz

**Hedef:** JPEG görüntüsünün içinde gizlenmiş WAV ses dosyasını bulmak, spectrogram analiziyle hex string'i çıkarmak ve flag'i elde etmek

**Strateji:** File analysis → Binwalk → Audio extraction → Spectrogram analysis → Flag

**İpuçları:**
- 🎯 Challenge adı **"Sessiz/SessizlikLutfen"** (Silence/Silence Please) → Görüntüde gizli **SES** dosyası!
- 🔍 JPEG dosyası: 3829x1847 piksel, 1937 KB
- 💡 Binwalk'ta RIFF WAV verisi bulundu (offset: 1496285)
- 🎵 WAV özellikleri: PCM, 2 kanal, 22050 Hz, 16 bit
- 🔑 Spectrogram'da görsel olarak gizlenmiş hex string

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Dosya Tipini Kontrol Et**
```bash
file SessizlikLutfen.jpg
```

**Çıktı:**
```
SessizlikLutfen.jpg: JPEG image data, JFIF standard 1.00, resolution (DPI), 
density 400x400, segment length 16, baseline, precision 8, 3829x1847, components 3
```

**Analiz:**
- ✅ Geçerli bir JPEG dosyası
- 💡 Boyut: 3829x1847 piksel (~7.1 megapiksel)
- 🎯 Standart JPEG metadata, şüpheli bir şey yok

---

### 📸 **Adım 2: Metadata Kontrolü**
```bash
exiftool SessizlikLutfen.jpg
```

**Çıktı:**
```
ExifTool Version Number         : 13.44
File Name                       : SessizlikLutfen.jpg
Directory                       : .
File Size                       : 1937 kB
File Type                       : JPEG
Image Width                     : 3829
Image Height                    : 1847
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:4:4 (1 1)
```

**Analiz:**
- ✅ Normal JPEG metadata
- 🚨 Gizli comment veya description yok
- 💡 Başka yöntemler denemeli

---

### 🔨 **Adım 3: Binwalk ile Dosya Yapısını Analiz Et**

Binwalk, dosya içinde gizlenmiş diğer dosyaları bulabilen güçlü bir araçtır.

```bash
binwalk SessizlikLutfen.jpg
```

**Çıktı:**
```
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             JPEG image data, JFIF standard 1.00
178358        0x2B8B6         JBOOT STAG header, image id: 16, timestamp 0x92B48E01
1496285       0x16D4DD        RIFF audio data (WAV), PCM, 2 channels, 22050 sample rate
```

**Analiz:**
- ✅ **KRITIK BULGU!** Offset 1496285'te gizli bir **WAV dosyası** var!
- 🎵 WAV özellikleri: PCM format, Stereo (2 kanal), 22050 Hz
- 🎯 Challenge adı "Sessiz" → Görüntüde gizli ses! Mükemmel uyum!
- 💡 Bu WAV dosyasını çıkarmalıyız

---

### 💾 **Adım 4: WAV Dosyasını Çıkar**

`dd` komutuyla binary offset'ten itibaren veriyi extract edebiliriz.

```bash
# Offset 1496285'ten başlayarak WAV dosyasını çıkar
dd if=SessizlikLutfen.jpg of=hidden.wav bs=1 skip=1496285
```

**Çıktı:**
```
441048+0 records in
441048+0 records out
441048 bytes (441 kB, 431 KiB) copied, 0.892341 s, 494 kB/s
```

**Dosyayı kontrol et:**
```bash
file hidden.wav
ls -lh hidden.wav
```

**Çıktı:**
```
hidden.wav: RIFF (little-endian) data, WAVE audio, Microsoft PCM, 16 bit, stereo 22050 Hz
-rw-rw-r-- 1 kali kali 431K Feb 13 10:15 hidden.wav
```

**Analiz:**
- ✅ WAV dosyası başarıyla çıkarıldı!
- 💡 Boyut: 431 KB
- 🎵 Geçerli bir ses dosyası: 16-bit PCM, stereo, 22050 Hz
- 🎯 Şimdi bu ses dosyasını analiz etmeliyiz

---

### 🎨 **Adım 5: Spectrogram Oluştur**

Ses dosyalarında gizli mesajlar genellikle **spectrogram**'da görsel olarak saklanır.

```bash
# Sox ile spectrogram oluştur
sox hidden.wav -n spectrogram -o spectrogram.png

# Spectrogram'ı görüntüle
xdg-open spectrogram.png
```

**Alternatif (Audacity ile):**
```bash
audacity hidden.wav
# Audacity'de: Track dropdown > Spectrogram seçeneğini seç
```

**Analiz:**
- ✅ Spectrogram başarıyla oluşturuldu
- 💡 Sox komutu ses dosyasını frekans-zaman grafiğine çevirir
- 🎯 Gizli mesajlar bu grafikte görsel olarak saklanır

---

### 🔍 **Adım 6: Spectrogram'da Gizli Mesajı Bul**

Spectrogram görüntüsünü açtığımızda şunu gördük:

![Spectrogram Analysis](spectrogram.png)

**Görsel Analiz:**
```
Frequency (kHz)
     11 ┌────────────────────────────────────────────────┐
     10 │                                                │
      9 │                                                │
      8 │                                                │
      7 │                                                │
      6 │ 54A8F963A252E414A1B85D244A848A56              │ ← HEX STRING!
      5 │ 54A8F963A252E414A1B85D244A848A56              │
      4 │                                                │
      3 │                                                │
        └────────────────────────────────────────────────┘
           0    1    2    3    4    5 (seconds)
```

**Bulunan Hex String:**
```
54A8F963A252E414A1B85D244A848A56
```

**Analiz:**
- ✅ Spectrogram'da **sarı/turuncu** renkte açıkça görünen bir hex string!
- 💡 32 karakter (16 byte) → 128 bit
- 🔍 Hex karakterler: 0-9 ve A-F
- 🎯 Bu hex string flag ile ilgili olmalı

---

### 🧪 **Adım 7: Hex String'i Decode Denemeleri**

Hex string'i farklı yöntemlerle decode etmeyi denedik:

```bash
# 1. Hex'i ASCII'ye çevir
echo "54A8F963A252E414A1B85D244A848A56" | xxd -r -p
# Çıktı: T¨ùc¢Rä¡¸]$JV (anlamlı değil)

# 2. Python ile
python3 -c "print(bytes.fromhex('54A8F963A252E414A1B85D244A848A56'))"
# Çıktı: b'T\xa8\xf9c\xa2R\xe4\x14\xa1\xb8]$J\x84\x8aV' (anlamlı değil)

# 3. Base64'e çevir
python3 << EOF
import base64
hex_str = "54A8F963A252E414A1B85D244A848A56"
data = bytes.fromhex(hex_str)
print(f"Base64: {base64.b64encode(data).decode()}")
EOF
# Çıktı: VKj5Y6JS5BShuF0kSoSKVg==

# 4. Tersine çevir
python3 -c "print('54A8F963A252E414A1B85D244A848A56'[::-1])"
# Çıktı: 65A848A442D58B1A414E252A369F8A45

# 5. MD5 hash lookup (CrackStation.net)
# Hash: 54A8F963A252E414A1B85D244A848A56
# Sonuç: Not found
```

**Analiz:**
- ❌ Hex → ASCII: Anlamlı metin çıkmadı
- ❌ Base64: VKj5Y6JS5BShuF0kSoSKVg== (flag değil)
- ❌ Reversed: Farklı hex ama yine anlamlı değil
- ❌ MD5 lookup: Bilinen hash değil
- 💡 Belki hex string'in kendisi flag'in içeriğidir?

---

### 🚩 **Adım 8: Flag Formatını Dene**

Beklenen flag formatı `Flag{gizliAnahtar}` olduğu için, hex string'i direkt bu formatta denemeliyiz:

```bash
# Orijinal hex string
echo "Flag{54A8F963A252E414A1B85D244A848A56}"

# Küçük harf
echo "Flag{54a8f963a252e414a1b85d244a848a56}"

# Base64 versiyonu
echo "Flag{VKj5Y6JS5BShuF0kSoSKVg==}"

# Tersine çevrilmiş
echo "Flag{65A848A442D58B1A414E252A369F8A45}"
```

**TEST SONUCU:**
```
✅ DOĞRU FLAG: Flag{54A8F963A252E414A1B85D244A848A56}
```

**Analiz:**
- ✅ Flag bulundu! 🎉
- 💡 Hex string'in kendisi "gizliAnahtar" kısmıydı
- 🎯 Büyük harf formatında kabul edildi
- 🔑 32 karakterlik hex string = flag'in içeriği

---

## 🚩 **FLAG**
```
Flag{54A8F963A252E414A1B85D244A848A56}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">📁<br><b>file</b><br><sub>File type detection</sub></td>
<td align="center">📸<br><b>exiftool</b><br><sub>Metadata extraction</sub></td>
<td align="center">🔍<br><b>binwalk</b><br><sub>Embedded file analysis</sub></td>
<td align="center">💾<br><b>dd</b><br><sub>Binary extraction</sub></td>
<td align="center">🎵<br><b>sox</b><br><sub>Spectrogram generation</sub></td>
</tr>
<tr>
<td align="center">🎨<br><b>audacity</b><br><sub>Audio analysis (optional)</sub></td>
<td align="center">🔧<br><b>xxd</b><br><sub>Hex manipulation</sub></td>
<td align="center">🐍<br><b>python3</b><br><sub>Encoding/decoding</sub></td>
<td align="center">📦<br><b>base64</b><br><sub>Base64 operations</sub></td>
<td align="center">🔎<br><b>grep</b><br><sub>Pattern searching</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**
```
🎯 Sessiz Challenge
       ↓
📁 Dosya tipini kontrol et
   → file SessizlikLutfen.jpg
   → JPEG image data
       ↓
📸 Metadata analizi
   → exiftool SessizlikLutfen.jpg
   → Normal JPEG metadata
   → Gizli bilgi yok
       ↓
🔍 Binwalk ile embedded dosya taraması
   → binwalk SessizlikLutfen.jpg
   → KRITIK: Offset 1496285'te WAV dosyası bulundu! ✅
   → RIFF audio data (WAV), PCM, 2 ch, 22050 Hz
       ↓
💾 WAV dosyasını çıkar
   → dd if=SessizlikLutfen.jpg of=hidden.wav bs=1 skip=1496285
   → 431 KB WAV dosyası oluşturuldu
   → file hidden.wav → Geçerli RIFF WAV dosyası
       ↓
🎨 Spectrogram oluştur
   → sox hidden.wav -n spectrogram -o spectrogram.png
   → Frekans-zaman grafiği oluşturuldu
       ↓
🔍 Spectrogram'da gizli mesajı bul
   → Spectrogram görüntüsünü aç
   → 6 kHz civarında görsel olarak yazılmış text!
   → Hex string: 54A8F963A252E414A1B85D244A848A56 ✅
       ↓
🚩 Flag formatını dene
   → Flag{54A8F963A252E414A1B85D244A848A56}
   → ✅ DOĞRU FLAG!
```
