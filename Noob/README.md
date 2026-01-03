# 🎯 Noob - OSINT Challenge

<div align="center">

![OSINT](https://img.shields.io/badge/OSINT-Challenge-blue?style=for-the-badge&logo=searchengineland)
![Difficulty](https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target)
![Bonus](https://img.shields.io/badge/Bonus-+2%20Points-success?style=for-the-badge&logo=star)

</div>

---

## 📋 Soru Bilgileri

**Kategori:** OSINT  
**Seviye:** Orta  
**Bonus:** +2 Puan  
**Açıklama:** OSINT yeteneklerinizi görelim. Soruyu her çözene +2 bonusu var.

**Challenge Dosyası:** [📥 Google Drive](https://drive.google.com/file/d/1U0XEBJnfS8A-5txVWLHd4eV-X2BCG-6Z/view?usp=drivesdk)

---

## 🔍 Analiz

### Adım 1: İlk İpucu

Linkten indirilen **JPG** dosyasını incelediğimizde bir kullanıcı adı keşfettik:
```
@noobctfplayer
```

---

## ✅ Çözüm Adımları

### 🔎 **1. Username OSINT**

Kullanıcı adını sosyal medya platformlarında taramak için OSINT araçlarını kullandık:

- 🔗 [instantusername.com](https://instantusername.com/)
- 🔗 [namechk.com](https://namechk.com/)

**Arama parametresi:** `noobctfplayer`

---

### 🐦 **2. Twitter/X Hesabı Bulma**

Sosyal medya taraması sonucunda **X (Twitter)** platformunda hedef hesabı bulduk:

🔗 **Hesap:** https://x.com/noobctfplayer

**Hesapta tespit edilen ipuçları:**

| İpucu Konumu | Detay |
|--------------|-------|
| 📝 **Bio** | `https://www.dcode.fr/` linki |
| 💬 **Postlar** | Her post **"AT"** ile ilgili |
| 🎨 **Banner** | Gizli şifre: `m0lywvw1pKi0bnfh` |

---

### 🧩 **3. Şifre Çözme**

**İpuçları analizi:**
- dcode.fr sitesi → Şifre çözme platformu
- "AT" vurgusu → Şifreleme türü ipucu
- Banner'daki metin → Encode edilmiş mesaj

**Şifreleme türü araştırması:**

dcode.fr'de **"AT"** araması yaptığımızda **7 farklı** şifreleme türü bulundu. Hepsini denedikten sonra doğru yöntemin **Chiffre Atbash** olduğunu anladık.

---

### 🔓 **4. Decryption**

**Decode işlemi:**

🔗 **Site:** https://www.dcode.fr/chiffre-atbash

**Input:** `m0lywvw1pKi0bnfh`  
**Output:** `n0obded1kPr0ymus`

---

## 🚩 **FLAG**
```
n0obded1kPr0ymus
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>instantusername.com</b><br><sub>Sosyal medya hesap tarama</sub></td>
<td align="center">✅<br><b>namechk.com</b><br><sub>Username + Domain sorgulama</sub></td>
<td align="center">🔐<br><b>dcode.fr</b><br><sub>Atbash Cipher decode</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**
```
📸 JPG Dosyası → 🔍 Kullanıcı adı (@noobctfplayer)
                           ↓
              🌐 OSINT Araçları (instantusername, namechk)
                           ↓
              🐦 Twitter/X Hesabı Bulma
                           ↓
              📝 Profil Analizi:
                 - Bio: dcode.fr linki
                 - Postlar: "AT" teması
                 - Banner: Gizli şifre
                           ↓
              🔐 Chiffre Atbash Decode
                           ↓
              🚩 FLAG: n0obded1kPr0ymus
```

