# 🎯 Detay - Forensics Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Forensics-Challenge-darkblue?style=for-the-badge" alt="Forensics">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Network Analysis-purple?style=for-the-badge" alt="Network Analysis">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Forensics  
**Seviye:** Orta  
**Puan:** 500

**Challenge Dosyası:** [📥 Google Drive - AmacinNeydiki.pcapng](https://drive.google.com/file/d/1OtDcr7GHwl6GFToRPP7VlNw2DvLGu0BJ/view?usp=drivesdk)

---

## 🔍 Analiz

**Hedef:** Pcap dosyasındaki ağ trafiğini inceleyerek `/flag` ve `/fl4g` isteklerinin sırasında gizlenmiş binary mesajı çözmek

**Strateji:** Protokol analizi → HTTP trafik incelemesi → Binary encoding tespiti → Flag extraction

**İpuçları:**
- 🎯 `.pcapng` dosyası → Ağ trafiği kaydı
- 🔍 HTTP trafiği → `/flag` ve `/fl4g` endpoint'leri
- 💡 İki farklı URI → Binary encoding (0/1) olabilir
- 🔑 500 puan = Detaylı analiz gerektirir

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Protokol Hiyerarşisini İncele**

```bash
tshark -r AmacinNeydiki.pcapng -q -z io,phs
```

**Çıktı:**
```
===================================================================
Protocol Hierarchy Statistics
Filter: 

frame                                    frames:1982 bytes:165356
  null                                   frames:1982 bytes:165356
    ip                                   frames:1975 bytes:164044
      tcp                                frames:1974 bytes:163968
        http                             frames:288 bytes:39818
          data-text-lines                frames:144 bytes:10298
      udp                                frames:1 bytes:76
        data                             frames:1 bytes:76
    ipv6                                 frames:7 bytes:1312
      udp                                frames:7 bytes:1312
        mdns                             frames:7 bytes:1312
===================================================================
```

**Analiz:**
- ✅ 288 HTTP frame mevcut
- 💡 144 data-text-lines → Response'lar
- 🎯 1 UDP data paketi → İlginç, incelenecek
- 🔑 HTTP trafiği ana odak noktası

---

### 📊 **Adım 2: HTTP İsteklerini İncele**

```bash
tshark -r AmacinNeydiki.pcapng -Y http -T fields -e http.request.method -e http.request.uri -e http.response.code 2>/dev/null | head -20
```

**Çıktı:**
```
GET     /flag   400
GET     /fl4g   401
GET     /flag   400
GET     /fl4g   401
...
```

**Analiz:**
- ✅ İki farklı endpoint: `/flag` → 400, `/fl4g` → 401
- 💡 400: "Flag's not here", 401: "Flag Not Found"
- 🎯 Sürekli tekrarlanan istekler → Pattern olabilir
- 🔑 İki farklı URI → Binary encoding şüphesi

---

### 🔍 **Adım 3: Response Body'leri Decode Et**

```bash
tshark -r AmacinNeydiki.pcapng -Y "http" -T fields -e http.response.code -e http.file_data 2>/dev/null | sort -u
```

**Çıktı:**
```
400     466c61672773206e6f7420686572650a
401     466c6167204e6f7420466f756e640a
```

```bash
echo "466c61672773206e6f7420686572650a" | xxd -r -p
echo "466c6167204e6f7420466f756e640a" | xxd -r -p
```

**Çıktı:**
```
Flag's not here
Flag Not Found
```

**Analiz:**
- ✅ Response body'ler decode edildi
- 💡 Flag yok, sadece yanıltıcı mesajlar
- 🎯 Asıl veri URI sırasında gizli olabilir

---

### 🎯 **Adım 4: Binary Encoding ile Flag'i Çöz**

`/flag` = 0, `/fl4g` = 1 olarak kabul edip sıradaki istekleri binary'ye çevirelim.

```bash
tshark -r AmacinNeydiki.pcapng -Y "http.request" -T fields -e tcp.srcport -e http.request.uri 2>/dev/null | python3 -c "
import sys
bits = []
for line in sys.stdin:
    parts = line.strip().split()
    if len(parts) == 2:
        uri = parts[1]
        if uri == '/flag':
            bits.append('0')
        elif uri == '/fl4g':
            bits.append('1')

print('Bits:', ''.join(bits))
print('Toplam:', len(bits))

binary = ''.join(bits)
chars = []
for i in range(0, len(binary)-7, 8):
    byte = int(binary[i:i+8], 2)
    chars.append(chr(byte))
print('Decoded:', ''.join(chars))
"
```

**Çıktı:**
```
Bits: 010100100101001101111011010100100110010101100110011100100011001101110011011010000101010001101000001100110101000000110100011001110011001101111101
Toplam: 144
Decoded: RS{Refr3shTh3P4g3}
```

**KRİTİK BULGULAR:**
- ✅ **FLAG BULUNDU!** 🎉
- 💡 144 bit → 18 karakter
- 🎯 `/flag` = 0, `/fl4g` = 1 binary encoding
- 🔑 İsteklerin sırası gizli mesajı oluşturuyordu

---

## 🚩 **FLAG**
```
RS{Refr3shTh3P4g3}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🦈<br><b>tshark</b><br><sub>Network analysis</sub></td>
<td align="center">🐍<br><b>Python</b><br><sub>Binary decode</sub></td>
<td align="center">🔧<br><b>xxd</b><br><sub>Hex decode</sub></td>
<td align="center">🐚<br><b>bash</b><br><sub>Command line</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 Detay Forensics Challenge
       ↓
📂 Dosyayı incele
   → AmacinNeydiki.pcapng
   → 1982 frame, 288 HTTP frame
   → 144 data-text-lines
       ↓
📊 HTTP trafik analizi
   → GET /flag → 400 "Flag's not here"
   → GET /fl4g → 401 "Flag Not Found"
   → İki farklı URI sürekli tekrarlanıyor
       ↓
🔍 Pattern tespiti
   → /flag = 0, /fl4g = 1
   → 144 istek = 144 bit = 18 karakter
       ↓
💡 Binary decode
   → İsteklerin sırası binary'ye çevrildi
   → 8 bitlik gruplara bölündü
   → ASCII karakterlere dönüştürüldü
       ↓
✅ FLAG: RS{Refr3shTh3P4g3}
```
