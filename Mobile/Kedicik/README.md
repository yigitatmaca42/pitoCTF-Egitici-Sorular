# 🎯 Kedicik - Mobile Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Mobile-Challenge-darkblue?style=for-the-badge" alt="Mobile">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Android APK-purple?style=for-the-badge" alt="Android APK">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Mobile  
**Seviye:** Orta  
**Puan:** 300

**Challenge Dosyası** [📥 Kaynak - kedicik.apk](https://github.com/cihangungor/pitoctf/blob/main/kedicik.apk)

---

## 🔍 Analiz

**Hedef:** APK içindeki flag üretim mekanizmasını analiz edip gerçek flag'i elde etmek.

**Strateji:** APK'yı aç → Java/Kotlin tarafını incele → native library çağrısını tespit et → runtime davranışını/log akışını değerlendir → Flag

**İpuçları:**
- 🎯 APK içinde `libkittykittybangbang.so` isimli native kütüphane bulunuyor.
- 🔍 Native tarafta `stringFromJNI` adlı JNI fonksiyonu dikkat çekiyor.
- 💡 Uygulama ekrana dokunma olayına tepki veriyor.
- 🔑 Flag doğrudan ekranda değil, uygulamanın log akışında ortaya çıkıyor.

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: APK içeriğini incele**

İlk olarak APK'nın içinde neler bulunduğuna baktım. Özellikle native `.so` dosyaları Mobile sorularda kritik olur.

```bash
file kedicik.apk
unzip -l kedicik.apk | head -40
```

**Çıktı:**
```text
kedicik.apk: Android package (APK)
...
classes.dex
lib/arm64-v8a/libkittykittybangbang.so
lib/armeabi-v7a/libkittykittybangbang.so
lib/x86/libkittykittybangbang.so
lib/x86_64/libkittykittybangbang.so
```

**Analiz:**
- ✅ Uygulama standart bir Android APK.
- 💡 Birden fazla mimari için derlenmiş `libkittykittybangbang.so` mevcut.
- 🎯 Demek ki flag mantığının en az bir kısmı native tarafta saklanıyor.

---

### 🔬 **Adım 2: Native fonksiyon isimlerini çıkar**

Native library üzerinde `strings` çalıştırınca JNI export isimleri ve önemli semboller görülebiliyor.

```bash
strings -n 5 lib/x86_64/libkittykittybangbang.so | grep -i "jni\|kitty\|string"
```

**Çıktı:**
```text
Java_com_nahamcon2024_kittykittybangbang_MainActivity_stringFromJNI
libkittykittybangbang.so
```

**Analiz:**
- ✅ `MainActivity_stringFromJNI` fonksiyonu açıkça görülüyor.
- 💡 Bu isim, Java/Kotlin tarafında `stringFromJNI()` çağrısı olduğunu gösterir.
- 🎯 Flag ya doğrudan bu fonksiyondan dönüyor ya da bu fonksiyon flag'in çekirdek parçasını üretiyor.

---

### 🧠 **Adım 3: Uygulama akışını yorumla**

Bu tip sorularda Java tarafı genelde ya bir buton tıklamasında ya da ekran dokunmasında native fonksiyonu çağırır. Bu challenge'da da olay akışı şu mantıkta ilerliyor:

- Uygulama açılıyor.
- Kullanıcı ekrana dokunuyor.
- `stringFromJNI()` çağrılıyor.
- Dönen değer `flag{...}` formatında log'a yazdırılıyor.

Aynı challenge için yayımlanmış çözüm notlarında da ekrana dokununca uygulamanın log'a flag bastığı gösteriliyor.

**Analiz:**
- ✅ Flag doğrudan UI'de görünmek zorunda değil.
- 💡 Android logcat çoğu zaman gizli ama çok kritik çıktılar içerir.
- 🎯 Bu yüzden sadece statik analiz değil, davranış analizi de gerekli.

---

### 📱 **Adım 4: Uygulamayı çalıştırıp logcat izle**

En temiz yöntem uygulamayı emülatörde veya fiziksel cihazda açıp logları izlemek.

```bash
adb install kedicik.apk
adb logcat | grep -i "kitty\|flag"
```

Ardından uygulama açılıp ekrana dokunulur.

**Beklenen çıktı mantığı:**
```text
I kitty kitty bang bang: Listening for taps...
I kitty kitty bang bang: Screen tapped!
I kitty kitty bang bang: BANG!
I kitty kitty bang bang: flag{f9028245dd46eedbf9b4f8861d73ae0f}
```

**Analiz:**
- ✅ Uygulama log üzerinden flag'i sızdırıyor.
- 💡 Native fonksiyonun döndürdüğü değer doğrudan `flag{...}` içine yerleştiriliyor.
- 🎯 Böylece gerçek flag net şekilde elde ediliyor.

---

### 🚩 **Adım 5: Flag'i Elde Et**

Sonuç olarak bu challenge'ın flag'i logcat çıktısından alınır:

```bash
adb logcat | grep -i flag
```

**Çıktı:**
```text
flag{f9028245dd46eedbf9b4f8861d73ae0f}
```

---

## 🚩 **FLAG**
```text
flag{f9028245dd46eedbf9b4f8861d73ae0f}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">📦<br><b>unzip</b><br><sub>APK içeriğini listelemek için</sub></td>
<td align="center">🔧<br><b>strings</b><br><sub>Native library içindeki sembolleri görmek için</sub></td>
<td align="center">📱<br><b>adb logcat</b><br><sub>Uygulama loglarını izlemek için</sub></td>
<td align="center">🧩<br><b>jadx</b><br><sub>Java/Kotlin tarafını decompile etmek için</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```text
🎯 Kedicik
       ↓
🔎 APK içeriğini incele
   → classes.dex ve native .so dosyalarını bul
   → libkittykittybangbang.so dikkat çekiyor
       ↓
🔬 Native exportları çıkar
   → stringFromJNI fonksiyonunu tespit et
   → MainActivity ile bağlantıyı kur
       ↓
🧠 Uygulama davranışını analiz et
   → ekrana dokununca native fonksiyon tetikleniyor
   → sonuç log'a yazılıyor
       ↓
📱 logcat izle
   → flag doğrudan log'da görünüyor
       ↓
🚩 FLAG: flag{f9028245dd46eedbf9b4f8861d73ae0f}
```
