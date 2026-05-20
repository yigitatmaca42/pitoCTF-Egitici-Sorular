# 🎯 Wifi - Forensics Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Forensics-Challenge-darkblue?style=for-the-badge" alt="Forensics">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-802.11 SSID Stego-purple?style=for-the-badge" alt="SSID Stego">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Forensics  
**Seviye:** Orta  
**Puan:** 500

**Challenge Dosyası:** [📥 Google Drive - Wifi.pcap](https://drive.google.com/file/d/1aget8LqjDY7iYJcqD5LjZE4grbsNvPZh/view?usp=drivesdk)

---

## 🔍 Analiz

**Hedef:** pcap dosyasını analiz ederek flag'i bulmak

**Strateji:** Dosya tipi tespiti → SSID'leri çek → Hex decode → Base64 decode → Flag

**İpuçları:**
- 🎯 802.11 WiFi paket yakalamı → SSID alanları incelenmeli
- 🔍 SSID'ler hex formatında → ASCII'ye çevrilmeli
- 💡 ASCII değerler Base64 encoded → decode edilmeli
- 🔑 Onlarca sahte SSID arasında flag gizlenmiş

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Dosyayı İncele**

```bash
file Wifi.pcap
```

**Çıktı:**
```
Wifi.pcap: pcap capture file, microsecond ts (little-endian) - version 2.4 
(802.11 with radiotap header, capture length 65535)
```

**Analiz:**
- ✅ 802.11 (WiFi) paket yakalamı
- 💡 Radiotap header ile kaydedilmiş
- 🎯 Beacon frame'lerdeki SSID'ler incelenmeli

---

### 📡 **Adım 2: SSID'leri Çek ve Decode Et**

Beacon frame'lerden (type_subtype == 8) SSID'leri çekip decode ettik:

```bash
tshark -r Wifi.pcap -Y "wlan.fc.type_subtype == 8" -T fields -e wlan.ssid 2>/dev/null | sort -u | while read hex; do
    ascii=$(echo "$hex" | xxd -r -p 2>/dev/null)
    decoded=$(echo "$ascii" | base64 -d 2>/dev/null)
    echo "HEX: $hex"
    echo "ASCII: $ascii"
    echo "DECODED: $decoded"
    echo "---"
done
```

**Analiz:**
- ✅ SSID'ler hex formatında geldi
- 💡 Hex → ASCII → Base64 decode zinciri uygulandı
- 🎯 Her SSID aslında base64 encoded bir mesaj içeriyor

---

### 🧠 **Adım 3: Decode Sonuçlarını İncele**

Onlarca sahte SSID arasında dikkat çeken mesajlar:

```
→ "7hi5 is n0t the n3twork name you 4r3 l0oking f0r"
→ "n0t h3re either"
→ "k33p s34rching"
→ "beacon fram3s, 50 h0t ri6ht n0w"
→ "wifi is my passion"
→ "wh3n in d0ubt, hack hard3r"
→ "y0u prob4bly shouldn't 7ry to do this manually"
→ "4l1 the c001 kid5 4r3 p14yin6 wi7h 802.11"
→ "10000o00000o00oo774 7r4ffic t0d4y"
```

**KRİTİK BULGU — Flag içeren SSID:**
```
HEX: 596d4e305a6e7430647a426663473878626e52664e46396e4d7a4e66597a42755a7a4e7a4e326b77626e304b
ASCII: YmN0Znt0dzBfcG8xbnRfNF9nMzNfYzBuZzNzN2kwbn0K
DECODED: bctf{tw0_po1nt_4_g33_c0ng3s7i0n}
```

**Analiz:**
- ✅ Hex → ASCII → Base64 decode zinciriyle flag elde edildi
- 💡 SSID'in ASCII değeri bir Base64 string'i
- 🎯 Base64 decode edince flag ortaya çıktı

---

## 🚩 **FLAG**
```
bctf{tw0_po1nt_4_g33_c0ng3s7i0n}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🦈<br><b>tshark</b><br><sub>SSID çekme</sub></td>
<td align="center">🔄<br><b>xxd</b><br><sub>Hex → ASCII</sub></td>
<td align="center">🔓<br><b>base64</b><br><sub>Base64 decode</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 Wifi Forensics Challenge
       ↓
📄 Wifi.pcap
   → 802.11 WiFi capture
   → Radiotap header
       ↓
📡 tshark ile Beacon frame analizi
   → wlan.fc.type_subtype == 8
   → SSID'ler hex olarak geldi
       ↓
🔄 Hex → ASCII dönüşümü (xxd)
   → Her SSID bir Base64 string'e dönüştü
   → Yüzlerce sahte mesaj: "not here", "k33p searching"...
       ↓
🔓 Base64 decode
   → Tek tek tüm SSID'ler decode edildi
   → Sahte yönlendirmeler arasında flag bulundu
       ↓
🚩 FLAG: bctf{tw0_po1nt_4_g33_c0ng3s7i0n}
```
