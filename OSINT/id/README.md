# 🎯 id - OSINT Challenge

<p align="center">
  <img src="https://img.shields.io/badge/OSINT-Challenge-darkblue?style=for-the-badge" alt="OSINT">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Platform_ID-purple?style=for-the-badge" alt="Platform ID">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** OSINT  
**Seviye:** Orta  
**Puan:** 500

**Challenge Açıklaması:** Elimizde bir id var. Ne olduğunu biz de anlamadık.

**Verilen ID:** `6RwS0tsR0Ibh7CktxMj3jU`

---

## 🔍 Analiz

**Hedef:** Verilen ID'nin hangi platforma ait olduğunu tespit edip, içinde gizlenmiş flag'i bulmak

**Strateji:** ID format analizi → Platform tespiti → İçerik incelemesi → Akrostiş analizi → Flag

**İpuçları:**
- 🎯 Elimizde sadece bir ID var: **`6RwS0tsR0Ibh7CktxMj3jU`**
- 🔍 **22 karakter**, büyük/küçük harf ve rakamlardan oluşuyor (Base62)
- 💡 Bu format **Spotify ID**'leriyle birebir örtüşüyor
- 🕵️ Playlist açıklamasındaki flag bir **tuzak**
- 🔑 Gerçek flag şarkı isimlerinde **akrostiş** olarak gizli

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: ID Format Analizi**

```
ID: 6RwS0tsR0Ibh7CktxMj3jU
Uzunluk: 22 karakter
İçerik: Büyük/küçük harf + rakam (Base62)
Özel karakter: Yok
```

**Bilinen 22 karakterli ID formatları:**

| Platform | Format | Uyuşuyor mu? |
|---|---|---|
| Spotify | 22 karakter Base62 | ✅ |
| YouTube | 11 karakter | ❌ |
| Discord | 18-19 rakam | ❌ |
| NanoID | 21 karakter | ❌ |

**Analiz:**
- ✅ 22 karakterlik Base62 format → Spotify ID'si
- 💡 Spotify; kullanıcı, şarkı, albüm ve playlist için bu formatı kullanır
- 🎯 Tüm Spotify URL tiplerini denemek gerekiyor

---

### 📄 **Adım 2: Spotify URL Denemeleri**

```bash
https://open.spotify.com/user/6RwS0tsR0Ibh7CktxMj3jU
https://open.spotify.com/track/6RwS0tsR0Ibh7CktxMj3jU
https://open.spotify.com/album/6RwS0tsR0Ibh7CktxMj3jU
https://open.spotify.com/playlist/6RwS0tsR0Ibh7CktxMj3jU
```

**Çıktı:**
```
/user     → Ana sayfaya yönlendirdi ❌
/track    → Hata ❌
/album    → Hata ❌
/playlist → ✅ Playlist açıldı!
```

**Analiz:**
- ✅ `playlist` URL'i çalıştı
- 🎯 **"BayrakKimde"** isimli playlist bulundu
- 💡 Sahibi: **Flag Bizd3n Sorulur**

---

### 🔨 **Adım 3: Playlist İncelemesi**

Playlist açıldığında açıklamada şu yazıyordu:

```
flag{budegil}
```

**Analiz:**
- 🚨 Bu flag **geçersiz** — tuzak!
- 💡 Playlist içinde **17 şarkı** var
- 🎯 Şarkı isimlerini daha dikkatli incelemek gerekiyor

---

### 🎯 **Adım 4: Akrostiş Analizi**

Playlist'teki şarkıların baş harfleri sırayla okunduğunda:

| # | Şarkı Adı | Sanatçı | Baş Harf |
|---|---|---|---|
| 1 | Fark Var | Ceza | **F** |
| 2 | Legend of the Eagle Bearer | The Flight, Assassin's Creed | **L** |
| 3 | Aksiyon | Toygar Işıklı | **A** |
| 4 | GELME İSTEMEM | BLOK3 | **G** |
| 5 | Shots Fired | Le Castle Vania | **S** |
| 6 | Patlat | BLOK3 | **P** |
| 7 | Oldu Olanlar - Sago K 2022 Remix | Sagopa Kajmer | **O** |
| 8 | Toroslu Yollar | Çetin İçten, Pikapatr Raks | **T** |
| 9 | İlk Kural Saygı | Killa Hakan, Ezhel, GRiNGO | **İ** |
| 10 | Freedom Fighters | Thomas Bergersen | **F** |
| 11 | Yar Bensiz | Emre Fel | **Y** |
| 12 | Merhabalar | Emre Fel | **M** |
| 13 | Olsun | Sertab Erener | **O** |
| 14 | Doyamam | Ezhel | **D** |
| 15 | UYUZ | BLOK3 | **U** |
| 16 | Onları da Anlıyorum | Sagopa Kajmer | **O** |
| 17 | napıyosun mesela ? | BLOK3 | **N** |

**Analiz:**
- ✅ Baş harfler sırayla okunduğunda: **F-L-A-G-S-P-O-T-İ-F-Y-M-O-D-U-O-N**
- 🎯 FLAG formatına soktuğumuzda: **FLAG{SPOTIFYMODUON}**
- 🚩 Flag bulundu!

---

## 🚩 **FLAG**
```
FLAG{SPOTIFYMODUON}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🎵<br><b>Spotify</b><br><sub>Platform tespiti</sub></td>
<td align="center">🌐<br><b>Tarayıcı</b><br><sub>URL deneme</sub></td>
<td align="center">👀<br><b>Manuel analiz</b><br><sub>Akrostiş okuma</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**
```
🎯 ID Challenge
       ↓
🔍 ID Format Analizi
   → 22 karakter, Base62
   → Spotify ID formatı ile eşleşiyor
       ↓
🌐 Spotify URL Denemeleri
   → /user     → Ana sayfa ❌
   → /track    → Hata ❌
   → /album    → Hata ❌
   → /playlist → ✅ Açıldı!
       ↓
🎵 Playlist Bulundu: "BayrakKimde"
   → Sahibi: Flag Bizd3n Sorulur
   → Açıklamada: flag{budegil} → TUZAK! 🚨
       ↓
📋 17 Şarkı İncelendi
   → Baş harfler sırayla okundu
   → F-L-A-G-S-P-O-T-İ-F-Y-M-O-D-U-O-N
       ↓
🚩 Flag
   → FLAG{SPOTIFYMODUON}
```
