# 🔐 Motivasyon Flagi - Crypto Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Crypto-Challenge-darkblue?style=for-the-badge&logo=lock" alt="Crypto">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Caesar%20Cipher-purple?style=for-the-badge&logo=key" alt="Caesar">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Crypto  
**Seviye:** Kolay  
**Puan:** 200
**Açıklama:** Son gün soru çözmeye çalışanlara özel bir flagim var. Motivasyon olsun :)

**Şifreli Metin:** `Synt{GeraXnyxvlbeSyntyrevNynyvz}`

---

## 🔍 Analiz

İlk bakışta şifreli metnin `Synt{...}` formatında olduğunu görüyoruz. Bu `Flag{...}` formatına benziyor. Harf sayısı değişmemiş ve özel karakterler korunmuş, bu da **Caesar Cipher** olduğunu gösteriyor.

---

## ✅ Çözüm Adımları

### 🌐 **1. dCode Sitesine Git**

**Site:** https://www.dcode.fr/caesar-cipher

### 🔓 **2. Decode İşlemi**

1. **Caesar Cipher Decoder** sayfasını aç
2. Şifreli metni gir: `Synt{GeraXnyxvlbeSyntyrevNynyvz}`
3. **Test all possible shifts** butonuna tıkla
4. İlk çıkan sonuçta flag'i al: `Flag{TrenKalkiyorFlagleriAlalim}`

**Sonuç:**
```
Flag{TrenKalkiyorFlagleriAlalim}
```

---

## 🚩 **FLAG**

```
Flag{TrenKalkiyorFlagleriAlalim}
```

---

## 🛠️ **Kullanılan Araçlar**

**Kullanılan Site:**
- 🌐 **dCode:** https://www.dcode.fr/caesar-cipher

---

## 💭 **Çözüm Akış Şeması**

```
🔐 Motivasyon Flagi
        ↓
🔍 Analiz: Caesar Cipher (ROT13)
        ↓
🌐 dCode'a git ve decode et
        ↓
🚩 Flag{TrenKalkiyorFlagleriAlalim}
```
