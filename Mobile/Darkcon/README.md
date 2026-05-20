# 🎯 Darkcon - Mobile Challenge

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

**Challenge Dosyası:** [📥 GitHub - Darkcon.apk](https://github.com/cihangungor/pitoctf/blob/main/Darkcon.apk)

---

## 🔍 Analiz

**Hedef:** Android APK içindeki native library'yi tersine mühendislik ile analiz ederek flag'i bulmak

**Strateji:** APK unzip → smali analizi → Firebase Firestore sorgusu → Native library disassembly → Catalan XOR decode

**İpuçları:**
- 🎯 APK = Android uygulaması → apktool ile smali koduna bak
- 🔍 Native library (`libnative-lib.so`) var → `magic()` ve `looper()` fonksiyonları
- 💡 Firebase Firestore kullanımı → `encrypted_flag` cloud'da saklanıyor
- 🔑 `looper(n)` = n. Catalan sayısını hesaplıyor → XOR ile decode

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Dosya Tipini Tespit Et**

```bash
file Darkcon.apk
```

**Çıktı:**
```
Darkcon.apk: Android package (APK), with zipflinger virtual entry, with APK Signing Block
```

---

### 📊 **Adım 2: String Analizi**

```bash
strings Darkcon.apk | grep -iE "flag|pito|ctf|key|secret|password|dark"
```

**Kritik Bulgular:**
```
flag_or_something_else
google_api_key
Java_com_application_darkcon_MainActivity_stringFromJNI
Java_com_application_darkcon_MyReceiver_magic
Theme.Darkcon
darkcon
```

**Analiz:**
- ✅ `Java_com_application_darkcon_MyReceiver_magic` → Native library içinde `magic()` fonksiyonu var
- 💡 `flag_or_something_else` → Flag ile ilgili bir alan
- 🎯 Native library analizi gerekiyor

---

### 🔍 **Adım 3: APK'yı Unzip Et**

```bash
unzip -o Darkcon.apk -d Darkcon_extracted
find Darkcon_extracted/lib/ -name "*.so"
```

**Çıktı:**
```
Darkcon_extracted/lib/x86_64/libnative-lib.so
Darkcon_extracted/lib/armeabi-v7a/libnative-lib.so
Darkcon_extracted/lib/x86/libnative-lib.so
Darkcon_extracted/lib/arm64-v8a/libnative-lib.so
```

**Analiz:**
- ✅ 4 farklı mimari için native library mevcut
- 💡 `libnative-lib.so` → `magic()` ve `looper()` fonksiyonları içeriyor
- 🎯 apktool ile smali kodu incelenmeli

---

### 🧠 **Adım 4: apktool ile Smali Kodunu İncele**

```bash
apktool d Darkcon.apk -o Darkcon_apktool -f
cat Darkcon_apktool/AndroidManifest.xml
find Darkcon_apktool/smali* -name "*.smali" | grep -iE "main|receiver|magic"
```

**AndroidManifest Kritik Bulgular:**
```xml
<receiver android:enabled="true" android:exported="true" 
          android:name="com.application.darkcon.MyReceiver">
    <intent-filter>
        <action android:name="flag_checker"/>
    </intent-filter>
</receiver>
```

**Analiz:**
- ✅ `MyReceiver` → `flag_checker` action'ını dinliyor
- 💡 APK `debuggable="true"` → Kolayca analiz edilebilir
- 🎯 `MyReceiver.smali` ve `data_receiver.smali` incelenmeli

---

### 💻 **Adım 5: Smali Kod Analizi**

```bash
cat Darkcon_apktool/smali_classes2/com/application/darkcon/MyReceiver.smali
cat Darkcon_apktool/smali_classes2/com/application/darkcon/data_receiver.smali
cat "Darkcon_apktool/smali_classes2/com/application/darkcon/MyReceiver\$loader.smali"
cat "Darkcon_apktool/smali_classes2/com/application/darkcon/data_receiver\$1.smali"
```

**MyReceiver.smali Kritik Bulgular:**
```smali
.method public native magic([B[J)Z    # flag bytes + long array → boolean
.end method

# onReceive: "flag" extra'sını alıyor
const-string v0, "flag"
invoke-virtual {p2, v0}, Landroid/content/Intent;->getStringExtra(...)
```

**data_receiver$1.smali Kritik Bulgular:**
```smali
# Firebase'den "encrypted_flag" field'ını çekiyor
const-string v3, "encrypted_flag"
invoke-virtual {v2, v3}, Lcom/google/firebase/firestore/DocumentSnapshot;->getString(...)
sput-object v3, Lcom/application/darkcon/data_receiver;->data:Ljava/lang/String;
```

**MyReceiver$loader.smali Kritik Bulgular:**
```smali
# encrypted_flag'i "," ile split edip long array'e çeviriyor
const-string v1, ","
invoke-virtual {v0, v1}, Ljava/lang/String;->split(...)[Ljava/lang/String;

# magic(flag.getBytes(), longArray) çağırıyor
invoke-virtual {v2, v3, v1}, Lcom/application/darkcon/MyReceiver;->magic([B[J)Z
```

**Analiz:**
- ✅ Firebase `encryption` collection → `encrypted_flag` field → virgülle ayrılmış long array
- 💡 `magic(flag.getBytes(), longArray)` → `true` dönerse "Well thats the correct flag"
- 🎯 Firebase Project ID bulunmalı

---

### 🔑 **Adım 6: Firebase Project ID Bul**

```bash
cat Darkcon_apktool/res/values/strings.xml | grep -iE "firebase|project|api|key|app_id|google"
```

**Çıktı:**
```xml
<string name="google_api_key">AIzaSyALOnUoUXigIDD1VJkzt2lUyiMG41u2FiE</string>
<string name="google_app_id">1:309642636102:android:9f3cf0c3f3aeada9491347</string>
<string name="google_storage_bucket">ezpz-91e11.appspot.com</string>
<string name="project_id">ezpz-91e11</string>
```

**Analiz:**
- ✅ **Project ID:** `ezpz-91e11`
- ✅ **API Key:** `AIzaSyALOnUoUXigIDD1VJkzt2lUyiMG41u2FiE`
- 🎯 Firestore REST API ile `encrypted_flag` çekilebilir

---

### 🚀 **Adım 7: Firestore REST API ile Encrypted Flag Al**

```bash
curl -s "https://firestore.googleapis.com/v1/projects/ezpz-91e11/databases/(default)/documents/encryption?key=AIzaSyALOnUoUXigIDD1VJkzt2lUyiMG41u2FiE"
```

**Çıktı:**
```json
{
  "documents": [
    {
      "fields": {
        "encrypted_flag": {
          "stringValue": "101,96,112,110,77,101,202,470,1506,4758,16815,58877,..."
        }
      }
    }
  ]
}
```

**Analiz:**
- ✅ `encrypted_flag` = virgülle ayrılmış long array (59 eleman)
- 💡 Bu değerler `magic()` fonksiyonunda flag byte'ları ile karşılaştırılıyor
- 🎯 Native library disassembly ile algoritma çözülmeli

---

### 🔬 **Adım 8: Native Library Disassembly**

```bash
sudo apt install -y binutils-aarch64-linux-gnu
aarch64-linux-gnu-objdump -d Darkcon_extracted/lib/arm64-v8a/libnative-lib.so > /tmp/disasm.txt
grep -A 200 "e8d8" /tmp/disasm.txt | head -200
```

**magic() Kritik Assembly:**
```asm
# Her i için:
e9b0: bl  e4b0 <_Z6looperj@plt>    # looper(i) çağır
e9b8: eor x9, x8, x0               # input[i] XOR looper(i)
e9c0: cmp x10, x9                   # encrypted[i] ile karşılaştır
```

**looper() Kritik Assembly:**
```asm
# fib[0] = 1, fib[1] = 1
eb10: str x8, [x10, #8]   # fib[1] = 1
eb14: str x8, [x10]       # fib[0] = 1

# İç döngü: fib[i] += fib[j] * fib[i-1-j]
eb90: mul x8, x8, x13     # fib[j] * fib[i-1-j]
eb9c: add x8, x13, x8     # fib[i] += ...
```

**Analiz:**
- ✅ `magic()` algoritması: `encrypted[i] == flag[i] XOR looper(i)`
- ✅ `looper(n)` = n. **Catalan sayısını** hesaplıyor
- 💡 Flag'in içinde de yazıyor: `c4t4l4n_ov3rf10w`
- 🎯 Python ile tersine çevir

---

### 🚩 **Adım 9: Flag'i Decode Et**

```python
data = [101,96,112,110,77,101,202,470,1506,4758,16815,58877,208123,742855,
        2674489,9694735,35357570,129644713,477638735,1767263206,2269153033,
        2991430638,1288250377,3757197244,1413958429,43422424,2072914473,
        2325361044,2600037558,3008195127,3276256895,4169229947,300814809,
        3929270464,2526730686,2527522239,645964816,1351610749,573153031,
        1347646066,1945953402,3824419424,480774039,2833665279,2366904092,
        2809807660,3295802436,3644429150,720643560,906311378,992169127,
        1211139059,1465960990,4269303883,3179939394,4095898594,580984841,
        3596758568,1063564231,3288906933]

def looper(n):
    size = n + 1
    fib = [0] * size
    fib[0] = 1
    if size > 1:
        fib[1] = 1
    for i in range(2, size):
        for j in range(i):
            fib[i] += fib[j] * fib[i-1-j]
    return fib[n]

flag = ""
for i in range(len(data)):
    c = data[i] ^ looper(i)
    flag += chr(c & 0xFF)

print("Flag:", flag)
```

**Çıktı:**
```
Flag: darkCON{th3_w31rd_c0mb1nat10n_of_fr1da_4nd_c4t4l4n_ov3rf10w}
```

**Analiz:**
- ✅ **FLAG BULUNDU!** 🎉
- 💡 `looper()` Catalan sayı dizisini hesaplıyor
- 🎯 Her flag byte'ı = `encrypted[i] XOR catalan(i)`

---

## 🚩 **FLAG**
```
darkCON{th3_w31rd_c0mb1nat10n_of_fr1da_4nd_c4t4l4n_ov3rf10w}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">📦<br><b>apktool</b><br><sub>APK decompile</sub></td>
<td align="center">🔧<br><b>objdump</b><br><sub>Native lib disassembly</sub></td>
<td align="center">🌐<br><b>curl</b><br><sub>Firestore REST API</sub></td>
<td align="center">🐍<br><b>Python</b><br><sub>Catalan XOR decode</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 Darkcon Mobile Challenge
       ↓
📂 APK analizi
   → Native library: libnative-lib.so
   → magic() ve looper() fonksiyonları
       ↓
🧠 apktool smali analizi
   → MyReceiver → "flag_checker" broadcast
   → data_receiver → Firebase "encryption" collection
   → MyReceiver$loader → magic(flag.getBytes(), longArray)
   → encrypted_flag field → virgülle ayrılmış long array
       ↓
🔑 Firebase credentials bul
   → strings.xml → project_id: ezpz-91e11
   → API key: AIzaSyALOnUoUXigIDD1VJkzt2lUyiMG41u2FiE
       ↓
🌐 Firestore REST API
   → curl ile encrypted_flag çekildi
   → 59 adet long değer
       ↓
🔬 Native library disassembly
   → magic(): encrypted[i] == flag[i] XOR looper(i)
   → looper(n): n. Catalan sayısını hesaplıyor
       ↓
🐍 Python decode
   → flag[i] = encrypted[i] XOR catalan(i)
       ↓
🚩 FLAG: darkCON{th3_w31rd_c0mb1nat10n_of_fr1da_4nd_c4t4l4n_ov3rf10w}
```
