# 🕺 SonDans - Misc Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Misc-Challenge-darkblue?style=for-the-badge&logo=puzzle" alt="Misc">
  <img src="https://img.shields.io/badge/Difficulty-Zor-darkred?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Multi Stage-purple?style=for-the-badge&logo=layers" alt="Multi">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Misc  
**Seviye:** Zor  
**Puan:** 700

**Açıklama:** 
```
Emulatör bence şart gibi. Tüm login aşamalarını geç. 
Son Dansı gördüğünde PROFESYONELLERİN düşünebileceği bir şey düşün. 
EEEE son dansı gördük flag nerde diye kızma! Flag orada değil. 
Flag için gerekli bilgiler oradaydı. Son Dansı Gördüysen artık tüm 
bilgiler elinde. Sadece bakman gerek.

Eğer sondansı gördüysen buradan devam et! Aşağıdaki bağlantıda 
profesyonel bir php sayfası var. Ona yukarıda elde ettiğin 
bilgilerle bir post isteği atman lazım.

64.226.74.243:1453/sayfaadiniSonDansiGördüysenBulabilirsin.php 
Not: php sayfası için brute denemeyin. Uzun bir hash. Öyle çıkmaz!
```

**Challenge Dosyası:** [📥 Google Drive - sondans.apk](https://drive.google.com/file/d/187a9pJxF8p706bJuP9K_w5SkajLpU-hO/view?usp=sharing)

**Flag Formatı:** `Flag{}`


---

## 🔍 İlk Bakış ve Analiz

Soru açıklamasına bakınca şunları anlıyoruz:
- 📱 Bir Android APK var
- 🔐 Login aşamalarını geçmemiz gerekiyor
- 🎬 "Son Dans" diye bir şey göreceğiz
- 🧠 PROFESYONELLERİN düşüneceği bir şey düşünmeliyiz
- 🌐 Sonunda bir PHP sayfasına POST isteği atacağız
- ⚠️ PHP sayfası adı uzun bir hash - brute force yok!

Başlayalım! 🚀

---

## ✅ Çözüm Adımları

### 📱 **Adım 1: APK'yı İndirip İnceliyoruz**

**Önce APK'yı indiriyoruz:**

```bash
wget "https://drive.google.com/uc?export=download&id=187a9pJxF8p706bJuP9K_w5SkajLpU-hO" -O sondans.apk
```

**Dosya tipini kontrol ediyoruz:**

```bash
file sondans.apk
```

```
sondans.apk: Zip archive data, at least v1.0 to extract
```

Hmm, bir APK dosyası... Android uygulaması demek ki. Login aşamalarını geçmemiz gerekiyor ama bunu nasıl yapacağız? Önce APK'nın içine bakmamız lazım!

---

### 🔓 **Adım 2: APK'yı Decompile Ediyoruz**

APK'yı açmak için `apktool` kullanacağız:

```bash
# Eğer apktool kurulu değilse
sudo apt install apktool -y

# APK'yı decompile et
apktool d sondans.apk -o sondans

# Dizine gir
cd sondans

# İçeriği kontrol et
ls -la
```

```
AndroidManifest.xml
smali/
res/
assets/
```

Güzel! APK başarıyla decompile edildi. Şimdi içinde neler var bakalım...

---

### 🔍 **Adım 3: Activity'leri Keşfediyoruz**

Android uygulamalarında "Activity" denen ekranlar var. Hangi activity'ler var bakalım:

```bash
cat AndroidManifest.xml | grep -i "activity"
```

```xml
<activity android:name="com.example.thelastdance.MainActivity"/>
<activity android:name="com.example.thelastdance.fa" android:label="fa"/>
<activity android:name="com.example.thelastdance.ok"/>
<activity android:name="com.example.thelastdance.qa"/>
<activity android:name="com.example.thelastdance.la"/>
<activity android:name="com.example.thelastdance.rq"/>
<activity android:name="com.example.thelastdance.ma"/>
```

Interesting! `fa`, `ok`, `la`, `qa`, `rq` gibi kısa isimli activity'ler var. 
- `fa` → "fa" label'ı var, belki **F**inal **A**ctivity? (Son Dans?)
- `ok` → Belki login ekranı?
- `la` → Belki **L**ogin **A**ctivity veya **L**aunch **A**ctivity?

Şimdi bu activity'lerin kodlarına bakmamız lazım. Smali klasöründe olmalılar.

---

### 🔐 **Adım 4: Login Bilgilerini Arıyoruz**

Activity'lerin smali kodlarına bakalım:

```bash
cd smali/com/example/thelastdance/
ls -la
```

```
fa.smali
la.smali
ok.smali
qa.smali
rq.smali
```

Hepsini tek tek inceleyeceğiz. Önce `la.smali` ile başlayalım:

```bash
grep "const-string" la.smali | head -20
```

```smali
const-string v5, "0123456789qwertyuiopasdfghjklzx"
const-string v2, "0279wrtiopasjklz"
const-string v4, "AES"
const-string v1, "Qu/+ykvgsyAkGpihkknrkQ=="
const-string v2, "MDI3OXdydGlvcGFzamtseg=="
const-string v0, "key"
```

Ooo! AES şifreleme var ve Base64 encoded string'ler görüyoruz. Bu Base64'ü decode edelim:

```bash
echo "MDI3OXdydGlvcGFzamtseg==" | base64 -d
```

```
0279wrtiopasjklz
```

Bu bir key gibi görünüyor. Peki username nerede? `fa.smali`'ye bakalım:

```bash
grep "const-string" fa.smali | head -30
```

```smali
const-string v3, "ZDE5MjIxZmNiMWIyZg=="
const-string v0, "cfa67e82"
const-string v2, "28"
```

Bu da Base64'e benziyor! Decode edelim:

```bash
echo "ZDE5MjIxZmNiMWIyZg==" | base64 -d
```

```
d19221fcb1b2f
```

**Vay be! Username'i bulduk:** `d19221fcb1b2f`

Ama password nerede? 🤔

---

### 🎬 **Adım 5: "Son Dans" GIF'ini Buluyoruz**

Soru açıklamasında "Son Dansı gördüğünde..." diyor. Demek ki bir GIF veya görsel dosya var. Arayalım:

```bash
cd ~/Desktop/sondans
find . -name "*.gif"
```

```
./res/drawable/tld.gif
```

**Buldum!** `tld.gif` → "The Last Dance" GIF'i! Kopyalayalım:

```bash
cp res/drawable/tld.gif ~/Desktop/tld.gif
cd ~/Desktop

# GIF'e bakalım
file tld.gif
```

```
tld.gif: GIF image data, version 89a, 1280 x 842
```

1280x842 boyutunda bir GIF. Soru açıklaması "PROFESYONELLERİN düşüneceği bir şey düşün" diyor... 

GIF'in içinde gizli data olabilir mi? Frame'lere ayırıp pixel pixel bakmamız gerekebilir! 🎨

---

### 🎨 **Adım 6: GIF Frame Analizi**

GIF'i frame'lere ayıralım:

```bash
mkdir gif_frames
cd gif_frames
convert ../tld.gif -coalesce frame_%02d.png

# Kaç frame var?
ls frame_*.png | wc -l
```

```
66
```

66 frame var! Şimdi soru şu: Bu frame'lerin içinde nasıl data gizlenmiş?

"PROFESYONELLERİN düşüneceği bir şey" dediyse... Belki koordinatlar? 🤔

---

### 🔢 **Adım 7: Matematik Formüllerini Buluyoruz**

`smali` klasöründe matematik işlemleri aratalım:

```bash
cd ~/Desktop/sondans
find smali/ -name "*.smali" | xargs grep -l "Math"
```

`g1/d.smali` dosyasında Math işlemleri var! İnceleyelim:

```bash
cat smali/g1/d.smali | grep -A50 "28561"
```

```smali
const-wide v1, 0x40dbe44000000000L    # 28561.0
invoke-static {v1, v2}, Ljava/lang/Math;->sqrt(D)D
move-result-wide v1

const-wide/high16 v3, 0x4024000000000000L    # 10.0
invoke-static {v3, v4}, Ljava/lang/Math;->log10(D)D
move-result-wide v3

sub-double/2addr v1, v3
const-wide/high16 v3, 0x4000000000000000L    # 2.0
div-double/2addr v1, v3
const-wide/high16 v5, 0x4008000000000000L    # 3.0
div-double/2addr v1, v5

invoke-static {v3, v4, v5, v6}, Ljava/lang/Math;->pow(DD)D
move-result-wide v7
add-double/2addr v7, v1
add-double/2addr v7, v5

invoke-static {v7, v8}, Ljava/lang/Math;->round(D)J
```

Vay be! Karmaşık matematik formülleri var. Devamını okuyunca Y için de formül var:

```smali
const-wide/high16 v5, 0x4034000000000000L    # 20.0
invoke-static {v5, v6}, Ljava/lang/Math;->log10(D)D
move-result-wide v5

const-wide/high16 v7, 0x4014000000000000L    # 5.0
invoke-static {v3, v4, v7, v8}, Ljava/lang/Math;->pow(DD)D
move-result-wide v7

add-double/2addr v7, v5
div-double/2addr v7, v3
const-wide/high16 v2, 0x402e000000000000L    # 15.0
add-double/2addr v7, v2

invoke-static {v7, v8}, Ljava/lang/Math;->round(D)J
```

X ve Y koordinatları hesaplanıyor! Python ile hesaplayalım:

```bash
cd ~/Desktop

cat > calculate_xy.py << 'EOF'
import math

# X hesaplama: (sqrt(28561) - log10(10)) / 2 / 3 + 2^3 + 3
x = round((math.sqrt(28561) - math.log10(10)) / 2 / 3 + math.pow(2, 3) + 3)
print(f"X = {x}")

# Y hesaplama: (log10(20) + 2^5) / 2 + 15
y = round((math.log10(20) + math.pow(2, 5)) / 2 + 15)
print(f"Y = {y}")

# Toplam
print(f"X + Y = {x + y}")
EOF

python3 calculate_xy.py
```

```
X = 39
Y = 32
X + Y = 71
```

**Mükemmel!** X=39, Y=32 koordinatlarını bulduk! 

Şimdi GIF frame'lerindeki (39, 32) koordinatına bakmamız lazım! 🎯

---

### 🔍 **Adım 8: Pixel Steganografi - Her Frame'i İnceliyoruz**

Her frame'deki (39, 32) pikselinin rengine bakalım:

```bash
cd ~/Desktop/gif_frames

for f in frame_*.png; do
  echo "$f: $(convert $f -format '%[pixel:p{39,32}]' info:)"
done
```

```
frame_00.png: srgb(0,0,85)
frame_01.png: srgb(0,0,85)
...
frame_15.png: srgb(36,0,85)     # ← Farklı!
frame_16.png: srgb(36,0,85)     # ← Farklı!
...
frame_25.png: srgb(36,36,85)    # ← Daha farklı!
frame_26.png: srgb(36,36,85)
frame_27.png: srgb(36,36,85)
...
```

**İlginç!** Bazı frame'lerde renk değişiyor! Hangi frame'lerde değişiyor not alalım:

```
15, 16, 25, 26, 27, 29, 30, 46, 47, 48, 50, 51, 52, 64, 65
```

Hmm... Bu sayılar bir anlam ifade ediyor mu? ASCII karakterler olabilir! 🤔

```bash
cd ~/Desktop

cat > analyze_frames.py << 'EOF'
# Renk değişen frame numaraları
frames = [15, 16, 25, 26, 27, 29, 30, 46, 47, 48, 50, 51, 52, 64, 65]

print(f"Frame numaraları: {frames}")
print()

# ASCII karakterlere çevir
ascii_chars = ''.join(chr(f) for f in frames)
print(f"ASCII karakterler: {repr(ascii_chars)}")
print()

# Hex string
hex_string = ''.join(format(n, '02x') for n in frames)
print(f"Hex: {hex_string}")
EOF

python3 analyze_frames.py
```

```
Frame numaraları: [15, 16, 25, 26, 27, 29, 30, 46, 47, 48, 50, 51, 52, 64, 65]

ASCII karakterler: '\x0f\x10\x19\x1a\x1b\x1d\x1e./0234@A'

Hex: 0f10191a1b1d1e2e2f303233344041
```

Control karakterler ve `./0234@A` var. İlginç ama henüz ne işe yaradığını bilmiyoruz...

---

### 🔐 **Adım 9: Pro Parametresini Oluşturma**

`fa.smali` dosyasında "pro" diye bir parametre oluşturuluyormuş. Bakalım:

```bash
cd ~/Desktop/sondans
cat smali/com/example/thelastdance/fa.smali | grep -A10 "249001"
```

```smali
const-wide v2, 0x410e654800000000L    # 249001.0
invoke-static {v2, v3}, Ljava/lang/Math;->sqrt(D)D
invoke-static {v2, v3}, Ljava/lang/Math;->round(D)J

const-string v2, "28"

const-string v3, "ZDE5MjIxZmNiMWIyZg=="
invoke-static {}, Ljava/util/Base64;->getEncoder()
invoke-virtual {v3}, Ljava/lang/String;->getBytes()[B
invoke-virtual {v2, v3}, Ljava/util/Base64$Encoder;->encodeToString([B)

const-wide v2, 0x407ba00000000000L    # 442.0
invoke-static {v2, v3}, Ljava/lang/Math;->floor(D)D

const-string v0, "cfa67e82"

# "=" → "p" değiştir ve lowercase
```

Pro parametresi şöyle oluşturuluyor:
1. sqrt(249001) = 499
2. "28"
3. Base64(Base64("d19221fcb1b2f"))
4. floor(442) = 442
5. "cfa67e82"
6. Sonra "=" → "p" ve lowercase

Python ile oluşturalım:

```bash
cd ~/Desktop

cat > create_pro_param.py << 'EOF'
import math
import base64

print("=== Pro Parametresi Oluşturma ===\n")

# Part 1: sqrt(249001)
part1 = str(round(math.sqrt(249001)))
print(f"Part 1: {part1}")

# Part 2: "28"
part2 = "28"
print(f"Part 2: {part2}")

# Part 3: base64(base64(username))
part3 = base64.b64encode("ZDE5MjIxZmNiMWIyZg==".encode()).decode()
print(f"Part 3: {part3}")

# Part 4: floor(442)
part4 = str(round(math.floor(442)))
print(f"Part 4: {part4}")

# Part 5: "cfa67e82"
part5 = "cfa67e82"
print(f"Part 5: {part5}")

# Birleştir
pro_param = part1 + part2 + part3 + part4 + part5

# "=" → "p" ve lowercase
pro_param = pro_param.replace("=", "p").lower()

print(f"\n=== SONUÇ ===")
print(f"Pro parameter: {pro_param}")
print(f"Length: {len(pro_param)}")
print(f"\nBu PHP sayfası adı olabilir!")
print(f"URL: http://64.226.74.243:1453/{pro_param}.php")
EOF

python3 create_pro_param.py
```

```
=== Pro Parametresi Oluşturma ===

Part 1: 499
Part 2: 28
Part 3: WkRFNU1qSXhabU5pTVdJeVpnPT0=
Part 4: 442
Part 5: cfa67e82

=== SONUÇ ===
Pro parameter: 49928wkrfnu1qsxhabu5ptvdjevpnpt0p442cfa67e82
Length: 44

Bu PHP sayfası adı olabilir!
URL: http://64.226.74.243:1453/49928wkrfnu1qsxhabu5ptvdjevpnpt0p442cfa67e82.php
```

**Vay be!** 44 karakterlik bir hash elde ettik! Soru açıklamasında "uzun bir hash" diyordu. Bu olmalı! 🎯

---

### 🌐 **Adım 10: PHP Sayfasını Test Ediyoruz**

Bakalım bu sayfa var mı:

```bash
curl -s "http://64.226.74.243:1453/49928wkrfnu1qsxhabu5ptvdjevpnpt0p442cfa67e82.php"
```

```json
{
  "status":"error",
  "msg":"Son dansı görmedin mi? Bana \"id\": \"int(x+y)\" ile istek at"
}
```

**EVET! Sayfa bulundu!** 🎉

Bize `"id": "int(x+y)"` ile POST isteği atmamızı söylüyor. 

X=39, Y=32 → X+Y=71

Hemen deneyelim:

```bash
curl -X POST "http://64.226.74.243:1453/49928wkrfnu1qsxhabu5ptvdjevpnpt0p442cfa67e82.php" \
  -H "Content-Type: application/json" \
  -d '{"id": 71}'
```

```json
{"status":"fail","msg":"Yanlış id"}
```

**Yanlış mı?** 🤔 Ama 39+32=71... 

Belki de 71 değil de başka bir şey? Brute force deneyelim (1-100 arası):

---

### 🎯 **Adım 11: Brute Force ile Doğru ID'yi Buluyoruz**

```bash
cd ~/Desktop

cat > find_flag.py << 'EOF'
import requests

url = "http://64.226.74.243:1453/49928wkrfnu1qsxhabu5ptvdjevpnpt0p442cfa67e82.php"

print("Testing IDs from 1 to 100...")
print("=" * 50)

for test_id in range(1, 101):
    try:
        response = requests.post(url, json={"id": test_id}, timeout=5)
        
        # Eğer "Yanlış id" yoksa göster
        if "Yanlış id" not in response.text and "YanlÄ±ÅŸ id" not in response.text:
            print(f"\nid={test_id}: {response.text}")
        
        # Flag var mı?
        if "flag" in response.text.lower() or "Flag" in response.text:
            print(f"\n{'='*50}")
            print(f"🎉 FLAG BULUNDU! ID = {test_id}")
            print(f"{'='*50}")
            print(f"\n{response.text}")
            break
            
    except Exception as e:
        print(f"Error: {e}")
        
print("\nDone!")
EOF

python3 find_flag.py
```

```
Testing IDs from 1 to 100...
==================================================

id=72: {"status":"success","flag":"FLAG{BuSoruTamDansEtmelikmisAmaBuDansBirazBasDondurucuOlduSiberYildizCTFtenCihanHocayaHatira}"}

==================================================
🎉 FLAG BULUNDU! ID = 72
==================================================

{"status":"success","flag":"FLAG{BuSoruTamDansEtmelikmisAmaBuDansBirazBasDondurucuOlduSiberYildizCTFtenCihanHocayaHatira}"}

Done!
```

**FLAG BULUNDU!** 🎉🎉🎉

Doğru ID 72'ymiş! 71 değil 72! (Muhtemelen off-by-one ya da başka bir hesaplama var ama brute force ile bulduk)

---

## 🚩 **FLAG**

```
FLAG{BuSoruTamDansEtmelikmisAmaBuDansBirazBasDondurucuOlduSiberYildizCTFtenCihanHocayaHatira}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🤖<br><b>Apktool</b><br><sub>APK decompile</sub></td>
<td align="center">🔍<br><b>grep</b><br><sub>Pattern search</sub></td>
<td align="center">🎨<br><b>ImageMagick</b><br><sub>GIF analizi</sub></td>
<td align="center">🐍<br><b>Python</b><br><sub>Automation</sub></td>
</tr>
<tr>
<td align="center">🌐<br><b>curl</b><br><sub>HTTP requests</sub></td>
<td align="center">🔐<br><b>base64</b><br><sub>Decoding</sub></td>
<td align="center">📊<br><b>find</b><br><sub>File search</sub></td>
<td align="center">💻<br><b>Kali Linux</b><br><sub>Analiz ortamı</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akışı**

```
🕺 SonDans Challenge
            ↓
📱 APK İndir → file sondans.apk
            ↓
🔓 Decompile → apktool d sondans.apk
            ↓
🔍 Activity'leri Bul → AndroidManifest.xml
            ↓
🔐 Username Bul → fa.smali → d19221fcb1b2f
            ↓
🎬 GIF Bul → res/drawable/tld.gif
            ↓
🎨 Frame'lere Ayır → 66 frame
            ↓
🔢 X,Y Hesapla → g1/d.smali → X=39, Y=32
            ↓
🔍 Pixel Analizi → (39,32) koordinatı
   → Frame 15,16,25,26,27,29,30,46,47,48,50,51,52,64,65
            ↓
🔐 Pro Parameter → fa.smali analizi
   → 49928wkrfnu1qsxhabu5ptvdjevpnpt0p442cfa67e82
            ↓
🌐 PHP Sayfası → /[hash].php bulundu
   → "id": "int(x+y)" istiyor
            ↓
🎯 Brute Force → id=1 to 100
   → id=72 DOĞRU!
            ↓
🚩 FLAG{BuSoruTamDansEtmelikmisAmaBuDansBirazBasDondurucuOlduSiberYildizCTFtenCihanHocayaHatira}
```
