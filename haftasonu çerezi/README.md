# 🍪 Haftasonu Çerezi - Misc Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Misc-Challenge-darkblue?style=for-the-badge&logo=puzzle" alt="Misc">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Crypto%2BStego-purple?style=for-the-badge&logo=lock" alt="Crypto+Stego">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Misc  
**Seviye:** Orta  
**Puan:** 100

**Challenge Dosyası:** [📥 Google Drive - Flags.jpg](https://drive.google.com/file/d/1EHj3t_73rciYhbJ4C3iWNSR03tM7h82E/view?usp=sharing)

---

## 🔍 Analiz

**Hedef:** Bayrak resimleri → Decode → Hash crack → Steghide

**Strateji:** Maritime Signals → NTLM hash → Steghide extract

---

## ✅ Çözüm Adımları

### 🚩 **1. Bayrak Şifrelemesini Tanımlama**

Dosyayı indirip açtığımızda **bayraklarla şifrelenmiş** bir görsel görüyoruz.

**Araştırma:** "Flag cipher" veya "Navy signals"

**Sonuç:** **Maritime Signals Code (Deniz Sinyalleri)**

---

### 🔓 **2. Maritime Signals Decode**

**Site:** https://www.dcode.fr/maritime-signals-code

**Adımlar:**

1. Bayrakları tek tek decode ediyoruz
2. Çıkan sonuçlar **3 parçaya** ayrılmış

**Decode Sonucu:**

```
DD1951B5F76789461994B7AAF628452A
```

> 💡 Bu bir **hash** gibi görünüyor!

---

### 🔐 **3. Hash Crack**

Hash'i online hash cracker'da deniyoruz.

**Site:** https://crackstation.net/

**Hash:**
```
DD1951B5F76789461994B7AAF628452A
```

**Sonuç:**

| Hash Type | Plain Text |
|-----------|------------|
| **NTLM** | p4ssw0rd123 |

> 🎯 Şifre bulundu: `p4ssw0rd123`

---

### 🖼️ **4. Steghide ile Gizli Dosya Çıkarma**

Bulunan şifre muhtemelen **steghide** için kullanılıyor.

**Komut:**

```bash
steghide extract -sf Flags.jpg -p p4ssw0rd123 -xf steghide.txt
```

**Parametreler:**
- `-sf` : Source file (Flags.jpg)
- `-p` : Password (p4ssw0rd123)
- `-xf` : Extract to file (steghide.txt)

**Çıktı:**
```
wrote extracted data to "steghide.txt"
```

---

### 📝 **5. Flag Dosyasını Okuma**

```bash
cat steghide.txt
```

**İçerik:**
```
FLAG{haftasonucereziAfiyetOlsun}
```

---

## 🚩 **FLAG**

```
FLAG{haftasonucereziAfiyetOlsun}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🚩<br><b>dCode.fr</b><br><sub>Maritime signals</sub></td>
<td align="center">🔐<br><b>CrackStation</b><br><sub>Hash cracker</sub></td>
<td align="center">🖼️<br><b>Steghide</b><br><sub>Steganography tool</sub></td>
</tr>
</table>

**Kullanılan Siteler:**
- 🚩 **Maritime Signals:** https://www.dcode.fr/maritime-signals-code
- 🔐 **Hash Cracker:** https://crackstation.net/
- 📥 **Challenge File:** Google Drive

---

## 💻 **Kullanılan Komutlar**

```bash
# Steghide ile gizli dosya çıkarma
steghide extract -sf Flags.jpg -p p4ssw0rd123 -xf steghide.txt

# Çıkarılan dosyayı okuma
cat steghide.txt
```

---

## 💭 **Çözüm Akış Şeması**

```
🍪 Misc Challenge: "Haftasonu Çerezi"
                    ↓
        📥 Flags.jpg dosyasını indir
                    ↓
        🚩 Bayraklarla şifrelenmiş görsel
                    ↓
        🔍 Maritime Signals Code olduğunu anla
                    ↓
        🌐 dCode.fr → Maritime Signals Decode
                    ↓
        🔐 Hash bulundu: DD1951B5F76789461994B7AAF628452A
                    ↓
        💻 CrackStation.net → NTLM hash crack
                    ↓
        🔑 Şifre: p4ssw0rd123
                    ↓
        🖼️ Steghide extract ile gizli dosya çıkar
                    ↓
        📝 steghide.txt → Flag okundu
                    ↓
        🚩 FLAG{haftasonucereziAfiyetOlsun}
```
