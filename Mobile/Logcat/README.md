# 📱 Logcat - Mobil Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Mobile-Challenge-darkblue?style=for-the-badge&logo=android" alt="Mobile">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-APK Analysis-purple?style=for-the-badge&logo=apk" alt="APK">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Mobil  
**Seviye:** Kolay  
**Puan:** 200

**Challenge Dosyası:** [📥 Google Drive - Logcat.apk](https://drive.google.com/file/d/1aQl_pj7dnhFn-dg0ZqbpFyEt-ZfuSR7P/view?usp=drivesdk)


---

## 🔍 Analiz

**Hedef:** Android APK dosyasını decompile edip flag'i bulmak

**Strateji:** APK unpack → Apktool decompile → Smali analizi → Flag extraction

**İpuçları:**
- 📱 **Android APK:** Java/Kotlin uygulaması
- 🔍 **Apktool:** APK decompile aracı
- 📝 **Smali:** Android bytecode formatı
- 🔎 **MainActivity:** Ana uygulama sınıfı

---

## ✅ Çözüm Adımları

### 📥 **1. Dosya Analizi**

**APK dosyasını indirip kontrol ediyoruz:**

```bash
file Logcat.apk
```

**Çıktı:**
```
Logcat.apk: Android package (APK), with APK Signing Block
```

> ✅ **Android APK** dosyası doğrulandı!

---

### 📦 **2. APK İçeriğini İnceleme**

**APK dosyası aslında bir ZIP arşivi:**

```bash
unzip Logcat.apk -d Logcat
cd Logcat
ls -la
```

**Çıktı:**
```
AndroidManifest.xml
classes.dex
classes2.dex
classes3.dex
resources.arsc
res/
META-INF/
```

**Analiz:**
- 📝 **AndroidManifest.xml:** Uygulama yapılandırması
- 🔢 **classes.dex:** Derlenmiş Java/Kotlin kodu
- 📊 **resources.arsc:** Uygulama kaynakları

---

### 🔤 **3. String Analizi**

**Önce strings ile hızlı bakış:**

```bash
strings Logcat.apk | grep -i skydays
```

**Çıktı:**
```
skydays2025
Base.Theme.Skydays2025
Theme.Skydays2025
```

**Flag aramayı deniyoruz:**

```bash
strings Logcat.apk | grep -i "skydays{"
```

**Sonuç:**
```
(Boş - Flag görünmüyor)
```

> 💡 Flag string'lerde değil, kod içinde gizli olabilir!

---

### 🔧 **4. Apktool ile Decompile**

**Apktool kullanarak APK'yı decompile ediyoruz:**

```bash
cd ~/Desktop
apktool d Logcat.apk -o Logcat_decoded
```

**Çıktı:**
```
I: Using Apktool 2.7.0-dirty on Logcat.apk
I: Loading resource table...
I: Decoding AndroidManifest.xml with resources...
I: Decoding file-resources...
I: Decoding values */* XMLs...
I: Baksmaling classes.dex...
I: Baksmaling classes3.dex...
I: Baksmaling classes2.dex...
I: Copying assets and libs...
I: Copying unknown files...
I: Copying original files...
```

> ✅ **Decompile başarılı!**

---

### 📂 **5. Smali Dosyalarını Bulma**

**Dizin yapısını inceliyoruz:**

```bash
cd Logcat_decoded
find . -name "*.smali" | head -20
```

**Önemli dosyalar:**
```
./smali_classes3/ytu/skydays2025/mobile2/MainActivity$1.smali
./smali_classes3/ytu/skydays2025/mobile2/MainActivity.smali
```

> 🎯 **MainActivity** - Ana uygulama sınıfı!

---

### 🔍 **6. Flag Arama**

**Skydays ile ilgili tüm dosyalarda arama:**

```bash
grep -r "skydays" .
```

**Flag için özel arama:**

```bash
grep -r "flag" . | grep -i sky
```

**En önemli sonuç:**
```
./smali_classes3/ytu/skydays2025/mobile2/MainActivity$1.smali:
    const-string v4, "Do\u011fru pin, flag:SKYDAYS{logcatkullanin}"
```

> 🚩 **FLAG BULUNDU!**

---

### 📝 **7. MainActivity$1.smali İnceleme**

**Dosyayı açıyoruz:**

```bash
cat ./smali_classes3/ytu/skydays2025/mobile2/MainActivity\$1.smali | grep -A5 -B5 "SKYDAYS"
```

**İlgili kod bölümü:**

```smali
.method public onClick(Landroid/view/View;)V
    .locals 8
    
    # PIN kontrolü
    iget-object v0, p0, Lytu/skydays2025/mobile2/MainActivity$1;->val$pinInput:Landroid/widget/EditText;
    invoke-virtual {v0}, Landroid/widget/EditText;->getText()Landroid/text/Editable;
    
    # Database sorgusu
    iget-object v2, p0, Lytu/skydays2025/mobile2/MainActivity$1;->val$db:Landroid/database/sqlite/SQLiteDatabase;
    
    # Doğru PIN ise
    const-string v4, "Doğru pin, flag:SKYDAYS{logcatkullanin}"
    
    # TextView'e yazdır
    iget-object v3, p0, Lytu/skydays2025/mobile2/MainActivity$1;->val$resultText:Landroid/widget/TextView;
    invoke-virtual {v3, v4}, Landroid/widget/TextView;->setText(Ljava/lang/CharSequence;)V
.end method
```

**Algoritma:**
1. Kullanıcıdan PIN input alınıyor
2. Database'de kontrol ediliyor
3. Doğruysa flag gösteriliyor: **"Doğru pin, flag:SKYDAYS{logcatkullanin}"**

---

## 🚩 **FLAG**

```
SKYDAYS{logcatkullanin}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>file</b><br><sub>Dosya tipi analizi</sub></td>
<td align="center">📦<br><b>unzip</b><br><sub>APK extraction</sub></td>
<td align="center">🔤<br><b>strings</b><br><sub>String analizi</sub></td>
<td align="center">🔧<br><b>apktool</b><br><sub>APK decompile</sub></td>
<td align="center">🔎<br><b>grep</b><br><sub>Text arama</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**
```bash
# Dosya analizi
file Logcat.apk

# APK extraction
unzip Logcat.apk -d Logcat

# String analizi
strings Logcat.apk | grep -i skydays

# Apktool decompile
apktool d Logcat.apk -o Logcat_decoded

# Smali dosyalarını bulma
find . -name "*.smali"

# Flag arama
grep -r "flag" . | grep -i sky
```

---

## 💭 **Çözüm Akışı**

```
📱 "Logcat" Challenge
            ↓
📥 Logcat.apk indirildi
            ↓
🔍 file Logcat.apk → Android APK
            ↓
📦 unzip → classes.dex bulundu
            ↓
🔤 strings → "skydays2025" görüldü
            ↓
🔧 apktool d Logcat.apk
            ↓
📂 Smali dosyaları decode edildi
            ↓
🔍 MainActivity$1.smali bulundu
            ↓
📝 grep -r "flag" . | grep -i sky
            ↓
🎯 const-string "Doğru pin, flag:SKYDAYS{logcatkullanin}"
            ↓
🚩 SKYDAYS{logcatkullanin}
```
