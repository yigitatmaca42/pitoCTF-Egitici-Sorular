# 🔍 Kim o? - OSINT Challenge

<p align="center">
  <img src="https://img.shields.io/badge/OSINT-Challenge-darkblue?style=for-the-badge&logo=search" alt="OSINT">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Domain%20Research-purple?style=for-the-badge&logo=globe" alt="Domain Research">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** OSINT 
**Seviye:** Kolay  
**Puan:** 100  
**Açıklama:** Kapı mı çaldı? Kim o?

**Challenge URL:** https://ctf.pitoctf.online/

---

## 🔍 Analiz

### OSINT Nedir?

**OSINT (Open Source Intelligence)**, açık kaynaklardan bilgi toplama ve analiz etme sürecidir. CTF yarışmalarında genellikle:

| Teknik | Açıklama |
|---------|----------|
| 🌐 **WHOIS Lookup** | Domain kayıt bilgilerini sorgulama |
| 📧 **Email Intelligence** | E-posta adresi analizi |
| 🏢 **Domain Registration** | Kayıt sahibi bilgileri |
| 📍 **Geolocation** | Konum bilgisi araştırması |

Bu challenge'da **domain bilgilerini** araştırmamız gerekiyor.

---

## ✅ Çözüm Adımları

### 🎯 **1. Challenge'ı Anlama**

Challenge başlığı: **"Kim o?"**  
Açıklama: **"Kapı mı çaldı? Kim o?"**

> 💡 **İpucu:** Bu sorular bize **domain sahibinin kim olduğunu** bulmamız gerektiğini söylüyor.

**Hedef Domain:** `pitoctf.online`

---

### 🌐 **2. WHOIS Sorgusu**

Domain hakkında bilgi toplamak için **WHOIS lookup** yapıyoruz.

**Kullanılan Site:** https://who.is/

**Adımlar:**

1. https://who.is/ adresine gidiyoruz
2. Arama kutusuna `pitoctf.online` yazıyoruz
3. **WHOIS Lookup** butonuna tıklıyoruz

**Alternatif RDAP Sorgusu:**
```
https://who.is/rdap/pitoctf.online
```

---

### 📊 **3. WHOIS Sonuçlarını İnceleme**

WHOIS sorgusu sonucunda şu bilgileri görüyoruz:

#### **Registrar Contact (Kayıt İletişim Bilgileri)**

---

#### **Domain Contacts - Registrant Contact**

**Address:**
```
Flag{birileriwhoissorgusumuyapti}
```

> 🎯 **FLAG BULUNDU!** Domain kayıt bilgilerinin "Address" alanında flag gizlenmiş!

---

## 🚩 **FLAG**

```
Flag{birileriwhoissorgusumuyapti}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>WHOIS Lookup</b><br><sub>Domain sorgulama</sub></td>
<td align="center">🌐<br><b>who.is</b><br><sub>Online WHOIS tool</sub></td>
<td align="center">📊<br><b>RDAP</b><br><sub>Registration Data Access Protocol</sub></td>
</tr>
</table>

**Kullanılan Siteler:**
- 🔍 **WHOIS Lookup:** https://who.is/
- 📊 **RDAP Query:** https://who.is/rdap/pitoctf.online
- 🎯 **Challenge Site:** https://ctf.pitoctf.online/

---

## 💭 **Çözüm Akış Şeması**

```
🔍 OSINT Challenge: "Kim o?"
                    ↓
        🎯 Domain Belirleme: pitoctf.online
                    ↓
        🌐 WHOIS Lookup Sitesine Git (who.is)
                    ↓
        🔎 Domain Sorgulama
                    ↓
        📊 WHOIS Sonuçlarını İnceleme
                    ↓
        📧 Registrar Contact → GoDaddy bilgileri
                    ↓
        🏢 Registrant Contact → Domain sahibi bilgileri
                    ↓
        📍 Address kısmını kontrol et
                    ↓
        🚩 FLAG: Flag{birileriwhoissorgusumuyapti}
```
