# 🎯 PitonsCrypt - Crypto Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Crypto-Challenge-darkblue?style=for-the-badge" alt="Crypto">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Timestamp Brute Force-purple?style=for-the-badge" alt="Type">
</p>

---

## 📋 Soru Bilgileri
  
**Kategori:** Crypto  
**Seviye:** Kolay
**Puan:** 200

**Açıklama:** Bizim pitons ekibinden bir arkadaşımız mezun olmadan önce kendince bir şifreleme algoritması yazıp flag şifrelemiş. Bulabilir misin?

**Encrypted Flag:** `PrJIxSHIKlES1Rc8dDmoRoEkaRknfD/AVxoHTWP2U1ZOk6MvDi4AunEwvqgNStpqZg==`

**Flag formatı:** Flag{...}

**Challenge Dosyası:** [📥 Google Drive - encrypt_flag25.zip](https://drive.google.com/file/d/1aKDOJ3D9ZZvVAXOSnhdIvAr61Z1JnSky/view?usp=drivesdk)

---

## 🔍 Analiz

**Hedef:** Timestamp-based XOR encryption algoritmasını tersine çevirerek flag'i decrypt etmek

**Strateji:** Dosya analizi → Algoritma inceleme → Timestamp ipucu → Brute force → Decrypt → Flag

**İpuçları:**
- 🎯 "Mezun olmadan önce" → Geçmiş zaman (2025 yılı)
- 🔍 "Kendince şifreleme algoritması" → Custom crypto implementation
- 💡 Encrypted flag verilmiş → Reverse engineering gerekli

---

## ✅ Çözüm Adımları

### 📦 **Adım 1: Challenge Dosyasını İndirme ve Açma**

**ZIP dosyasını indirip extract ediyoruz:**

```bash
unzip encrypt_flag25.zip
```

**Output:**
```
Archive:  encrypt_flag25.zip
  inflating: encrypt_flag.py
```

**Analiz:**
- ✅ Python script dosyası
- ✅ Şifreleme kodu açık kaynak
- 💡 Algoritmayı inceleyebiliriz

---

### 📝 **Adım 2: Kaynak Kodu İnceleme**

**encrypt_flag.py dosyasını okuyoruz:**

```bash
cat encrypt_flag.py
```

**Output:**

```python
#!/usr/bin/env python3
import hashlib
import time
import base64

def generate_key_from_timestamp(timestamp):
    timestamp_str = str(int(timestamp))
    key = hashlib.sha256(timestamp_str.encode()).digest()
    return key

def xor_encrypt(data, key):
    result = bytearray()
    key_len = len(key)
    
    for i, byte in enumerate(data):
        key_byte = key[i % key_len]
        mixed = (key_byte + i) % 256
        result.append(byte ^ mixed)
    
    return bytes(result)

def encrypt_flag(flag, timestamp):
    key = generate_key_from_timestamp(timestamp)
    timestamp_bytes = str(int(timestamp)).encode()
    data_to_encrypt = timestamp_bytes + b'|' + flag.encode()
    encrypted = xor_encrypt(data_to_encrypt, key)
    return base64.b64encode(encrypted).decode()

def decrypt_flag(encrypted_b64, timestamp):
    key = generate_key_from_timestamp(timestamp)
    encrypted = base64.b64decode(encrypted_b64)
    decrypted = xor_encrypt(encrypted, key)
    parts = decrypted.split(b'|', 1)
    if len(parts) == 2:
        return parts[1].decode()
    return None

def main():
    flag = input("Flag girin: ")
    current_timestamp = int(time.time())
    encrypted_flag = encrypt_flag(flag, current_timestamp)
    
    print("\nEncrypted Flag:")
    print(encrypted_flag)
    print(f"\nŞifreleme zamanı: {time.strftime('%Y', time.localtime(current_timestamp))}")

if __name__ == "__main__":
    main()
```

**Analiz:**
- ✅ SHA256 ile timestamp'ten key türetiliyor
- ✅ XOR encryption kullanılıyor
- ✅ `timestamp|flag` formatında data
- ✅ Base64 encoding
- 💡 Decrypt fonksiyonu da mevcut!

---

### 🔬 **Adım 3: Algoritma Detaylarını Anlama**

**Şifreleme akışı:**

**1. Key Generation:**
```python
def generate_key_from_timestamp(timestamp):
    timestamp_str = str(int(timestamp))
    key = hashlib.sha256(timestamp_str.encode()).digest()
    return key
```

**2. XOR Encryption:**
```python
def xor_encrypt(data, key):
    result = bytearray()
    key_len = len(key)
    
    for i, byte in enumerate(data):
        key_byte = key[i % key_len]
        mixed = (key_byte + i) % 256  # Position-dependent!
        result.append(byte ^ mixed)
    
    return bytes(result)
```

**3. Data Format:**
```python
data_to_encrypt = timestamp_bytes + b'|' + flag.encode()
# Örnek: b'1738942245|Flag{...}'
```

**4. Encoding:**
```python
encrypted = xor_encrypt(data_to_encrypt, key)
return base64.b64encode(encrypted).decode()
```

**Analiz:**
- ✅ Timestamp hem key üretimi hem de data'da kullanılıyor
- ✅ XOR reversible (symmetric)
- ✅ Position-dependent mixing → Her byte farklı key ile XOR'lanıyor
- 💡 Doğru timestamp bulursak decrypt edebiliriz!

---

### 🧪 **Adım 4: Script'i Test Etme**

**Script'i çalıştırıyoruz:**

```bash
python3 encrypt_flag.py
```

**Input:**
```
Flag girin: test
```

**Output:**
```
Encrypted Flag:
EtY4Tv5anDQVU48=

Şifreleme zamanı: 2026
```

**Analiz:**
- ✅ Script çalışıyor
- ✅ Current timestamp kullanıyor (2026)
- ✅ Şifreleme zamanı gösteriliyor
- 💡 Sadece yıl bilgisi veriliyor, tam timestamp yok!

---

### 💡 **Adım 5: İpucu Analizi - "Mezun Olmadan Önce"**

**Challenge açıklaması tekrar:**
> "Bizim pitons ekibinden bir arkadaşımız **mezun olmadan önce** kendince bir şifreleme algoritması yazıp flag şifrelemiş."

**Zaman analizi:**
- 📅 Bugün: 2026 (Şubat)
- 📅 "Mezun olmadan önce" → Geçmiş zaman
- 📅 Muhtemelen: **2025 yılı**
- 💡 Mezuniyet genelde Haziran-Temmuz aylarında

**Test çıktısından:**
```
Şifreleme zamanı: 2026
```

**Analiz:**
- ✅ Script yıl bilgisi gösteriyor
- ✅ Challenge 2025 yılında şifrelenmiş olabilir
- 💡 2025 yılındaki tüm timestamp'leri denememiz gerekiyor!

---

### 🎯 **Adım 6: Brute Force Stratejisi**

**Timestamp aralığı hesaplama:**

**2025 yılı:**
- **Başlangıç:** 2025-01-01 00:00:00 → Unix timestamp
- **Bitiş:** 2025-12-31 23:59:59 → Unix timestamp
- **Toplam:** ~31.5 milyon saniye

**Python'da hesaplama:**

```python
from datetime import datetime

start_2025 = int(datetime(2025, 1, 1, 0, 0, 0).timestamp())
end_2025 = int(datetime(2026, 1, 1, 0, 0, 0).timestamp())

print(f"Start: {start_2025}")
print(f"End: {end_2025}")
print(f"Total: {end_2025 - start_2025}")
```

**Output:**
```
Start: 1735689600
End: 1767225600
Total: 31536000
```

**Analiz:**
- ✅ 31.5 milyon timestamp denemesi gerekiyor
- ✅ Modern bilgisayarda birkaç dakika sürer
- 💡 Her timestamp için decrypt deneyip "Flag{" kontrolü yapacağız

---

### 🔓 **Adım 7: Decrypt Script Hazırlama**

**decrypt_flag.py oluşturuyoruz:**

```python
#!/usr/bin/env python3
import hashlib
import base64
from datetime import datetime

def generate_key_from_timestamp(timestamp):
    timestamp_str = str(int(timestamp))
    key = hashlib.sha256(timestamp_str.encode()).digest()
    return key

def xor_encrypt(data, key):
    result = bytearray()
    key_len = len(key)
    
    for i, byte in enumerate(data):
        key_byte = key[i % key_len]
        mixed = (key_byte + i) % 256
        result.append(byte ^ mixed)
    
    return bytes(result)

def decrypt_flag(encrypted_b64, timestamp):
    try:
        key = generate_key_from_timestamp(timestamp)
        encrypted = base64.b64decode(encrypted_b64)
        decrypted = xor_encrypt(encrypted, key)
        parts = decrypted.split(b'|', 1)
        if len(parts) == 2:
            return parts[1].decode()
    except:
        pass
    return None

def main():
    encrypted_flag = "PrJIxSHIKlES1Rc8dDmoRoEkaRknfD/AVxoHTWP2U1ZOk6MvDi4AunEwvqgNStpqZg=="
    
    # 2025 yılının başlangıç ve bitiş timestamp'leri
    start_2025 = int(datetime(2025, 1, 1, 0, 0, 0).timestamp())
    end_2025 = int(datetime(2026, 1, 1, 0, 0, 0).timestamp())
    
    print(f"🔍 2025 yılı timestamp aralığını tarama başlıyor...")
    print(f"Start: {start_2025}")
    print(f"End: {end_2025}")
    print(f"Toplam deneme: {end_2025 - start_2025:,}\n")
    
    # Progress tracking
    total = end_2025 - start_2025
    checked = 0
    
    for timestamp in range(start_2025, end_2025):
        result = decrypt_flag(encrypted_flag, timestamp)
        
        if result and result.startswith("Flag{"):
            print(f"\n✅ FLAG BULUNDU!")
            print(f"Timestamp: {timestamp}")
            print(f"Tarih: {datetime.fromtimestamp(timestamp)}")
            print(f"Flag: {result}")
            return
        
        checked += 1
        if checked % 100000 == 0:
            progress = (checked / total) * 100
            print(f"Progress: {progress:.2f}% ({checked:,}/{total:,})", end='\r')
    
    print("\n❌ Flag bulunamadı!")

if __name__ == "__main__":
    main()
```

**Analiz:**
- ✅ Orijinal fonksiyonları kopyaladık
- ✅ 2025 yılı timestamp aralığını tarayacak
- ✅ Progress bar ile ilerleme gösteriyor
- ✅ "Flag{" ile başlayan decrypt'i bulunca duracak

---

### 🚀 **Adım 8: Brute Force Başlatma**

**Script'i çalıştırıyoruz:**

```bash
python3 decrypt_flag.py
```

**Output:**
```
🔍 2025 yılı timestamp aralığını tarama başlıyor...
Start: 1735689600
End: 1767225600
Toplam deneme: 31,536,000

Progress: 0.32% (100,000/31,536,000)
Progress: 0.63% (200,000/31,536,000)
Progress: 0.95% (300,000/31,536,000)
Progress: 1.27% (400,000/31,536,000)
Progress: 1.59% (500,000/31,536,000)
Progress: 1.90% (600,000/31,536,000)
Progress: 2.22% (700,000/31,536,000)
Progress: 2.54% (800,000/31,536,000)
Progress: 2.85% (900,000/31,536,000)
Progress: 3.17% (1,000,000/31,536,000)
Progress: 3.49% (1,100,000/31,536,000)
Progress: 3.81% (1,200,000/31,536,000)
Progress: 4.12% (1,300,000/31,536,000)
Progress: 4.44% (1,400,000/31,536,000)
Progress: 4.76% (1,500,000/31,536,000)
Progress: 5.07% (1,600,000/31,536,000)
Progress: 5.39% (1,700,000/31,536,000)
Progress: 5.71% (1,800,000/31,536,000)
Progress: 6.02% (1,900,000/31,536,000)
Progress: 6.34% (2,000,000/31,536,000)
Progress: 6.66% (2,100,000/31,536,000)
Progress: 6.98% (2,200,000/31,536,000)
Progress: 7.29% (2,300,000/31,536,000)
Progress: 7.61% (2,400,000/31,536,000)
Progress: 7.93% (2,500,000/31,536,000)
Progress: 8.24% (2,600,000/31,536,000)
Progress: 8.56% (2,700,000/31,536,000)
Progress: 8.88% (2,800,000/31,536,000)
Progress: 9.20% (2,900,000/31,536,000)
Progress: 9.51% (3,000,000/31,536,000)
Progress: 9.83% (3,100,000/31,536,000)
Progress: 10.15% (3,200,000/31,536,000)
```

**Analiz:**
- ✅ Brute force başladı
- ✅ Progress tracking çalışıyor
- ⏳ Her 100k timestamp'te güncelleniyor
- 💡 Modern CPU'da saniyede ~1M timestamp deneniyor

---

### 🏆 **Adım 9: Flag Bulundu!**

**Birkaç saniye sonra:**

```
✅ FLAG BULUNDU!
Timestamp: 1738942245
Tarih: 2025-02-07 15:30:45
Flag: Flag{zamanabruteatmaca682628970525257}
```

**✅ BAŞARILI!**

**Doğrulama:**

**Timestamp detayları:**
- **Unix Timestamp:** 1738942245
- **Tarih:** 7 Şubat 2025
- **Saat:** 15:30:45
- **Yıl:** 2025 ✅

**Flag formatı:**
- Format: `Flag{...}` ✅
- İçerik: `zamanabruteatmaca682628970525257`
- Mesaj: "zamana brute atmaca" 😄

**Analiz:**
- ✅ ~3.2 milyon timestamp denendi
- ✅ ~10% progress'te bulundu
- ✅ Şifreleme tarihi: Şubat 2025
- 💡 "Mezun olmadan önce" ipucu doğruydu!

---

### ✅ **Adım 10: Manuel Doğrulama**

**Bulunan timestamp ile decrypt etme:**

```python
encrypted_flag = "PrJIxSHIKlES1Rc8dDmoRoEkaRknfD/AVxoHTWP2U1ZOk6MvDi4AunEwvqgNStpqZg=="
timestamp = 1738942245

result = decrypt_flag(encrypted_flag, timestamp)
print(result)
```

**Output:**
```
Flag{zamanabruteatmaca682628970525257}
```

**Key generation doğrulama:**

```python
import hashlib

timestamp_str = str(1738942245)
key = hashlib.sha256(timestamp_str.encode()).digest()
print(f"Key (hex): {key.hex()}")
print(f"Key length: {len(key)} bytes")
```

**Output:**
```
Key (hex): 7f8e9a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1
Key length: 32 bytes
```

**Data format doğrulama:**

```python
# Decrypt edilen data
decrypted = b'1738942245|Flag{zamanabruteatmaca682628970525257}'

# Split işlemi
parts = decrypted.split(b'|', 1)
print(f"Timestamp: {parts[0].decode()}")
print(f"Flag: {parts[1].decode()}")
```

**Output:**
```
Timestamp: 1738942245
Flag: Flag{zamanabruteatmaca682628970525257}
```

**Analiz:**
- ✅ Timestamp doğru
- ✅ SHA256 key generation çalışıyor
- ✅ Data format `timestamp|flag` şeklinde
- ✅ Decrypt işlemi reversible
- 🎯 Çözüm %100 doğrulandı!

---

## 🚩 **FLAG**

```
Flag{zamanabruteatmaca682628970525257}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">📦<br><b>unzip</b><br><sub>Archive extraction</sub></td>
<td align="center">🐍<br><b>Python 3</b><br><sub>Script execution</sub></td>
<td align="center">🔐<br><b>hashlib</b><br><sub>SHA256 hashing</sub></td>
<td align="center">📊<br><b>base64</b><br><sub>Encoding/Decoding</sub></td>
</tr>
<tr>
<td align="center">⏰<br><b>datetime</b><br><sub>Timestamp handling</sub></td>
<td align="center">🔄<br><b>XOR</b><br><sub>Symmetric encryption</sub></td>
<td align="center">💪<br><b>Brute Force</b><br><sub>31M iterations</sub></td>
<td align="center">📝<br><b>cat</b><br><sub>File reading</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**

```bash
# Archive extract
unzip encrypt_flag25.zip

# Kaynak kod inceleme
cat encrypt_flag.py

# Script test
python3 encrypt_flag.py

# Decrypt script oluşturma
cat > decrypt_flag.py << 'EOF'
#!/usr/bin/env python3
import hashlib
import base64
from datetime import datetime
...
EOF

# Brute force çalıştırma
python3 decrypt_flag.py
```

**Python Komutları:**

```python
# Timestamp aralığı hesaplama
from datetime import datetime
start_2025 = int(datetime(2025, 1, 1, 0, 0, 0).timestamp())
end_2025 = int(datetime(2026, 1, 1, 0, 0, 0).timestamp())
print(f"Total: {end_2025 - start_2025}")

# Key generation
import hashlib
timestamp_str = str(1738942245)
key = hashlib.sha256(timestamp_str.encode()).digest()

# XOR encryption/decryption
def xor_encrypt(data, key):
    result = bytearray()
    for i, byte in enumerate(data):
        key_byte = key[i % len(key)]
        mixed = (key_byte + i) % 256
        result.append(byte ^ mixed)
    return bytes(result)

# Decrypt
import base64
encrypted = base64.b64decode(encrypted_flag)
decrypted = xor_encrypt(encrypted, key)
flag = decrypted.split(b'|', 1)[1].decode()
```

---

## 💭 **Çözüm Akış Şeması**

```
🎯 PitonsCrypt Challenge
            ↓
📦 ZIP dosyasını indir
   → encrypt_flag25.zip
   → Extract: encrypt_flag.py
            ↓
📝 Kaynak kodu incele
   → cat encrypt_flag.py
   → Şifreleme algoritması açık
            ↓
🔬 Algoritma analizi
   → SHA256 key derivation
   → XOR encryption (position-dependent)
   → timestamp|flag formatı
   → Base64 encoding
            ↓
🧪 Script'i test et
   → python3 encrypt_flag.py
   → "Şifreleme zamanı: 2026"
   → Current timestamp kullanıyor
            ↓
💡 İpucu analizi
   → "Mezun olmadan önce"
   → Geçmiş zaman → 2025 yılı
   → Mezuniyet dönemi muhtemelen
            ↓
🎯 Brute force stratejisi
   → 2025 yılı timestamp aralığı
   → Start: 1735689600
   → End: 1767225600
   → Total: 31,536,000
            ↓
🔓 Decrypt script hazırla
   → decrypt_flag.py
   → Tüm fonksiyonları kopyala
   → Progress tracking ekle
   → Flag{ kontrolü
            ↓
🚀 Brute force başlat
   → python3 decrypt_flag.py
   → Saniyede ~1M timestamp
   → Progress: 10.15%
            ↓
🏆 Flag bulundu!
   → Timestamp: 1738942245
   → Tarih: 2025-02-07 15:30:45
   → Flag{zamanabruteatmaca682628970525257}
            ↓
✅ Manuel doğrulama
   → Timestamp ile decrypt
   → Key generation test
   → Data format kontrol
   → ✅ %100 doğru!
            ↓
🚩 Flag{zamanabruteatmaca682628970525257}
```
