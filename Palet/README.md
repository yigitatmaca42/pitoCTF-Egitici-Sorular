# 🎨 Palette - Mobil Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Mobile-Challenge-darkblue?style=for-the-badge&logo=android" alt="Mobile">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-APK Analysis-purple?style=for-the-badge&logo=apk" alt="APK">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Mobil  
**Seviye:** Orta  
**Puan:** 300  

**Challenge Dosyası:** [📥 varit.apk](https://github.com/cihangungor/pitoctf/blob/main/varit.apk)

**İpucu:** "Hep jadx bakmayın istedim. apk tool da güzel. Renkleri daha güzel :)"

**Firstblood:** +50 puan

---

## 🔍 Analiz

**Hedef:** Android APK dosyasını analiz edip palette renklerinden flag oluşturmak

**Strateji:** APK extract → Palette image analizi → colors.xml inceleme → Flag

**İpuçları:**
- 🎨 **Palette:** Renk paletleri önemli
- 📱 **Android colors.xml:** Renk tanımlamaları
- 🔍 **APK Tool:** Decode edilen XML dosyaları
- 🎯 **Format:** STMCTF{...} formatında flag

---

## ✅ Çözüm Adımları

### 📥 **1. Dosya Analizi**

**APK dosyasını indirip kontrol ediyoruz:**

```bash
file varit.apk
```

**Çıktı:**
```
varit.apk: Android package (APK), with APK Signing Block
```

> ✅ **Android APK** dosyası doğrulandı!

---

### 📦 **2. APK İçeriğini Extract Etme**

**APK dosyası aslında bir ZIP arşivi, basit unzip ile açıyoruz:**

```bash
unzip varit.apk -d varit_extracted
cd varit_extracted
ls -la
```

**Çıktı:**
```
AndroidManifest.xml
classes.dex
classes2.dex
classes3.dex
classes4.dex
resources.arsc
res/
META-INF/
```

**Analiz:**
- 📄 **AndroidManifest.xml:** Uygulama yapılandırması (binary format)
- 🔢 **classes.dex:** Derlenmiş Java/Kotlin kodu
- 📊 **resources.arsc:** Uygulama kaynakları (binary format)
- 📁 **res/:** Resim, layout ve diğer kaynak dosyaları

---

### 🎨 **3. Resim Dosyalarını İnceleme**

**Drawable klasöründeki resimleri listeliyoruz:**

```bash
find varit_extracted/res/drawable/ -type f -name "*.png"
```

**Önemli dosyalar:**
```
varit_extracted/res/drawable/palette.png    ← 🎯 Dikkat!
varit_extracted/res/drawable/header.png
varit_extracted/res/drawable/icon.png
varit_extracted/res/drawable/ctf_logo.png
```

> 🎨 **palette.png** dosyası şüpheli görünüyor!

---

### 🖼️ **4. Palette.png Analizi**

**Resmi görüntülemek için kopyalıyoruz:**

```bash
cp varit_extracted/res/drawable/palette.png .
```

**Resmi açıyoruz:**
```bash
xdg-open palette.png
# veya
eog palette.png
```

**Resimde ne var:**
```
📝 Üstte: "STMCTF{" yazısı
🎨 6 renkli kutu (2 sıra x 3 sütun):
📝 Sağ altta: "}" işareti
```

> 💡 Flag formatı gösteriliyor ama renk kodları ne?

---

### 🔤 **5. Python ile Renk Analizi**

**Resimden renkleri çıkarmak için Python kullanıyoruz:**

```python
from PIL import Image
import collections

# Palette.png'yi aç
img = Image.open('varit_extracted/res/drawable/palette.png')
img_rgb = img.convert('RGB')

# Tüm pikselleri al ve renkleri say
pixels = list(img_rgb.getdata())
color_counts = collections.Counter(pixels)

# En çok kullanılan renkleri göster
print("En çok kullanılan renkler:")
for i, (color, count) in enumerate(color_counts.most_common(10), 1):
    hex_color = '#{:02x}{:02x}{:02x}'.format(color[0], color[1], color[2])
    print(f"{i}. RGB{color} = {hex_color} - {count} piksel")
```

**Çıktı:**
```
En çok kullanılan renkler:
1. RGB(220, 219, 210) = #dcdbd2 - 1310189 piksel   ← Background
2. RGB(255, 255, 255) = #ffffff - 151252 piksel    ← Beyaz çerçeve
3. RGB(183, 183, 164) = #b7b7a4 - 33856 piksel     ← Renk 5
4. RGB(221, 190, 169) = #ddbea9 - 33672 piksel     ← Renk 2
5. RGB(165, 165, 141) = #a5a58d - 33672 piksel     ← Renk 4
6. RGB(107, 112, 92) = #6b705c - 33672 piksel      ← Renk 6
7. RGB(203, 153, 126) = #cb997e - 33489 piksel     ← Renk 1
8. RGB(255, 232, 214) = #ffe8d6 - 33489 piksel     ← Renk 3
```

---

### 📍 **6. Renk Kutularının Konumlarını Bulma**

**Renklerin resimde hangi sırada olduğunu bulmak için konumlarına bakıyoruz:**

```python
# Renk kutularını konum bazlı bul
for y in range(0, height, 100):
    for x in range(0, width, 100):
        color = img_rgb.getpixel((x, y))
        # Background ve beyaz değilse
        if color not in [(220, 219, 210), (255, 255, 255)]:
            hex_color = '#{:02x}{:02x}{:02x}'.format(color[0], color[1], color[2])
            print(f"Konum ({x:4d}, {y:4d}): {hex_color}")
```

**Çıktı:**
```
Konum ( 200,  700): #cb997e
Konum ( 400,  700): #ddbea9
Konum ( 700,  700): #ffe8d6
Konum ( 200, 1000): #a5a58d
Konum ( 400, 1000): #b7b7a4
Konum ( 700, 1000): #6b705c
```

**Sıralı renkler (yukarıdan aşağıya, soldan sağa):**
```
1. #cb997e
2. #ddbea9
3. #ffe8d6
4. #a5a58d
5. #b7b7a4
6. #6b705c
```

---

### 🎯 **7. İlk Flag Denemesi**

**Bulduğumuz renkleri flag formatına sokuyoruz:**

```
STMCTF{cb997e_ddbea9_ffe8d6_a5a58d_b7b7a4_6b705c}
```

**Deneme:**
```
❌ YANLIŞ!
```

> 🤔 Neden yanlış? İpucunu tekrar okuyalım: "apk tool da güzel. Renkleri daha güzel"

---

### 🔧 **8. APKTool ile Decode (Doğru Yöntem)**

**İpucu bize APKTool kullanmamızı söylüyor:**

```bash
apktool d varit.apk -o varit_decoded
```

> 💡 **APKTool** binary XML dosyalarını okunabilir hale getirir!

**Decode edilen dosyaları inceliyoruz:**

```bash
cd varit_decoded
ls res/values/
```

**Çıktı:**
```
colors.xml      ← 🎯 Bingo!
strings.xml
styles.xml
```

---

### 🎨 **9. colors.xml İnceleme**

**colors.xml dosyasını açıyoruz:**

```bash
cat res/values/colors.xml
```

**İlgili kısımlar:**
```xml
<color name="one">#ffcb997e</color>
<color name="two">#ffddbea9</color>
<color name="three">#ffffe8d6</color>
<color name="four">#ffa5a58d</color>
<color name="five">#ffb7b7a4</color>
<color name="six">#ff6b705c</color>
```

> 🚩 **EUREKA!** Renkler `#ff` ile başlıyor!

**Android renk formatı analizi:**
```
#AARRGGBB
 ││├─┴─ Blue (BB)
 │├──── Green (GG)
 ├───── Red (RR)
 └───── Alpha/Opacity (AA)

#ff = 255 (tam opak)
```

---

### 🏆 **10. Flag Oluşturma**

**Doğru renk kodları (alpha channel ile):**

```
one:   ffcb997e
two:   ffddbea9
three: ffffe8d6
four:  ffa5a58d
five:  ffb7b7a4
six:   ff6b705c
```

**Flag:**
```
STMCTF{ffcb997e_ffddbea9_ffffe8d6_ffa5a58d_ffb7b7a4_ff6b705c}
```

**Deneme:**
```
✅ DOĞRU!
```

---

## 🚩 **FLAG**

```
STMCTF{ffcb997e_ffddbea9_ffffe8d6_ffa5a58d_ffb7b7a4_ff6b705c}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">📄<br><b>file</b><br><sub>Dosya tipi analizi</sub></td>
<td align="center">📦<br><b>unzip</b><br><sub>APK extraction</sub></td>
<td align="center">🔧<br><b>apktool</b><br><sub>APK decode</sub></td>
<td align="center">🐍<br><b>Python/PIL</b><br><sub>Resim analizi</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**

```bash
# Dosya analizi
file varit.apk

# APK extraction (basit yöntem)
unzip varit.apk -d varit_extracted

# Resim dosyalarını bulma
find varit_extracted/ -name "palette.png"

# Python ile renk analizi
python3 analyze_palette.py

# APKTool ile decode (doğru yöntem!)
apktool d varit.apk -o varit_decoded

# colors.xml inceleme
cat varit_decoded/res/values/colors.xml
```

**Python analiz scripti:**

```python
from PIL import Image
import collections

img = Image.open('varit_extracted/res/drawable/palette.png')
img_rgb = img.convert('RGB')
width, height = img.size

# Renk kutularının konumlarını bul
for y in range(0, height, 100):
    for x in range(0, width, 100):
        color = img_rgb.getpixel((x, y))
        if color not in [(220, 219, 210), (255, 255, 255)]:
            hex_color = '#{:02x}{:02x}{:02x}'.format(color[0], color[1], color[2])
            print(f"Konum ({x:4d}, {y:4d}): {hex_color}")
```

---

## 💭 **Çözüm Akışı**

```
🎨 "Palette" Challenge
            ↓
📥 varit.apk indirildi
            ↓
📄 file varit.apk → Android APK
            ↓
📦 unzip → res/drawable/palette.png bulundu
            ↓
🖼️ palette.png → STMCTF{ + 6 renk + }
            ↓
🐍 Python/PIL → Renk kodları çıkarıldı
   #cb997e, #ddbea9, #ffe8d6, #a5a58d, #b7b7a4, #6b705c
            ↓
❌ STMCTF{cb997e_...} → YANLIŞ!
            ↓
💡 İpucu hatırlandı: "apk tool da güzel"
            ↓
🔧 apktool d varit.apk
            ↓
📝 res/values/colors.xml bulundu
            ↓
🎨 Android color format: #AARRGGBB
   one=#ffcb997e, two=#ffddbea9, ...
            ↓
🚩 FLAG: STMCTF{ffcb997e_ffddbea9_ffffe8d6_ffa5a58d_ffb7b7a4_ff6b705c}
```
