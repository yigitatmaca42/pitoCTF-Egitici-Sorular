# 🎯 firm - IoT Challenge

<p align="center">
  <img src="https://img.shields.io/badge/IoT-Challenge-darkblue?style=for-the-badge" alt="IoT">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Firmware Reverse-purple?style=for-the-badge" alt="Firmware">
</p>

---

## 📋 Challenge Bilgileri

**Kategori:** IoT  
**Zorluk:** Orta  
**Puan:** 500

**Açıklama:** 0x1000!=0

**Challenge Dosyası:** [📥 Google Drive - iot_firmware.bin](https://drive.google.com/file/d/1nBtN51cPTkQJrMIoXvBwywpp2DgTfJBL/view?usp=sharing)

---

## 🔍 Çözüm

### Adım 1: Firmware Dosyasını İnceleme

```bash
# Dosya tipini kontrol et
file iot_firmware.bin
```

**Çıktı:**
```
iot_firmware.bin: data
```

---

### Adım 2: String Analizi

```bash
# Dosyadaki okunabilir metinleri çıkar
strings iot_firmware.bin
```

**Sonuç:**
```
FWBIN001
Version: 2.4.1
IoT Device Firmware
Manufacturer: SecureIoT Inc.
DEVICE_NAME=SecureIoT_Gateway
AES_KEY=CokSecretKey4134
ENCRYPTION=AES-CBC-128
iv mi lazim? Iyi bak.
CONFIG_DATA_HEX
IV_START
IV_END
[DEBUG] System initialized
[DEBUG] Encryption module loaded
[INFO] Boot sequence complete
```

**Analiz:** 
- Firmware versiyonu: 2.4.1
- Şifreleme kullanılıyor: AES-CBC-128
- AES Key bulundu: `CokSecretKey4134`
- Türkçe ipucu: "iv mi lazim? Iyi bak." (IV'ye dikkat!)

---

### Adım 3: Hex Dump ile Binary İnceleme

```bash
# İlk 100 satırı hex formatında görüntüle
xxd iot_firmware.bin | head -100
```

**Gözlemler:**
```
00000000: 4657 4249 4e30 3031  FWBIN001........
00000010: 5665 7273 696f 6e3a  Version: 2.4.1..
...
```

**Header bilgileri:**
- Magic bytes: `FWBIN001`
- Versiyon bilgisi ASCII formatında
- Çok fazla null byte padding var

---

### Adım 4: 0x1000 Offset'ini İnceleme (Hint: 0x1000!=0)

```bash
# 0x1000 adresindeki veriyi görüntüle
xxd -s 0x1000 iot_firmware.bin | head -50
```

**Çıktı:**
```
00001000: f3b2 5b16 4ab7 9408 5d51 b0f1 b823 0622  ..[.J...]Q...#."
00001010: 955d 3e3a 4753 f984 a7e1 7a90 661e 9005  .]>:GS....z.f...
00001020: 8b81 c6de 0b87 44a5 4b8d c72c 1653 e54b  ......D.K..,.S.K
00001030: 0000 0000 0000 0000 0000 0000 0000 0000  ................
```

**Gözlem:** 0x1000 adresinde şifrelenmiş data var! (48 byte)

---

### Adım 5: IV (Initialization Vector) Bulma

```bash
# IV_START ve IV_END arasındaki veriyi bul
xxd iot_firmware.bin | grep -A 5 "IV_START"
```

**Çıktı:**
```
00008100: 4956 5f53 5441 5254 dead beef cafe 0123  IV_START.......#
00008110: 4567 89ab cdef 3441 4956 5f45 4e44 0000  Eg....4AIV_END..
```

**IV değeri:** `deadbeefcafe0123456789abcdef3441` (16 byte)

---

### Adım 6: Şifrelenmiş Datayı Çıkarma

```bash
# 0x1000 adresinden 48 byte şifrelenmiş veriyi çıkar
dd if=iot_firmware.bin of=encrypted_data.bin bs=1 skip=4096 count=48
```

**Çıktı:**
```
48+0 records in
48+0 records out
48 bytes copied, 5.4576e-05 s, 880 kB/s
```

---

### Adım 7: Key ve IV Dosyalarını Hazırlama

```bash
# IV'yi binary formatına çevir
echo "deadbeefcafe0123456789abcdef3441" | xxd -r -p > iv.bin
xxd iv.bin
```

**Çıktı:**
```
00000000: dead beef cafe 0123 4567 89ab cdef 3441  .......#Eg....4A
```

```bash
# Key'i dosyaya yaz (16 byte olmalı)
echo -n "CokSecretKey4134" > key.bin
xxd key.bin
```

**Çıktı:**
```
00000000: 436f 6b53 6563 7265 744b 6579 3431 3334  CokSecretKey4134
```

---

### Adım 8: AES-128-CBC ile Decrypt İşlemi

```bash
# OpenSSL kullanarak decrypt et
openssl enc -d -aes-128-cbc \
  -in encrypted_data.bin \
  -out decrypted.txt \
  -K $(xxd -p key.bin | tr -d '\n') \
  -iv $(xxd -p iv.bin | tr -d '\n')
```

---

### Adım 9: Flag'i Okuma

```bash
# Decrypt edilmiş veriyi görüntüle
cat decrypted.txt
```

**Çıktı:**
```
FLAG{firmware_reverse_crypto_birarada_iotKolay}
```

---

## 🚩 Flag

```
FLAG{firmware_reverse_crypto_birarada_iotKolay}
```

---

## 🛠️ Kullanılan Araçlar

- **file** - Dosya tipi tespiti
- **strings** - ASCII string çıkarma
- **xxd** - Hex dump ve binary dönüşümler
- **dd** - Binary data extraction
- **openssl** - AES decryption
- **grep** - Pattern matching
- **bash** - Scripting ve automation

---

## 📊 Çözüm Akış Şeması

```
🎯 Firmware Challenge
       ↓
📦 iot_firmware.bin dosyasını aç
   → 64KB binary file
       ↓
🔍 String analizi
   → FWBIN001, Version 2.4.1
   → AES_KEY=CokSecretKey4134
   → ENCRYPTION=AES-CBC-128
       ↓
💭 Türkçe ipucu
   → "iv mi lazim? Iyi bak."
       ↓
📍 Hex dump analizi
   → Header: FWBIN001
   → Metadata: Version, Manufacturer
       ↓
🎯 Hint çözümü: 0x1000!=0
   → 0x1000 offset'inde data var
       ↓
🔓 0x1000 adresini kontrol
   → f3b25b164ab794085d51b0f1b8230622...
   → Şifrelenmiş data bulundu! (48 byte)
       ↓
🔑 IV_START marker'ını bul
   → deadbeefcafe0123456789abcdef3441
   → IV bulundu! (16 byte)
       ↓
📦 Data extraction
   → dd ile encrypted_data.bin çıkar
   → Key: CokSecretKey4134
   → IV: deadbeefcafe0123456789abcdef3441
       ↓
🔓 OpenSSL ile decrypt
   → AES-128-CBC mode
   → Key ve IV ile decryption
       ↓
🚩 FLAG{firmware_reverse_crypto_birarada_iotKolay}
```
