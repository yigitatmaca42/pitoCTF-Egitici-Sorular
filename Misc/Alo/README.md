# 📞 Alo - Misc Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Misc-Challenge-darkblue?style=for-the-badge" alt="Misc">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Network%20Forensics-purple?style=for-the-badge" alt="Network Forensics">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Misc  
**Seviye:** Orta  
**Puan:** 500

**Challenge Açıklaması:** Aloo sesin gelmiyor?

**Challenge Dosyası:** [📥 Google Drive - AloSesinGelmiyor.wav](https://drive.google.com/file/d/1AcnOo6t42tutCYjNlSJZFYL9k2yz-bdr/view?usp=drivesdk)

---

## 🔍 Analiz

**Hedef:** WAV dosyası içine gizlenmiş ağ trafiğini analiz edip flag'i çıkarmak

**Strateji:** WAV'ı raw byte incele → gizli pcapng bul → TCP akışlarını yeniden birleştir → HTTP'den görseli çıkar → OCR ile flag'i oku

**İpuçları:**
- 🎯 "Alo sesin gelmiyor" → iletişim/konuşma teması, network trafiği olabilir
- 🔍 WAV yalnızca 0.39 saniyelik ve çok küçük → gerçek ses değil, veri taşıyıcı
- 💡 Raw byte'larda `Dumpcap (Wireshark)` ve `AMD Ryzen` gibi metinler görünüyor
- 🔑 pcapng magic bytes `0x0a0d0d0a` offset 56'da mevcut

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Dosya Analizi**

```bash
file AloSesinGelmiyor.wav
# RIFF (little-endian) data, WAVE audio, Microsoft PCM, 16 bit, stereo 44100 Hz

ffprobe AloSesinGelmiyor.wav
# Duration: 00:00:00.39, bitrate: 1412 kb/s
```

**Analiz:**
- ✅ Geçerli bir WAV formatı ama yalnızca 0.39 saniye — şüpheli kısa
- 💡 `Packet corrupt` uyarısı → dosya aslında ses verisi içermiyor
- 🎯 İçerik muhtemelen başka bir formatta veri

---

### 🧠 **Adım 2: Raw Byte İncelemesi**

```python
with open('AloSesinGelmiyor.wav', 'rb') as f:
    raw = f.read()

# Printable string segmentleri tarandı
# Sonuç: okunabilir metinler bulundu!
```

**Bulunan kritik metinler:**
- `AMD Ryzen 7 7840HS w/ Radeon 780M Graphics`
- `Dumpcap (Wireshark) 4.2.2`
- `hello there host computer` / `hello there vm`
- `yeah i think it starts with bcactf{`
- `uhh... listening_ ... i forgot the rest but i have it in an image somewhere`
- `port 5500`
- `GET /rest_of_flag.jpg HTTP/1.1`

---

### 📦 **Adım 3: Gizli pcapng Dosyasının Tespiti**

WAV içinde **pcapng** (Wireshark network capture) magic bytes tespit edildi:

```python
# pcapng magic: 0x0a0d0d0a
pcapng_start = raw.find(b'\x0a\x0d\x0d\x0a')
# → Offset 56'da bulundu!

with open('capture.pcapng', 'wb') as f:
    f.write(raw[pcapng_start:])
```

**Çıktı:**
```
pcapng magic at offset: 56
Wrote 68736 bytes as capture.pcapng
JPEG found at offset: 11820
```

**Analiz:**
- ✅ WAV dosyasının audio data alanına pcapng gizlenmiş
- 💡 Wireshark 4.2.2 ile Linux 6.6.9 üzerinde alınmış bir capture
- 🎯 İçeride HTTP trafiği ve bir JPEG görseli var

---

### 🌐 **Adım 4: TCP Akışlarının Yeniden Birleştirilmesi**

pcapng dosyası manuel olarak parse edildi, EPB (Enhanced Packet Block) blokları ayrıştırıldı:

```python
import struct

# EPB blokları: block_type=0x00000006
# Ethernet → IPv4 → TCP payload extraction
# TCP akışları seq numarasına göre sıralandı ve birleştirildi
```

**Tespit edilen TCP akışları:**

| Kaynak | Hedef | Port | İçerik |
|--------|-------|------|--------|
| 10.0.2.15:60884 | 192.168.1.178:4999 | SpotUDP | Mesajlaşma |
| 192.168.1.178:5500 | 10.0.2.15:56780 | HTTP | rest_of_flag.jpg (46044 bytes) |

---

### 💬 **Adım 5: Network Trafiği — Konuşma**

Port 4999 (SpotUDP) üzerinden iki bilgisayar arasındaki mesajlaşma:

```
[10.0.2.15]      → hello there host computer
[192.168.1.178]  → hello there vm
[10.0.2.15]      → do you know what the flag is
[192.168.1.178]  → yeah i think it starts with bcactf{
[10.0.2.15]      → ok and the rest of it?
[192.168.1.178]  → uhh... listening_ ... i forgot the rest but i have
                   it in an image somewhere, i'll send it to you
[10.0.2.15]      → you still there?? it's been a long time
[192.168.1.178]  → yeah i'm sorry i'm literally just making the flag on the fly
                   port 5500
[10.0.2.15]      → ok lemme check it out
[10.0.2.15]      → thanks i think i got it hopefully
```

**Analiz:**
- ✅ Flag'in `bcactf{listening_` ile başladığı öğrenildi
- 💡 Geri kalanı bir görselde — HTTP üzerinden gönderilecek
- 🎯 Port 5500'deki HTTP sunucusundan `rest_of_flag.jpg` alındı

---

### 🖼️ **Adım 6: HTTP'den Görsel Extraction**

```python
# En büyük TCP akışı: 192.168.1.178:5500 → 10.0.2.15:56780
with open('stream_5500_56780.bin', 'rb') as f:
    data = f.read()  # 46400 bytes

hdr_end = data.find(b'\r\n\r\n')
jpeg_body = data[hdr_end+4:]
# → rest_of_flag.jpg: 46044 bytes, 1900x504 px JPEG
```

**HTTP Response:**
```
HTTP/1.1 200 OK
Content-Type: image/jpeg
Content-Length: 46044
```

---

### 🔡 **Adım 7: Flag Görselinden OCR**

```python
import pytesseract
from PIL import Image

img = Image.open('rest_of_flag.jpg')
# Boyut: 1900x504
text = pytesseract.image_to_string(img, config='--psm 7')
# Sonuç: "in_a28270fb0dbfd}"
```

---

### 🔗 **Adım 8: Flag'in Birleştirilmesi**

| Kaynak | İçerik |
|--------|--------|
| Chat mesajı | `bcactf{` |
| Chat mesajı | `listening_` |
| rest_of_flag.jpg (OCR) | `in_a28270fb0dbfd}` |

---

## 🚩 **FLAG**
```
bcactf{listening_in_a28270fb0dbfd}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🐍<br><b>Python (struct)</b><br><sub>pcapng manuel parse</sub></td>
<td align="center">🔍<br><b>ffprobe / file</b><br><sub>WAV metadata analizi</sub></td>
<td align="center">🖼️<br><b>Pillow (PIL)</b><br><sub>JPEG extraction</sub></td>
<td align="center">📝<br><b>Pytesseract</b><br><sub>OCR metin okuma</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
📞 Alo — Sesin Gelmiyor?
       ↓
📁 AloSesinGelmiyor.wav (WAV formatı ama ses yok!)
   → file / ffprobe → 0.39s, corrupt packet uyarısı
       ↓
🧠 Raw byte analizi
   → "Dumpcap (Wireshark)" metni bulundu
   → pcapng magic 0x0a0d0d0a → offset 56'da ✅
       ↓
📦 capture.pcapng (68 KB) extraction
   → EPB blokları parse edildi
   → TCP akışları seq'e göre reassemble edildi
       ↓
     ┌──────────┴──────────┐
     │                     │
  Port 4999              Port 5500
  (SpotUDP chat)         (HTTP)
     │                     │
  "bcactf{"           GET /rest_of_flag.jpg
  "listening_"             │
                     46044 bytes JPEG
                           │
                          OCR
                           │
                   "in_a28270fb0dbfd}"
       ↓
🚩 FLAG: bcactf{listening_in_a28270fb0dbfd}
```
