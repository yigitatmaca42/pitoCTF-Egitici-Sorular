# 🎯 :) :) - Crypto Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Crypto-Challenge-darkblue?style=for-the-badge" alt="Crypto">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Emoji Encoding-purple?style=for-the-badge" alt="Encoding">
</p>

---

## 📋 Soru Bilgileri
  
**Kategori:** Crypto  
**Seviye:** Kolay  
**Puan:** 500

**Challenge:** 
```
😫😫🙆😳👌😵👒😶🙊🙉👳👘👨👹👥😱👬👪🙁😲👌😵👔👓👤🙃😵👱👩👨👤👳😶👗👵🙊👫👘👤🙍🙉😫👖👢😵😵🙉😶👺👳👦🙍👣🙅🙇👥🙋😷🙉👴😸👰😸👖👓👢👔👒😯👮😹😷😯👵👪👧😽😽
```

---

## 🔍 Analiz

**Hedef:** Emoji encoding'i çözmek

**Strateji:** Emoji pattern analizi → TxtMoji tespiti → Decode → Flag

**İpuçları:**
- 🎯 Sadece emoji'lerden oluşan mesaj
- 🔍 "text emoji crypto" araması
- 💡 txtmoji.com sitesi

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Emoji Encoding Araştırması**

Google'da **"text emoji crypto"** araması yapıldığında txtmoji.com sitesi çıkıyor:
```
https://txtmoji.com/
```

**Analiz:**
- ✅ TxtMoji Emoji Encryption
- ✅ Şifreli emoji mesajları
- 💡 Şifre ile decode edilir

---

### 🌐 **Adım 2: txtmoji.com Sitesinde Decode**

**Site:** https://txtmoji.com/

1. Emoji string'i decode kutusuna yapıştır:
```
😫😫🙆😳👌😵👒😶🙊🙉👳👘👨👹👥😱👬👪🙁😲👌😵👔👓👤🙃😵👱👩👨👤👳😶👗👵🙊👫👘👤🙍🙉😫👖👢😵😵🙉😶👺👳👦🙍👣🙅🙇👥🙋😷🙉👴😸👰😸👖👓👢👔👒😯👮😹😷😯👵👪👧😽😽
```

2. Şifre olarak **"1234"** gir

3. **"Decode"** butonuna tıkla

4. **Sonuç:**
```
Flag{neolduDcodefrdenCikmadiMi?}
```

---

## 🚩 **FLAG**
```
Flag{neolduDcodefrdenCikmadiMi?}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🌐<br><b>txtmoji.com</b><br><sub>Online decoder</sub></td>
<td align="center">🔐<br><b>Password: 1234</b><br><sub>Decrypt key</sub></td>
<td align="center">😀<br><b>Emoji Crypto</b><br><sub>Emoji encryption</sub></td>
</tr>
</table>

**Kullanılan Siteler:**
- https://txtmoji.com/

---

## 💭 **Çözüm Akış Şeması**
```
🎯 :) :) Challenge
       ↓
🎨 Emoji string analiz
   → Sadece emoji'lerden oluşuyor
   → Çok sayıda emoji var
       ↓
🔍 Google: "text emoji crypto"
   → txtmoji.com sitesi bulundu
   → TxtMoji Emoji Encryption
       ↓
🌐 txtmoji.com'da decode
   → Emoji string yapıştır
   → Şifre: 1234
   → Decode butonuna tıkla
       ↓
📝 Sonuç alındı
   → Flag{neolduDcodefrdenCikmadiMi?}
       ↓
🚩 Flag{neolduDcodefrdenCikmadiMi?}
```
