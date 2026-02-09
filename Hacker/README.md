# 🎯 Hacker - OSINT Challenge

<p align="center">
  <img src="https://img.shields.io/badge/OSINT-Challenge-darkblue?style=for-the-badge" alt="OSINT">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Geolocation-purple?style=for-the-badge" alt="Geolocation">
</p>

---

## 📋 Challenge Bilgileri

**Kategori:** OSINT  
**Zorluk:** Orta  
**Puan:** 500

**Açıklama:** Beyaz şapkalı bir hacker arıyoruz. Elimizde sadece bu fotoğraf var. Hacker buralarda bir yerde sıra beklemiş.

**Challenge Dosyası:** [📥 Google Drive - hacker.jpg](https://drive.google.com/file/d/1USoOnf2mv0CyU4epmSsN0fkTZDvLDPpX/view?usp=drivesdk)

**Flag Formatı:** `Flag{hackerınUzerindekiYazi}`

---

## 🔍 Çözüm

### Adım 1: Fotoğrafı İnceleme

Fotoğrafta gözlemlenenler:
- Arka planda bir **anıt** var
- Anıt üzerinde **2 bayrak** görünüyor
- Bayraklardan biri **Türk bayrağı**
- Diğer bayrak **Güney Kore** veya **Japon bayrağı**'na benziyor

---

### Adım 2: Google Lens ile Görsel Arama

```bash
# Google Lens'e fotoğrafı yükle
# https://lens.google.com/
```

**Sonuçlar:**
- **Galata Kulesi** ile ilgili sonuçlar çıkıyor
- Konum: İstanbul, Türkiye
- Arka plandaki anıt belirlenmeye çalışılıyor

---

### Adım 3: Anıtı Belirleme

Fotoğraftaki bayraklar ve konum bilgisinden yola çıkarak:

- Türk bayrağı + Güney Kore/Japon bayrağı
- Galata Kulesi yakınında
- **Sonuç:** İpek Yolu Dostluk Anıtı

**Doğrulama:**
```
Anıt: İpek Yolu Dostluk Anıtı
Konum: Galata Kulesi, Beyoğlu, İstanbul
```

---

### Adım 4: Google Maps'te Konum Tespiti

```bash
# Google Maps'te Galata Kulesi'ne git
# https://www.google.com/maps/place/Galata+Kulesi
```

**Aksiyon:**
1. Galata Kulesi'ne zoom yap
2. Street View'e geç
3. Etrafta gezinmeye başla
4. İpek Yolu Dostluk Anıtı'nı ara

---

### Adım 5: Fotoğrafın Çekildiği Noktayı Bulma

Galata Kulesi çevresinde Street View ile gezinirken, verilen fotoğrafın tam olarak nerede çekildiğini bul:

**Bulunan Konum:**
```
[📍 Google Maps - Fotoğraf Parçasındaki Konum](https://www.google.com/maps/@41.0257756,28.9741534,3a,22.3y,21.09h,85.06t/data=!3m7!1e1!3m5!1s42Nt3aKkI7I7WWYvR035Ow!2e0!6shttps:%2F%2Fstreetviewpixels-pa.googleapis.com%2Fv1%2Fthumbnail%3Fcb_client%3Dmaps_sv.tactile%26w%3D900%26h%3D600%26pitch%3D4.944906872361415%26panoid%3D42Nt3aKkI7I7WWYvR035Ow%26yaw%3D21.092777331830316!7i16384!8i8192?entry=ttu&g_ep=EgoyMDI2MDIwNC4wIKXMDSoASAFQAw%3D%3D)
```

**Gözlem:** Bu noktada İpek Yolu Dostluk Anıtı ve bayraklar görünüyor

---

### Adım 6: Sıradaki Kişileri Arama

**Challenge ipuçları:**
- "Beyaz şapkalı bir hacker"
- "Sıra beklemiş"
- Flag: "Hacker'ın üzerindeki yazı"

**Strateji:** Aynı tarihli Street View görüntülerinde Galata Kulesi **giriş kuyruğunu** kontrol et

---

### Adım 7: Galata Kulesi Sırasını İnceleme

Google Maps'te aynı tarihteki Street View görüntülerinde Galata Kulesi giriş sırasına bak:

**Bulunan Konum:**
```
[📍 Google Maps - Aranan Hackerın Konumu]https://www.google.com/maps/place/Galata+Kulesi/@41.0255695,28.9743216,3a,37y,276.8h,86.54t/data=!3m7!1e1!3m5!1s8sWAOFSB8SUvAp0BCeMEBA!2e0!6shttps:%2F%2Fstreetviewpixels-pa.googleapis.com%2Fv1%2Fthumbnail%3Fcb_client%3Dmaps_sv.tactile%26w%3D900%26h%3D600%26pitch%3D3.4559854587262606%26panoid%3D8sWAOFSB8SUvAp0BCeMEBA%26yaw%3D276.8019604273197!7i16384!8i8192
```

---

### Adım 8: Beyaz Şapkalı Hacker'ı Bulma

**Sırada görülen kişi:**
- **Beyaz kapşonlu** ✓
- **Sırada bekliyor** ✓
- Sırtında **"RICKandMORTY"** yazıyor ✓

**Tüm kriterler uyuyor!**

---

### Adım 9: Flag'i Oluşturma

```
Hacker'ın üzerindeki yazı: RICKandMORTY
Flag formatı: Flag{hackerınUzerindekiYazi}
```

---

## 🚩 Flag

```
Flag{RICKandMORTY}
```

---

## 🛠️ Kullanılan Araçlar

- **Google Lens** - Görsel tanıma
- **Google Maps** - Konum tespiti
- **Street View** - Detaylı görüntü analizi
- **Google Search** - Anıt araştırması

---

## 📊 Çözüm Akış Şeması

```
🎯 Hacker Challenge
       ↓
📸 Fotoğrafı incele
   → Anıt + 2 bayrak (TR + KR/JP)
       ↓
🔍 Google Lens araması
   → Galata Kulesi sonuçları
       ↓
🗺️ Konum tespiti
   → İpek Yolu Dostluk Anıtı
   → Galata Kulesi, İstanbul
       ↓
📍 Google Maps'te gezinme
   → Street View aktif
   → Anıt konumunu bul
       ↓
🎯 Exact location bulma
   → Fotoğrafın çekildiği nokta
   → Koordinat: 41.0257756, 28.9741534
       ↓
🕐 Tarih bazlı arama
   → Aynı tarihteki Street View'ler
   → Galata Kulesi giriş sırası
       ↓
👤 Kişi analizi
   → Beyaz kapşonlu ✓
   → Sırada bekliyor ✓
   → Sırtında "RICKandMORTY" ✓
       ↓
🚩 Flag{RICKandMORTY}
```

---

## EKSTRA

**Galata Kulesi yorumlarına bakıyoruz ve şu ayarları yapıyoruz**

### Yorumlarda arayın
  **Flag{}**

### En yeni

**En aşağı iniyoruz ve şu yorumu buluyoruz**

```
Yorumu Atan: siber güvenlik

Yorum Tarihi 5 yıl önce

Yorum: Barış Varşol yarışmada trol flag ile yarışmayı baltaladığın için ananı ve 7 sülaleni saygı ile anıyoruz merak etme sen umarım bu yorumu görürsün...

Flag{1_am_4l13n_in_ISTANBUL} ile ...
```
