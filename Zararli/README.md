# 🦠 Zararlı - Malware Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Malware-Challenge-darkblue?style=for-the-badge&logo=virus" alt="Malware">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Bonus-+2%20Points-success?style=for-the-badge&logo=star" alt="Bonus">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Malware  
**Seviye:** Orta  
**Soru No:** 71  
**Bonus:** +2 Puan  
**Açıklama:** Çalıştırma istersen. Virüs var gibi.

**Challenge Dosyası:** [📥 GitHub - Zararlidosya.exe](https://github.com/cihangungor/pitoctf/blob/main/Zararlidosya.exe)

---

## 🔍 Analiz

### İlk Değerlendirme

Soruda **"Çalıştırma istersen. Virüs var gibi."** uyarısı var. Bu bize ne yapacağımızı söylüyor:

| İpucu | Çıkarım |
|-------|---------|
| ⚠️ **"Çalıştırma istersen"** | Dosyayı çalıştırmayacağız |
| 🦠 **"Virüs var gibi"** | Zararlı olabilir |
| 🔬 **Analiz Yöntemi** | **Statik Analiz** yapacağız |

---

## ✅ Çözüm Adımları

### 📝 **1. Strings ile İlk İnceleme**

İlk adımda dosyanın içindeki metinleri inceliyoruz. Belki direkt bir ipucu vardır.

**Komut:**
```bash
strings Zararlidosya.exe
```

**Sonuç:** ❌ Çok uzun çıktı (61997 satır!)

**Problem:** Çıktı çok fazla ve içinde gereksiz bilgiler var. Aramamızı **daraltmamız** gerekiyor.

---

### 🔍 **2. Filtreleme ile Hedefli Arama**

Flag, password ve key kelimelerini içeren satırları filtreliyoruz.

**Komut:**
```bash
strings Zararlidosya.exe | grep -i "flag\|password\|key" > Zararlidosya.txt
```

**Komut Parametreleri:**

| Parametre | Açıklama |
|-----------|----------|
| `strings` | Binary dosyadaki okunabilir metinleri çıkarır |
| `grep -i` | Case-insensitive arama (büyük/küçük harf duyarsız) |
| `"flag\|password\|key"` | Bu kelimeleri ara (OR mantığı) |
| `> Zararlidosya.txt` | Çıktıyı dosyaya kaydet |

**Sonuç:** ✅ **61997 satır → 302 satıra** düştü!

---

### 🤖 **3. AI ile Hızlı Analiz**

302 satır hala uzun. Daha hızlı analiz için **AI bot** kullanıyoruz.

**İşlem:** `Zararlidosya.txt` dosyasını bota yüklüyoruz

**Soru:** "Burada şüpheli bir şey var mı?"

**Botun Cevabı:**
```
RmxhZ3sweDQ2NkM2MTY3N0I2ODY1NzI3MzY1Nzk3NjM0NzQ2MTZFMzE2MzY5NkU3RH0=
```

✅ **Base64 encoded bir string bulundu!**

---

### 🔓 **4. Base64 Decode (1. Katman)**

İlk şifre katmanını çözüyoruz.

**Komut:**
```bash
echo "RmxhZ3sweDQ2NkM2MTY3N0I2ODY1NzI3MzY1Nzk3NjM0NzQ2MTZFMzE2MzY5NkU3RH0=" | base64 -d
```

**Çıktı:**
```
Flag{0x466C61677B686572736579763474616E3163696E7D}
```

**Analiz:** İçinde `0x` prefix'i olan bir **hex değeri** var!

---

### 🔐 **5. Hex Decode (2. Katman)**

Flag formatının içindeki hex veriyi decode ediyoruz.

**Hex Değeri:** `466C61677B686572736579763474616E3163696E7D`

**Komut:**
```bash
echo "466C61677B686572736579763474616E3163696E7D" | xxd -r -p
```

**Komut Parametreleri:**

| Parametre | Açıklama |
|-----------|----------|
| `xxd` | Hex dump aracı |
| `-r` | Reverse (hex'ten text'e) |
| `-p` | Plain hexdump style |

**Çıktı:**
```
Flag{herseyv4tan1cin}
```

---

## 🚩 **FLAG**

```
Flag{herseyv4tan1cin}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">📝<br><b>strings</b><br><sub>Binary text extraction</sub></td>
<td align="center">🔍<br><b>grep</b><br><sub>Pattern matching</sub></td>
<td align="center">🤖<br><b>AI Bot</b><br><sub>Hızlı analiz</sub></td>
<td align="center">🔓<br><b>base64</b><br><sub>Base64 decode</sub></td>
<td align="center">🔐<br><b>xxd</b><br><sub>Hex decode</sub></td>
</tr>
</table>

**Alternatif Online Araçlar:**
- 🔓 **Base64 Decode:** https://www.base64decode.org/
- 🔐 **Hex to ASCII:** https://www.rapidtables.com/convert/number/hex-to-ascii.html

---

## 💻 **Kullanılan Komutlar**

```bash
# 1. Tüm stringleri çıkar (çok uzun!)
strings Zararlidosya.exe

# 2. Flag/password/key kelimelerini filtrele
strings Zararlidosya.exe | grep -i "flag\|password\|key" > Zararlidosya.txt

# 3. Base64 decode
echo "RmxhZ3sweDQ2NkM2MTY3N0I2ODY1NzI3MzY1Nzk3NjM0NzQ2MTZFMzE2MzY5NkU3RH0=" | base64 -d

# 4. Hex decode
echo "466C61677B686572736579763474616E3163696E7D" | xxd -r -p
```

---

## 💭 **Çözüm Akış Şeması**

```
🦠 Zararlı Dosya: Zararlidosya.exe
                    ↓
        ⚠️ "Çalıştırma istersen. Virüs var gibi."
                    ↓
        🔬 Statik Analiz Kararı (Çalıştırmayacağız!)
                    ↓
        📝 strings Zararlidosya.exe
                    ↓
        ❌ Çok uzun çıktı (61997 satır)
                    ↓
        🔍 Filtreleme: grep -i "flag\|password\|key"
                    ↓
        ✅ 302 satıra düştü → Zararlidosya.txt
                    ↓
        🤖 AI Bot Analizi: "Şüpheli bir şey var mı?"
                    ↓
        🔍 Base64 String Bulundu!
                    ↓
        🔓 Base64 Decode
                    ↓
        Flag{0x466C61677B686572736579763474616E3163696E7D}
                    ↓
        🔐 Hex Decode (0x prefix'inden sonraki kısım)
                    ↓
        🚩 FLAG: Flag{herseyv4tan1cin}
```
