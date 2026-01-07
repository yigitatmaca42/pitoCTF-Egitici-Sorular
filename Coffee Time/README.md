# ☕ Coffee Time - OSINT Challenge

<p align="center">
  <img src="https://img.shields.io/badge/OSINT-Challenge-darkblue?style=for-the-badge&logo=search" alt="OSINT">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Geolocation-purple?style=for-the-badge&logo=map" alt="Geolocation">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** OSINT  
**Seviye:** Orta  
**Puan:** 200  
**Açıklama:** Google Drive'da bir fotoğraf dosyası veriliyor

**Challenge Dosyası:** [📥 Google Drive - Coffee Image](https://drive.google.com/file/d/1-Vi6lqzZ5_dxa6Fc9D3v6Cjd3zYHdQ7d/view?usp=sharing)

---

## 🔍 Analiz

**Hedef:** Fotoğraf üzerinden konum ve flag bulmak

**Strateji:** Image metadata → Username OSINT → Geolocation → Google Maps

---

## ✅ Çözüm Adımları

### 📥 **1. Dosyayı İndirme & Metadata Analizi**

Google Drive'dan `coffee.jpg` dosyasını indiriyoruz.

**Strings ile analiz:**

```bash
strings coffee.jpg
```

**Çıktıda bulduklarımız:**

```
JFIF
Exif
@coffeeforjava
```

> 🎯 **Önemli:** `@coffeeforjava` username'i bulundu!

---

### 🔎 **2. Username OSINT - X (Twitter) Hesabı**

`@` işaretini kaldırıp **WhatsMyName.me**'de arıyoruz:

```
coffeeforjava
```

**Sonuç:** X (Twitter) hesabı bulundu!

**URL:** https://x.com/coffeeforjava

---

### 📱 **3. X Hesabını İnceleme**

**Hesap Bilgileri:**

- **İsim:** Coffe for Java
- **Username:** @coffeeforjava
- **Tarih:** 31 Ara 2025

**Paylaşımlar:**

**Tweet 1:**
```
Just a quite place.
```
(Sokak fotoğrafı eklemiş)

**Tweet 2:**
```
Before having coffee somewhere, I always check the reviews. 
The Grind has great coffee for writing java.
```

> 🎯 **İpucu:** **The Grind** adlı kahve dükkanından bahsediyor!

---

### 🔍 **4. Google Lens ile Fotoğraf Analizi**

X'te paylaşılan sokak fotoğrafını **Google Lens**'e yüklüyoruz.

**Sonuç:** https://gohikevirginia.com/things-to-do-in-wytheville-va/

**Konum Bulundu:**
```
Wytheville, Virginia
```

**Sitede keşif:**

```
#10: Stroll Historic Downtown Wytheville.
```

**Fotoğrafta görülen:**
- Devasa sarı kalem heykeli/pankartı
- **"WYTHEVILLE OFFICE SUPPLY"** dükkanı

---

### 🗺️ **5. Google Maps Araştırması**

**1. Adım:** Google Maps'te arama:

```
Wytheville Office Supply
```

**2. Adım:** Konuma gidiyoruz ve **karşı kaldırıma** bakıyoruz

**3. Bulduğumuz:**
```
The Grind
```

> ✅ X'te bahsedilen kahve dükkanı!

---

### 💬 **6. Google Maps Yorumlarında Flag**

**The Grind** yorumlarında `java` kelimesini arıyoruz.

**Bulunan Yorum:**

**Jim Bossem** - 1 yorum, 1 fotoğraf

```
This place is fantastic! The staff is always great and friendly. Probably offering up the best java around! As a programmer I know how important Java is, haha!

Flag{this_is_good_java}
```

**Ayrıca:** Jim Bossem'in yüklediği kahve bardağı fotoğrafında da flag yazıyor!

---

## 🚩 **FLAG**

```
Flag{this_is_good_java}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>Strings</b><br><sub>Metadata analizi</sub></td>
<td align="center">🌐<br><b>WhatsMyName</b><br><sub>Username OSINT</sub></td>
<td align="center">📸<br><b>Google Lens</b><br><sub>Reverse image search</sub></td>
<td align="center">🗺️<br><b>Google Maps</b><br><sub>Geolocation</sub></td>
</tr>
</table>

**Kullanılan Siteler:**
- 🔍 **Strings:** Terminal komutu
- 🌐 **WhatsMyName:** https://whatsmyname.me/
- 🐦 **X Profile:** https://x.com/coffeeforjava
- 📸 **Google Lens:** https://lens.google.com/
- 🗺️ **Google Maps:** The Grind, Wytheville, VA

---

## 💭 **Çözüm Akış Şeması**

```
☕ OSINT Challenge: "Coffee Time"
                    ↓
        📥 Fotoğrafı İndir (coffee.jpg)
                    ↓
        🔍 strings coffee.jpg → @coffeeforjava
                    ↓
        🌐 WhatsMyName → X hesabı bulundu
                    ↓
        🐦 X profili: "The Grind" kahveciden bahsediyor
                    ↓
        📸 Sokak fotoğrafını Google Lens'e yükle
                    ↓
        📍 Konum: Wytheville, Virginia
                    ↓
        🗺️ Google Maps: Wytheville Office Supply ara
                    ↓
        ☕ Karşı kaldırımda "The Grind" bul
                    ↓
        💬 Yorumlarda "java" ara
                    ↓
        👤 Jim Bossem yorumunda flag var
                    ↓
        🚩 Flag{this_is_good_java}
```
