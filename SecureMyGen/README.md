# 📱 SecureMyGen - Mobile Challenge 

<p align="center">
  <img src="https://img.shields.io/badge/Mobile-Challenge-darkblue?style=for-the-badge" alt="Mobile">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-APK Analysis + AES Decryption-purple?style=for-the-badge" alt="APK">
</p>

---

## 📋 Soru Bilgileri
  
**Kategori:** Mobil  
**Seviye:** Orta  
**Puan:** 500  

**Challenge Dosyası:** [📥 Github - SecureMyGen.apk](https://github.com/cihangungor/pitoctf/blob/main/SecureMyGen.apk)

---

## 🔍 Analiz

**Hedef:** Android APK dosyasını decompile ederek içindeki AES encrypted flag'i çözmek

**Strateji:** APK extraction → Decompilation → Encryption analizi → Key bulma → AES decryption → Flag

**İpuçları:**
- 🎯 "SecureMyGen" → Secure Generator - AES şifreleme var
- 📱 APK dosyası → Android uygulaması analizi
- 🔐 Smali kodu içinde hardcoded key olabilir

---

## ✅ Çözüm Adımları

### 🔧 **Adım 1: Dosya Kontrolü - File Type Verification**

**APK dosyasının tipini kontrol ediyoruz:**

```bash
file SecureMyGen.apk
```

**Response:**

```
SecureMyGen.apk: Android package (APK), with AndroidManifest.xml
```

**Analiz:**
- ✅ Android package (APK)
- ✅ AndroidManifest.xml içeriyor
- 💡 APK aslında bir ZIP dosyası

---

### 📦 **Adım 2: APK Extraction - İçeriği Çıkarma**

**APK dosyasını unzip ile çıkartıyoruz:**

```bash
unzip SecureMyGen.apk -d SecureMyGen
ls -la SecureMyGen/
```

**Response:**

```
Archive: SecureMyGen.apk
  inflating: SecureMyGen/AndroidManifest.xml
  extracting: SecureMyGen/classes.dex
  extracting: SecureMyGen/classes2.dex
  extracting: SecureMyGen/classes3.dex
  inflating: SecureMyGen/resources.arsc
  inflating: SecureMyGen/res/...
```

**Key Files Found:**
- ✅ `AndroidManifest.xml` - Uygulama manifest
- ✅ `classes.dex` - Ana Dalvik bytecode
- ✅ `classes2.dex` - Ek bytecode
- ✅ `classes3.dex` - Ek bytecode
- ✅ `resources.arsc` - Kaynak dosyaları

---

### 🔨 **Adım 3: APKTool Decompilation - Smali Kod Elde Etme**

**APK'yı smali koduna çeviriyoruz:**

```bash
# Eski dizini temizle
rm -rf SecureMyGen

# APKTool ile decompile et
apktool d SecureMyGen.apk
```

**Response:**

```
I: Using Apktool 2.x.x
I: Loading resource table...
I: Decoding AndroidManifest.xml with resources...
I: Loading resource table from file: framework-res.apk
I: Regular manifest package...
I: Decoding file-resources...
I: Decoding values */* XMLs...
I: Baksmaling classes.dex...
I: Baksmaling classes2.dex...
I: Baksmaling classes3.dex...
I: Copying assets and libs...
I: Copying unknown files...
I: Copying original files...
```

**Analiz:**
- ✅ Decompilation başarılı
- ✅ Smali kodları elde edildi
- ✅ 3 ayrı dex dosyası decode edildi

---

### 🔍 **Adım 4: Encryption Code Search - AES Aramak**

**AES şifreleme ile ilgili dosyaları arıyoruz:**

```bash
find SecureMyGen/smali* -name "*.smali" | xargs grep -l "Cipher\|AES" | head -5
```

**Response:**

```
SecureMyGen/smali_classes3/com/securegenaes/hackmasters_securemygen/Aa.smali
SecureMyGen/smali_classes3/com/securegenaes/hackmasters_securemygen/Ca.smali
```

**🎯 Şüpheli Dosyalar Bulundu!**
- ✅ `Aa.smali` - Ana encryption sınıfı gibi görünüyor
- ✅ `Ca.smali` - Yardımcı sınıf olabilir

---

### 💎 **Adım 5: Aa.smali Analizi - CRITICAL FINDINGS!**

**Aa.smali dosyasını inceliyoruz:**

```bash
cat SecureMyGen/smali_classes3/com/securegenaes/hackmasters_securemygen/Aa.smali
```

**🎉 ÖNEMLİ BULGULAR!**

**1. Encrypted Flag (Base64):**
```smali
const-string v0, "IRZJFYZmLFl8aSldn0FcSyzLu4Upm+cMJvdz5H3fmik="
```

**2. Encryption Method:**
```smali
const-string v1, "AES/ECB/PKCS5Padding"
```

**3. Fake Key (Red Herring):**
```smali
const-string v1, "whatsthislol"
```

**Analiz:**
- ✅ **Encrypted flag base64 encoded**
- ✅ **AES ECB mode** kullanılıyor
- ⚠️ "whatsthislol" gerçek key değil - kandırmaca!
- 💡 Gerçek key başka yerde olmalı

---

### 🔎 **Adım 6: Real Key Hunting - Gerçek Anahtarı Bulmak**

**Tüm const-string değerlerini tarıyoruz:**

```bash
grep "const-string" SecureMyGen/smali_classes3/com/securegenaes/hackmasters_securemygen/Aa.smali
```

**Response (excerpt):**

```smali
const-string v1, "8d"
const-string v1, "12"
const-string v1, "76"
const-string v1, "84"
const-string v1, "cb"
const-string v1, "c3"
const-string v1, "7c"
const-string v1, "17"
const-string v1, "61"
const-string v1, "6d"
const-string v1, "80"
const-string v1, "6c"
const-string v1, "f5"
const-string v1, "04"
const-string v1, "73"
const-string v1, "cc"
```

**🎁 BINGO! Hex Değerleri Bulundu!**

**16 adet hex string:**
```
"8d", "12", "76", "84", "cb", "c3", "7c", "17",
"61", "6d", "80", "6c", "f5", "04", "73", "cc"
```

**Analiz:**
- ✅ 16 hex değer = 16 byte
- ✅ 16 byte = 128-bit AES key
- 💡 Muhtemelen metodlar (`a()`, `b()`, `c()`) bu hex'leri döndürüyor
- 💡 Birleştirilerek AES key'i oluşturuluyor

---

### 🧬 **Adım 7: Code Analysis - Smali Kod İncelemesi**

**Smali metodlarını analiz ediyoruz:**

```smali
# b(String) metodu - Hex string'i byte array'e çevirir
.method public static b(Ljava/lang/String;)[B
    # Hex string'den byte dizisi oluştur
    ...
.end method

# bb([B[B) metodu - AES ECB ile decrypt yapar
.method public static bb([B[B)[B
    const-string v1, "AES/ECB/PKCS5Padding"
    invoke-static {v1}, Ljavax/crypto/Cipher;->getInstance(Ljava/lang/String;)Ljavax/crypto/Cipher;
    # AES decryption işlemi
    ...
.end method
```

**Kod Akışı:**
1. ✅ Hex string'ler metodlardan döner (`a()`, `b()`, `c()`, ...)
2. ✅ Hex string'ler birleştirilir
3. ✅ `b(String)` ile byte array'e çevrilir
4. ✅ `bb([B[B)` ile AES ECB decrypt yapılır

---

### 🐍 **Adım 8: Python Decryption Script - FLAG ELDE ETME!**

**Python ile AES decrypt ediyoruz:**

```python
import base64
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad

# Encrypted flag (Base64)
encrypted_b64 = "IRZJFYZmLFl8aSldn0FcSyzLu4Upm+cMJvdz5H3fmik="
encrypted = base64.b64decode(encrypted_b64)

# Hex değerlerini sırayla birleştir
hex_values = ["8d", "12", "76", "84", "cb", "c3", "7c", "17", 
              "61", "6d", "80", "6c", "f5", "04", "73", "cc"]
key_hex = ''.join(hex_values)

print(f"Key hex: {key_hex}")
print(f"Key length: {len(key_hex)//2} bytes")

# Hex'i byte'a çevir
key = bytes.fromhex(key_hex)

# AES ECB ile decrypt et
cipher = AES.new(key, AES.MODE_ECB)
decrypted = cipher.decrypt(encrypted)
flag = unpad(decrypted, AES.block_size).decode()

print("FLAG:", flag)
```

**🎉 SCRIPT ÇALIŞTIRILDI! Response:**

```
Key hex: 8d127684cbc37c17616d806cf50473cc
Key length: 16 bytes
FLAG: HM2024{aeS_secureMy4PP_ecB}
```

**✅ BAŞARILI! FLAG ELDE EDİLDİ!**

---

## 🏁 **FLAG**

```
HM2024{aeS_secureMy4PP_ecB}
```

**Flag Anlamı:**
- `aeS` → AES encryption
- `secureMy4PP` → Secure My App (APK'nın ismi)
- `ecB` → ECB mode (güvensiz mode!)

---

## 🧬 Teknik Analiz

### **APK Dosya Yapısı**

**Android APK aslında bir ZIP arşividir:**
```bash
# APK = ZIP
unzip SecureMyGen.apk

# İçindekiler:
├── AndroidManifest.xml    # Uygulama bilgileri
├── classes.dex             # Dalvik bytecode (Java/Kotlin kodu)
├── classes2.dex            # Ek dex dosyaları
├── classes3.dex
├── resources.arsc          # Compiled kaynaklar
├── res/                    # UI kaynakları (layout, drawable)
├── META-INF/               # İmza dosyaları
└── lib/                    # Native library'ler (.so)
```

**DEX (Dalvik Executable):**
- ✅ Android'in çalıştırılabilir formatı
- ✅ Java bytecode'un optimized versiyonu
- ✅ APKTool ile Smali'ye çevrilebilir

---

### **Smali Kod Analizi**

**Smali nedir?**
- ✅ Dalvik bytecode'un human-readable hali
- ✅ APKTool ile decompilation sonucu
- ✅ Assembly benzeri syntax

**Örnek Smali Kodu:**
```smali
# String tanımlama
const-string v0, "IRZJFYZmLFl8aSldn0FcSyzLu4Upm+cMJvdz5H3fmik="

# Method çağrısı
invoke-static {v0}, Landroid/util/Base64;->decode(Ljava/lang/String;I)[B
move-result-object v1

# AES Cipher oluşturma
const-string v2, "AES/ECB/PKCS5Padding"
invoke-static {v2}, Ljavax/crypto/Cipher;->getInstance(Ljava/lang/String;)Ljavax/crypto/Cipher;
```

---

### **AES ECB Mode Analizi**

**Neden ECB güvensiz?**

```
Plaintext:  [BLOCK1] [BLOCK2] [BLOCK3]
               ↓         ↓         ↓
            [ENC]     [ENC]     [ENC]  (Same key, no IV)
               ↓         ↓         ↓
Ciphertext: [CIPH1]  [CIPH2]  [CIPH3]
```

**Problem:**
- ❌ Aynı plaintext block → Aynı ciphertext block
- ❌ Pattern'ler korunur (örneğin resimler)
- ❌ IV (Initialization Vector) yok
- ❌ Parallel attack mümkün

**Güvenli Alternatifler:**
- ✅ **AES-GCM** (en önerilen)
- ✅ **AES-CBC** (IV ile)
- ✅ **AES-CTR** (nonce ile)

**Challenge'daki implementasyon:**
```java
// GÜVENSİZ KOD (Challenge'dan)
Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
cipher.init(Cipher.DECRYPT_MODE, secretKey);
byte[] decrypted = cipher.doFinal(encrypted);
```

---

### **Hardcoded Key Problemi**

**Challenge'daki key:**
```smali
# 16 farklı const-string ile dağıtılmış
const-string v1, "8d"
const-string v1, "12"
...
const-string v1, "cc"

# Birleştirilmiş key:
8d127684cbc37c17616d806cf50473cc
```

**Neden tehlikeli?**
- ❌ APK reverse engineering ile bulunabilir
- ❌ String obfuscation yeterli değil
- ❌ Key rotation imkansız
- ❌ Tüm kullanıcılar aynı key'i kullanıyor

---

## 🛡️ Nasıl Önlenir?

### **1. Secure Key Management**

```java
// GÜVENSİZ - Hardcoded key ❌
byte[] key = new byte[]{0x8d, 0x12, 0x76, ...};

// GÜVENLİ - Android Keystore ✅
KeyGenerator keyGen = KeyGenerator.getInstance(
    KeyProperties.KEY_ALGORITHM_AES, "AndroidKeyStore");

KeyGenParameterSpec keyGenSpec = new KeyGenParameterSpec.Builder(
    "MyKeyAlias",
    KeyProperties.PURPOSE_ENCRYPT | KeyProperties.PURPOSE_DECRYPT)
    .setBlockModes(KeyProperties.BLOCK_MODE_GCM)
    .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)
    .build();

keyGen.init(keyGenSpec);
SecretKey secretKey = keyGen.generateKey();
```

**Android Keystore Avantajları:**
- ✅ Hardware-backed (TEE/SE)
- ✅ Key extraction imkansız
- ✅ Biometric authentication ile korunabilir
- ✅ Root detection

---

### **2. Secure Encryption Mode**

```java
// GÜVENSİZ - ECB mode ❌
Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");

// GÜVENLİ - GCM mode ✅
Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");

// Random IV oluştur
byte[] iv = new byte[12];
SecureRandom random = new SecureRandom();
random.nextBytes(iv);
GCMParameterSpec spec = new GCMParameterSpec(128, iv);

cipher.init(Cipher.ENCRYPT_MODE, secretKey, spec);
byte[] encrypted = cipher.doFinal(plaintext);

// IV'yi ciphertext'in başına ekle
byte[] result = new byte[iv.length + encrypted.length];
System.arraycopy(iv, 0, result, 0, iv.length);
System.arraycopy(encrypted, 0, result, iv.length, encrypted.length);
```

---

### **3. Code Obfuscation**

**ProGuard/R8 Kullanımı:**
```proguard
# build.gradle
android {
    buildTypes {
        release {
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'),
                          'proguard-rules.pro'
        }
    }
}

# proguard-rules.pro
-keep class com.myapp.security.** { *; }
-keepclassmembers class * {
    @com.myapp.annotations.DoNotObfuscate *;
}

# String encryption
-obfuscatedictionary obfuscation-dictionary.txt
```

**Native Code (JNI):**
```cpp
// C++ ile key'i gizle
extern "C"
JNIEXPORT jbyteArray JNICALL
Java_com_myapp_Crypto_getKey(JNIEnv *env, jobject obj) {
    // Key'i runtime'da oluştur veya decrypt et
    unsigned char key[] = {0x8d, 0x12, 0x76, ...};
    
    jbyteArray result = env->NewByteArray(16);
    env->SetByteArrayRegion(result, 0, 16, (jbyte*)key);
    return result;
}
```

---

### **4. Root Detection**

```java
// Root detection
public boolean isDeviceRooted() {
    // Check for su binary
    String[] paths = {
        "/system/app/Superuser.apk",
        "/sbin/su",
        "/system/bin/su",
        "/system/xbin/su"
    };
    
    for (String path : paths) {
        if (new File(path).exists()) return true;
    }
    
    // Check build tags
    String buildTags = android.os.Build.TAGS;
    if (buildTags != null && buildTags.contains("test-keys")) {
        return true;
    }
    
    return false;
}

// Uygulama başlangıcında kontrol et
if (isDeviceRooted()) {
    // Kritik işlemleri engelle veya uyar
    Toast.makeText(this, "Rooted device detected!", Toast.LENGTH_LONG).show();
    finish();
}
```

---

### **5. Certificate Pinning**

```java
// Network güvenliği için certificate pinning
OkHttpClient client = new OkHttpClient.Builder()
    .certificatePinner(new CertificatePinner.Builder()
        .add("api.myapp.com", "sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=")
        .build())
    .build();
```

---

## 🛠️ **Kullanılan Komutlar**

```bash
# Dosya tipi kontrolü
file SecureMyGen.apk

# APK extraction
unzip SecureMyGen.apk -d SecureMyGen
ls -la SecureMyGen/

# APKTool decompilation
rm -rf SecureMyGen
apktool d SecureMyGen.apk

# AES şifreleme araması
find SecureMyGen/smali* -name "*.smali" | xargs grep -l "Cipher\|AES" | head -5

# Aa.smali analizi
cat SecureMyGen/smali_classes3/com/securegenaes/hackmasters_securemygen/Aa.smali

# Const-string araması
grep "const-string" SecureMyGen/smali_classes3/com/securegenaes/hackmasters_securemygen/Aa.smali

# Python decryption script
python3 decrypt.py
```

**decrypt.py:**
```python
import base64
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad

encrypted_b64 = "IRZJFYZmLFl8aSldn0FcSyzLu4Upm+cMJvdz5H3fmik="
encrypted = base64.b64decode(encrypted_b64)

hex_values = ["8d", "12", "76", "84", "cb", "c3", "7c", "17", 
              "61", "6d", "80", "6c", "f5", "04", "73", "cc"]
key = bytes.fromhex(''.join(hex_values))

cipher = AES.new(key, AES.MODE_ECB)
flag = unpad(cipher.decrypt(encrypted), AES.block_size).decode()
print(flag)
```

---

## 💭 **Çözüm Akışı**

```
📱 SecureMyGen Mobile Challenge
            ↓
🔧 File type check
   → file SecureMyGen.apk
   → Android package confirmed
            ↓
📦 APK extraction
   → unzip SecureMyGen.apk
   → classes.dex, classes2.dex, classes3.dex
            ↓
🔨 APKTool decompilation
   → apktool d SecureMyGen.apk
   → Smali kod elde edildi
            ↓
🔍 AES code search
   → find ... | grep "AES"
   → Aa.smali bulundu ✅
            ↓
💎 Aa.smali analizi
   → Encrypted flag: IRZJFYZmLFl8aSldn0FcSyzLu4Upm+cMJvdz5H3fmik=
   → Encryption: AES/ECB/PKCS5Padding
   → Fake key: "whatsthislol"
            ↓
🔎 Real key hunting
   → grep "const-string"
   → 16 hex değer bulundu: 8d, 12, 76, ...
            ↓
🧬 Code analysis
   → b(String) → Hex to byte array
   → bb([B[B) → AES ECB decrypt
            ↓
🐍 Python decryption
   → Key: 8d127684cbc37c17616d806cf50473cc
   → AES ECB decrypt
            ↓
🏁 HM2024{aeS_secureMy4PP_ecB}
```
