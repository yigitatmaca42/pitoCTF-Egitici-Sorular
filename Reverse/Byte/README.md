# 🔄 Byte - Reverse Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Reverse-Challenge-darkblue?style=for-the-badge&logo=python" alt="Reverse">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Bytecode-purple?style=for-the-badge&logo=code" alt="Bytecode">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Reverse  
**Seviye:** Kolay  
**Puan:** 200

**Challenge Dosyası:** [📥 Google Drive - chall.txt](https://drive.google.com/file/d/1riE7acg2PVrcIkmoNI6F9Xb_tKj6qIhd/view?usp=sharing)


---

## 🔍 Analiz

**Hedef:** Python bytecode'u analiz edip şifrelenmiş mesajı decrypt etmek

**Strateji:** Bytecode okuma → Python koduna çevirme → XOR key decode → Fernet decrypt

---

## ✅ Çözüm Adımları

### 📥 **1. Dosyayı İndirme**

Google Drive'dan `chall.txt` dosyasını indiriyoruz.

**Dosya içeriği:** Python bytecode disassembly çıktısı

---

### 🔍 **2. Bytecode Analizi**

**Bytecode'u incelediğimizde şu bilgileri çıkarıyoruz:**

**Import edilen kütüphaneler:**
```python
import base64
from cryptography.fernet import Fernet
```

**Encrypted mesaj:**
```python
encMessage = b'gAAAAABj7Xd90ySo11DSFyX8t-9QIQvAPmU40mWQfpq856jFl1rpwvm1kyE1w23fyyAAd9riXt-JJA9v6BEcsq6LNroZTnjExjFur_tEp0OLJv0c_8BD3bg='
```

**Base64 encoded key:**
```python
encoded_key = b'7PXy9PSZmf/r5pXB79LW1cj/7JT6ltPEmfjk8sHljfr6x/LyyfjymNXR5Z0='
```

**XOR işlemi:**
```python
# Her byte 160 ile XOR'lanıyor
for k_b in key_bytes:
    key.append(k_b ^ 160)
```

---

### 🔓 **3. Python Koduna Dönüştürme**

Bytecode'u analiz ederek orijinal Python kodunu yeniden yazıyoruz:

```python
import base64
from cryptography.fernet import Fernet

# Encrypted message
encMessage = b'gAAAAABj7Xd90ySo11DSFyX8t-9QIQvAPmU40mWQfpq856jFl1rpwvm1kyE1w23fyyAAd9riXt-JJA9v6BEcsq6LNroZTnjExjFur_tEp0OLJv0c_8BD3bg='

# Base64 decode the key
key_bytes = base64.b64decode(b'7PXy9PSZmf/r5pXB79LW1cj/7JT6ltPEmfjk8sHljfr6x/LyyfjymNXR5Z0=')

# XOR each byte with 160
key = []
for k_b in key_bytes:
    key.append(k_b ^ 160)

key = bytes(key)

# Decrypt with Fernet
fernet = Fernet(key)
decMessage = fernet.decrypt(encMessage).decode()

print(decMessage)
```

---

### 🛠️ **4. Çözüm Scripti**

**solve.py:**

```python
import base64
from cryptography.fernet import Fernet

# Encrypted message
encMessage = b'gAAAAABj7Xd90ySo11DSFyX8t-9QIQvAPmU40mWQfpq856jFl1rpwvm1kyE1w23fyyAAd9riXt-JJA9v6BEcsq6LNroZTnjExjFur_tEp0OLJv0c_8BD3bg='

# Encoded key
encoded_key = b'7PXy9PSZmf/r5pXB79LW1cj/7JT6ltPEmfjk8sHljfr6x/LyyfjymNXR5Z0='

# Base64 decode
key_bytes = base64.b64decode(encoded_key)

# XOR with 160
key = []
for k_b in key_bytes:
    key.append(k_b ^ 160)

key = bytes(key)

print(f"Decoded Key: {key}")

# Decrypt
fernet = Fernet(key)
decMessage = fernet.decrypt(encMessage).decode()

print(f"\nFlag: {decMessage}")
```

---

### 🎯 **5. Script Çalıştırma**

```bash
python3 solve.py
```

**Çıktı:**

```
Decoded Key: b'LURTT99_KF5aOrvuh_L4Z6sd9XDRaE-ZZgRRiXR8uqE='

Flag: FLAG{FLY_L1k3_0xR4V3N}
```

---

## 🚩 **FLAG**

```
FLAG{FLY_L1k3_0xR4V3N}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🐍<br><b>Python 3</b><br><sub>Script çalıştırma</sub></td>
<td align="center">📦<br><b>base64</b><br><sub>Decode işlemi</sub></td>
<td align="center">🔐<br><b>cryptography</b><br><sub>Fernet decrypt</sub></td>
</tr>
</table>

**Kurulum:**
```bash
# Gerekli kütüphaneyi yükle
pip install cryptography

# Scripti çalıştır
python3 solve.py
```

---

## 💭 **Çözüm Akışı**

```
🔄 Bytecode Challenge: "Byte"
            ↓
📥 chall.txt indirildi
            ↓
🔍 Bytecode analiz edildi
            ↓
📝 Python koduna dönüştürüldü
            ↓
🔑 Key: Base64 decode + XOR 160
            ↓
🔓 Fernet ile decrypt
            ↓
🚩 FLAG{FLY_L1k3_0xR4V3N}
```
