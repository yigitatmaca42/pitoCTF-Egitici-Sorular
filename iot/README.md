# 🎯 iot - IoT Challenge

<p align="center">
  <img src="https://img.shields.io/badge/IoT-Challenge-darkblue?style=for-the-badge" alt="IoT">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-MQTT Steganography-purple?style=for-the-badge" alt="MQTT">
</p>

---

## 📋 Challenge Bilgileri

**Kategori:** IoT  
**Zorluk:** Orta  
**Puan:** 500

**Açıklama:** Bir siber güvenlik firması olarak, şüpheli ağ trafiği yakalayan bir müşteriniz var. Evindeki akıllı ev sistemi normal çalışıyor gibi görünüyor ama garip davranışlar fark etmiş. Sizden pcap dosyasını analiz etmenizi ve bir sorun olup olmadığını belirlemenizi istiyorlar.

**Challenge Dosyası:** [📥 Google Drive - iot_challenge.pcap](https://drive.google.com/file/d/1sZ37scM65ue5-jDOYOiLIjND0bJG_rvR/view?usp=drivesdk)

---

## 🔍 Çözüm

### Adım 1: PCAP Dosyasını İnceleme

```bash
# Dosya tipini kontrol et
file iot_challenge.pcap

# Temel bilgileri öğren
capinfos iot_challenge.pcap
```

**Çıktı:**
```
File name:           iot_challenge.pcap
Number of packets:   55
File size:           7,563 bytes
Capture duration:    257.000000 seconds
```

---

### Adım 2: Protokol Analizi

```bash
# Protokol dağılımını kontrol et
tshark -r iot_challenge.pcap -qz io,phs
```

**Sonuç:**
```
Protocol Hierarchy Statistics
frame                    frames:55 bytes:6659
  eth                    frames:55 bytes:6659
    ip                   frames:55 bytes:6659
      tcp                frames:55 bytes:6659
        mqtt             frames:52 bytes:6497
```

**Analiz:** Tüm trafik MQTT protokolü kullanıyor (IoT cihazlar için yaygın)

---

### Adım 3: MQTT Trafiğini İnceleme

```bash
# MQTT mesajlarını listele
tshark -r iot_challenge.pcap -Y mqtt
```

**Gözlemler:**
- **Topic'ler:**
  - `home/kitchen/temperature` (sıcaklık sensörü)
  - `home/livingroom/light` (ışık kontrolü)
  - `home/bedroom/motion` (hareket sensörü)

- **Mesaj formatı:** Hex encoded JSON

---

### Adım 4: MQTT Payload'larını Decode Etme

```bash
# Hex mesajları ASCII'ye çevir
echo "7b2274656d70223a2032322e37302c2022756e6974223a202243222c202273656e736f725f6964223a202274656d705f303031227d" | xxd -r -p
```

**Çıktı:**
```json
{"temp": 22.70, "unit": "C", "sensor_id": "temp_001"}
{"status": "on", "brightness": 75}
{"motion": false}
```

---

### Adım 5: Sıcaklık Değerlerini Analiz Etme

```bash
# Tüm sıcaklık değerlerini çıkar
tshark -r iot_challenge.pcap -Y 'mqtt.topic == "home/kitchen/temperature"' \
  -T fields -e mqtt.msg | while read hex; do
    echo "$hex" | xxd -r -p | grep -oP '"temp":\s*\K[0-9.]+'
done
```

**Sıcaklık değerleri:**
```
22.70
22.76
22.65
22.71
22.123
22.105
22.111
22.116
...
```

**Gözlem:** Bazı değerler 3 haneli ondalık kısma sahip (22.123, 22.105) - Bu şüpheli!

---

### Adım 6: Steganografi Keşfi

Sıcaklık değerlerinin ondalık kısımları ASCII değerleri gibi görünüyor:

```bash
# Ondalık kısımları çıkar ve ASCII'ye çevir
tshark -r iot_challenge.pcap -Y 'mqtt.topic == "home/kitchen/temperature"' \
  -T fields -e mqtt.msg | while read hex; do
    temp=$(echo "$hex" | xxd -r -p | grep -oP '"temp":\s*\K[0-9.]+')
    decimal=$(echo "$temp" | cut -d. -f2)
    printf "\x$(printf %x $decimal)"
done
```

**Decoding:**
```
70  → 'F'
76  → 'L'
65  → 'A'
71  → 'G'
123 → '{'
105 → 'i'
111 → 'o'
116 → 't'
105 → 'i'
108 → 'l'
101 → 'e'
111 → 'o'
114 → 'r'
116 → 't'
97  → 'a'
109 → 'm'
105 → 'i'
98  → 'b'
105 → 'i'
114 → 'r'
97  → 'a'
122 → 'z'
105 → 'i'
115 → 's'
105 → 'i'
116 → 't'
97  → 'a'
108 → 'l'
105 → 'i'
109 → 'm'
100 → 'd'
101 → 'e'
100 → 'd'
105 → 'i'
107 → 'k'
125 → '}'
```

---

## 🚩 Flag

```
FLAG{iotileortamibirazisitalimdedik}
```

---

## 💡 Açıklama

Bu challenge'da **IoT Data Exfiltration via MQTT Steganography** zafiyeti bulunmaktadır.

**Saldırı Yöntemi:**
1. IoT cihazı compromise edilmiş
2. Normal görünen sensör verilerinde gizli veri taşınıyor
3. Sıcaklık değerlerinin ondalık kısmında ASCII karakterler gizlenmiş
4. Her sıcaklık okuması (22.XXX formatında) bir karakter içeriyor

**Tespit Yöntemi:**
- Normal sıcaklık değerleri: ~22.1°C civarında beklenir
- Ancak bazı değerler 22.70, 22.123 gibi anormal
- Bu değerlerin ondalık kısmı ASCII karakter kodlarına karşılık geliyor

**Güvenlik Önerileri:**
- IoT trafiğini şifrele (TLS/SSL)
- Anomali tespiti uygula
- Sensör değerlerini validate et
- Normal değer aralıklarını tanımla
- MQTT authentication kullan

---

## 🛠️ Kullanılan Araçlar

- **Wireshark/tshark** - PCAP analizi
- **xxd** - Hex to ASCII dönüşümü
- **bash** - Scripting ve automation
- **jq** - JSON parsing

---

## 📊 Saldırı Akış Şeması

```
🎯 IoT Challenge
       ↓
📦 PCAP dosyasını aç
   → 55 paket, 257 saniye
       ↓
🔍 Protokol analizi
   → %100 MQTT trafiği
       ↓
📡 MQTT topic'leri
   → home/kitchen/temperature
   → home/livingroom/light
   → home/bedroom/motion
       ↓
🔓 Hex payload'ları decode
   → JSON formatında sensör verileri
       ↓
🌡️ Sıcaklık değerleri
   → 22.70, 22.76, 22.65, 22.71, 22.123...
       ↓
🔎 Anomali tespiti
   → 3 haneli ondalık kısımlar şüpheli
       ↓
💡 Steganografi keşfi
   → Ondalık kısım = ASCII kod
       ↓
🔢 ASCII dönüşümü
   → 70→F, 76→L, 65→A, 71→G, 123→{
       ↓
🚩 FLAG{iotileortamibirazisitalimdedik}
```
