# 🔐 Mesaj - Crypto Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Crypto-Challenge-darkblue?style=for-the-badge&logo=lock" alt="Crypto">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Baudot_Code-purple?style=for-the-badge&logo=binary" alt="Baudot">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Crypto  
**Seviye:** Kolay  
**Puan:** 200

**Açıklama:** Cihan hocanın size bir mesajı var:

FLAG{ 00110 10101 00110 10000 00011 10000 00110 10010 10010 00001 
01010 10110 11000 01010 10000 00101 10011 00110 11010 11010 00001 
01010 10010 00011 11001 00101 11000 01100 00101 00001 10010 00011 
11100 00101 11000 01010 00111 00101 00111 01100 00011 00101 00111 
00101 10010 00111 10110 00011 01010 00011 01100 10000 00001 10001 
10010 00001 01010 10010 00001 01110 11000 01100 01101 00110 11010 
11010 11000 01100 01001 00001 01010 }

Flag içinde Selam sorusuna ipucu var.


---

## 🔍 Analiz

**Hedef:** Binary formatındaki şifreli mesajı decode ederek flag elde etmek

**Strateji:** Binary analizi → Cipher tespiti → Decode → Flag

**İpuçları:**
- 🔢 5 bitlik binary gruplar → Normal binary (8 bit) değil
- 💡 "Selam sorusuna ipucu var" → Decode edince başka bir challenge için hint çıkacak
- 🎯 Özel bir encoding sistemi kullanılmış

---

## ✅ Çözüm Adımları

### 🔍 **Adım 1: Binary Verisini İnceleme**

**Flag içeriğini temizliyoruz:**

```
00110 10101 00110 10000 00011 10000 00110 10010 10010 00001 01010 10110 11000 01010 10000 00101 10011 00110 11010 11010 00001 01010 10010 00011 11001 00101 11000 01100 00101 00001 10010 00011 11100 00101 11000 01010 00111 00101 00111 01100 00011 00101 00111 00101 10010 00111 10110 00011 01010 00011 01100 10000 00001 10001 10010 00001 01010 10010 00001 01110 11000 01100 01101 00110 11010 11010 11000 01100 01001 00001 01010
```

**Gözlemler:**
- 📊 Toplam 70 adet binary grup var
- 🔢 Her grup **5 bit** uzunluğunda (örn: `00110`, `10101`)
- 🤔 Normal ASCII binary kodlaması 8 bit kullanır
- 💡 5 bitlik özel bir encoding sistemi olmalı!

**İlk 10 grubu analiz edelim:**
```
00110 → 6  (decimal)
10101 → 21 (decimal)
00110 → 6  (decimal)
10000 → 16 (decimal)
00011 → 3  (decimal)
```

> 💭 5 bit binary, 0-31 arası sayıları temsil edebilir. Bu özel bir cipher olmalı!

---

### 🔎 **Adım 2: Cipher Identifier ile Analiz**

**dCode.fr Cipher Identifier kullanıyoruz:**

```
URL: https://www.dcode.fr/cipher-identifier
```

**Binary veriyi yapıştırıyoruz:**
```
00110 10101 00110 10000 00011 10000 00110 10010 10010 00001 01010 10110 11000 01010 10000 00101 10011 00110 11010 11010 00001 01010 10010 00011 11001 00101 11000 01100 00101 00001 10010 00011 11100 00101 11000 01010 00111 00101 00111 01100 00011 00101 00111 00101 10010 00111 10110 00011 01010 00011 01100 10000 00001 10001 10010 00001 01010 10010 00001 01110 11000 01100 01101 00110 11010 11010 11000 01100 01001 00001 01010
```

**Cipher Identifier sonucu:**

```
🎯 Detected: Baudot Code (98% confidence)
```

**Baudot Code nedir?**
- 📡 Telgraf sistemlerinde kullanılan 5-bitlik encoding
- 🔢 Her karakter 5 bit ile temsil edilir (0-31 arası)
- 📜 1870'lerde Émile Baudot tarafından geliştirildi
- 🌐 Modern teleprinter ve teletypewriter'ların temeli

> ✅ **Baudot Code tespit edildi!** Şimdi decode edelim.

---

### 🔓 **Adım 3: Baudot Code Decoder**

**dCode.fr Baudot Code Decoder'a gidiyoruz:**

```
URL: https://www.dcode.fr/baudot-code
```

**Binary veriyi "Decode" kısmına yapıştırıyoruz:**

```
00110 10101 00110 10000 00011 10000 00110 10010 10010 00001 01010 10110 11000 01010 10000 00101 10011 00110 11010 11010 00001 01010 10010 00011 11001 00101 11000 01100 00101 00001 10010 00011 11100 00101 11000 01010 00111 00101 00111 01100 00011 00101 00111 00101 10010 00111 10110 00011 01010 00011 01100 10000 00001 10001 10010 00001 01010 10010 00001 01110 11000 01100 01101 00110 11010 11010 11000 01100 01001 00001 01010
```

**"DECODE" butonuna tıklıyoruz...**

**🎉 Çıktı:**

```
IYITATILLERPORTSWIGGERLABSONSELAMSORUSUNASUSLUPARANTEZLERLECONFIGGONDER
```

**Decode başarılı!** ✅

---

## 🏁 **FLAG**

```
FLAG{IYITATILLERPORTSWIGGERLABSONSELAMSORUSUNASUSLUPARANTEZLERLECONFIGGONDER}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🌐<br><b>dCode.fr</b><br><sub>Cipher identifier</sub></td>
<td align="center">🔓<br><b>Baudot Decoder</b><br><sub>Binary to text</sub></td>
</tr>
</table>

**Kullanılan Web Araçları:**
```
https://www.dcode.fr/cipher-identifier
https://www.dcode.fr/baudot-code
```

---

## 💭 **Çözüm Akışı**

```
🔐 "Mesaj" Crypto Challenge
            ↓
📊 Flag içeriği analizi
   → 70 adet 5-bitlik binary grup
   → Normal binary 8 bit olur!
            ↓
🔎 dCode Cipher Identifier
   → https://www.dcode.fr/cipher-identifier
   → Binary veriyi yapıştır
            ↓
🎯 Baudot Code tespit edildi!
   → %98 confidence
   → 5-bit telgraf encoding sistemi
            ↓
🔓 Baudot Code Decoder
   → https://www.dcode.fr/baudot-code
   → Binary → Text decode
            ↓
📝 Decode sonucu:
   IYITATILLERPORTSWIGGERLABSONSELAMSORUSUNASUSLUPARANTEZLERLECONFIGGONDER
            ↓
🧩 Mesajı ayırma:
   → IYI TATILLER
   → PORT SWIGGER LAB
   → SON SELAM SORUSUNA
   → SUSLU PARANTEZLERLE
   → CONFIG GONDER
            ↓
💡 Selam challenge'ı için ipucu!
   → {CONFIG} veya {SECRET_KEY} payload kullanılacak
   → Format string / Template injection olabilir
            ↓
🏁 FLAG{IYITATILLERPORTSWIGGERLABSONSELAMSORUSUNASUSLUPARANTEZLERLECONFIGGONDER}
```
