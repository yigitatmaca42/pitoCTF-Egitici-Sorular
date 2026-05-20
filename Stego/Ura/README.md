# 🖼️ Ura - Stego Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Stego-Challenge-darkblue?style=for-the-badge&logo=image" alt="Stego">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Steghide-purple?style=for-the-badge&logo=lock" alt="Steghide">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Stego  
**Seviye:** Kolay  
**Puan:** 200

**Challenge Dosyası:** [📥 Google Drive - Uraa.jpg](https://drive.google.com/file/d/1ilHy5vabMgv2e6msKlfkm7BhAS8dZgGE/view?usp=drivesdk)


---

## 🔍 Analiz

**Hedef:** JPEG dosyasında steghide ile gizlenmiş flag'i çıkarmak

**Strateji:** File check → Steghide tespit → Password cracking → Flag extraction

---

## ✅ Çözüm Adımları

### 📥 **1. Dosyayı İndirme**

Google Drive'dan `Uraa.jpg` dosyasını indiriyoruz.

---

### 🔍 **2. Temel Analiz**

**Dosya tipini kontrol:**

```bash
file Uraa.jpg
```

**Çıktı:**
```
Uraa.jpg: JPEG image data, JFIF standard 1.01
```

**Metadata kontrolü:**

```bash
exiftool Uraa.jpg
```

**Çıktı:**
```
File Name                       : Uraa.jpg
File Size                       : 79 kB
Image Width                     : 564
Image Height                    : 824
```

**String analizi:**

```bash
strings Uraa.jpg | grep -i flag
# Sonuç yok - flag gizlenmiş olmalı
```

---

### 🔐 **3. Steghide Tespiti**

**Steghide ile test:**

```bash
steghide extract -sf Uraa.jpg
```

**Çıktı:**
```
Enter passphrase: 
```

> 🎯 **Steghide ile gizlenmiş!** Şifre gerekiyor.

---

### 🛠️ **4. Stegseek Kurulumu**

**Kali Linux:**

```bash
sudo apt update
sudo apt install stegseek
```

**Rockyou.txt wordlist'i hazırlama:**

```bash
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
```

---

### 🔓 **5. Password Cracking**

**Stegseek ile şifre kırma:**

```bash
stegseek Uraa.jpg /usr/share/wordlists/rockyou.txt
```

**Çıktı:**

```
StegSeek 0.6 - https://github.com/RickdeJager/StegSeek
[i] Found passphrase: "urahara1"         
[i] Original filename: "flag.txt".
[i] Extracting to "Uraa.jpg.out".
```

> ✅ **Şifre bulundu:** `urahara1`  
> 📝 **İpucu:** Bleach anime karakteri "Kisuke Urahara" referansı!

---

### 🎯 **6. Flag'i Okuma**

**Çıkarılan dosyayı okuma:**

```bash
cat Uraa.jpg.out
```

**Çıktı:**

```
0xL4ugh{W4RM_UP_STE94N0_G0OD_J0B}
```

---

## 🚩 **FLAG**

```
0xL4ugh{W4RM_UP_STE94N0_G0OD_J0B}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>file</b><br><sub>Dosya tipi tespiti</sub></td>
<td align="center">📊<br><b>exiftool</b><br><sub>Metadata analizi</sub></td>
<td align="center">🔐<br><b>steghide</b><br><sub>Stego extraction</sub></td>
<td align="center">⚡<br><b>stegseek</b><br><sub>Password cracking</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**
```bash
# Temel analiz
file Uraa.jpg
exiftool Uraa.jpg
strings Uraa.jpg | grep -i flag

# Steghide testi
steghide extract -sf Uraa.jpg

# Stegseek ile şifre kırma
stegseek Uraa.jpg /usr/share/wordlists/rockyou.txt

# Flag okuma
cat Uraa.jpg.out
```

---

## 💭 **Çözüm Akış Şeması**

```
🖼️ Stego Challenge: "Ura"
                    ↓
        📥 Uraa.jpg dosyasını indir
                    ↓
        🔍 file Uraa.jpg → JPEG image
                    ↓
        📊 exiftool → Normal metadata
                    ↓
        🔤 strings → Flag bulunamadı
                    ↓
        🔐 steghide extract → Şifre istedi!
                    ↓
        ⚡ stegseek ile brute force
                    ↓
        ✅ Şifre bulundu: "urahara1"
                    ↓
        📝 flag.txt çıkarıldı
                    ↓
        🚩 0xL4ugh{W4RM_UP_STE94N0_G0OD_J0B}
```
