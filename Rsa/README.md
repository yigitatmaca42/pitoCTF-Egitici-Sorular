# 🔐 Rsa - Crypto Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Crypto-Challenge-darkblue?style=for-the-badge&logo=lock" alt="Crypto">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-RSA%20Encryption-purple?style=for-the-badge&logo=shield" alt="RSA">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Crypto  
**Seviye:** Kolay  
**Puan:** 100  

**Challenge Dosyası:** [📥 Google Drive - Rsa.txt](https://drive.google.com/file/d/1rcbU-YGD3YNGiU1qta2pmS9uFA7b04Zr/view?usp=drivesdk)

---

## 🔍 Analiz

**Hedef:** RSA şifrelenmiş mesajı decrypt etmek

**Verilen:**

```
n (Modulus): 5032189615332819537112976822349589378730798408513723516065074630081106503758415210427578051757871842379259866570098301946206888469902434898092142149331291

e (Exponent): 65537

c (Ciphertext): 2385531761880574571521044618041107490447656236932398098489180893651448509508705100922368494855388138214488483034456098213857219598953002865357066717722496
```

---

## ✅ Çözüm Adımları

### 🌐 **1. Online RSA Solver Kullanımı**

**Site:** https://www.dcode.fr/rsa-cipher

**Adımlar:**

1. dCode RSA Cipher sayfasına gidiyoruz
2. **"Decrypt RSA"** bölümünü seçiyoruz
3. Parametreleri giriyoruz:
   - **n (Modulus):** `5032189615332819537112976822349589378730798408513723516065074630081106503758415210427578051757871842379259866570098301946206888469902434898092142149331291`
   - **e (Public Exponent):** `65537`
   - **c (Ciphertext):** `2385531761880574571521044618041107490447656236932398098489180893651448509508705100922368494855388138214488483034456098213857219598953002865357066717722496`
4. **"Decrypt"** butonuna tıklıyoruz

---

### 🔓 **2. RSA Decrypt Sonucu**

**Plaintext (Çözülen Mesaj):**

```
SiberVatan{hackerlar_verinin_arkeologlaridir}
```

> 🎯 **Flag bulundu!**

---

## 🚩 **FLAG**

```
SiberVatan{hackerlar_verinin_arkeologlaridir}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔐<br><b>dCode RSA</b><br><sub>RSA decoder</sub></td>
<td align="center">🔢<br><b>FactorDB</b><br><sub>Number factorization</sub></td>
<td align="center">🐍<br><b>RsaCtfTool</b><br><sub>Automated RSA tool</sub></td>
</tr>
</table>

**Kullanılan Siteler:**
- 🔐 **RSA Cipher:** https://www.dcode.fr/rsa-cipher
- 🔢 **FactorDB:** http://factordb.com/
- 📥 **Challenge File:** Google Drive

---

## 💭 **Çözüm Akış Şeması**

```
🔐 Crypto Challenge: "Rsa"
                    ↓
        📥 Rsa.txt dosyasını indir
                    ↓
        📝 n, e, c parametrelerini al
                    ↓
        🌐 dCode RSA Cipher'a git
                    ↓
        🔢 n, e, c değerlerini gir
                    ↓
        🔓 Decrypt butonuna tıkla
                    ↓
        📝 Plaintext: SiberVatan{hackerlar_verinin_arkeologlaridir}
                    ↓
        🚩 Flag bulundu!
```
