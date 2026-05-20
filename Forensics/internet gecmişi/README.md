# 🎯 internet geçmişi - Forensics Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Forensics-Challenge-darkblue?style=for-the-badge" alt="Digital Forensics">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Memory%20Forensics-purple?style=for-the-badge" alt="Memory Forensics">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Forensics  
**Seviye:** Orta  
**Puan:** 500

**Challenge Dosyası:** [📥 Google Drive - Remember.raw](https://drive.google.com/file/d/1GXLpI45Tqok9KEj-SNqQ-f0dMOi8_ls2/view?usp=drivesdk)

---

## 🔍 Analiz

**Hedef:** RAM dump içindeki internet geçmişinden flag bilgisini tespit etmek

**Strateji:** `strings` komutu ile memory dump'ı tarayarak URL ve arama sorgularını çıkarmak

**İpuçları:**
- 🎯 Memory dump'larda browser geçmişi RAM'de açık metin olarak kalabilir
- 🔍 Bing arama sonuçları `q=` parametresiyle URL'lerde saklanır
- 💡 Flag değeri URL-encode veya Base64 ile gizlenmiş olabilir
- 🕵️ `strings` + `grep` kombinasyonu Volatility kurmaya gerek kalmadan işe yarar

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Flag Formatını Doğrudan Ara**
```bash
strings ~/Desktop/Remember.raw | grep -iE "CTF\{|FLAG\{|flag\{|ctf\{"
```

**Analiz:**
- ❌ Sonuç yok → Flag doğrudan metin olarak saklanmamış
- 💡 Encode edilmiş veya URL içine gömülmüş olabilir

---

### 🌐 **Adım 2: Bing Arama Geçmişini Tara**
```bash
strings ~/Desktop/Remember.raw | grep -iE "bing\.com/search\?q="
```

**Çıktı:**
```
http://www.bing.com/search?q={searchTerms}&FORM=IE8SRC
http://www.bing.com/search?q={searchTerms}&FORM=IE8SRC
```

**Analiz:**
- ❌ Sadece tarayıcı template'i mevcut, gerçek arama kaydı yok
- 💡 Video araması yapılmış olabilir, farklı endpoint denenebilir

---

### 🎬 **Adım 3: Tüm q= Parametrelerini Tara**
```bash
strings ~/Desktop/Remember.raw | grep -iE "q=\w+" | grep -v "searchTerms"
```

**Önemli Çıktılar:**
```
https://www.bing.com/videos/search?q=ZmxhZ3tLZWVwX3VzaW5nX3ZvbGF0aWxpdHl9&FORM=HDRSC3
Location: https://www.bing.com/videos/search?q=ZmxhZ3tLZWVwX3VzaW5nX3ZvbGF0aWxpdHl9&FORM=HDRSC3
ferer: https://www.bing.com/videos/search?q=ZmxhZ3tLZWVwX3VzaW5nX3ZvbGF0aWxpdHl9&FORM=HDRSC3
```

**Analiz:**
- ✅ `q=ZmxhZ3tLZWVwX3VzaW5nX3ZvbGF0aWxpdHl9` → Tekrar eden şüpheli parametre
- 🚨 Standart bir arama terimi değil → **Base64 encoded!**
- 💡 Birden fazla HTTP kaydında (Location, Referer) görünüyor → kesinlikle arama yapılmış

---

### 🔓 **Adım 4: Base64 Decode Et**
```bash
echo "ZmxhZ3tLZWVwX3VzaW5nX3ZvbGF0aWxpdHl9" | base64 -d
```

**Çıktı:**
```
flag{Keep_using_volatility}
```

**Analiz:**
- ✅ Base64 decode başarılı → Flag açık metin olarak elde edildi
- 🎯 Kullanıcı Bing'de Base64 kodlanmış bir sorgu aramış, bu veri RAM'de kalmış
- 💡 `strings` + `grep` kombinasyonu Volatility kurmaya gerek kalmadan yeterliydi

---

## 🚩 **FLAG**
```
flag{Keep_using_volatility}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔤<br><b>strings</b><br><sub>Binary string extraction</sub></td>
<td align="center">🔍<br><b>grep</b><br><sub>Pattern matching</sub></td>
<td align="center">🔓<br><b>base64</b><br><sub>Decode aracı</sub></td>
<td align="center">🧠<br><b>RAM Dump</b><br><sub>Remember.raw</sub></td>
<td align="center">🌐<br><b>Bing Video Search</b><br><sub>Arama geçmişi kaynağı</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**
```
🎯 internet geçmişi Challenge
       ↓
🔎 Flag formatını doğrudan ara (CTF{, flag{ vs.)
   → Sonuç yok → encode edilmiş olabilir
       ↓
🌐 Bing arama geçmişini tara (q= parametresi)
   → Sadece browser template'i → gerçek sorgu yok
       ↓
🎬 Tüm q= parametrelerini tara
   → q=ZmxhZ3tLZWVwX3VzaW5nX3ZvbGF0aWxpdHl9 tespit edildi
   → Bing video search URL'lerinde tekrar ediyor → şüpheli!
       ↓
🔓 Base64 decode et
   → flag{Keep_using_volatility}
       ↓
🚩 flag{Keep_using_volatility}
```
