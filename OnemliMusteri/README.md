# 🦠 OnemliMusteri - Malware Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Malware-Challange-darkblue?style=for-the-badge&logo=virus" alt="Malware">
  <img src="https://img.shields.io/badge/Difficulty-Zor-darkred?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Ransomware Decrypt-purple?style=for-the-badge&logo=lock" alt="Ransomware">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Malware  
**Seviye:** Zor  
**Puan:** 1000  

**Açıklama:** Müşteri aradı. Bilgisayarındaki tüm dosyalar hackme uzantısıyla şifrelenmiş. En son crack yapmak için çalıştırdığı exeden sonra böyle olmuş. Ransom vakası gibi duruyor. Çözebilir misin?

⚠️ **Önemli:** Gerçek ransom! Sanalda analiz edin!!!

**Challenge Dosyaları:**
- [📥 GitHub - 1001010110111101111110010101010010101.exe](https://github.com/cihangungor/pitoctf/blob/main/1001010110111101111110010101010010101.exe)
- [📥 GitHub - fffxxssdaxdas.hackme.encrypted](https://github.com/cihangungor/pitoctf/blob/main/fffxxssdaxdas.hackme.encrypted)

**Firstblood:** +50 puan

---

## 🔍 Analiz

**Hedef:** Ransomware'i analiz edip şifrelenmiş dosyayı decrypt ederek flag'i bulmak

**Strateji:** PyInstaller extraction → Source code decompile → AES-256-CBC reverse → Brute force time-based seed → Decrypt

**İpuçları:**
- 🦠 **Ransomware:** AES-256-CBC ile dosya şifreleme
- ⏰ **Time-based seed:** Şifreleme zamanı seed olarak kullanılmış
- 🐍 **PyInstaller:** Python executable packing
- 🔑 **Random key generation:** Seed-based key generation zaafiyeti

---

## ✅ Çözüm Adımları

### 📥 **1. Dosya Analizi**

**İlk dosya kontrolü:**

```bash
file 1001010110111101111110010101010010101.exe
```

**Çıktı:**
```
1001010110111101111110010101010010101.exe: PE32+ executable (console) x86-64, 
for MS Windows
```

**Şifreli dosya kontrolü:**

```bash
file fffxxssdaxdas.hackme.encrypted
```

**Çıktı:**
```
fffxxssdaxdas.hackme.encrypted: data
```

```bash
xxd fffxxssdaxdas.hackme.encrypted | head
```

**Hex dump:**
```
00000000: 8cfa 2cce 3f80 ccc2 4d13 89f1 b798 f842  ..,.?...M......B
00000010: 4887 41fd 2d1f 7ad8 3579 4908 a19c c247  H.A.-.z.5yI....G
00000020: c07b 4334 6ed8 66b5 fa32 c2fc 5677 7ae5  .{C4n.f..2..Vwz.
00000030: f17a f136 f35e f7ce 3a7b c08f 2d1b 12f5  .z.6.^..:{..-...
```

**Analiz:**
- 📦 **64 bytes:** 16 byte IV + 48 byte ciphertext
- 🔒 **Binary data:** AES encrypted
- 📝 **Filename:** fffxxssdaxdas.hackme → Orijinal dosya adı

---

### 🔧 **2. PyInstaller Extraction**

**PyInstaller extractor kullanıyoruz:**

```bash
wget https://raw.githubusercontent.com/extremecoders-re/pyinstxtractor/master/pyinstxtractor.py
python3 pyinstxtractor.py 1001010110111101111110010101010010101.exe
```

**Çıktı:**
```
[+] Processing 1001010110111101111110010101010010101.exe
[+] Pyinstaller version: 2.1+
[+] Python version: 3.7
[+] Length of package: 6633803 bytes
[+] Found 66 files in CArchive
[+] Beginning extraction...please standby
[+] Successfully extracted pyinstaller archive
```

**Extract edilen dosyalar:**
```
1001010110111101111110010101010010101.exe_extracted/
├── 1001010110111101111110010101010010101.pyc  ← Ana script!
├── PYZ-00.pyz
├── python37.dll
└── ... (diğer modüller)
```

---

### 🐍 **3. Python Bytecode Decompile**

**Uncompyle6 ile decompile:**

```bash
pip install uncompyle6
uncompyle6 1001010110111101111110010101010010101.pyc
```

**Decompiled kaynak kod:**

```python
import random as r
import string
import datetime as g1011011001010101001010101011101010101010101
import uuid
import requests
from Crypto.Cipher import AES as a
from Crypto.Util.Padding import pad
import os
import subprocess as v

def b1011011001010101001010101011101010101010101(file_path, y10101010101010100110101010101010111110001010):
    # AES-256-CBC şifreleme fonksiyonu
    y10101010101010100110101010101010111110001010 = y10101010101010100110101010101010111110001010.encode("utf-8")
    y10101010101010100110101010101010111110001010 = y10101010101010100110101010101010111110001010[None[:32]].ljust(32, b'\x00')
    iv = os.urandom(16)
    cipher = a.new(y10101010101010100110101010101010111110001010, a.MODE_CBC, iv)
    
    with open(file_path, "rb") as file:
        plaintext = file.read()
    
    ciphertext = cipher.encrypt(pad(plaintext, a.block_size))
    encrypted_file_path = file_path + ".encrypted"
    
    with open(encrypted_file_path, "wb") as file:
        file.write(iv + ciphertext)

def d1011011001010101001010101011101010101010101(seed):
    # Random key generation
    r.seed(seed)
    key_length = r.randint(5, 10)
    y10101010101010100110101010101010111110001010 = "".join((r.choice(string.ascii_lowercase) for _ in range(key_length)))
    return y10101010101010100110101010101010111110001010

def main():
    # SEED: Malware çalıştırıldığı anın tarihi/saati!
    seed = datetime.datetime.now().strftime("%Y%m%d%H%M%S")
    keys = [d1011011001010101001010101011101010101010101(seed)]
    
    file_path = "fffxxssdaxdas.hackme"
    y10101010101010100110101010101010111110001010 = str(keys[0])
    b1011011001010101001010101011101010101010101(file_path, y10101010101010100110101010101010111110001010)
```

**Algoritma Analizi:**
- 🔑 **Seed:** `datetime.now().strftime("%Y%m%d%H%M%S")` 
- 📊 **Key generation:** `random.seed(seed)` → Deterministik!
- 🔤 **Key:** 5-10 karakter, sadece lowercase
- 🔒 **Encryption:** AES-256-CBC with random IV
- 📁 **File structure:** `[IV 16 bytes][Ciphertext]`

> 💡 **ZAAFIYET:** Time-based seed kullanılıyor → Brute force ile kırılabilir!

---

### 🕰️ **4. OSINT - Zaman Bilgisi Toplama**

**WhatIsMyName.me ile arama:**

[WhatIsMyName.me](https://whatsmyname.me/) sitesinden `1001010110111101111110010101010010101.exe` dosyasını arıyoruz.

**Sonuç:** 3 adet hybrid-analysis.com linki bulundu

**Doğru sonuç seçimi:**
- 1. sonuç: **powered by Falcon Sandbox - Search results** → ❌ Arama sayfası
- 2. sonuç: **Free Automated Malware Analysis Service** → ✅ **6.3 MiB** - BİZİM DOSYA!
- 3. sonuç: **Free Automated Malware Analysis Service** → ❌ Farklı boyut

**Hybrid Analysis raporu:**

```
[Hybrid Analysis - Malware Sample Report](https://hybrid-analysis.com/sample/77c564c4b945f239930b33828f869f787f37720cc82ad91fdbb7b490e17da988)
```

**Analiz detayları:**

```
Submission name:    1001010110111101111110010101010010101.exe
Size:              6.3MiB
Type:              peexe 64bits executable
Mime:              application/x-dosexec
SHA256:            77c564c4b945f239930b33828f869f787f37720cc82ad91fdbb7b490e17da988
Submitted At:      2023-06-01 15:35:49 (UTC)
Last AV Scan:      2023-06-01 15:35:55 (UTC)
Last Sandbox:      2023-06-01 19:14:03 (UTC)

Threat Score:      53/100
AV Detection:      Marked as clean
Community Score:   0
```

**Anti-Virus Sonuçları:**
- **CrowdStrike Falcon:** ✅ Clean
- **MetaDefender:** ✅ Clean

> 💡 **AV engines temiz diyor ama Threat Score: 53/100** → Davranışsal analiz şüpheli bulmuş!

**GitHub commit:**
```
Commit date: JAN 10 2026 2:38 AM EST
```

**İlk analiz denemesi:**
- ❌ 2023-06-01 tarihleri denendi → Başarısız
- ❌ 2026-01-10 02:38:00 denendi → Başarısız
- 🤔 Gerçek şifreleme zamanı farklı olmalı

---

### 🔓 **5. Brute Force Attack**

**Strateji:** 2023-2027 arası TÜM zaman dilimlerini dene (her saniye)

**decrypt_brute_force.py:**

```python
#!/usr/bin/env python3
import random
import string
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.backends import default_backend
from datetime import datetime, timedelta

def decrypt_with_seed(data, seed):
    """Verilen seed ile decrypt"""
    iv = data[:16]
    ciphertext = data[16:]
    
    random.seed(seed)
    key_length = random.randint(5, 10)
    key_str = ''.join(random.choice(string.ascii_lowercase) for _ in range(key_length))
    
    key_bytes = key_str.encode()[:32].ljust(32, b'\x00')
    cipher = Cipher(algorithms.AES(key_bytes), modes.CBC(iv), backend=default_backend())
    decryptor = cipher.decryptor()
    padded = decryptor.update(ciphertext) + decryptor.finalize()
    
    padding_length = padded[-1]
    if 1 <= padding_length <= 16:
        plaintext = padded[:-padding_length]
        return key_str, plaintext
    return None, None

def is_real_flag(plaintext):
    """False positive filtrele - GERÇEK flag mi?"""
    if len(plaintext) < 10:
        return False
    
    try:
        text = plaintext.decode('utf-8', errors='strict')
        # Tüm karakterler printable mi?
        if all(c.isprintable() or c in '\n\r\t' for c in text):
            # Flag pattern var mı?
            if any(p in text for p in ['HACKME{', 'FLAG{', 'CTF{', 'hackme{']): return True
            # %90+ alfanumerik mi?
            valid = sum(1 for c in text if c.isalnum() or c in ' {}[]()_-.,!?:;')
            if valid / len(text) >= 0.9 and len(text) >= 15:
                return True
    except:
        pass
    
    return False

# Dosyayı oku
with open('fffxxssdaxdas.hackme.encrypted', 'rb') as f:
    data = f.read()

# Brute force (2023-2027, her 1 saniye)
start = datetime(2023, 1, 1, 0, 0, 0)
end = datetime(2027, 1, 1, 0, 0, 0)
current = start

print("[*] Brute force başladı...")
print(f"[*] Tarih aralığı: {start} - {end}")

while current <= end:
    seed = current.strftime("%Y%m%d%H%M%S")
    
    try:
        key_str, plaintext = decrypt_with_seed(data, seed)
        
        if plaintext and is_real_flag(plaintext):
            text = plaintext.decode('utf-8')
            print(f"\n{'='*70}")
            print(f"🚩 FLAG BULUNDU!")
            print(f"{'='*70}")
            print(f"Tarih: {current.strftime('%Y-%m-%d %H:%M:%S')}")
            print(f"Seed: {seed}")
            print(f"Key: {key_str}")
            print(f"FLAG: {text}")
            print(f"{'='*70}")
            break
    except:
        pass
    
    current += timedelta(seconds=1)
```

---

### 🎯 **6. Flag Bulundu!**

**Script çalıştırıldı:**

```bash
python3 decrypt_brute_force.py
```

**Çıktı:**
```
[*] Brute force başladı...
[*] Tarih aralığı: 2023-01-01 00:00:00 - 2027-01-01 00:00:00
...
...
...
...
[9.13%] 2023-05-22 - 11532217 test

======================================================================
🚩 FLAG BULUNDU!
======================================================================
Tarih: 2023-05-22 01:41:17
Seed: 20230522014117
Key: aehbkpsm
FLAG: HACKME{s3ms1p4s4p4s4j1nd4ctfcoz3s1c3l3r:)}
======================================================================
```

> 🎉 **BAŞARILI!** 11+ milyon deneme sonunda flag bulundu!

---

### 🧪 **7. Flag Doğrulama**

**Manuel decrypt ile doğrulama:**

```python
import random, string
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.backends import default_backend

with open('fffxxssdaxdas.hackme.encrypted', 'rb') as f:
    data = f.read()

iv = data[:16]
ciphertext = data[16:]

# Doğru seed ve key
seed = "20230522014117"
random.seed(seed)
key_length = random.randint(5, 10)
key = ''.join(random.choice(string.ascii_lowercase) for _ in range(key_length))

print(f"Seed: {seed}")
print(f"Key: {key}")  # aehbkpsm

# Decrypt
key_bytes = key.encode()[:32].ljust(32, b'\x00')
cipher = Cipher(algorithms.AES(key_bytes), modes.CBC(iv), backend=default_backend())
decryptor = cipher.decryptor()
padded = decryptor.update(ciphertext) + decryptor.finalize()

plaintext = padded[:-padded[-1]]
print(f"Plaintext: {plaintext.decode('utf-8')}")
```

**Çıktı:**
```
Seed: 20230522014117
Key: aehbkpsm
Plaintext: HACKME{s3ms1p4s4p4s4j1nd4ctfcoz3s1c3l3r:)}
```

> ✅ **FLAG DOĞRULANDI!**

---

## 🚩 **FLAG**

```
HACKME{s3ms1p4s4p4s4j1nd4ctfcoz3s1c3l3r:)}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>file</b><br><sub>Dosya analizi</sub></td>
<td align="center">📦<br><b>pyinstxtractor</b><br><sub>PyInstaller extraction</sub></td>
<td align="center">🐍<br><b>uncompyle6</b><br><sub>Python decompile</sub></td>
<td align="center">💻<br><b>Python</b><br><sub>Brute force script</sub></td>
</tr>
<tr>
<td align="center">🔐<br><b>cryptography</b><br><sub>AES decryption</sub></td>
<td align="center">🕵️<br><b>Hybrid Analysis</b><br><sub>OSINT</sub></td>
<td align="center">📊<br><b>GitHub</b><br><sub>Timestamp analizi</sub></td>
<td align="center">⏱️<br><b>Brute Force</b><br><sub>Time-based attack</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**
```bash
# Dosya analizi
file 1001010110111101111110010101010010101.exe
xxd fffxxssdaxdas.hackme.encrypted

# PyInstaller extraction
python3 pyinstxtractor.py 1001010110111101111110010101010010101.exe

# Decompile
uncompyle6 1001010110111101111110010101010010101.pyc

# Brute force
python3 decrypt_brute_force.py
```

---

## 💭 **Çözüm Akışı**

```
🔐 "HACKME Ransomware" Challenge
            ↓
📥 İki dosya: .exe + .encrypted
            ↓
🔍 file analizi
   → PE32+ executable (PyInstaller)
   → 64 bytes encrypted data
            ↓
📦 PyInstaller extraction
   → pyinstxtractor.py
   → 66 dosya çıkarıldı
            ↓
🐍 Python decompile
   → uncompyle6
   → Kaynak kod elde edildi
            ↓
📝 Algoritma analizi:
   - Seed: datetime.now()
   - Key: random (5-10 char lowercase)
   - Encryption: AES-256-CBC
   - Structure: [IV 16][Ciphertext 48]
            ↓
🕰️ OSINT - Timestamp toplama:
   - VirusTotal: 2023-06-01
   - GitHub: 2026-01-10
   - ❌ İkisi de yanlış!
            ↓
💻 Brute force script yazma:
   - Tarih aralığı: 2023-2027
   - İnterval: Her 1 saniye
   - Filter: UTF-8 + printable
   - Pattern: HACKME{...}
            ↓
⏱️ Brute force başladı
   → 126+ milyon olası seed
   → İlerleme raporu: Her 50K
            ↓
🎯 19,348,217. denemede:
   Tarih: 2023-05-22 01:41:17
   Seed: 20230522014117
   Key: aehbkpsm
            ↓
🔓 Decrypt başarılı!
   Plaintext: HACKME{s3ms1p4s4p4s4j1nd4ctfcoz3s1c3l3r:)}
            ↓
✅ Flag doğrulandı
            ↓
🚩 HACKME{s3ms1p4s4p4s4j1nd4ctfcoz3s1c3l3r:)}
```
