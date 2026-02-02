# 🔐 Ssl pinning bypass - Mobil Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Mobile-Challenge-darkblue?style=for-the-badge&logo=android" alt="Mobile">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Reverse Engineering-purple?style=for-the-badge&logo=apk" alt="APK">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Mobil  
**Seviye:** Orta  
**Puan:** 300  

**Challenge Dosyası:** [📥 Google Drive - Sslpinnng.apk](https://drive.google.com/file/d/1YsrJLthZfoDgH1q7o5QfQ-HqQp_93Wep/view?usp=drivesdk)

**Firstblood:** +50 puan

---

## 🔍 Analiz

**Hedef:** Android APK dosyasını reverse engineering yaparak AES şifreli password'ü decrypt etmek

**Strateji:** APK decompile → Smali analizi → AES key reconstruction → Decryption

**İpuçları:**
- 📱 **SSL Pinning:** Sertifika doğrulama bypass
- 🔐 **AES Encryption:** Symmetric encryption
- 🔑 **Key Construction:** Dinamik key oluşturma
- 🎯 **Format:** HTB{...} formatında flag

---

## ✅ Çözüm Adımları

### 📥 **1. Dosya Analizi**

**APK dosyasını indirip kontrol ediyoruz:**
```bash
file Sslpinnng.apk
```

**Çıktı:**
```
Sslpinnng.apk: Zip archive data, at least v2.0 to extract
```

> ✅ **Android APK** dosyası doğrulandı!

---

### 🔧 **2. APK Decompile**

**Apktool kullanarak APK'yı decompile ediyoruz:**
```bash
apktool d Sslpinnng.apk -o Sslpinnng_decoded
cd Sslpinnng_decoded
```

**Çıktı:**
```
I: Using Apktool 2.7.0 on Sslpinnng.apk
I: Loading resource table...
I: Decoding AndroidManifest.xml with resources...
I: Decoding file-resources...
I: Decoding values */* XMLs...
I: Baksmaling classes.dex...
I: Copying assets and libs...
I: Copying unknown files...
I: Copying original files...
```

**Dizin yapısı:**
```
AndroidManifest.xml
apktool.yml
res/
smali/
```

> ✅ **Decompile başarılı!**

---

### 📂 **3. MainActivity.smali Analizi**

**Ana sınıfı buluyoruz:**
```bash
find . -name "*MainActivity*"
```

**Sonuç:**
```
./smali/com/example/pinned/MainActivity.smali
./smali/com/example/pinned/MainActivity$a.smali
```

**MainActivity.smali'yi inceliyoruz:**
```bash
cat ./smali/com/example/pinned/MainActivity.smali | head -100
```

**Önemli bulgular:**
- 🌐 URL: `https://pinned.com:443/pinned.php`
- 👤 Username: `bnavarro`
- 🔑 Password: `1234567890987654`
- 🔐 Encrypted password (Base64): `zlg4rjdEd0Xvwel80q98Cc1Z1TPpCsLPGt63lw+sVsk3ED9a84hYDGfQn/gdiEvh`

---

### 🔍 **4. Yardımcı Fonksiyonları Analiz Etme**

**c/b/a/ klasöründeki fonksiyonların return değerlerini buluyoruz:**
```bash
for f in {a,b,c,d,e,f,h,i}; do
  tail -15 ./smali/c/b/a/$f.smali | grep -E "const|get|return"
done
```

**h.b() fonksiyonu için:**
```bash
grep -A 40 "method public static b()" ./smali/b/q/h.smali
```

**Her fonksiyon bir ArrayList<String> oluşturup belirli bir index döndürüyor:**

| Fonksiyon | Return Index | Return Value |
|-----------|--------------|--------------|
| a.a() | 1 | "fQn/gdiEvh" |
| b.a() | 8 | "dEd0Xv" |
| c.a() | 5 | "98Cc1Z" |
| d.a() | 2 | "zlg4rj" |
| e.a() | 2 | "lw+sVs" |
| f.a() | 1 | "k3ED9a" |
| h.a() | 0 | "wel80q" |
| h.b() | 6 | "2jOu89" |
| i.a() | 9 | "1TPpCs" |

---

### 🔑 **5. AES Key Reconstruction**

**MainActivity.smali satır 524-710 arası key oluşturma algoritması:**
```smali
invoke-static {}, Lb/q/h;->b()Ljava/lang/String;  # h.b() -> "2jOu89"
invoke-virtual {v9, v10}, Ljava/lang/String;->charAt(I)C  # charAt(3)

invoke-static {}, Lc/b/a/c;->a()Ljava/lang/String;  # c.a() -> "98Cc1Z"
invoke-virtual {v12, v11}, Ljava/lang/String;->charAt(I)C  # charAt(0)

# ... (devam ediyor)
```

**Python scripti ile key hesaplıyoruz:**
```python
#!/usr/bin/env python3

# String arrays
hb = "2jOu89"
ha = "wel80q"
ca = "98Cc1Z"
aa = "fQn/gdiEvh"
ia = "1TPpCs"
ba = "dEd0Xv"
ea = "lw+sVs"
da = "zlg4rj"

# Key construction
key = ""
key += hb[3]           # 'u'
key += hb[0]           # '2'
key += ca[0]           # '9'
key += aa[8].upper()   # 'V'
key += ha[1]           # 'e'
key += hb[0]           # '2'
key += ia[5].upper()   # 'S'
key += aa[7]           # 'E'
key += ba[4]           # 'X'
key += ea[4]           # 'V'
key += ea[4]           # 'V'
key += ia[5].upper()   # 'S'
key += da[3]           # '4'
key += da[5]           # 'j'
key += ha[1]           # 'e'
key += ha[1]           # 'e'

print(f"🔑 AES Key: {key}")
print(f"🔑 Key Length: {len(key)} characters")
```

**Çıktı:**
```
🔑 AES Key: u29Ve2SEXVVS4jee
🔑 Key Length: 16 characters
```

> ✅ **16 byte AES key başarıyla oluşturuldu!**

---

### 🔓 **6. AES Decryption**

**Python decryption script:**
```python
#!/usr/bin/env python3
from Crypto.Cipher import AES
import base64

# AES Key
aes_key = "u29Ve2SEXVVS4jee"

# Encrypted password (Base64)
encrypted_b64 = "zlg4rjdEd0Xvwel80q98Cc1Z1TPpCsLPGt63lw+sVsk3ED9a84hYDGfQn/gdiEvh"
encrypted_data = base64.b64decode(encrypted_b64)

print(f"[*] AES Key: {aes_key}")
print(f"[*] Encrypted length: {len(encrypted_data)} bytes")

# AES/ECB/PKCS5Padding (Java default)
cipher = AES.new(aes_key.encode('utf-8'), AES.MODE_ECB)
decrypted = cipher.decrypt(encrypted_data)

# Decrypted data
result = decrypted.decode('utf-8', errors='ignore')

print(f"\n{'='*70}")
print(f"🎉 DECRYPTED PASSWORD: {result}")
print(f"{'='*70}")
```

**Çalıştırıyoruz:**
```bash
python3 decrypt_aes.py
```

**Çıktı:**
```
[*] AES Key: u29Ve2SEXVVS4jee
[*] Encrypted length: 48 bytes

======================================================================
🎉 DECRYPTED PASSWORD: HTB{trust_n0_1_n0t_3v3n_@_c3rt!}2eXv 
======================================================================
```

> 🎉 **FLAG BULUNDU!**

---

## 🚩 **FLAG**
```
HTB{trust_n0_1_n0t_3v3n_@_c3rt!}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">📄<br><b>file</b><br><sub>Dosya tipi analizi</sub></td>
<td align="center">🔧<br><b>apktool</b><br><sub>APK decompile</sub></td>
<td align="center">🔍<br><b>grep</b><br><sub>Text arama</sub></td>
<td align="center">🐍<br><b>Python</b><br><sub>Key reconstruction</sub></td>
<td align="center">🔐<br><b>PyCryptodome</b><br><sub>AES decryption</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**
```bash
# Dosya analizi
file Sslpinnng.apk

# APK decompile
apktool d Sslpinnng.apk -o Sslpinnng_decoded

# MainActivity analizi
cat ./smali/com/example/pinned/MainActivity.smali

# Fonksiyon return değerlerini bulma
for f in {a,b,c,d,e,f,h,i}; do
  tail -15 ./smali/c/b/a/$f.smali | grep -E "const|get|return"
done

# h.b() fonksiyonu
grep -A 40 "method public static b()" ./smali/b/q/h.smali

# Key reconstruction ve decryption
python3 calculate_aes_key.py
python3 decrypt_aes.py
```

---

## 💭 **Çözüm Akışı**
```
🔐 "SSL Pinning Bypass" Challenge
            ↓
📥 Sslpinnng.apk indirildi
            ↓
📄 file Sslpinnng.apk → Android APK
            ↓
🔧 apktool d Sslpinnng.apk
            ↓
📂 MainActivity.smali bulundu
            ↓
🔍 Encrypted password tespit edildi
            ↓
📊 c/b/a/ fonksiyonları analiz edildi
            ↓
🔑 AES key algoritması çözüldü
            ↓
🧮 Key: u29Ve2SEXVVS4jee
            ↓
🔓 AES/ECB decrypt
            ↓
🎉 HTB{trust_n0_1_n0t_3v3n_@_c3rt!}
            ↓
🚩 FLAG: HTB{trust_n0_1_n0t_3v3n_@_c3rt!}
```
