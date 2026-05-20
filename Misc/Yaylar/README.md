# 🎯 Yaylar - Misc Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Crypto-Challenge-darkblue?style=for-the-badge" alt="Misc">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Crypto-purple?style=for-the-badge" alt="Crypto">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Crypto  
**Seviye:** Orta  
**Puan:** 500

**Challenge Dosyası:** [📥 Google Drive - Yaylar](https://drive.google.com/file/d/10Fqdk0VW1Tkeha_Wirojp3UlHh4rU4Ow/view?usp=drivesdk)

---

## 🔍 Analiz

**Hedef:** Görseldeki sembolik yazıyı çözümleyip içindeki iç içe şifreli mesajdan flag'e ulaşmak

**Strateji:** Dorabella Cipher → Mesaj Ayrıştırma → Vigenere Cipher → Flag

**İpuçları:**
- 🎯 Görsel **Dorabella Cipher** alfabesiyle yazılmış
- 🔍 Decode edilen metin hem **ciphertext** hem **key** içeriyor
- 💡 `BULAMAZSINKI` → Vigenere key
- 🕌 Ramazan temalı flag (İftar referansı)
- 🔗 İç içe şifreleme: klasik sembolik şifre + Vigenere

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Şifre Tipini Tanı**

Görseldeki sembollere bakılır — harflere benzeyen ama Latin alfabesine uymayan kıvrımlı semboller.

**Analiz:**
- ✅ Semboller Dorabella Cipher alfabesiyle eşleşiyor
- 💡 Dorabella Cipher: Edward Elgar'ın 1897'de yazdığı, hâlâ tam çözülemeyen tarihi bir şifre
- 🔗 Çözüm aracı: [dcode.fr/dorabella-cipher](https://www.dcode.fr/dorabella-cipher)

---

### 📄 **Adım 2: Dorabella Cipher'ı Decode Et**

Görsel 5 satır sembol içeriyor. dcode.fr aracıyla her satır decode edilir.

**Çıktı:**
```
GFLGUFSS
ZNUIDMLA
FKZDLUKE
YBULAMAZ
SINKI
```

**Birleştirilmiş metin:**
```
GFLGUFSSZNUIDMLAFKZDLUKEYBULAMAZSINKI
```

**Analiz:**
- ✅ 5 satır başarıyla decode edildi
- 💡 Metin anlamsız görünüyor → başka bir şifreleme katmanı var

---

### 🔑 **Adım 3: Mesajı Ayrıştır**

Birleşik metne dikkatli bakılır:

```
GFLGUFSSZNUIDMLAFKZDLU  →  Ciphertext
KEY                     →  Ayraç (KEY kelimesi geçiyor!)
BULAMAZSINKI            →  Vigenere Key
```

**Ayrıştırılmış hali:**
```
Ciphertext : GFLGUFSSZNUIDMLAFKZDLU
Key        : BULAMAZSINKI
```

**Analiz:**
- ✅ `KEY` kelimesi tam ortada ayraç görevi görüyor
- 💡 Bu bir **Vigenere Cipher** — ciphertext ve key gömülü olarak verilmiş

---

### 🔓 **Adım 4: Vigenere Cipher'ı Decode Et**

[dcode.fr/vigenere-cipher](https://www.dcode.fr/vigenere-cipher) aracı kullanılır.

**Girdi:**
```
Ciphertext : GFLGUFSSZNUIDMLAFKZDLU
Key        : BULAMAZSINKI
```

**Çıktı:**
```
FLAGIFTARAKACSAATKALDH
```

**Analiz:**
- ✅ Decode başarılı
- 🚨 Son harf `H` beklenmedik → `I` olması gerekiyor (encoding hatası)
- 💡 `FLAG` prefix'i görünüyor → doğru yoldayız

---

### 🚩 **Adım 5: Flag'i Oluştur**

Son karakterdeki encoding hatasını düzelt (`H` → `I`) ve parantezleri ekle:

```
FLAGIFTARAKACSAATKALDI
→ FLAG{IFTARAKACSAATKALDI}
```

**Analiz:**
- ✅ Flag geçerli formata getirildi
- 🕌 "İftar vakası aat kaldı" → Ramazan temalı mesaj

---

## 🚩 **FLAG**
```
FLAG{IFTARAKACSAATKALDI}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔮<br><b>dcode.fr</b><br><sub>Dorabella Cipher</sub></td>
<td align="center">🔑<br><b>dcode.fr</b><br><sub>Vigenere Cipher</sub></td>
<td align="center">👁️<br><b>Manuel Analiz</b><br><sub>Mesaj ayrıştırma</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**
```
🎯 Yaylar Challenge
       ↓
🔎 Görseldeki sembolleri tanı
   → Kıvrımlı semboller → Dorabella Cipher
       ↓
📄 Dorabella Cipher decode et
   → dcode.fr/dorabella-cipher
   → GFLGUFSSZNUIDMLAFKZDLUKEYBULAMAZSINKI
       ↓
🔑 Mesajı ayrıştır
   → GFLGUFSSZNUIDMLAFKZDLU → Ciphertext
   → KEY → Ayraç
   → BULAMAZSINKI → Vigenere Key
       ↓
🔓 Vigenere Cipher decode et
   → dcode.fr/vigenere-cipher
   → FLAGIFTARAKACSAATKALDH
       ↓
✏️ Son karakter düzelt
   → H → I (encoding hatası)
       ↓
🚩 Flag oluştur
   → FLAG{IFTARAKACSAATKALDI}
```
