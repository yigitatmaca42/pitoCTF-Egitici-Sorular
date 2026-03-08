# 📡 Mac - OSINT Challenge

<p align="center">
  <img src="https://img.shields.io/badge/OSINT-Challenge-darkblue?style=for-the-badge&logo=search" alt="OSINT">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-WiFi%20Geolocation-purple?style=for-the-badge&logo=wifi" alt="WiFi Geolocation">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** OSINT
**Seviye:** Orta  
**Puan:** 300  
**Açıklama:** Uzun zamandır peşinde olduğumuz siyah şapkalı hacker mac adresini sızdırmış. Kaçmadan konumunu bulmamız lazım.

**Mac Bilgisi:** `b4:de:31:7b:99:cf`

**Flag Formatı:** `Flag{CaddeAdi_EnYakinSokakAdi}`

**Firstblood Bonus:** +50 puan

---

## 🔍 Analiz

**Hedef:** MAC adresi üzerinden WiFi konumunu bulmak

**Strateji:** WiGLE.net → MAC sorgulama → Konum tespiti → Sokak isimleri

---

## ✅ Çözüm Adımları

### 🌐 **1. WiGLE.net'e Giriş**

**Site:** https://wigle.net/

WiGLE.net, dünya çapındaki WiFi ağlarının MAC adreslerini ve konumlarını içeren bir veritabanıdır.

---

### 🔎 **2. Advanced Search Kullanımı**

**Adımlar:**

1. WiGLE.net ana sayfasına gidiyoruz
2. **"View"** menüsünden **"Advanced Search"** seçiyoruz
3. **"BSSID/MAC"** alanına MAC adresini yazıyoruz:

```
b4:de:31:7b:99:cf
```

4. **"Query"** butonuna tıklıyoruz

---

### 📊 **3. Arama Sonuçları**

**Bulunan Network Bilgileri:**

| Bilgi | Değer |
|-------|-------|
| **Net ID** | B4:DE:31:7B:99:CF |
| **SSID** | Turk Telekom WiFi |
| **Type** | Infra |
| **First Seen** | 2020-06-25T19:00:00.000Z |
| **Most Recently** | 2020-06-27T01:00:00.000Z |
| **Channel** | 48 |
| **Est. Lat** | 41.0355679 |
| **Est. Long** | 28.98241043 |

> 🎯 **Koordinatlar İstanbul'u gösteriyor!**

---

### 🗺️ **4. Interactive Map'i Açma**

**"Click for interactive map"** linkine tıklıyoruz.

**Haritada Görünen:**

- 📍 Konum: **İstanbul, Türkiye**
- 🗺️ Tam koordinat detayı görünüyor

---

### 📍 **5. Sokak İsimlerini Bulma**

Haritadan yakınlaştırarak sokak isimlerini kontrol ediyoruz:

**Bulunan:**

- **Cadde:** İstiklal Caddesi
- **Yakındaki Sokaklar:**
  - Bekar Sokak
  - Küçük Parmakkapı Sokak

---

### 🚩 **6. Flag Formatını Oluşturma**

**Flag Kuralları (Challenge'dan):**

- ✅ Tüm harfler **küçük** olmalı
- ✅ İngilizce karakterler olmalı (Türkçe karakter yok)
- ✅ Format: `Flag{CaddeAdi_EnYakinSokakAdi}`

**Deneme 1:**
```
Flag{İstiklal_Bekar}
```
❌ Türkçe karakter var

**Deneme 2:**
```
Flag{İstiklal_KüçükParmakKapı}
```
❌ Türkçe karakter var

**Deneme 3:**
```
Flag{istiklal_kucukparmakkapi}
```
❌ Yanlış

**Deneme 4:**
```
Flag{istiklal_bekar}
```
✅ **DOĞRU!**

---

## 🚩 **FLAG**

```
Flag{istiklal_bekar}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">📡<br><b>WiGLE.net</b><br><sub>WiFi geolocation</sub></td>
<td align="center">🗺️<br><b>Interactive Map</b><br><sub>Konum detayı</sub></td>
<td align="center">🌍<br><b>OpenStreetMap</b><br><sub>Sokak isimleri</sub></td>
</tr>
</table>

**Kullanılan Siteler:**
- 📡 **WiGLE.net:** https://wigle.net/
- 🗺️ **WiGLE Map:** https://wigle.net/map
- 🎯 **Challenge Site:** https://ctf.pitoctf.online/

---

## 💭 **Çözüm Akış Şeması**

```
📡 OSINT Challenge: "Mac"
                    ↓
        📝 MAC Adresi: b4:de:31:7b:99:cf
                    ↓
        🌐 WiGLE.net Advanced Search
                    ↓
        🔎 BSSID/MAC alanına MAC yaz → Query
                    ↓
        📊 Sonuçlar: Turk Telekom WiFi
                    ↓
        📍 Koordinat: 41.0355679, 28.98241043
                    ↓
        🗺️ Click for interactive map
                    ↓
        🇹🇷 Konum: İstanbul, Türkiye
                    ↓
        📍 Zoom yaparak sokak isimlerine bak
                    ↓
        🛣️ İstiklal Caddesi + Bekar Sokak
                    ↓
        📝 Flag formatı: küçük harf + ingilizce karakter
                    ↓
        🚩 Flag{istiklal_bekar}
```

---

## 🔐 **Ekstra Bilgi**

**Not:** Bu challenge aslında **SiberVatan** yarışmasından alınmış. Orijinal versiyonda sadece **cadde adı** isteniyordu.

**SiberVatan Versiyonu:**
- Sadece cadde adı yeterliydi
- Flag formatı farklı olabilir

**pitoCTF Versiyonu:**
- Cadde + En yakın sokak
- Flag formatı: `Flag{cadde_sokak}`
