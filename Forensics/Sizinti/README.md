# 🔓 Sizinti - Forensics Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Forensics-Challenge-darkblue?style=for-the-badge" alt="Forensics">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Network%20Forensics-purple?style=for-the-badge" alt="Network Forensics">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Forensics  
**Seviye:** Orta
**Puan:** 500

**Challenge Açıklaması:** Ne sunuculara mı sızmışlar?

**Challenge Dosyası:** [📥 Google Drive - Sizinti.zip](https://drive.google.com/file/d/1hXjWCk-Izp34A0PkttDP1wqthG5a0lDG/view?usp=drivesdk)

---

## 🔍 Analiz

**Hedef:** Bir web sunucusuna yapılan saldırıyı pcap dosyasından analiz etmek ve flag'i bulmak

**Strateji:** pcap aç → HTTP trafiğini analiz et → Saldırgan faaliyetlerini tespit et → Yüklenen webshell'i decode et → Flag'i çıkar

**İpuçları:**
- 🎯 "Sizinti" → data exfiltration ya da sunucu ele geçirme teması
- 🔍 pcap dosyası = network capture analizi gerekli
- 💡 Port 80 üzerinde yoğun HTTP trafiği var
- 🔑 Obfuscate edilmiş PHP webshell içinde flag gizli

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Dosya Analizi**

```bash
file challenge.pcap
# pcap capture file, microsecond ts (little-endian) - version 2.4 (Ethernet)

# İstatistikler:
# Toplam paket: 19.256
# Süre: ~13 dakika
# Port 80: 18.441 paket (HTTP)
```

**Analiz:**
- ✅ Standart Ethernet/IPv4 pcap dosyası
- 💡 Port 80 üzerinde çok yoğun HTTP trafiği var
- 🎯 Hedef sunucu: `172.31.9.156` (targetaggregator.intergalacticministry.com)

---

### 🌐 **Adım 2: HTTP Trafiği Analizi**

TCP akışları reassemble edilerek HTTP istekleri listelendi:

```python
# Saldırgan IP'leri:
# 146.70.38.48 → En aktif saldırgan (directory bruteforce + webshell)
# 37.46.113.165 → Büyük PDF dosyası yükledi
# 212.102.35.152 → SQLi denemeleri

# Toplam 226 HTTP akışı tespit edildi
```

**Bulunan HTTP İstekleri (kritik):**

| IP | İstek | Açıklama |
|----|-------|----------|
| `146.70.38.48` | `POST /map-update.php` | Webshell yükleme |
| `146.70.38.48` | `GET /uploads/galacticmap.php?cmd=whoami` | Komut çalıştırma |
| `146.70.38.48` | `GET /uploads/galacticmap.php?cmd=ls` | Dizin listeleme |
| `146.70.38.48` | `GET /uploads/galacticmap.php?cmd=ls+/` | Root dizin listeleme |
| `146.70.38.48` | `GET /*/galacticmap.php` (500+ istek) | Directory bruteforce |

---

### 📁 **Adım 3: Webshell Upload Tespiti**

Saldırgan `POST /map-update.php` ile obfuscate edilmiş PHP webshell yükledi:

```
Content-Disposition: form-data; name="uploaded_file"; filename="galacticmap.php"
Content-Type: application/x-php
```

**Yüklenen webshell parametreleri:**
```php
$pPziZoJiMpcu = 82;          // Deobfuscation key
$liGBOKxsOGMz = array();     // Temp array
$iyzQ5h8qf6 = "";             // Obfuscated string
```

---

### 🔐 **Adım 4: Webshell Decode**

PHP kodu obfuscate edilmiş — `$pPziZoJiMpcu = 82` key ile shuffle yapılmış.

**Deobfuscation algoritması:**
- String 82 farklı döngü ile interleave edilmiş
- Her döngü 65 karakter içeriyor (5330 / 82 = 65)
- `decoded[i] = shuffled[(i * 82) % 5330]` formülü uygulandı

```python
n = len(combined)   # 5330
result = ['?'] * n
for i in range(n):
    src = (i * 82) % n
    result[i] = combined[src]
decoded = ''.join(result)
```

**Decode edilen webshell (kritik satır):**

```php
$dir = realpath($dir.'/'.$value);
##flag = HTB{W0w_ROt_A_DaY}
$dirs = scandir($dir);
```

---

### ⚡ **Adım 5: Saldırı Timeline'ı**

```
21:56:01  Saldırgan keşif → GET /
21:56:49  SQL injection denemesi → POST /results_display.php
21:57:31  Webshell yükleme → POST /map-update.php (galacticmap.php)
21:58:19  Directory bruteforce (500+ GET isteği)
22:01:27  Webshell keşfedildi → GET /uploads/galacticmap.php
22:01:31  Komut çalıştırma → ?cmd=whoami  → "www-data"
22:01:37  Dizin listeleme → ?cmd=ls
22:01:41  Root listeleme → ?cmd=ls+/
```

---

## 🚩 **FLAG**
```
HTB{W0w_ROt_A_DaY}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🐍<br><b>Python (struct)</b><br><sub>pcap manuel parse / TCP reassembly</sub></td>
<td align="center">🔍<br><b>Python (math)</b><br><sub>Webshell deobfuscation</sub></td>
<td align="center">🌐<br><b>HTTP analizi</b><br><sub>Stream reconstruction</sub></td>
<td align="center">📝<br><b>String analizi</b><br><sub>PHP obfuscation decode</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🔓 Sizinti — Forensics Challenge
       ↓
📁 challenge.pcap (9.4 MB, 19.256 paket)
   → Ethernet/IPv4/TCP, Port 80 ağırlıklı
       ↓
🌐 HTTP trafiği analizi
   → 226 TCP akışı tespit edildi
   → 4 farklı kaynak IP bulundu
       ↓
🚨 Saldırgan tespit: 146.70.38.48
   → POST /map-update.php → galacticmap.php yüklendi
   → 500+ GET isteği (directory bruteforce)
   → ?cmd=whoami → "www-data" ✅
       ↓
🔐 Webshell deobfuscation
   → $pPziZoJiMpcu = 82 (shuffle key)
   → n=5330, gcd(82,5330)=82 → 82 ayrı döngü
   → decoded[i] = shuffled[(i*82) % 5330]
       ↓
💡 PHP kodunda flag bulundu:
   → ##flag = HTB{W0w_ROt_A_DaY}
       ↓
🚩 FLAG: HTB{W0w_ROt_A_DaY}
```
