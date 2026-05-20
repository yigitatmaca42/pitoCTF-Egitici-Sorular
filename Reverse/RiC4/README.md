# 🎯 RiC4 - Reverse Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Reverse-Challenge-darkblue?style=for-the-badge" alt="Reverse">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-PyInstaller RC4-purple?style=for-the-badge" alt="PyInstaller RC4">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Reverse  
**Seviye:** Orta  
**Puan:** 500

**Challenge Dosyası:** [📥 Google Drive - Rc.exe](https://drive.google.com/file/d/1u6yhF5K50HcR9GwapPp1oQp_zm0KkmB1/view?usp=drivesdk)

---

## 🔍 Analiz

**Hedef:** Exe dosyasını tersine mühendislik ile analiz edip flag'i bulmak

**Strateji:** Dosya tipi tespiti → PyInstaller tespiti → Python kaynak kodunu çıkarma → RC4 decrypt

**İpuçları:**
- 🎯 `Rc.exe` adı RC4 şifrelemeye işaret ediyor
- 🔍 `PyMarshal_ReadObjectFromString` strings çıktısında görünüyor → PyInstaller ile paketlenmiş
- 💡 `chal.pyc` içinde RC4 implementasyonu ve şifreli flag mevcut
- 🔑 Key = `Rivest` (RC4'ün mucidi Ron Rivest'ten geliyor)

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Dosyayı İncele**

```bash
file Rc.exe
```

**Çıktı:**
```
Rc.exe: PE32+ executable for MS Windows 5.02 (console), x86-64, 7 sections
```

**Analiz:**
- ✅ Windows 64-bit PE executable
- 💡 Strings incelendi, `PyMarshal_ReadObjectFromString` ve `PyRun_SimpleStringFlags` görüldü
- 🎯 PyInstaller ile paketlenmiş Python uygulaması!

---

### 🐍 **Adım 2: PyInstaller Tespiti**

```bash
strings Rc.exe | grep -i "flag\|ctf\|pito\|{"
```

**Kritik Çıktı:**
```
PyMarshal_ReadObjectFromString
PyRun_SimpleStringFlags
python311.dll
```

**Analiz:**
- ✅ Python 3.11 ile paketlenmiş PyInstaller binary'si
- 💡 `wine Rc.exe` denenince `python311.dll` hatası verdi → `_internal` klasörü gerekiyor
- 🎯 pyinstxtractor ile Python dosyalarını çıkaralım

---

### 📦 **Adım 3: PyInstaller'dan Python Dosyalarını Çıkar**

pyinstxtractor pip'te bulunmadığı için GitHub'dan direkt indirildi:

```bash
wget https://raw.githubusercontent.com/extremecoders-re/pyinstxtractor/master/pyinstxtractor.py
python3 pyinstxtractor.py Rc.exe
```

**Çıktı:**
```
[+] Processing Rc.exe
[+] Pyinstaller version: 2.1+
[+] Python version: 3.11
[+] Length of package: 1336887 bytes
[+] Found 10 files in CArchive
[+] Possible entry point: pyiboot01_bootstrap.pyc
[+] Possible entry point: pyi_rth_inspect.pyc
[+] Possible entry point: chal.pyc
[+] Successfully extracted pyinstaller archive: Rc.exe
```

```bash
ls Rc.exe_extracted/
```

**Çıktı:**
```
chal.pyc  pyiboot01_bootstrap.pyc  pyimod01_archive.pyc
pyimod02_importers.pyc  pyimod03_ctypes.pyc  pyimod04_pywin32.pyc
pyi_rth_inspect.pyc  PYZ-00.pyz  PYZ-00.pyz_extracted  struct.pyc
```

**Analiz:**
- ✅ `chal.pyc` → ana challenge dosyası tespit edildi
- 🎯 Bu .pyc dosyasını decompile etmek gerekiyor

---

### 🔬 **Adım 4: .pyc Dosyasını Decompile Et**

`uncompyle6` Python 3.11'i desteklemediği için `pycdc` kullanıldı:

```bash
git clone https://github.com/zrax/pycdc
cd pycdc
cmake .
make
```

```bash
./pycdc ~/Desktop/soru/Rc.exe_extracted/chal.pyc
```

**Çıktı (kaynak kod):**
```python
import codecs
import base64
import sys

def KSA(key):
    S = list(range(256))
    j = 0
    for i in range(256):
        j = (j + S[i] + key[i % len(key)]) % 256
        S[i], S[j] = S[j], S[i]
    return S

def PRGA(S):
    # Generator - keystream üretir
    pass

def encrypt(plaintext):
    key = 'Rivest'
    keystream = PRGA(KSA(key))
    enc = []
    for i in range(len(plaintext)):
        enc.append(plaintext[i] ^ next(keystream))
    byte_string = bytes(enc)
    b64_enc = base64.b64encode(byte_string)
    return b64_enc

flag = input('Please enter the flag: ')
enc_flag = encrypt(flag)
if enc_flag == b'aJlQBCwh9JM2RHmXLy+PA6cUIFDC3A==':
    print('Correct!')
print('Wrong!')
```

**KRİTİK BULGULAR:**
- ✅ RC4 şifrelemesi kullanılmış (KSA + PRGA = RC4)
- 💡 Key = `'Rivest'` → RC4'ün mucidi Ron **Rivest**'ten geliyor, challenge adı da **RiC4**!
- 🎯 Hedef ciphertext: `aJlQBCwh9JM2RHmXLy+PA6cUIFDC3A==`
- 🔑 RC4 simetrik şifreleme → aynı fonksiyon ile decrypt edilebilir

---

### 🚀 **Adım 5: RC4 Decrypt**

RC4 simetrik olduğu için encrypt fonksiyonu aynı zamanda decrypt fonksiyonudur:

```python
import base64

def KSA(key):
    S = list(range(256))
    j = 0
    for i in range(256):
        j = (j + S[i] + key[i % len(key)]) % 256
        S[i], S[j] = S[j], S[i]
    return S

def PRGA(S):
    i = 0
    j = 0
    while True:
        i = (i + 1) % 256
        j = (j + S[i]) % 256
        S[i], S[j] = S[j], S[i]
        yield S[(S[i] + S[j]) % 256]

def decrypt(enc_b64):
    key = [ord(c) for c in 'Rivest']
    enc = base64.b64decode(enc_b64)
    keystream = PRGA(KSA(key))
    dec = []
    for byte in enc:
        dec.append(byte ^ next(keystream))
    return bytes(dec).decode()

print(decrypt(b'aJlQBCwh9JM2RHmXLy+PA6cUIFDC3A=='))
```

**Çıktı:**
```
AOFCTF{rc4_go_brrrrrr}
```

---

## 🚩 **FLAG**
```
AOFCTF{rc4_go_brrrrrr}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">📦<br><b>pyinstxtractor</b><br><sub>PyInstaller extraction</sub></td>
<td align="center">🔬<br><b>pycdc</b><br><sub>.pyc decompiler</sub></td>
<td align="center">🐍<br><b>Python3</b><br><sub>RC4 decrypt</sub></td>
<td align="center">🔍<br><b>strings</b><br><sub>Binary analiz</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 RiC4 Reverse Challenge
       ↓
🔍 file Rc.exe
   → PE32+ Windows 64-bit executable
       ↓
🔎 strings Rc.exe
   → PyMarshal_ReadObjectFromString görüldü
   → Python 3.11 ile paketlenmiş PyInstaller binary!
       ↓
📦 pyinstxtractor.py Rc.exe
   → 10 dosya çıkarıldı
   → chal.pyc tespit edildi ✅
       ↓
🔬 pycdc chal.pyc
   → uncompyle6 Python 3.11'i desteklemedi
   → pycdc ile kaynak kod elde edildi
   → RC4 (KSA + PRGA) implementasyonu
   → key = 'Rivest'
   → hedef = 'aJlQBCwh9JM2RHmXLy+PA6cUIFDC3A=='
       ↓
🚀 Python ile RC4 decrypt
   → RC4 simetrik → aynı fonksiyon decrypt eder
   → base64 decode → XOR keystream
       ↓
🚩 FLAG: AOFCTF{rc4_go_brrrrrr}
```
