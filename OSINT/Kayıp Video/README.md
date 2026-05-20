# 📹 Kayıp Video - OSINT Challenge

<p align="center">
  <img src="https://img.shields.io/badge/OSINT-Challenge-darkblue?style=for-the-badge&logo=search" alt="OSINT">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Wayback Machine-purple?style=for-the-badge&logo=youtube" alt="Wayback">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** OSINT  
**Seviye:** Orta  
**Puan:** 300

**Açıklama:** Bu kanalda bir video vardı. Şimdi bulamıyorum.

**YouTube Kanalı:** [🔗 Russian Top Secret Service](https://www.youtube.com/@RussianTopSecretServiceUnOff)


---

## 🔍 Analiz

**Hedef:** Silinmiş YouTube videosunu Wayback Machine'de bulup gizli QR kodu decode etmek

**Strateji:** Archive.org → Video snapshot → QR kod → Google Maps → Flag

**İpuçları:**
- 🕰️ **Wayback Machine:** Silinmiş içerikleri arşivden bul
- 📅 **Tarih analizi:** Yoğun aktivite günleri önemli
- 🎬 **Video analizi:** Frame by frame inceleme
- 🗺️ **QR kod:** Google Maps lokasyonu

---

## ✅ Çözüm Adımları

### 🔍 **1. İlk Keşif**

**YouTube kanalına gidiyoruz:**

```
https://www.youtube.com/@RussianTopSecretServiceUnOff
```

**Sonuç:**
```
❌ Kanal boş görünüyor
❌ Video yok
```

> 💡 Video silinmiş olabilir! Wayback Machine'e bakalım.

---

### 🕰️ **2. Wayback Machine Analizi**

**Archive.org'a gidiyoruz:**

```
https://web.archive.org/
```

**Kanalın URL'sini yapıştırıyoruz:**
```
https://www.youtube.com/@RussianTopSecretServiceUnOff
```

**Takvimi inceliyoruz:**

- 📅 **2023 yılı** en son kayıtlar
- 🗓️ **20 Mayıs 2023** tarihinde yoğun aktivite
- 🔵 **3 mavi nokta** (normal snapshot)
- 🟢 **1 yeşil nokta** (farklı snapshot) → `17:00:42`

> 🎯 Yeşil nokta = İçerikte değişiklik var!

---

### 📸 **3. Snapshot Seçimi**

**17:00:42 saatine tıklıyoruz:**

```
https://web.archive.org/web/20230520170043/https://www.youtube.com/@RussianTopSecretServiceUnOff
```

**Karşımıza çıkan:**
```
✅ 1 video var!
⏱️ Video süresi: 42 saniye
```

**Videoyu açmak için:**
- Sağ tık → `Open link in new tab`

---

### 🎬 **4. Video Analizi**

**Videoyu izlemeye başlıyoruz:**

**Gözlemler:**
- 💬 Yorumlar bot gibi görünüyor
- 🎥 Video içeriğini dikkatli izliyoruz

**30. saniyede:**
```
⚡ 1 saniyelik QR kod görünüyor!
```

> 🎯 QR kodu yakalamak için video durdurulmalı!

**QR kod screenshot:**
- Video 30. saniyede durdur
- Screenshot al
- QR kod net görünüyor ✅

---

### 📱 **5. QR Kod Decode**

**QR Scanner sitesine gidiyoruz:**

```
https://qrscanner.net/tr
```

**QR kodu yüklüyoruz ve okutuyoruz.**

**Çıkan link:**
```
https://goo.gl/maps/NThoxkDWDfB1fHEj6
```

---

### 🗺️ **6. Google Maps Lokasyonu**

**Linke tıklayıp gidiyoruz:**

```
https://goo.gl/maps/NThoxkDWDfB1fHEj6
```

**Lokasyon:**
```
📍 Russian Foreign Intelligence Service (SVR)
🇷🇺 Rusya Dış İstihbarat Servisi
```

> 💡 Challenge adı "Russian Top Secret Service" - Lokasyon bağlantısı mantıklı!

---

### 💬 **7. Yorumları İnceleme**

**Google Maps yorumlarına bakıyoruz:**

**DPWD PLAYZ** adlı kullanıcının yorumu:

```
HACKME{PEŞİNDEYİZVESENİİYİTANIYORUZ}
```

> 🚩 **FLAG BULUNDU!**

---

## 🚩 **FLAG**

```
HACKME{PEŞİNDEYİZVESENİİYİTANIYORUZ}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🕰️<br><b>Wayback Machine</b><br><sub>Archive.org</sub></td>
<td align="center">🎬<br><b>YouTube</b><br><sub>Video platformu</sub></td>
<td align="center">📱<br><b>QR Scanner</b><br><sub>QR kod okuma</sub></td>
<td align="center">🗺️<br><b>Google Maps</b><br><sub>Lokasyon</sub></td>
</tr>
</table>

**Kullanılan Siteler:**
```
📦 https://web.archive.org/
📱 https://qrscanner.net/tr
🗺️ https://goo.gl/maps/
```

---

## 💭 **Çözüm Akışı**

```
📹 "Kayıp Video" Challenge
            ↓
🔍 YouTube kanalı boş
            ↓
🕰️ Wayback Machine'e git
            ↓
📅 2023 - Mayıs 20 analizi
            ↓
🟢 17:00:42 snapshot seçildi
            ↓
🎬 42 saniyelik video bulundu
            ↓
⏱️ 30. saniyede QR kod keşfedildi
            ↓
📱 QR Scanner ile decode
            ↓
🗺️ Google Maps linki
            ↓
📍 Russian Foreign Intelligence Service
            ↓
💬 DPWD PLAYZ yorumunda flag
            ↓
🚩 HACKME{PEŞİNDEYİZVESENİİYİTANIYORUZ}
```
