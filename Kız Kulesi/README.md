# 🗼 Kız Kulesi - Mobil Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Mobile-Challange-darkblue?style=for-the-badge&logo=android" alt="Mobile">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Reverse-purple?style=for-the-badge&logo=lock" alt="Reverse">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Mobil  
**Seviye:** Orta  
**Puan:** 500  

**Açıklama:** Flagi emülatörde bulduğunuz şekilde aynen girmeniz gerekiyor. 
Özel bir format eklemedim.

**Challenge Dosyası:** [Google Drive - kizkulesi.apk](https://drive.google.com/file/d/15XLNWR0yjFqcqeLWFzORll36NmVCANtr/view?usp=sharing)

**Firstblood:** +50 puan

---

## 🔍 İlk Bakış ve Analiz

Hmm, bir Android APK dosyası var. "Emülatörde bulduğunuz şekilde" diyor ama ben emülatör kurmak yerine önce statik analiz yapmayı tercih ediyorum. Bakalım içinde neler varmış! 🕵️

---

## ✅ Çözüm Adımları

### 📱 **Adım 1: APK'yı İndirip İnceliyoruz**

Önce dosyayı indirelim ve ne tür bir dosya olduğuna bakalım:

```bash
wget "https://drive.google.com/uc?export=download&id=15XLNWR0yjFqcqeLWFzORll36NmVCANtr" -O kizkulesi.apk

file kizkulesi.apk
```

```
kizkulesi.apk: Zip archive data, at least v2.0 to extract
```

Tamam, bir APK dosyası. Şimdi bunu decompile edip içine bakmamız lazım!

---

### 🔓 **Adım 2: APK'yı Decompile Ediyoruz**

Apktool ile APK'yı açalım:

```bash
# Eğer apktool kurulu değilse
sudo apt install apktool -y

# APK'yı decompile et
apktool d kizkulesi.apk -o kizkulesi_decompiled

# Dizine gir
cd kizkulesi_decompiled

# İçeriğe bakalım
ls -la
```

```
AndroidManifest.xml
smali/
smali_classes2/
smali_classes3/
res/
```

Güzel! APK başarıyla açıldı. Şimdi nerede flag olabilir diye düşünelim... 🤔

---

### 🔍 **Adım 3: strings.xml Dosyasını Keşfediyoruz**

Android uygulamalarında string değerleri genellikle `res/values/strings.xml` dosyasında saklanır. Oraya bakalım:

```bash
cat res/values/strings.xml | grep -i "encrypted\|flag"
```

```xml
<string name="encrypted_1">VG1rNWRVOUhSa3hpUm1oT1VrWm5ORnBHY0ZwaU1uaERVMFZ6ZUZWUlBUMD0=</string>
<string name="encrypted_1_1">WVRKR2RWcEhiSGxhUjJ4MFltMUdObUpIYkRWWldFcHdZbEU5UFE9PQ==</string>
<string name="encrypted_2">Vm14a2QxVXhWblJVYmtwVFlXeGFiMVZyWkc5amJGcFhWbXR3YkZacmJEVmFWVlpYVlcxRmVGZHVXbGRXZWtaMVZGUkdkMDB4UWxWTlJEQTk=</string>
<string name="encrypted_3">VmtSQk5XRldiRFZrU0VKcllsWktlVll3WkZaa01WSnhVV3BhYTFaVmIzaFdWM1JxVUZFOVBRPT0=</string>
```

Ooo! 4 tane encrypted string var! Bunlar Base64'e benziyor. Ama flag'i bulmak için bu değerlerin nasıl işlendiğini anlamamız lazım. MainActivity'ye bakalım! 👀

---

### 📂 **Adım 4: MainActivity Smali Kodunu İnceliyoruz**

```bash
cd smali_classes3/com/example/kizkulesi/
ls -la
```

```
MainActivity.smali
```

MainActivity'de ne var bakalım. Önce "areYouAtKizKulesi" metodunu bulalım çünkü bu ilginç bir isim:

```bash
grep -A5 "areYouAtKizKulesi" MainActivity.smali
```

```smali
.method private final areYouAtKizKulesi()Z
    .locals 1
    
    .line 56
    const/4 v0, 0x0
    
    return v0
.end method
```

**İlginç!** Bu metod her zaman `false` (0) döndürüyor. Yani "Kız Kulesi'nde değilsin" diyor! 

Peki bu metod nerede kullanılıyor?

```bash
grep -B5 -A10 "invoke.*areYouAtKizKulesi" MainActivity.smali | head -20
```

```smali
invoke-direct {p0}, Lcom/example/kizkulesi/MainActivity;->isRootedOrEmulator()Z
move-result v6
if-nez v6, :cond_4

invoke-direct {p0}, Lcom/example/kizkulesi/MainActivity;->areYouAtKizKulesi()Z
move-result v6
if-nez v6, :cond_0
goto/16 :goto_3
```

Hmm... `if-nez v6, :cond_0` ne demek? 🤔

"if-nez" = "if v6 == 0" demek. Yani eğer `areYouAtKizKulesi()` false döndürürse (ki her zaman false döndürüyor), `:cond_0` bloğuna git!

Yani aslında uygulama emülatörde çalışmasa bile `:cond_0` bloğundaki kodu çalıştırıyor! Akıllıca! 😎

---

### 🔢 **Adım 5: Flag İşleme Mantığını Anlıyoruz**

`:cond_0` bloğunda neler oluyor bakalım:

```bash
grep -A100 ":cond_0" MainActivity.smali | head -50
```

Kodu incelediğimizde şu döngüleri görüyoruz:

1. **goto_0 bloğu:** `i=1`'den `4`'e kadar (3 kere)
   - `decode(v1)` → flag11'i 3 kere decode et
   - `decode(v0)` → flag1'i 3 kere decode et  
   - `decode(v2)` → flag2'yi 3 kere decode et

2. **goto_1 bloğu:** `i=1`'den `6`'ya kadar (5 kere)
   - `decode(v2)` → flag2'yi 5 kere daha decode et (toplam 8)

3. **goto_2 bloğu:** `i=1`'den `5`'e kadar (4 kere)
   - `encode(v1)` → flag11'i 4 kere encode et
   - `decode(v3)` → flag3'ü 4 kere decode et

4. **Flag birleştirme:**
   ```smali
   new-instance v6, Ljava/lang/StringBuilder;
   invoke-virtual {v6, v0}, append  # v0 ekle
   invoke-virtual {v6, v2}, append  # v2 ekle
   invoke-virtual {v6, v3}, append  # v3 ekle
   ```

5. **Final flag:**
   ```smali
   const-string v10, "RkxBRz17SE0yMDIzLQ=="
   invoke-direct {p0, v10}, decode  # "FLAG={HM2023-"
   
   invoke-direct {p0, v6}, decrypt  # v6'yı AES decrypt et
   
   const-string v10, "fQ=="
   invoke-direct {p0, v10}, decode  # "}"
   ```

Vay be! Şimdi anladık:

```
Final Flag = "FLAG={HM2023-" + AES_decrypt(v0 + v2 + v3) + "}"
```

Şimdi kodu yazmaya başlayabiliriz! 💻

---

### 🔐 **Adım 6: Decrypt ve Encode Metodlarını İnceliyoruz**

AES decrypt metodunu bulalım:

```bash
grep -A30 "method.*decrypt" MainActivity.smali
```

```smali
.method private final decrypt(Ljava/lang/String;)Ljava/lang/String;
    const-string v1, "1234567890123456"
    const-string v2, "AES"
    new-instance v1, Ljavax/crypto/spec/IvParameterSpec;
    const/16 v2, 0x10
    new-array v2, v2, [B
    invoke-direct {v1, v2}, <init>
    const-string v2, "AES/CBC/PKCS5Padding"
```

Harika! AES şifreleme detaylarını bulduk:
- **Key:** `1234567890123456`
- **IV:** 16 byte sıfır
- **Mode:** AES/CBC/PKCS5Padding

---

### 🐍 **Adım 7: Python Scripti Yazıyoruz**

Şimdi tüm bilgileri birleştirelim ve flag'i çözelim:

```python
cat > solve_flag.py << 'EOF'
import base64
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad

# Encrypted değerler (strings.xml'den)
enc1 = "VG1rNWRVOUhSa3hpUm1oT1VrWm5ORnBHY0ZwaU1uaERVMFZ6ZUZWUlBUMD0="
enc2 = "Vm14a2QxVXhWblJVYmtwVFlXeGFiMVZyWkc5amJGcFhWbXR3YkZacmJEVmFWVlpYVlcxRmVGZHVXbGRXZWtaMVZGUkdkMDB4UWxWTlJEQTk="
enc3 = "VmtSQk5XRldiRFZrU0VKcllsWktlVll3WkZaa01WSnhVV3BhYTFaVmIzaFdWM1JxVUZFOVBRPT0="

print("="*70)
print("KIZ KULESI FLAG SOLVER")
print("="*70)

def multi_decode(data, times):
    """Base64'ü birden fazla kere decode et"""
    result = data
    for i in range(times):
        try:
            result = base64.b64decode(result).decode('utf-8')
            print(f"  Decode {i+1}: {result[:50]}...")
        except:
            print(f"  HATA - artık base64 değil")
            break
    return result

def aes_decrypt(ciphertext):
    """AES CBC ile decrypt et"""
    key = b"1234567890123456"
    iv = bytes(16)  # 16 byte sıfır
    
    # Base64 decode
    cipher_bytes = base64.b64decode(ciphertext)
    
    # AES decrypt
    cipher = AES.new(key, AES.MODE_CBC, iv)
    plaintext = unpad(cipher.decrypt(cipher_bytes), 16)
    return plaintext.decode('utf-8')

# 1. v0 = 3x decode(enc1)
print("\n1. v0 = 3x decode(enc1)")
v0 = multi_decode(enc1, 3)
print(f"   v0 = {v0}")

# 2. v2 = 8x decode(enc2) ama 5. decodeden sonra düz metin
print("\n2. v2 = 5x decode(enc2)")
v2 = enc2
for i in range(5):
    v2 = base64.b64decode(v2).decode('utf-8')
    print(f"  Decode {i+1}: {v2[:50]}...")
print(f"   v2 = {v2}")

# 3. v3 = 4x decode(enc3)
print("\n3. v3 = 4x decode(enc3)")
v3 = multi_decode(enc3, 4)
print(f"   v3 = {v3}")

# 4. v6 = v0 + v2 + v3 (string concatenation)
print("\n4. v6 = v0 + v2 + v3")
v6 = v0 + v2 + v3
print(f"   v6 = {v6}")

# 5. v8 = AES_decrypt(v6)
print("\n5. v8 = AES_decrypt(v6)")
v8 = aes_decrypt(v6)
print(f"   v8 = {v8}")

# 6. Final Flag
print("\n6. Final Flag")
beginning = base64.b64decode("RkxBRz17SE0yMDIzLQ==").decode()
ending = base64.b64decode("fQ==").decode()
flag = beginning + v8 + ending

print("\n" + "="*70)
print(f"FINAL FLAG: {flag}")
print("="*70)
EOF

python3 solve_flag.py
```

---

### 🎯 **Adım 8: Scripti Çalıştırıp Flag'i Buluyoruz**

```bash
python3 solve_flag.py
```

```
======================================================================
KIZ KULESI FLAG SOLVER
======================================================================

1. v0 = 3x decode(enc1)
  Decode 1: Tmk5dU9HRkxiRmhOUkZnNFpGcFpiMnhDU0VzeFVRPT0=...
  Decode 2: Ni9uOGFLbFhNRFg4ZFpZb2xCSEsxUQ==...
  Decode 3: 6/n8aKlXMDX8dZYolBHK1Q...
   v0 = 6/n8aKlXMDX8dZYolBHK1Q

2. v2 = 5x decode(enc2)
  Decode 1: Vmxkd1UxVnRUbkpTYWxab1VrZG9jbFpXVmtwbFZrbDVaVVZXVW...
  Decode 2: VldwU1VtTnJSalZoUkdoclZWVkplVkl5ZUVWUmExWnZWVzFuTT...
  Decode 3: VWpSUmNrRjVhRGhrVVVJeVIyeEVRa1ZvVW1nM1p3PT0=...
  Decode 4: UjRRckF5aDhkUUIyR2xEQkVoUmg3Zw==...
  Decode 5: R4QrAyh8dQB2GlDBEhRh7g...
   v2 = R4QrAyh8dQB2GlDBEhRh7g

3. v3 = 4x decode(enc3)
  Decode 1: VkRBNWFWbDVkSEJrYlZKeVYwZFZkMVJxUWpaa1ZVb3hWV3RqUF...
  Decode 2: VDA5aVl5dHBkbVJyV0dVd1RqQjZkVUoxVWtjPQ==...
  Decode 3: T09iYytpdmRrWGUwTjB6dUJ1Ukc=...
  Decode 4: OObc+ivdkXe0N0zuBuRG...
   v3 = OObc+ivdkXe0N0zuBuRG

4. v6 = v0 + v2 + v3
   v6 = 6/n8aKlXMDX8dZYolBHK1QR4QrAyh8dQB2GlDBEhRh7gOObc+ivdkXe0N0zuBuRG

5. v8 = AES_decrypt(v6)
   v8 = 561feccec11015b8be7ff470e15d5c1e

6. Final Flag

======================================================================
FINAL FLAG: FLAG={HM2023-561feccec11015b8be7ff470e15d5c1e}
======================================================================
```

**BAŞARDIK!** 🎉🎉🎉

---

## 🚩 **FLAG**

```
FLAG={HM2023-561feccec11015b8be7ff470e15d5c1e}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🤖<br><b>Apktool</b><br><sub>APK decompile</sub></td>
<td align="center">🔍<br><b>grep</b><br><sub>Pattern search</sub></td>
<td align="center">🐍<br><b>Python</b><br><sub>Scripting</sub></td>
<td align="center">🔐<br><b>PyCryptodome</b><br><sub>AES decrypt</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akışı**

```
🗼 Kız Kulesi Challenge
            ↓
📱 APK İndir → kizkulesi.apk
            ↓
🔓 Decompile → apktool d kizkulesi.apk
            ↓
🔍 strings.xml → 4 encrypted string bul
            ↓
📂 MainActivity.smali → İşleme mantığını anla
            ↓
🤔 areYouAtKizKulesi() → Her zaman false!
   → Ama yine de :cond_0 bloğu çalışıyor
            ↓
🔢 İşlem Sırası:
   v0 = 3x base64_decode(enc1)
   v2 = 5x base64_decode(enc2)
   v3 = 4x base64_decode(enc3)
   v6 = v0 + v2 + v3
            ↓
🔐 AES Decrypt → decrypt(v6)
   Key: "1234567890123456"
   IV: 16 byte zero
   Mode: AES/CBC/PKCS5Padding
            ↓
🎯 Final Flag → "FLAG={HM2023-" + decrypt(v6) + "}"
            ↓
🚩 FLAG={HM2023-561feccec11015b8be7ff470e15d5c1e}
```
