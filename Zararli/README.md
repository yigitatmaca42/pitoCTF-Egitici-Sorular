# 🦠 Zararli - Misc Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Misc-Challenge-darkblue?style=for-the-badge&logo=bug" alt="Misc">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-OSINT Crypto-purple?style=for-the-badge&logo=search" alt="OSINT">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Misc  
**Seviye:** Orta  
**Puan:** 500  

**Challenge Dosyası:** [📥 Google Drive - Zararli](https://drive.google.com/file/d/18Uy5ScGxVCkr3k7TICvAGfpkzC5xNKUB/view?usp=drivesdk)

**Firstblood:** +50 puan

---

## 🔍 Analiz

**Hedef:** "Zararlı" PDF dosyasını analiz edip flag'i bulmak

**Strateji:** PDF analizi → VirusTotal → OSINT → Flag

---

## ✅ Çözüm Adımları

### 📥 **1. Dosya İndirme ve İlk Analiz**

**Dosyayı indirip kontrol ediyoruz:**

```bash
file CokzararliAcma.pdf
```

**Çıktı:**
```
CokzararliAcma.pdf: PDF document, version 1.7, 1 page(s)
```

---

### 🔤 **2. String Analizi**

**PDF içindeki string'leri inceliyoruz:**

```bash
strings CokzararliAcma.pdf
```

**Çıktı:**
```
%PDF-1.7
1 0 obj
<</Pages 2 0 R /PieceInfo<</SPenSDK_PAGE_SINGLE<</LastModified(D:20260109140328)...
/Creator(Samsung Electronics)/ModDate(D:20260109140328+03'00')>>
...
```

**Analiz:**
- ✅ Geçerli PDF dosyası
- 📄 1 sayfa
- 📱 Samsung Electronics tarafından oluşturulmuş
- ❌ String'lerde flag bulunamadı

---

### 🔍 **3. VirusTotal Analizi**

**PDF'i VirusTotal'e yüklüyoruz:**

```
https://www.virustotal.com/gui/home/upload
```

**VirusTotal Sonuçları:**

```
SHA-256: 18884c99960787daee702fe0a136dad49bbca339832ee14d861fae0eb19e4053
Dosya: CokzararliAcma.pdf
Boyut: 27.58 KB
Detection: 0/63 ✅ (Temiz!)
```

**Analiz:**
- ✅ **Hiçbir antivirüs zararlı bulmadı**
- 🎭 **"Zararlı"** adı kandırmaca!

---

### 💬 **4. Community Sekmesi - İlk İpucu**

**VirusTotal'de "COMMUNITY" sekmesini kontrol ediyoruz:**

**Comments (1):**

```
👤 flagbizdensorulur
📅 3 days ago

Kandırdım. Zararlı değilim. Madem buraya geldin. 
Biraz da osint yap: 0xb4ykus
```

**Analiz:**
- 🎯 **OSINT ipucu:** `0xb4ykus`
- 🔍 Bu kullanıcı adını araştırmamız gerekiyor

---

### 🔎 **5. OSINT - WhatIsMyName**

**Kullanıcı adını sosyal medyada arıyoruz:**

```
https://whatsmyname.me/
```

**Arama:** `0xb4ykus`

**Sonuç:**

```
Platform: X (Twitter)
URL: https://x.com/0xb4ykus
```

---

### 🐦 **6. Twitter/X Hesabı İnceleme**

**Hesaba gidiyoruz:**

```
https://x.com/0xb4ykus
```

**Profil:**
```
👤 heckir olucak @0xb4ykus
```

**Tweet:**
```
10 saat yol gidilcekmiş hehehhe
```

**Fotoğraf analizi:**
- 💻 Laptop ekranında kamyon görseli
- 🎮 Gaming laptop (turuncu klavye)
- ⚠️ **ÖNEMLİ:** Laptop'un solunda **QR kod** var!

---

### 📱 **7. QR Kod Okuma**

**Fotoğrafı indirip QR kod okuyoruz:**

```
https://qrscanner.net/tr
```

**QR Kod İçeriği:**
```
C[ITQICkwesOt#t!w!~Oieb#{d#t!bm
```

**Analiz:**
- 🔐 Şifrelenmiş bir metin
- 🎯 XOR cipher gibi görünüyor
- 🔑 Brute force gerekiyor

---

### 🔓 **8. XOR Brute Force**

**CyberChef'e gidiyoruz:**

```
https://cyberchef.io/
```

**İşlemler:**
1. "XOR Brute Force" operasyonunu ekliyoruz
2. Input alanına şifreli metni yapıştırıyoruz:
   ```
   C[ITQICkwesOt#t!w!~Oieb#{d#t!bm
   ```
3. "Bake" butonuna basıyoruz

---

### 🎯 **9. Flag Bulundu!**

**Brute Force Sonuçları:**

```
Key = 0e: MUGZ_GMeyk}Az-z/y/pAgkl-uj-z/lc.
Key = 0f: LTF[^FLdxj|@{,{.x.q@fjm,tk,{.mb/
Key = 10: SKYDAYS{guc_d3d1g1n_yur3kt3d1r}
Key = 11: RJXE@XRzftb^e2e0f0o^xts2ju2e0s|
```

**Doğru Key: 0x10 (16)**

```
SKYDAYS{guc_d3d1g1n_yur3kt3d1r}
```

---

## 🚩 **FLAG**

```
SKYDAYS{guc_d3d1g1n_yur3kt3d1r}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">📄<br><b>file</b><br><sub>Dosya analizi</sub></td>
<td align="center">🔤<br><b>strings</b><br><sub>String extraction</sub></td>
<td align="center">🦠<br><b>VirusTotal</b><br><sub>Malware scan</sub></td>
<td align="center">🔍<br><b>WhatIsMyName</b><br><sub>OSINT tool</sub></td>
</tr>
<tr>
<td align="center">🐦<br><b>Twitter/X</b><br><sub>Social media</sub></td>
<td align="center">📱<br><b>QR Scanner</b><br><sub>QR kod okuma</sub></td>
<td align="center">🔐<br><b>CyberChef</b><br><sub>XOR Brute Force</sub></td>
<td align="center">💻<br><b>Kali Linux</b><br><sub>Analiz ortamı</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**
```bash
# Dosya analizi
file CokzararliAcma.pdf

# String analizi
strings CokzararliAcma.pdf

# PDF içeriğini görüntüleme
cat CokzararliAcma.pdf
```

**Online Araçlar:**
```
VirusTotal: https://www.virustotal.com/
WhatIsMyName: https://whatsmyname.me/
QR Scanner: https://qrscanner.net/tr
CyberChef: https://cyberchef.io/
```

---

## 💭 **Çözüm Akışı**

```
🦠 "Zararli" Challenge
            ↓
📥 CokzararliAcma.pdf indirildi
            ↓
📄 file CokzararliAcma.pdf
   → PDF document, version 1.7
            ↓
🔤 strings ile analiz
   → Flag bulunamadı
            ↓
🦠 VirusTotal'e yükleme
   SHA-256: 18884c99960787daee...
   Detection: 0/63 ✅ Temiz!
            ↓
💬 COMMUNITY sekmesi kontrol
   👤 flagbizdensorulur:
   "Kandırdım. Zararlı değilim.
    OSINT yap: 0xb4ykus"
            ↓
🔍 WhatIsMyName araması
   Platform: X (Twitter)
   URL: https://x.com/0xb4ykus
            ↓
🐦 Twitter hesabı inceleme
   Tweet: "10 saat yol gidilcekmiş"
   📸 Fotoğrafta QR kod tespit edildi!
            ↓
📱 QR kod okuma
   → C[ITQICkwesOt#t!w!~Oieb#{d#t!bm
            ↓
🔐 CyberChef XOR Brute Force
   Key 0x10 denendi
            ↓
🎯 Flag bulundu!
   SKYDAYS{guc_d3d1g1n_yur3kt3d1r}
            ↓
🚩 Flag submit edildi ✅
```
