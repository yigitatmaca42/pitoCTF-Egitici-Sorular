# 📧 Derin - Forensics Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Forensics-Challenge-darkblue?style=for-the-badge" alt="Forensics">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Email%20Forensics-purple?style=for-the-badge" alt="Email Forensics">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Forensics  
**Seviye:** Orta  
**Puan:** 500

**Challenge Dosyası:** [📥 Google Drive - izleritakipet.pcap](https://drive.google.com/file/d/1b4j-3-8ZVZ8TL58TG_QmrDwk-wVj6qT9/view?usp=drivesdk)

---

## 🔍 Analiz

**Hedef:** pcap dosyasındaki network trafiğini analiz ederek gizli iletilen dosyayı ortaya çıkarmak

**Strateji:** pcap aç → protokolleri tespit et → SMTP trafiğini çıkar → email eklerini al → ZIP parçalarını birleştir → PDF'i oku → flag'i bul

**İpuçları:**
- 🎯 "Derin" → derin analiz / "izleritakipet" dosya adı → iz takip et
- 🔍 Port 25 (SMTP) ağırlıklı trafik → email trafiği forensics
- 💡 Her email `"Attached is a part of the file"` diyor → parçalı gizli dosya transfer
- 🔑 Her email kendi şifreli ZIP'ini ve şifresini taşıyor

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Dosya Analizi**

```bash
file izleritakipet.pcap
# pcap capture file, microsecond ts (little-endian) - version 2.4 (Ethernet)
# Boyut: 7 MB
```

**Temel istatistikler:**
- Toplam paket: **3.915**
- Süre: ~33 dakika (2001 saniye)
- Port 25 (SMTP): **972 paket** — ağırlıklı protokol
- Port 80 (HTTP): 2769 paket (sadece Ubuntu update trafiği)

---

### 🌐 **Adım 2: TCP Stream Analizi**

```python
# 64 TCP akışı tespit edildi
# SMTP akışları: 60
# HTTP akışları: 4 (sadece apt güncelleme)
```

**Kritik bulgular:**
- `192.168.68.108` → `192.168.68.111:25` → lokal mail sunucusuna gönderilen emailler
- `192.168.68.111:25` → `204.141.43.44:25` → Zoho Mail'e iletim (STARTTLS ile şifreli)
- Gönderen: `caleb@thephreaks.com`
- Alıcı: `resources@thetalents.com`
- Konu: **"Secure File Transfer"**

---

### 📧 **Adım 3: Email İçerik Analizi**

Her email aynı formatı izliyor:

```
From: caleb@thephreaks.com
To: resources@thetalents.com
Subject: Secure File Transfer

Text body: "Attached is a part of the file. Password: <PASSWORD>"
Attachment: <SHA256_HASH>.zip
```

**15 email, 2 dakikada bir gönderilmiş (14:59 → 15:27)**

---

### 📦 **Adım 4: ZIP Eklerini Çıkarma**

15 emailden ZIP dosyaları ve şifreleri parse edildi:

| # | Zaman | Şifre | Dosya (SHA256) |
|---|-------|-------|----------------|
| 1 | 14:59 | `S3W8yzixNoL8` | `caf334...efcfd.zip` |
| 2 | 15:01 | `r5Q6YQEcGWEF` | `2c586c...7dbf.zip` |
| 3 | 15:03 | `TVm9aC1UycxF` | `9026fb...b487.zip` |
| 4 | 15:05 | `jISlbC8145Ox` | `6c4491...6b0.zip` |
| 5 | 15:07 | `AdtJYhF4sFgv` | `76b329...2736.zip` |
| 6 | 15:09 | `j2SRRDraIvUZ` | `5dc7a1...b6ef.zip` |
| 7 | 15:11 | `xh161WSXX7tB` | `81cda6...701d.zip` |
| 8 | 15:13 | `yH5vqnkm7Ixa` | `882d61...6706.zip` |
| 9 | 15:15 | `tJPUTUfceO1P` | `961eaf...7703.zip` |
| 10 | 15:17 | `2qKlZHZlBPQz` | `c0c845...a347.zip` |
| 11 | 15:19 | `mbkUvLZ1koxu` | `0b094d...efe7.zip` |
| 12 | 15:21 | `ZN4yKAYrtf8x` | `b96e04...20bd.zip` |
| 13 | 15:23 | `0eA143t4432M` | `759994...4414.zip` |
| 14 | 15:25 | `oea41WCJrWwN` | `6cca03...81c1.zip` |
| 15 | 15:27 | `gdOvbPtB0xCK` | `d0034e...bf85.zip` |

---

### 🔓 **Adım 5: ZIP'leri Açma**

Her ZIP kendi şifresiyle açıldı:

```python
for zipfile, password in pairs:
    subprocess.run(['7z', 'x', f'-p{password}',
                    f'-o/extracted/', zipfile, '-y'])
```

Her ZIP içinde: `phreaks_plan.pdf.part<N>` (221 byte, toplam 15 parça)

---

### 🔗 **Adım 6: PDF Parçalarını Birleştirme**

```python
parts = sorted(files, key=lambda x: int(x.split('part')[-1]))
combined = b''.join(open(p,'rb').read() for p in parts)
# Header: %PDF → PDF document, version 1.3, 2 pages
```

---

### 📄 **Adım 7: PDF Okuma**

PDF 2 sayfalık bir operasyon belgesi: **"Operation Spotlight: The Phreaks' Grand Scheme Against The Talents"**

Son paragrafta flag gizli:

```
Appendix: Communication Protocols
For secure communication, all operatives are required to use encrypted
channels only. Key for secure communication: HTB{Th3Phr3aksReadyT0Att4ck}.
```

---

## 🚩 **FLAG**
```
HTB{Th3Phr3aksReadyT0Att4ck}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🐍<br><b>Python (struct)</b><br><sub>pcap parse / TCP reassembly</sub></td>
<td align="center">📧<br><b>Python (email)</b><br><sub>MIME ayrıştırma</sub></td>
<td align="center">🔓<br><b>7-Zip</b><br><sub>Şifreli ZIP açma</sub></td>
<td align="center">📄<br><b>pypdf</b><br><sub>PDF metin çıkarma</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
📁 izleritakipet.pcap (7 MB, 3915 paket)
       ↓
🌐 Protokol analizi
   → Port 25 (SMTP): 972 paket — ağırlıklı
   → 60 SMTP TCP akışı tespit edildi
       ↓
📧 Email analizi
   → caleb@thephreaks.com → resources@thetalents.com
   → Konu: "Secure File Transfer"
   → 15 email, 2dk aralıklarla (14:59 - 15:27)
       ↓
📦 Her emailden:
   ├── Text body → Şifre (ör: S3W8yzixNoL8)
   └── Attachment → Şifreli ZIP dosyası
       ↓
🔓 15 ZIP → 7z -p<şifre> ile açıldı
   → Her ZIP: phreaks_plan.pdf.part<N> (221 byte)
       ↓
🔗 15 parça birleştirildi
   → phreaks_plan.pdf (3302 byte, 2 sayfa)
       ↓
📄 PDF okundu
   → "Operation Spotlight" belgesi
   → Appendix'te flag gizli
       ↓
🚩 FLAG: HTB{Th3Phr3aksReadyT0Att4ck}
```
