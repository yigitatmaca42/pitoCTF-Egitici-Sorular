# 🌊 Dijitaliz - OSINT Challenge

<p align="center">
  <img src="https://img.shields.io/badge/OSINT-Challenge-darkblue?style=for-the-badge&logo=searchengineland" alt="OSINT">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Bonus-+2%20Points-success?style=for-the-badge&logo=star" alt="Bonus">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** OSINT  
**Seviye:** Orta  
**Soru No:** 70  
**Bonus:** +2 Puan  
**Açıklama:** İzleri takip et bayrağı bul.

**Flag Formatı:** `pitoctf{}`

---

## 🔍 Analiz

### Resimdeki Önemli Etkenler

Verilen fotoğrafı incelediğimizde şu ipuçlarını buluyoruz:

| İpucu | Detay |
|-------|-------|
| 🥤 **Karton Bardak** | Üzerinde **"taze"** yazısı var |
| 💧 **Su Markası** | **"Çenesuyu"** |
| 🌊 **Lokasyon** | Deniz kenarı bir mekan |

**İlk Çıkarım:**
- Karton bardak çay bardağı olduğu belli
- Büyük ihtimalle burası bir **cafe**

---

## ✅ Çözüm Adımları

### 🏙️ **1. Şehir Tespiti**

**Çenesuyu** markası **Kocaeli** şehrine ait bir marka. Aramamızı burada yapacağız.

---

### ☕ **2. Cafe Araştırması**

İnternete bardaktaki **"taze"** yazısını arattık:

**Arama Terimi:** `taze cafe kocaeli`

**Bulunan İşletme:** **Taze Kruvasan Kahve**

---

### 🗺️ **3. Google Maps İncelemesi**

**Google Maps**'te "Taze Kruvasan Kahve Kocaeli" aratıyoruz ve **deniz kenarı** mekanları inceliyoruz.

**Tespitler:**
- Kocaeli'nde deniz kenarına yakın **3 tane** Taze Kruvasan Kahve mekanı var
- Aralarından **1 tanesi** direkt denizin dibinde
- Diğer ikisi biraz daha geride

**Doğru Mekan:** Denizin hemen dibindeki şube

🗺️ **Mekan Linki:** https://maps.app.goo.gl/HnQ9UrXRJjFHrPiv8

---

### 💬 **4. Google Maps Yorumları**

Mekanın yorumlarına bakarken **"Flag BizdenSorulur"** hesabının bir fotoğraf attığını görüyoruz.

**Fotoğraf İncelemesi:**
- Fotoğrafı açıp dikkatlice baktığımızda
- Aşağıda **turuncu yazıyla** şu yazıyor:

**Fotoğraftaki Turuncu Yazı:**
```
{SıcakÇayKırmızıÇizgimiz}
```

---

## 🚩 **FLAG**

**Flag Formatı:** `pitoctf{}`

```
pitoctf{SıcakÇayKırmızıÇizgimiz}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>Google Search</b><br><sub>Cafe araştırması</sub></td>
<td align="center">🗺️<br><b>Google Maps</b><br><sub>Lokasyon tespiti</sub></td>
<td align="center">💬<br><b>Google Reviews</b><br><sub>Yorum incelemesi</sub></td>
</tr>
</table>

**Kullanılan Linkler:**
- 🗺️ **Google Maps Mekan:** https://maps.app.goo.gl/HnQ9UrXRJjFHrPiv8
- 🔍 **Arama Terimi:** "taze cafe kocaeli"

---

## 💭 **Çözüm Akış Şeması**

```
📸 Fotoğraf Analizi
            ↓
🔍 İpuçları Toplama:
   - Karton bardak: "taze"
   - Su markası: "Çenesuyu"
   - Deniz kenarı lokasyon
            ↓
🏙️ Şehir Tespiti: Çenesuyu → Kocaeli
            ↓
☕ Google Search: "taze cafe kocaeli"
            ↓
✅ İşletme Bulma: Taze Kruvasan Kahve
            ↓
🗺️ Google Maps: 3 şube bulundu
            ↓
🌊 Lokasyon Filtreleme: Denizin dibindeki şube
            ↓
💬 Yorumlar İncelemesi: "Flag BizdenSorulur" hesabı
            ↓
📸 Fotoğraf İncelemesi: Turuncu yazı tespit
            ↓
🚩 FLAG: pitoctf{SıcakÇayKırmızıÇizgimiz}
```

---
