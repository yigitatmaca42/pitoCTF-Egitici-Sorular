# 🔍 Şişman - Forensics Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Forensics-Challenge-darkblue?style=for-the-badge" alt="Forensics">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Network%20Forensics-purple?style=for-the-badge" alt="Network Forensics">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Forensics  
**Seviye:** Orta  
**Puan:** 1000

**Challenge Dosyası:** [📥 Google Drive - sisman.pcapng](https://drive.google.com/file/d/1kx5ogkCOBazqRirI4H_uFhQSqDQ9UFNA/view?usp=sharing)

---

## 🔍 Analiz

**Hedef:** PCAP dosyasındaki network trafiğini analiz etmek ve 3 farklı yerde gizlenmiş flag parçalarını bulmak

**Strateji:** PCAP analiz → Paket yorumları → DNS subdomain'ler → FTP trafiği → Flag birleştir

**İpuçları:**
- 🎯 "Şişman" → büyük PCAP dosyası (181MB)
- 🔍 197k paket, ~11 dakika capture
- 💡 Paket yorumlarında ipuçları var
- 🔑 Flag 3 parça halinde gizlenmiş

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: PCAP Dosya Analizi**

```bash
file sisman.pcapng
capinfos sisman.pcapng
```

**Sonuçlar:**
- ✅ **Dosya boyutu:** 181MB
- 📊 **Paket sayısı:** 197,539 paket
- ⏱️ **Süre:** 695.5 saniye (~11.5 dakika)
- 🌐 **Protokol:** Raw IPv4, çoğunlukla HTTP trafiği

---

### 🧩 **Adım 2: Paket Yorumlarını İnceleme**

```bash
tshark -r sisman.pcapng -T fields -e frame.comment | grep -v "^$"
```

**Kritik Bulgular:**

| Frame | Yorum |
|-------|--------|
| 163038 | `_busismandosyadabuyorumuiyigordun}` |
| 163039 | `analytics-cdn.net domainine dikkat et. Puzzle'ın bir parcasi orada :)` |

**Analiz:**
- ✅ Flag'in **sonu** bulundu
- 💡 **analytics-cdn.net** domain'inde daha fazla parça var
- 🎯 "Puzzle parçası" = flag parçalı şekilde gizlenmiş

---

### 🌐 **Adım 3: DNS Subdomain Analizi**

```bash
# analytics-cdn.net domain'ini ara
tshark -r sisman.pcapng -Y "dns.qry.name contains \"analytics-cdn.net\"" -T fields -e frame.number -e dns.qry.name
```

**Hex-Encoded Subdomain'ler Bulundu:**
```
107027  00.5f646e7373756264.data.analytics-cdn.net
107029  01.6f6d61696e6c6572.data.analytics-cdn.net
107031  02.696e6567697a6c65.data.analytics-cdn.net
107033  03.6e64696d5f.data.analytics-cdn.net
```

**Hex Decode:**
```bash
echo "5f646e7373756264" | xxd -r -p    # _dnssub
echo "6f6d61696e6c6572" | xxd -r -p    # domainler
echo "696e6567697a6c65" | xxd -r -p    # inegizle  
echo "6e64696d5f" | xxd -r -p          # ndim_
```

**Birleşik Mesaj:** `_dnssubdomainlerinegizlendim_`

---

### 📡 **Adım 4: FTP Trafiği Keşfi**

```bash
# CTF string'ini tüm pakette ara
strings sisman.pcapng | grep -i ctf
```

**FTP Server Bulundu:**
```
220 Welcome to CTF FTP Server (vsFTPd 3.0.5)
USER ctfplayer
```

**FTP Session Analizi:**
```bash
tshark -r sisman.pcapng -Y "ftp" -T fields -e frame.number -e ftp.request.command -e ftp.request.arg -e ftp.response.arg
```

**FTP İşlemleri:**

| Frame | Komut | Dosya | Açıklama |
|--------|--------|--------|----------|
| 64021 | `RETR` | `secret.txt` | Dosya indiriliyor |
| 64022 | `150` | `64 bytes` | Transfer başlıyor |
| 64023 | `FTP-DATA` | - | **Dosya içeriği** |
| 64024 | `226` | - | Transfer tamamlandı |

---

### 🔓 **Adım 5: Secret.txt İçeriğini Çıkarma**

```bash
# Frame 64023'teki FTP-DATA'yı hex dump olarak görüntüle
tshark -r sisman.pcapng -Y "frame.number == 64023" -x
```

**Hex Decode:**
```
70 69 74 6f 7b 66 6c 61 67 33 70 61 72 74 64 6f 73 74 75 6d 62 75 65 6e 6b 6f 6c 61 79 c4 b1 79 64 c4 b1 5f
```

**ASCII Çevirisi:**
```
pito{flag3partdostumbuenkolayıydı_
```

---

### 🧩 **Adım 6: Flag Parçalarını Birleştirme**

**Bulunan 3 Parça:**

1. **FTP secret.txt:** `pito{flag3partdostumbuenkolayıydı_`
2. **DNS subdomain'ler:** `_dnssubdomainlerinegizlendim`  
3. **Paket yorumu:** `_busismandosyadabuyorumuiyigordun}`

**Final Birleştirme:**
```
pito{flag3partdostumbuenkolayıydı + _dnssubdomainlerinegizlendim + _busismandosyadabuyorumuiyigordun}
```

---

## 🚩 **FLAG**
```
pito{flag3partdostumbuenkolayıydı_dnssubdomainlerinegizlendim_busismandosyadabuyorumuiyigordun}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>Tshark</b><br><sub>PCAP analizi ve filtering</sub></td>
<td align="center">🌐<br><b>DNS Analysis</b><br><sub>Hex-encoded subdomain decode</sub></td>
<td align="center">📡<br><b>FTP Investigation</b><br><sub>File transfer analizi</sub></td>
<td align="center">🔧<br><b>Hex Tools</b><br><sub>xxd, hex dump analizi</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🔍 Şişman — Forensics Challenge
       ↓
📁 sisman.pcapng (181MB, 197k paket)
   → Network capture analysis
       ↓
🧩 Paket Yorumları İnceleme
   → Frame 163038: "_busismandosyadabuyorumuiyigordun}"
   → Frame 163039: "analytics-cdn.net puzzle parçası"
       ↓
🌐 DNS Subdomain Analysis
   → Hex-encoded subdomains bulundu
   → Decode: "_dnssubdomainlerinegizlendim_"
       ↓
📡 FTP Server Keşfi
   → "CTF FTP Server" bulundu
   → secret.txt dosyası transfer edildi
       ↓
🔓 FTP-DATA Extraction
   → Frame 64023: Hex dump
   → Decode: "pito{flag3partdostumbuenkolayıydı_"
       ↓
🧩 3 Parça Birleştirme
   → FTP + DNS + Paket Yorumu
       ↓
🚩 FLAG: pito{flag3partdostumbuenkolayıydı_dnssubdomainlerinegizlendim_busismandosyadabuyorumuiyigordun}
```
