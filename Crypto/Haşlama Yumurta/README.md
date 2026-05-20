# 🥚 Haşlama Yumurta - Crypto Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Crypto-Challenge-darkblue?style=for-the-badge&logo=lock" alt="Crypto">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Hash%20Cracking-purple?style=for-the-badge&logo=key" alt="Hash Cracking">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Crypto  
**Seviye:** Kolay  
**Puan:** 200  
**Açıklama:** Şifreyi bul, flagi al.

**Challenge URL:** 
```bash
nc verbal-sleep.picoctf.net 59566
```

---

## 🔍 Analiz

### Netcat Bağlantısı

Challenge'a bağlandığımızda **3 aşamalı** bir hash cracking senaryosu ile karşılaştık. Her aşamada farklı hash algoritması kullanılmış.

---

## ✅ Çözüm Adımları

### 🔌 **1. Bağlantı Kurma**
```bash
nc verbal-sleep.picoctf.net 59566
```

İlk giriş çıktısında bir hash değeri aldık.

---

### 🔐 **2. MD5 Hash Cracking (1. Aşama)**

**Hash Değeri:** `482c811da5d5b4bc6d497ffa98491e38`

**Hash Analizi:**
- Kısa hex uzunluğu
- Sayı ile başlıyor
- **32 karakter** → MD5 hash

**Decode sonucu:**

| Hash | Algoritma | Sonuç |
|------|-----------|-------|
| `482c811da5d5b4bc6d497ffa98491e38` | MD5 | `password123` |

✅ **1. Şifre:** `password123`

---

### 🔓 **3. SHA-1 Hash Cracking (2. Aşama)**

Yeni bir hash değeri aldık.

**Hash Değeri:** `b7a875fc1ea228b9061041b7cec4bd3c52ab3ce3`

**Hash Analizi:**
- Daha uzun hex değeri
- **40 karakter** → SHA-1 hash
- Sırayla güçlenen hash algoritmaları

**Decode sonucu:**

| Hash | Algoritma | Sonuç |
|------|-----------|-------|
| `b7a875fc1ea228b9061041b7cec4bd3c52ab3ce3` | SHA-1 | `letmein` |

✅ **2. Şifre:** `letmein`

---

### 🛡️ **4. SHA-256 Hash Cracking (3. Aşama)**

Son hash değerini aldık.

**Hash Değeri:** `916e8c4f79b25028c9e467f1eb8eee6d6bbdff965f9928310ad30a8d88697745`

**Hash Analizi:**
- En uzun hex değeri
- **64 karakter** → SHA-256 hash
- Hash algoritmaları sıralı: MD5 → SHA-1 → SHA-256

**Decode sonucu:**

| Hash | Algoritma | Sonuç |
|------|-----------|-------|
| `916e8c4f79b25028c9e467f1eb8eee6d6bbdff965f9928310ad30a8d88697745` | SHA-256 | `qwerty098` |

✅ **3. Şifre:** `qwerty098`

---

### 🎯 **5. Flag Elde Etme**

Tüm şifreleri doğru girdikten sonra flag verildi:
```
picoCTF{UseStr0nG_h@shEs_&PaSswDs!_eff9dbe0}
```

---

## 🚩 **FLAG**
```
picoCTF{UseStr0nG_h@shEs_&PaSswDs!_eff9dbe0}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔓<br><b>CrackStation</b><br><sub>Hash cracking servisi</sub></td>
<td align="center">🔌<br><b>Netcat</b><br><sub>Network bağlantı aracı</sub></td>
<td align="center">🔍<br><b>Hash Identifier</b><br><sub>Hash türü tespiti</sub></td>
</tr>
</table>

**Decode Sitesi:** [CrackStation](https://crackstation.net/)

---

## 💭 **Çözüm Akış Şeması**
```
🔌 Netcat Bağlantısı (nc verbal-sleep.picoctf.net 59566)
                           ↓
              📝 1. Hash → MD5 (32 karakter)
                           ↓
              🔓 Decode: password123
                           ↓
              📝 2. Hash → SHA-1 (40 karakter)
                           ↓
              🔓 Decode: letmein
                           ↓
              📝 3. Hash → SHA-256 (64 karakter)
                           ↓
              🔓 Decode: qwerty098
                           ↓
              🚩 FLAG: picoCTF{UseStr0nG_h@shEs_&PaSswDs!_eff9dbe0}
```

---

## 📚 **Hash Algoritmaları**

| Algoritma | Karakter Uzunluğu | Güvenlik Seviyesi |
|-----------|-------------------|-------------------|
| **MD5** | 32 karakter | ⚠️ Zayıf |
| **SHA-1** | 40 karakter | ⚠️ Orta |
| **SHA-256** | 64 karakter | ✅ Güçlü |

---

