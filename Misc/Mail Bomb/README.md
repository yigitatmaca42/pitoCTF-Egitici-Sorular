# 📧 MailBomb - Misc Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Misc-Challenge-darkblue?style=for-the-badge&logo=wireshark" alt="Misc">
  <img src="https://img.shields.io/badge/Difficulty-Medium-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-PCAP Analysis-purple?style=for-the-badge&logo=network" alt="PCAP">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Misc  
**Seviye:** Orta  
**Puan:** 400

**Challenge Dosyası:** [📥 MailBomb.pcapng](https://drive.google.com/file/d/14ZbFUJzGSHd6chias0qiaa8ZT1GfS1fG/view?usp=drivesdk)


---

## 🔍 Analiz

**Hedef:** PCAP dosyasını analiz edip 1000 email arasından gizli FLAG'i bulmak

**Strateji:** PCAP analysis → Email extraction → Size anomaly detection → Base64 decode → PNG extraction → Flag

**İpuçları:**
- 📧 **Email Forensics:** SMTP traffic analysis
- 📊 **Anomaly Detection:** Size-based filtering
- 🔐 **Multi-layer Encoding:** Base64 → Hex → Binary
- 🖼️ **Image Analysis:** PNG extraction from encoded data
- 🎯 **Format:** HCTF{...} format flag

---

## ✅ Çözüm Adımları

### 📥 **1. Dosya Analizi**

**PCAP dosyasını indirip file type kontrolü:**
```bash
file MailBomb.pcapng
```

**Çıktı:**
```
MailBomb.pcapng: pcapng capture file - version 1.0
```

**Analiz:**
- ✅ **PCAP format** doğrulandı
- 📦 **pcapng version 1.0** - Modern Wireshark format
- 🌐 Network traffic capture dosyası

> 🎯 Bu bir network capture dosyası - email trafiği içeriyor olabilir!

---

### 🔍 **2. Quick Flag Search**

**Basit bir strings araması ile başlıyoruz:**
```bash
strings MailBomb.pcapng | grep -i "flag"
```

**Çıktı:**
```
(hiçbir sonuç yok)
```

**Analiz:**
- ❌ Flag düz metin olarak bulunmuyor
- 🔐 Muhtemelen encoded veya gömülü durumda
- 🔬 Daha derin analiz gerekiyor

> 💡 Flag gizlenmiş - PCAP dosyasını detaylı incelememiz lazım!

---

### 🖥️ **3. Wireshark GUI Analysis**

**Wireshark ile dosyayı açıyoruz:**
```bash
wireshark MailBomb.pcapng &
[2] 936186

** (wireshark:936186) 23:38:33.717963 [GUI WARNING] -- QObject::disconnect...
[2]  - done       wireshark MailBomb.pcapng
```

**Wireshark'ta gözlemler:**
- 📧 **Çok sayıda SMTP trafiği** (localhost:25)
- 🔄 **127.0.0.1 → 127.0.0.1** (loopback)
- 💣 **Mail bomb pattern** - Sürekli email gönderimi
- 👤 **From Person → To Person** email'leri

> 🚨 "MailBomb" ismi gerçekten doğru - yüzlerce email var!

---

### 📊 **4. TCP Connection Analysis**

**TCP bağlantı sayısını öğreniyoruz:**
```bash
tshark -r MailBomb.pcapng -q -z conv,tcp | grep "^127" | wc -l
```

**Çıktı:**
```
1000
```

**Analiz:**
- 🔢 **1000 TCP bağlantısı!**
- 💥 Her bağlantı bir email gönderimi
- 🎯 1000 email arasında flag gizli

**Connection Details:**
```
127.0.0.1:57426  <-> 127.0.0.1:25     (10 packets, 35 kB)
127.0.0.1:38906  <-> 127.0.0.1:25     (9 packets, 2,177 bytes)
127.0.0.1:54376  <-> 127.0.0.1:25     (10 packets, 2,387 bytes)
...
(1000 satır)
```

> 💡 Port 25 = SMTP (Simple Mail Transfer Protocol)

---

### 📦 **5. Email Extraction**

**tshark ile tüm email'leri extract ediyoruz:**
```bash
mkdir extracted
tshark -r MailBomb.pcapng --export-objects "imf,extracted"
cd extracted
ls | wc -l
```

**Analiz:**
- ✅ **999 email dosyası** başarıyla extract edildi
- 📁 Her email bir `.eml` dosyası olarak kaydedildi
- 🔍 Şimdi anomali araması yapacağız

> 🎯 999 email arasında bir tanesi farklı olmalı!

---

### 🔎 **6. Size-Based Anomaly Detection**

**Email dosyalarını boyutlarına göre sıralıyoruz:**
```bash
ls -lhS *.eml | head -20
```

**Çıktı:**
```
-rw-r--r-- 1 kali kali  33K Jan 12 23:39 (643).eml    ← 🎯 ANOMALİ!
-rw-r--r-- 1 kali kali 1.1K Jan 12 23:39 (144).eml
-rw-r--r-- 1 kali kali 1.1K Jan 12 23:39 (528).eml
-rw-r--r-- 1 kali kali 1.1K Jan 12 23:39 (826).eml
-rw-r--r-- 1 kali kali 1.1K Jan 12 23:39 (925).eml
-rw-r--r-- 1 kali kali 1.1K Jan 12 23:39 (431).eml
-rw-r--r-- 1 kali kali 1.1K Jan 12 23:39 (469).eml
-rw-r--r-- 1 kali kali 1.1K Jan 12 23:39 (638).eml
-rw-r--r-- 1 kali kali 1.1K Jan 12 23:39 (809).eml
-rw-r--r-- 1 kali kali 1.1K Jan 12 23:39 (709).eml
-rw-r--r-- 1 kali kali 1.1K Jan 12 23:39 (955).eml
-rw-r--r-- 1 kali kali 1.1K Jan 12 23:39 (734).eml
-rw-r--r-- 1 kali kali 1.1K Jan 12 23:39 (101).eml
-rw-r--r-- 1 kali kali 1.1K Jan 12 23:39 (153).eml
-rw-r--r-- 1 kali kali 1.1K Jan 12 23:39 (916).eml
-rw-r--r-- 1 kali kali 1.1K Jan 12 23:39 (701).eml
-rw-r--r-- 1 kali kali 1.1K Jan 12 23:39 (980).eml
-rw-r--r-- 1 kali kali 1.1K Jan 12 23:39 (551).eml
-rw-r--r-- 1 kali kali 1.1K Jan 12 23:39 (340).eml
-rw-r--r-- 1 kali kali 1.1K Jan 12 23:39 (1).eml
```

**Analiz:**
- 🚨 **(643).eml = 33K** - ÇOK BÜYÜK!
- 📏 Diğer tüm dosyalar ~1.1K
- 📊 **Oran: 33KB / 1.1KB ≈ 30x daha büyük!**
- 🎯 Bu kesinlikle bizim hedefimiz!

> 🏆 **ANOMALY DETECTED!** (643).eml dosyası çok büyük - flag burada olmalı!

---

### 📧 **7. Suspicious Email Examination**

**Şüpheli email'in header'larına bakıyoruz:**
```bash
head -5 "(643).eml"
```

**Çıktı:**
```
From: From Person <root@hackappa.pub.com>
To: To Person <root@hackappa.pub.com>
Subject:
```

**Satır numaraları ile detaylı inceleme:**
```bash
cat -n "(643).eml" | head -10
```

**Çıktı:**
```
     1  From: From Person <root@hackappa.pub.com>
     2  To: To Person <root@hackappa.pub.com>
     3  Subject:
     4
     5
     6
     7  ODk1MDRlNDcwZDBhMWEwYTAwMDAwMDBkNDk0ODQ0NTIwMDAwMDJjNTAwMDAwMThhMDgwMjAwMDAwMDZi...
        (uzun base64 string devam ediyor)
```

**Analiz:**
- 📝 **Email header:** Standart SMTP formatı
- 🔐 **7. satırdan itibaren:** Uzun base64 encoded data
- 📏 **Data uzunluğu:** ~33KB base64 string
- 🎯 **Bu bizim hedefimiz!**

> 💡 7. satırda base64 encoded data başlıyor - decode etmemiz lazım!

---

### 🔓 **8. Base64 Decoding**

**7. satırdaki base64 string'i decode ediyoruz:**
```bash
sed -n '7p' "(643).eml" | base64 -d | xxd | head -3
```

**Çıktı:**
```
base64: invalid input
00000000: 3839 3530 3465 3437 3064 3061 3161 3061  89504e470d0a1a0a
00000010: 3030 3030 3030 3064 3439 3438 3434 3532  0000000d49484452
00000020: 3030 3030 3032 6335 3030 3030 3031 3861  000002c50000018a
```

**Analiz:**
- ⚠️ `base64: invalid input` - Bazı karakterler hatalı olabilir
- 🔍 **Hex output:** `89504e470d0a1a0a`
- 🖼️ **Bu ASCII hex string!** → `89 50 4e 47 0d 0a 1a 0a` = **PNG magic bytes!**
- 🎯 Data aslında: Base64 → Hex String → Binary PNG

**Encoding layers:**
```
Original Data:    PNG image (binary)
       ↓
Layer 1:          Hexadecimal string (ASCII)
       ↓
Layer 2:          Base64 encoding
       ↓
Final:            Email body content
```

> 🎨 **DISCOVERY!** Bu bir PNG dosyası - ama double-encoded! (Base64 + Hex)

---

### 🎨 **9. Complete Decoding Chain**

**Multi-layer decoding: Base64 → Hex → Binary PNG**
```bash
sed -n '7p' "(643).eml" | base64 -d | xxd -r -p > flag.png
base64: invalid input
```

**File verification:**
```bash
file flag.png
```

**Çıktı:**
```
flag.png: PNG image data, 709 x 394, 8-bit/color RGB, non-interlaced
```

**Analiz:**
- ✅ **PNG başarıyla oluşturuldu!**
- 📐 **Dimensions:** 709x394 pixels
- 🎨 **Format:** 8-bit RGB
- ⚠️ Base64 uyarısı önemli değil - dosya başarıyla decode edildi

**Decoding işlemi:**
```
sed -n '7p' "(643).eml"     → 7. satırı al
     ↓
base64 -d                   → Base64 decode (hex string çıkar)
     ↓
xxd -r -p                   → Hex string'i binary'ye çevir
     ↓
> flag.png                  → PNG dosyası oluştur
```

> 🎉 **FLAG.PNG OLUŞTURULDU!** Artık görselleştirebiliriz!

---

### 🚩 **10. Flag Visualization**

**PNG dosyasını açıyoruz:**
```bash
xdg-open flag.png
```

**Ekranda görünen:**
```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│                                                             │
│          HCTF{u_4r3_1N_mY_n3t!!_3y_pH1shing_k1nG}           │
│                                                             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

> 🏆 **FLAG BULUNDU!** Tebrikler, mail bomb'u çözdün!

---

## 🚩 **FLAG**
```
HCTF{u_4r3_1N_mY_n3t!!_3y_pH1shing_k1nG}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">📄<br><b>file</b><br><sub>Dosya tipi analizi</sub></td>
<td align="center">🦈<br><b>tshark</b><br><sub>CLI PCAP analizi</sub></td>
<td align="center">🖥️<br><b>Wireshark</b><br><sub>GUI PCAP analizi</sub></td>
<td align="center">🔐<br><b>base64</b><br><sub>Base64 decode</sub></td>
<td align="center">🔧<br><b>xxd</b><br><sub>Hex manipulation</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**
```bash
# Dosya analizi
file MailBomb.pcapng

# Quick flag search
strings MailBomb.pcapng | grep -i "flag"

# Wireshark GUI
wireshark MailBomb.pcapng &

# TCP connection count
tshark -r MailBomb.pcapng -q -z conv,tcp | grep "^127" | wc -l

# Email extraction
mkdir extracted
tshark -r MailBomb.pcapng --export-objects "imf,extracted"
cd extracted
ls | wc -l

# Size anomaly detection
ls -lhS *.eml | head -20

# Email examination
head -5 "(643).eml"
cat -n "(643).eml" | head -10

# Base64 preview
sed -n '7p' "(643).eml" | base64 -d | xxd | head -3

# Complete decode (Base64 → Hex → PNG)
sed -n '7p' "(643).eml" | base64 -d | xxd -r -p > flag.png

# Verify and view
file flag.png
xdg-open flag.png
```

---

## 💭 **Çözüm Akışı**
```
📧 "MailBomb" Challenge
            ↓
📄 file MailBomb.pcapng → PCAP file
            ↓
🔍 strings | grep flag → Sonuç yok
            ↓
🖥️ Wireshark analysis
   → 1000 SMTP connections (127.0.0.1:25)
            ↓
📊 tshark conversation analysis
   → 1000 TCP streams confirmed
            ↓
📦 Email extraction
   → 999 .eml files created
            ↓
🔎 Size analysis (ls -lhS)
   → (643).eml = 33KB ← ANOMALY!
   → Diğerleri = ~1.1KB
            ↓
📧 Email examination
   → Line 7: Base64 data başlıyor
            ↓
🔓 Base64 decode
   → Output: Hex string (ASCII)
   → "89504e470d0a1a0a..." detected
            ↓
🎨 Hex to Binary conversion
   → PNG magic bytes: 89 50 4e 47
   → flag.png created (709x394)
            ↓
🖼️ Image visualization
            ↓
🚩 HCTF{u_4r3_1N_mY_n3t!!_3y_pH1shing_k1nG}
```
