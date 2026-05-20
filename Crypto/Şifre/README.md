# 🎯 Şifre - Crypto Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Crypto-Challenge-darkblue?style=for-the-badge" alt="Crypto">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Hash Cracking-purple?style=for-the-badge" alt="Hash Cracking">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Crypto  
**Seviye:** Kolay  
**Puan:** 500

**Challange Açıklaması:** Cihan Hoca gizli bir proje için şifre oluşturmuş. Ama ne olduğunu anlayamadık. Sen çözer misin?

**Verilen Hash:** `cc5eec4ef98897fc17f31f8b1e86628f`

**Flag Formatı:** `Flag{şifre}`

---

## 🔍 Analiz

**Hedef:** Verilen hash'i crack ederek orijinal şifreyi bulmak

**Strateji:** Hash type identification → Online hash cracking → Flag

**İpuçları:**
- 🎯 32 karakterlik hexadecimal string → **MD5 hash!**
- 🔍 MD5: 128-bit hash algoritması (32 hex karakter = 128 bit)
- 💡 Eski ve zayıf algoritma → Kolayca kırılabilir
- 🌐 Online hash cracking servisleri kullanılabilir

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Hash Tipini Belirle**

```bash
# Hash uzunluğunu kontrol et
echo "cc5eec4ef98897fc17f31f8b1e86628f" | wc -c
```

**Çıktı:**
```
33  (32 karakter + newline)
```

**Analiz:**
- ✅ 32 hexadecimal karakter
- 💡 Olası hash tipleri:
  - **MD5** (en yaygın 32 char hash)
  - NTLM
  - LM
- 🎯 Challenge kategorisi "Crypto" ve basit → Muhtemelen **MD5**

---

### 🔨 **Adım 2: Hash'i Tanımla (hashid)**

```bash
hashid cc5eec4ef98897fc17f31f8b1e86628f
```

**Çıktı:**
```
Analyzing 'cc5eec4ef98897fc17f31f8b1e86628f'
[+] MD5 
[+] Domain Cached Credentials - MD4(MD4(($pass)).(strtolower($username)))
```

**Analiz:**
- ✅ İlk tahmin **MD5** olarak doğrulandı
- 💡 MD5 çok yaygın kullanılan hash tipi
- 🚨 MD5 artık güvenli değil (rainbow table ile kırılabilir)

---

### 🌐 **Adım 3: Online Hash Cracking Servisi Kullan**

Hash'i crack etmek için online bir servis kullanacağız: **https://md5hashing.net/hash**

**Adımlar:**
1. 🌐 https://md5hashing.net/hash sitesine git
2. 📝 **"Reverse hash decoder"** sekmesini seç
3. 🔍 **Hash type:** MD5 seç
4. 📋 **Hash:** `cc5eec4ef98897fc17f31f8b1e86628f` yapıştır
5. 🔓 **"Decode!"** butonuna tıkla

**Sonuç:**
```
Hash: cc5eec4ef98897fc17f31f8b1e86628f
Type: MD5
Decoded: P1T03dubounty
```

**Analiz:**
- ✅ Hash başarıyla decode edildi!
- 💡 Orijinal şifre: **`P1T03dubounty`**
- 🎯 Leet speak kullanılmış: `P1T0` → "Pito"
- 🐛 "bugbounty" kelimesinin modifiye edilmiş hali

---

### 🚩 **Adım 4: Flag Formatını Oluştur**

Challenge'ın istediği format: `Flag{şifre}`

```bash
echo "Flag{P1T03dubounty}"
```

**Analiz:**
- ✅ Şifre bulundu: `P1T03dubounty`
- 🎯 Flag: `Flag{P1T03dubounty}`
- 💡 Büyük/küçük harf hassasiyetine dikkat!

---

## 🚩 **FLAG**
```
Flag{P1T03dubounty}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>hashid</b><br><sub>Hash type identification</sub></td>
<td align="center">🌐<br><b>md5hashing.net</b><br><sub>Online hash cracking</sub></td>
<td align="center">🔧<br><b>wc</b><br><sub>Character counting</sub></td>
<td align="center">📝<br><b>echo</b><br><sub>String operations</sub></td>
</tr>
</table>

**Alternatif Araçlar:**
- 🔓 **CrackStation.net** - Geniş rainbow table
- 🕵️ **HashCat** - GPU-accelerated cracking
- 🔨 **John the Ripper** - Multi-format password cracker
- 🌈 **RainbowCrack** - Rainbow table tool

---

## 💡 **Ek Bilgiler**

### 📚 **MD5 Hash Hakkında**
- **Algoritma:** MD5 (Message Digest Algorithm 5)
- **Çıktı:** 128-bit hash (32 hexadecimal karakter)
- **Güvenlik:** ⚠️ Artık güvenli değil!
  - Collision attacks mümkün
  - Rainbow table ile hızlıca kırılabilir
  - Şifre hash'leme için kullanılmamalı
- **Modern Alternatifler:** 
  - ✅ bcrypt
  - ✅ Argon2
  - ✅ PBKDF2
  - ✅ scrypt

### 🔐 **Hash Cracking Yöntemleri**

**1. Rainbow Table Attack:**
```
Önceden hesaplanmış hash'ler
P1T03dubounty → cc5eec4ef98897fc17f31f8b1e86628f (tabloda var!)
```

**2. Dictionary Attack:**
```bash
hashcat -m 0 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

**3. Brute Force:**
```bash
hashcat -m 0 -a 3 hash.txt ?a?a?a?a?a?a?a?a
```

---

## 💭 **Çözüm Akış Şeması**
```
🎯 Şifre Challenge
       ↓
🔍 Hash tipini belirle
   → echo "cc5eec4ef98897fc17f31f8b1e86628f" | wc -c
   → 32 karakter → MD5 olabilir
       ↓
🔨 Hash'i tanımla
   → hashid cc5eec4ef98897fc17f31f8b1e86628f
   → MD5 ✅
       ↓
🌐 Online cracking servisi kullan
   → https://md5hashing.net/hash
   → Reverse hash decoder
   → Hash type: MD5
   → Hash: cc5eec4ef98897fc17f31f8b1e86628f
       ↓
🔓 Decode et
   → "Decode!" butonuna tıkla
   → Sonuç: P1T03dubounty ✅
       ↓
🚩 Flag formatını oluştur
   → Flag{P1T03dubounty}
```
