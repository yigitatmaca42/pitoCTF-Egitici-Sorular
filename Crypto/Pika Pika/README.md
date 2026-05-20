# ⚡ Pika pika? - Crypto Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Crypto-Challenge-darkblue?style=for-the-badge&logo=lock" alt="Crypto">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Esoteric%20Language-purple?style=for-the-badge&logo=code" alt="Esoteric Language">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Crypto  
**Seviye:** Kolay  
**Puan:** 200

**Challenge Dosyası:** [📥 Google Drive - pikapika.txt](https://drive.google.com/file/d/1l-vHKe8YuGjAGzyoDvKAPTVkdl2vGYlx/view?usp=drivesdk)

---

## 🔍 Analiz

**Hedef:** Pikachu dilinde yazılmış kodu decode etmek

**Strateji:** Cipher Identifier → Pikalang → Decode

---

## ✅ Çözüm Adımları

### 📥 **1. Dosyayı İndirme**

Google Drive'dan `pikapika.txt` dosyasını indiriyoruz.

**İçerik:**

```
pi pi pi pi pi pi pi pi pi pi pika pipi pi pipi pi pi pi pipi pi pi pi pi pi pi pi pipi pi pi pi pi pi pi pi pi pi pi pichu pichu pichu pichu ka chu pipi pipi pipi pikachu pipi pi pi pi pi pi pi pi pi pikachu ka ka ka ka ka ka ka ka ka ka ka pikachu pi pi pi pi pi pi pikachu pi pi pi pi pi pi pi pi pi pi pi pi pi pi pi pi pi pi pi pi pikachu ka ka ka ka ka ka ka ka ka ka ka pikachu ka pikachu ka ka ka ka pikachu ka ka pikachu pi pi pi pi pi pi pi pi pi pi pi pikachu ka ka ka ka ka pikachu pi pikachu pi pi pi pi pi pikachu ka ka ka ka ka ka ka ka pikachu pi pikachu ka ka ka ka ka ka ka ka ka pikachu pi pi pi pi pi pi pi pi pi pi pi pi pi pikachu ka ka ka ka ka ka ka ka ka ka ka ka ka ka pikachu ka pikachu pi pikachu pi pi pikachu pi pi pi pi pi pi pi pi pikachu pi pi pi pikachu ka ka ka ka ka ka ka ka ka ka ka ka ka ka pikachu pi pi pi pi pi pi pi pi pi pi pi pi pi pi pi pi pi pikachu ka ka ka ka ka ka ka pikachu pi pi pi pi pi pi pi pikachu pi pi pi pi pi pikachu ka ka ka ka ka ka ka ka ka ka ka ka ka pikachu pi pi pi pi pi pi pi pi pikachu pi pi pi pi pi pi pi pi pikachu
```

> 💡 "pi", "pika", "pikachu", "pipi", "pichu", "ka", "chu" kelimeleri tekrar ediyor!

---

### 🔍 **2. Cipher Identifier Kullanımı**

**Site:** https://www.dcode.fr/cipher-identifier

**Adımlar:**

1. Metni Cipher Identifier'a yapıştırıyoruz
2. **Analyze** butonuna tıklıyoruz

**Sonuç:**

```
Pikalang Language - High probability
```

> 🎯 **Pikalang** esoteric programming dili!

---

### ⚡ **3. Pikalang Decode**

**Site:** https://www.dcode.fr/pikalang-language

**Adımlar:**

1. Pikalang decoder sayfasına gidiyoruz
2. Metni **"Pikalang Code"** kutusuna yapıştırıyoruz
3. **"Decrypt/Translate"** butonuna tıklıyoruz

**Decode Sonucu:**

```
Flag{pokitopumneredegordunuzmu}
```

---

## 🚩 **FLAG**

```
Flag{pokitopumneredegordunuzmu}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>Cipher Identifier</b><br><sub>Şifre tanımlama</sub></td>
<td align="center">⚡<br><b>Pikalang Decoder</b><br><sub>Decode aracı</sub></td>
</tr>
</table>

**Kullanılan Siteler:**
- 🔍 **Cipher Identifier:** https://www.dcode.fr/cipher-identifier
- ⚡ **Pikalang Decoder:** https://www.dcode.fr/pikalang-language
- 📥 **Challenge File:** Google Drive

---

## 💭 **Çözüm Akış Şeması**

```
⚡ Crypto Challenge: "Pika pika?"
                    ↓
        📥 pikapika.txt dosyasını indir
                    ↓
        📝 İçeriği oku: "pi pi pika pikachu..."
                    ↓
        🔍 dCode Cipher Identifier
                    ↓
        ⚡ Pikalang dili tespit edildi
                    ↓
        🌐 Pikalang Decoder'a yapıştır
                    ↓
        🔓 Decrypt/Translate
                    ↓
        🚩 Flag{pokitopumneredegordunuzmu}
```
