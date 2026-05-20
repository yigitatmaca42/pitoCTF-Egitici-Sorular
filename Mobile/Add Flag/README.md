# 🎯 Add Flag - Mobile Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Mobile-Challenge-darkblue?style=for-the-badge" alt="Mobile">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-APK Reverse-purple?style=for-the-badge" alt="APK Reverse">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Mobile  
**Seviye:** Orta  
**Puan:** 500

**Challenge Dosyası:** [📥 GitHub - AddFlag.apk](https://github.com/cihangungor/pitoctf/blob/main/AddFlag.apk)

---

## 🔍 Analiz

**Hedef:** Android APK dosyasını tersine mühendislik ile analiz ederek flag'i bulmak

**Strateji:** APK decompile → Smali analizi → Native kütüphane incelemesi → SharedPreferences anahtarları → SQLCipher DB decrypt → AES-256-GCM decrypt

**İpuçları:**
- 🎯 APK = Android uygulaması → `apktool` ile smali koduna dönüştür
- 🔍 `net.sqlcipher` kullanımı → Şifreli SQLite veritabanı var
- 💡 Native `libUtility.so` → AES-256-GCM şifreleme
- 🔑 SharedPreferences default değerleri `res/xml/myprefs.xml` içinde

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Dosya Tipini Tespit Et**

```bash
file AddFlag.apk
```

**Çıktı:**
```
AddFlag.apk: Android package (APK), with zipflinger virtual entry
```

---

### 📦 **Adım 2: APK'yı apktool ile Extract Et**

```bash
apktool d AddFlag.apk -o AddFlag_extracted
ls AddFlag_extracted/
```

**Çıktı:**
```
AndroidManifest.xml  apktool.yml  assets  lib  original  res  smali
```

**Analiz:**
- ✅ `smali/` → Decompile edilmiş Dalvik bytecode
- 💡 `assets/flags.ty` → Şifreli veritabanı dosyası
- 🎯 `lib/` → Native `.so` kütüphaneleri mevcut

---

### 🧠 **Adım 3: Smali Kod Analizi**

```bash
grep -r "flag\|CTF\|Flag" AddFlag_extracted/smali/com/example/addflag/ -i
```

**Kritik Bulgular:**
```
com/example/addflag/addFlag.smali:    .local v0, "flag":Ljava/lang/String;
com/example/addflag/addFlag.smali:    const-string v4, "flag"
com/example/addflag/addFlag.smali:    const-string v5, "FLAGS"
com/example/addflag/DisplayFlagsActivity.smali:    const-string v10, "flag"
com/example/addflag/DisplayFlagsActivity.smali:    const-string v3, "FLAGS"
b/a/a/d.smali:    const-string v0, "flags.ty"
b/a/a/d.smali:    const-string v0, "DROP TABLE IF EXISTS FLAGS"
```

**Analiz:**
- ✅ `FLAGS` tablosu → SQLite veritabanında flag'ler saklanıyor
- 💡 `flags.ty` → Asset olarak gelen şifreli DB dosyası
- 🎯 `net.sqlcipher` → SQLCipher ile şifreli DB açılıyor

---

### 💻 **Adım 4: utility.smali & b/a/a/d.smali Analizi**

```bash
cat AddFlag_extracted/smali/com/example/addflag/utility.smali
cat AddFlag_extracted/smali/b/a/a/d.smali
```

**Kritik Bulgular:**
```java
// utility.smali - Native metotlar
.method public native decrypt([BILjava/lang/String;Ljava/lang/String;[B[B)I
.method public native encrypt(Ljava/lang/String;ILjava/lang/String;Ljava/lang/String;[B[B)I
// "Utility" native kütüphanesi yükleniyor
invoke-static {v0}, Ljava/lang/System;->loadLibrary(Ljava/lang/String;)V

// b/a/a/d.smali - DB adı ve şifreleme
const-string v0, "flags.ty"         ← DB dosya adı
invoke-virtual {v0, p1}, Lb/a/a/d;->e(...)  ← encrypt fonksiyonu
invoke-virtual {v0, v3, v4}, Lb/a/a/d;->d(...)  ← decrypt fonksiyonu
```

**Analiz:**
- ✅ `libUtility.so` native kütüphanesi encrypt/decrypt işlemlerini yapıyor
- 💡 Key ve IV, `b/a/a/e` sınıfından SharedPreferences ile çekiliyor
- 🎯 `string_0` ve `string_1` anahtarları kritik

---

### 🔍 **Adım 5: b/a/a/e.smali Analizi**

```bash
cat AddFlag_extracted/smali/b/a/a/e.smali
```

**Kritik Bulgular:**
```java
// c() metodu - string_0 çekiyor (AES Key)
const-string v1, "string_0"
invoke-interface {v0, v1, v2}, Landroid/content/SharedPreferences;->getString(...)

// a() metodu - string_1 çekiyor (IV/Nonce)
const-string v1, "string_1"
invoke-interface {v0, v1, v2}, Landroid/content/SharedPreferences;->getString(...)

// d() metodu - string_2 çekiyor (DB şifresi)
const-string v1, "string_2"
invoke-interface {v0, v1, v2}, Landroid/content/SharedPreferences;->getString(...)
```

**Analiz:**
- ✅ Üç ayrı SharedPreferences değeri kullanılıyor
- 💡 `string_0` → AES anahtarı, `string_1` → IV, `string_2` → DB şifresi
- 🎯 Default değerler `res/xml/` altında olmalı

---

### 🔑 **Adım 6: Native Kütüphane Analizi**

```bash
strings AddFlag_extracted/lib/x86/libUtility.so | head -30
```

**Çıktı:**
```
Java_com_example_addflag_utility_encrypt
Java_com_example_addflag_utility_decrypt
EVP_aes_256_gcm
EVP_EncryptInit_ex
EVP_DecryptInit_ex
```

**Analiz:**
- ✅ **AES-256-GCM** şifreleme algoritması kullanılıyor
- 💡 `EVP_` fonksiyonları → OpenSSL implementasyonu
- 🎯 Key, IV ve Tag ile decrypt mümkün

---

### 🗝️ **Adım 7: SharedPreferences Default Değerlerini Bul**

```bash
find AddFlag_extracted/res/ -name "*.xml" | xargs grep -l "string_0\|string_1\|string_2"
cat AddFlag_extracted/res/xml/myprefs.xml
```

**Çıktı:**
```xml
<?xml version="1.0" encoding="utf-8"?>
<PreferenceScreen xmlns:android="http://schemas.android.com/apk/res/android">
    <EditTextPreference android:key="string_0" android:defaultValue="8[V3@eL521#@R2XNX3?4vygXw4$2Jr" />
    <EditTextPreference android:key="string_1" android:defaultValue="Fh@S/xW]y$?q" />
    <EditTextPreference android:key="string_2" android:defaultValue="1l0v3ch0c0lA7e" />
</PreferenceScreen>
```

**KRİTİK BULGULAR:**
- ✅ **AES Key** (`string_0`): `8[V3@eL521#@R2XNX3?4vygXw4$2Jr`
- ✅ **IV/Nonce** (`string_1`): `Fh@S/xW]y$?q`
- ✅ **DB Şifresi** (`string_2`): `1l0v3ch0c0lA7e`

---

### 🗄️ **Adım 8: SQLCipher DB'yi Aç**

```bash
sqlcipher AddFlag_extracted/assets/flags.ty
```

```sql
PRAGMA key = '1l0v3ch0c0lA7e';
SELECT * FROM FLAGS;
```

**Çıktı:**
```
system|l5wMg7HQCuXMk3Dkf3GDlLX52+VM0bZcDCQIZjyVJlKZ3hh9LMIUY13zzlgimU3IAAAAAAAAAAAAAAAAAAAAAA==|90|Tfcv8SdMzbVGThvKPAOlRw==
```

**KRİTİK BULGULAR:**
- ✅ **owner:** `system`
- ✅ **flag (ciphertext):** `l5wMg7HQCuXMk3Dkf3GDlLX52+VM0bZcDCQIZjyVJlKZ3hh9LMIUY13zzlgimU3IAAAAAAAAAAAAAAAAAAAAAA==`
- ✅ **tag:** `Tfcv8SdMzbVGThvKPAOlRw==`
- 🎯 AES-256-GCM ile decrypt gerekiyor

---

### 🚀 **Adım 9: AES-256-GCM ile Decrypt Et**

```python
python3 << 'EOF'
from cryptography.hazmat.primitives.ciphers.aead import AESGCM
import base64, hashlib

key_str = "8[V3@eL521#@R2XNX3?4vygXw4$2Jr"
iv_str = "Fh@S/xW]y$?q"

key = hashlib.sha256(key_str.encode()).digest()
iv = iv_str.encode()

ciphertext_b64 = "l5wMg7HQCuXMk3Dkf3GDlLX52+VM0bZcDCQIZjyVJlKZ3hh9LMIUY13zzlgimU3IAAAAAAAAAAAAAAAAAAAAAA=="
tag_b64 = "Tfcv8SdMzbVGThvKPAOlRw=="

ciphertext = base64.b64decode(ciphertext_b64)
tag = base64.b64decode(tag_b64)

aesgcm = AESGCM(key)
ct_with_tag = ciphertext.rstrip(b'\x00') + tag
result = aesgcm.decrypt(iv, ct_with_tag, None)
print("FLAG:", result.decode())
EOF
```

**Çıktı:**
```
FLAG: BSidesTLV2024{4ndr01d_dc0mp1l3_4nd_h00k_15_fun!}
```

---

## 🚩 **FLAG**
```
BSidesTLV2024{4ndr01d_dc0mp1l3_4nd_h00k_15_fun!}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔧<br><b>apktool</b><br><sub>APK decompile</sub></td>
<td align="center">🔍<br><b>strings</b><br><sub>Native lib analizi</sub></td>
<td align="center">🗄️<br><b>sqlcipher</b><br><sub>Şifreli DB açma</sub></td>
<td align="center">🐍<br><b>Python3</b><br><sub>AES-256-GCM decrypt</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 AddFlag Mobile Challenge
       ↓
📂 APK analizi
   → apktool ile extract
   → assets/flags.ty → şifreli SQLCipher DB
   → lib/ → libUtility.so (AES-256-GCM)
       ↓
🧠 Smali kod analizi
   → utility.smali → native encrypt/decrypt
   → b/a/a/e.smali → SharedPreferences'tan key/IV/DB şifresi
   → string_0, string_1, string_2 anahtarları
       ↓
🔑 Default değerleri bul
   → res/xml/myprefs.xml
   → string_0: 8[V3@eL521#@R2XNX3?4vygXw4$2Jr (AES Key)
   → string_1: Fh@S/xW]y$?q (IV)
   → string_2: 1l0v3ch0c0lA7e (DB şifresi)
       ↓
🗄️ SQLCipher DB aç
   → sqlcipher flags.ty
   → PRAGMA key = '1l0v3ch0c0lA7e'
   → Şifreli flag ve tag alındı
       ↓
🐍 AES-256-GCM decrypt
   → Key = SHA256(string_0)
   → IV = string_1
   → Python cryptography kütüphanesi
       ↓
🚩 FLAG: BSidesTLV2024{4ndr01d_dc0mp1l3_4nd_h00k_15_fun!}
```
