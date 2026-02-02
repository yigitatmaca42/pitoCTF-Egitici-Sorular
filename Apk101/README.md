# 📱 Apk101 - Mobil Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Mobile-Challenge-darkblue?style=for-the-badge&logo=android" alt="Mobile">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-APK_Analysis-purple?style=for-the-badge&logo=apk" alt="APK">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Mobil  
**Seviye:** Kolay  
**Puan:** 200  

**Challenge Dosyası:** [📥 Google Drive - Mobil101.apk](https://drive.google.com/file/d/1wxckVZ8sOETML2NAKG4Yv-4ei7lmMbZm/view?usp=drivesdk)

**Firstblood:** +50 puan

---

## 🔍 Analiz

**Hedef:** Android APK dosyasını analiz edip flag'i bulmak

**Strateji:** APK unpack → String analysis → Flag extraction

**İpuçları:**
- 📱 **Android APK:** Java/Kotlin uygulaması
- 🔤 **Strings:** APK içindeki text veriler
- 🔍 **Regex:** Pattern matching ile flag bulma
- 🎯 **Format:** SKYDAYS{...} formatında flag

---

## ✅ Çözüm Adımları

### 📥 **1. Dosya Analizi**

**APK dosyasını indirip kontrol ediyoruz:**

```bash
file Mobil101.apk
```

**Çıktı:**
```
Mobil101.apk: Android package (APK), with APK Signing Block
```

> ✅ **Android APK** dosyası doğrulandı!

---

### 📦 **2. APK İçeriğini İnceleme**

**APK dosyası aslında bir ZIP arşivi:**

```bash
unzip Mobil101.apk -d Mobil101_extracted
cd Mobil101_extracted
ls -la
```

**Çıktı:**
```
AndroidManifest.xml
classes.dex
classes2.dex
resources.arsc
res/
META-INF/
```

**Analiz:**
- 📄 **AndroidManifest.xml:** Uygulama yapılandırması
- 🔢 **classes.dex:** Derlenmiş Java/Kotlin kodu
- 📊 **resources.arsc:** Uygulama kaynakları

---

### 🔤 **3. İlk String Analizi**

**Önce grep ile binary dosyalarda flag arıyoruz:**

```bash
grep -r "flag" .
grep -r "CTF" .
```

**Çıktı:**
```
grep: ./classes.dex: binary file matches
grep: ./resources.arsc: binary file matches
grep: ./classes2.dex: binary file matches
```

> 💡 Binary dosyalarda "flag" ve "CTF" kelimelerini buldu ama içeriği göremiyoruz!

---

### 🔍 **4. Detaylı String Extraction**

**strings komutu ile APK'dan tüm text verilerini çıkarıyoruz:**

```bash
strings ../Mobil101.apk | grep -i "flag\|ctf"
```

**İlk çıktı:**
```
behavior_saveFlags
defaultScrollFlagsEnabled
layout_scrollFlags
nestedScrollFlags
transitionFlags
flag
&CtFGrQR
CtFz
XI*cTf
```

**Analiz:**
- İlk satırlar Android framework string'leri (işimize yaramıyor)
- `&CtFGrQR`, `CtFz`, `XI*cTf` → Şüpheli parçalar!

> 🤔 Bunlar flag'in parçaları olabilir mi?

---

### 🎯 **5. Flag Format Arama**

**CTF flag'leri genelde `flag{...}` veya `Flag{...}` formatındadır:**

```bash
strings ../Mobil101.apk | grep -E "(CTF|ctf|flag|Flag|FLAG)\{"
```

**Sonuç:**
```
(Boş - Direkt flag formatı yok)
```

**Daha geniş regex pattern deniyoruz:**

```bash
strings ../Mobil101.apk | grep -E "^[A-Za-z0-9_]*\{.*\}$"
```

**Çıktı:**
```
P85{}
{:OJJg}
SKYDAYS{buyukhocaya}
{jP3}
3G{}
Lz{/p}
w{4}
{<C}
g{5m}
e{(}
{6t\}
bk{}
{JO}
l{yC&}
{Srq~}
Yf{}
```

> 🚩 **FLAG BULUNDU!**

---

### 🎉 **6. Flag Doğrulama**

**Bulduk:**
```
SKYDAYS{buyukhocaya}
```

**Format analizi:**
- ✅ **SKYDAYS** - CTF organizasyonu/platform adı
- ✅ **{buyukhocaya}** - Flag içeriği (Türkçe: "büyük hocaya")
- ✅ Süslü parantez formatı doğru

**Diğer satırlar:**
- `P85{}`, `3G{}`, `bk{}` → Boş süslü parantezler (framework kodu)
- `{:OJJg}`, `{jP3}` → Random stringler (flag değil)
- Sadece `SKYDAYS{buyukhocaya}` anlamlı ve flag formatına uyuyor!

---

## 🚩 **FLAG**

```
SKYDAYS{buyukhocaya}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">📄<br><b>file</b><br><sub>Dosya tipi analizi</sub></td>
<td align="center">📦<br><b>unzip</b><br><sub>APK extraction</sub></td>
<td align="center">🔤<br><b>strings</b><br><sub>String extraction</sub></td>
<td align="center">🔍<br><b>grep</b><br><sub>Text arama & regex</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**
```bash
# Dosya analizi
file Mobil101.apk

# APK extraction
unzip Mobil101.apk -d Mobil101_extracted
cd Mobil101_extracted

# Binary dosyalarda flag arama
grep -r "flag" .
grep -r "CTF" .

# String extraction ve flag bulma
strings ../Mobil101.apk | grep -i "flag\|ctf"

# Flag format regex ile arama
strings ../Mobil101.apk | grep -E "^[A-Za-z0-9_]*\{.*\}$"
```

---

## 💭 **Çözüm Akışı**

```
📱 "Mobil101" Challenge
            ↓
📥 Mobil101.apk indirildi
            ↓
📄 file Mobil101.apk → Android APK
            ↓
📦 unzip → classes.dex, resources.arsc bulundu
            ↓
🔍 grep -r "flag" → Binary dosyalarda match
            ↓
🔤 strings Mobil101.apk → Text extraction
            ↓
🔎 grep -i "flag\|ctf" → Şüpheli parçalar görüldü
            ↓
🎯 grep -E "^[A-Za-z0-9_]*\{.*\}$" → Regex pattern
            ↓
📝 SKYDAYS{buyukhocaya} bulundu!
            ↓
🚩 FLAG: SKYDAYS{buyukhocaya}
```
