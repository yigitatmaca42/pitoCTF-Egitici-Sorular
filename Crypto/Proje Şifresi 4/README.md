# 🔐 Proje Sifresi4 - Crypto Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Crypto-Challenge-darkblue?style=for-the-badge&logo=lock" alt="Crypto">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Affine%20Cipher-purple?style=for-the-badge&logo=key" alt="Affine">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Crypto  
**Seviye:** Orta  
**Puan:** 400  
**Açıklama:** Bulduğun şifre yine işe yaramadı? Kendi kendine bir şifreyi mail olarak atmış. Acaba bu mu?

**Challenge Dosyası:** [📥 GitHub - mail.jpeg](https://github.com/yigitatmaca42/pitoCTF-Egitici-Sorular/blob/main/Proje%20sifresi4/mail.jpeg)

**Flag Formatı:** `pitoctf{şifre}`

**İpucu:** decodefr cipher identifier

**Verilen:**
- Mail resmi içinde şifrelenmiş metin

---

## 🔍 Analiz

### Mail İçeriği

Mail resminde şu yazıyor:

```
Maildeki Şifre: "d1qa3lhOfyrablb2qbvzonknbhzf"
```

Bu şifrelenmiş bir metin. Hangi şifreleme yöntemi kullanılmış?

---

## ✅ Çözüm Adımları

### 🔍 **Adım 1: Cipher Türünü Belirleme**

İpucu "dcode.fr cipher identifier" diyor. Bu siteye gidiyoruz:

**Site:** https://www.dcode.fr/cipher-identifier

Şifreyi giriyoruz:
```
d1qa3lhOfyrablb2qbvzonknbhzf
```

**Sonuç:**

Site birkaç olasılık gösteriyor, ama **3. sırada** en yüksek ihtimalle **"Affine Cipher"** çıkıyor!

---

### 🔓 **Adım 2: Affine Cipher ile Decode**

Affine Cipher sayfasına gidiyoruz:

**Site:** https://www.dcode.fr/affine-cipher

Şifreyi yapıştırıp **"DECRYPT"** butonuna tıklıyoruz.

**Çıktı:**
```
s1fr3mcNkzorama2fayinedeacik
```

---

### ⚠️ **Adım 3: Hata Fark Etme**

Çıktıda bir şey garip:
```
s1fr3mcNkzorama2fayinedeacik
       ↑
     "N" harfi yanlış görünüyor
```

"çok" kelimesi olmalı ama "cNk" çıkmış. Bu harf şunlardan biri olabilir:
- `cok` (küçük o)
- `cOk` (büyük O)  
- `c0k` (sıfır)

---

### ✅ **Adım 4: Doğru Şifreyi Bulma**

3 ihtimali deniyoruz:

1. `s1fr3mcokzorama2fayinedeacik` (küçük o)
2. `s1fr3mcOkzorama2fayinedeacik` (büyük O)
3. `s1fr3mc0kzorama2fayinedeacik` (sıfır 0)

Flag formatıyla denediğimizde **doğru olan:**

```
s1fr3mc0kzorama2fayinedeacik
```

(Sıfır "0" kullanılan versiyon doğru!)

---

## 🚩 **FLAG**

```
pitoctf{s1fr3mc0kzorama2fayinedeacik}
```

---

## 💻 **Kullanılan Siteler**

```
1. Cipher türünü belirleme:
   https://www.dcode.fr/cipher-identifier

2. Affine Cipher decode:
   https://www.dcode.fr/affine-cipher
```

---

## 💭 **Basit Akış Şeması**

```
📧 Mail'de şifreli metin
       ↓
d1qa3lhOfyrablb2qbvzonknbhzf
       ↓
🔍 Cipher Identifier → Affine Cipher
       ↓
🔓 Affine Cipher Decode
       ↓
s1fr3mcNkzorama2fayinedeacik
       ↓
⚠️ "N" harfi yanlış görünüyor
       ↓
✅ Üç ihtimal dene: o, O, 0
       ↓
🚩 Doğrusu: s1fr3mc0kzorama2fayinedeacik
```
