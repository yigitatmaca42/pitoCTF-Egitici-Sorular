# 🎯 Kilit - Mobile Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Mobile-Challenge-darkblue?style=for-the-badge" alt="Mobile">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-APK%20Reverse-purple?style=for-the-badge" alt="APK Reverse">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Mobile  
**Seviye:** Orta  
**Puan:** 300

**Challenge Dosyası / URL:** [📥 Git Hub - Kilit.apk](https://github.com/cihangungor/pitoctf/blob/main/Kilit.apk)

---

## 🔍 Analiz

**Hedef:** APK içindeki gerçek flag’i üreten mantığı bulup flag’i çıkarmak.

**Strateji:** APK’yı aç → Java/XML tarafını incele → Gizli activity’yi bul → Native mantığı çöz → Flag

**İpuçları:**
- 🎯 `MainActivity` içindeki `secretpass` değeri özellikle fake görünüyor.
- 🔍 XML tarafında “unreachable activity” ifadesi doğrudan yönlendirme veriyor.
- 💡 Challenge sadece Java ile çözülmüyor, native `.so` dosyası da işin içinde.
- 🔑 Ekrana append edilen anahtar ile native fonksiyon birlikte gerçek çıktıyı üretiyor.

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: APK dosyasını inceleme**

İlk olarak APK dosyasını decompile ederek Java kodlarını, XML resource’larını ve native kütüphaneleri inceliyoruz.

```bash
apktool d Kilit.apk -o Kilit_apktool
jadx -d Kilit_jadx Kilit.apk
```

**Çıktı:**
```text
Kilit_apktool/
├── AndroidManifest.xml
├── res/
├── smali/
└── lib/

Kilit_jadx/
├── sources/
└── resources/
```

**Analiz:**
- ✅ APK içinde hem Java tarafı hem de native `.so` dosyaları mevcut.
- 💡 Bu da çözümün yalnızca string aramakla bitmeyeceğini gösteriyor.
- 🎯 Sonraki adımda activity’ler ve dikkat çeken sabit değerler incelenmeli.

---

### 🔬 **Adım 2: MainActivity içindeki fake flag’i tespit etme**

Decompile edilen Java kodlarında `MainActivity` içinde dikkat çeken bir string bulunuyor.

```bash
grep -R "th1s_is_n0t_the_flag" -n Kilit_jadx/sources
```

**Çıktı:**
```text
String secretpass = "th1s_is_n0t_the_flag_:(";
```

**Analiz:**
- ✅ Bu değer açıkça gerçek flag olmadığını söylüyor.
- 💡 Geliştirici burada kullanıcıyı yanlış yola sokmaya çalışmış.
- 🎯 O yüzden bu string yerine uygulamanın diğer activity ve resource dosyalarına bakmamız gerekiyor.

---

### 🧠 **Adım 3: XML içindeki kritik ipucunu bulma**

Resource dosyalarında yapılan incelemede `activity_flag.xml` içinde çok önemli bir yönlendirme görülüyor.

```bash
grep -R "unreachable activity" -n Kilit_apktool/res Kilit_jadx/resources
```

**Çıktı:**
```text
Look at the unreachable activity
```

**Analiz:**
- ✅ XML doğrudan “erişilemeyen activity”ye bakmamızı söylüyor.
- 💡 Demek ki manifestte veya kaynak kodda normal akışta açılmayan gizli bir activity var.
- 🎯 Sonraki adımda bu activity bulunup içeriği incelenmeli.

---

### 🔬 **Adım 4: UnreachableActivity içindeki gerçek anahtarı çıkarma**

Kod tarafında yapılan incelemede `UnreachableActivity` isimli bir activity bulunuyor. Bu activity içinde ekrana bir anahtar yazdırılıyor.

```bash
grep -R "wohoo_you_found_the_key" -n Kilit_jadx/sources
```

**Çıktı:**
```text
textView.append("wohoo_you_found_the_key");
```

**Analiz:**
- ✅ Uygulama ekrana doğrudan bir key append ediyor.
- 💡 Buradaki gerçek anahtar: `wohoo_you_found_the_key`
- 🎯 Bu anahtarın biraz sonra native fonksiyonla birlikte kullanıldığı görülüyor.

---

### 🧠 **Adım 5: Native fonksiyon çağrısını analiz etme**

Aynı activity içinde native tarafa giden fonksiyon çağrısı da yer alıyor.

```bash
grep -R "InternetUtil.getKey" -n Kilit_jadx/sources
```

**Çıktı:**
```text
InternetUtil.getKey(...)
```

**Analiz:**
- ✅ Flag üretim mantığının bir kısmı JNI üzerinden native koda taşınmış.
- 💡 Kodda `fakekeyfakekeyfakekey` gibi yanıltıcı bir değer görünse de asıl önemli olan ekrana yazdırılan anahtar.
- 🎯 Artık `libnative-lib.so` içindeki işlemleri incelemek gerekiyor.

---

### 🧠 **Adım 6: Native mantığı çözme**

Native kütüphanede sabit 32 baytlık veri ve XOR benzeri bir işlem bulunuyor. Bu veri, Java tarafında bulunan anahtar ile işleniyor.

```python
data = [/* native içindeki sabit byte dizisi */]
key = "wohoo_you_found_the_key"

out = []
for i in range(len(data)):
    out.append(data[i] ^ ord(key[i % len(key)]))

print(bytes(out))
```

**Çıktı:**
```text
Ara çıktı elde edildi ancak son hali henüz tamamlanmadı.
```

**Analiz:**
- ✅ Native tarafta temel işlem XOR mantığıyla çalışıyor.
- 💡 Bu çıktı tek başına final flag değil; Java tarafındaki ek dönüşümle tamamlanıyor.

---

### 🔬 **Adım 7: Java tarafındaki offset işlemini uygulama**

Native çıktı alındıktan sonra Java tarafında integer offset listesi ile bazı karakterler kaydırılıyor. Son olarak string’in son 4 karakteri atılıp `}` ekleniyor.

```python
flag = "FL1TZ{Gl0ub_gra33_ft0ur_Sbe7xxxx"
flag = flag[:-4] + "}"
print(flag)
```

**Çıktı:**
```text
FL1TZ{Gl0ub_gra33_ft0ur_Sbe7}
```

**Analiz:**
- ✅ Native çözüm ile Java tarafındaki son düzenleme birleşince flag elde ediliyor.
- 💡 Challenge, fake string + unreachable activity + native reverse kombinasyonu üzerine kurulmuş.
- 🎯 Final flag artık elimizde.

---

### 🚩 **Adım 8: Flag'i Elde Et**

Son adımda elde edilen çıktı doğrudan challenge flag’i oluyor.

```bash
echo "FL1TZ{Gl0ub_gra33_ft0ur_Sbe7}"
```

**Çıktı:**
```text
FL1TZ{Gl0ub_gra33_ft0ur_Sbe7}
```

---

## 🚩 **FLAG**
```text
FL1TZ{Gl0ub_gra33_ft0ur_Sbe7}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">📦<br><b>apktool</b><br><sub>APK dosyasını açmak için</sub></td>
<td align="center">☕<br><b>jadx</b><br><sub>Java kodlarını decompile etmek için</sub></td>
<td align="center">🐍<br><b>Python</b><br><sub>Native mantığı yeniden canlandırmak için</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```text
🎯 Kilit
       ↓
🔎 APK dosyasını aç
   → Java kodlarını incele
   → XML resource’larını kontrol et
       ↓
🔬 Fake flag’i ele
   → MainActivity içindeki yanıltıcı stringi tespit et
   → XML içindeki unreachable activity ipucunu bul
       ↓
🧠 Gizli activity + native çözüm
   → UnreachableActivity içindeki key’i çıkar
   → libnative-lib.so mantığını çöz
   → Java offset işlemini uygula
       ↓
🚩 FLAG: FL1TZ{Gl0ub_gra33_ft0ur_Sbe7}
```
