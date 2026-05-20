# 🎯 Old School - Forensics Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Forensics-Challenge-darkblue?style=for-the-badge" alt="Forensics">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-ICMP Steganography-purple?style=for-the-badge" alt="ICMP">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Forensics  
**Seviye:** Orta  
**Puan:** 400

**Challenge Dosyası:** [📥 Google Drive - Oldschool.pcap](https://drive.google.com/file/d/1RvAcX4kq3GQMXVMW_Ae5XcqZiVvYuoP4/view?usp=drivesdk)

---

## 🔍 Çözüm

### Adım 1: PCAP Dosyasını İnceleme
```bash
# Dosya tipini kontrol et
file Oldschool.pcap

# Temel bilgileri öğren
capinfos Oldschool.pcap
```

**Çıktı:**
```
File type:           Wireshark/tcpdump/... - pcap
Number of packets:   500+
```

---

### Adım 2: Protokol Analizi
```bash
# Protokol dağılımını kontrol et
tshark -r Oldschool.pcap -T fields -e _ws.col.Protocol | sort -u
```

**Sonuç:**
```
ARP
DNS
FTP
HTTP
ICMP
TCP
```

**Analiz:** Karışık trafik - FTP, HTTP ve ICMP paketleri mevcut

---

### Adım 3: FTP Trafiğini İnceleme
```bash
# FTP komutlarını listele
strings Oldschool.pcap | grep -E "220|230|USER|PASS|RETR|LIST|CWD"
```

**Gözlemler:**
- **FTP Banner:** `220 FTP Server - Flag? bulamazki :)` 
  - "bulamazki" = Türkçe: "sen bulamazsın" (İpucu!)
- **Kullanıcı:** anonymous / guest@
- **Komutlar:** USER, PASS, CWD /pub, LIST, RETR file.txt
```bash
# Unique FTP response'larını bul
strings Oldschool.pcap | grep "^220 " | sort -u
```

**FTP Anomalisi:**
```
220 CWD /pub
220 FTP Server - Flag? bulamazki :)
220 LIST
220 PASS guest@
220 RETR file.txt
220 USER anonymous
```

**Gözlem:** Normal FTP'de "220" sadece banner için kullanılır, komutlarla birlikte görülmez - Bu şüpheli!

---

### Adım 4: HTTP Trafiğini Kontrol Etme
```bash
# HTTP içinde flag ara
strings Oldschool.pcap | grep -i "flag"
```

**Sonuç:**
```
flag{fakeflag}
flag{fakeflag}
flag{fakeflag}
```

**Analiz:** HTTP trafiğindeki tüm flag'ler sahte (decoy)

---

### Adım 5: ICMP Trafiğini Keşfetme
```bash
# ICMP paketlerini listele
tshark -r Oldschool.pcap -Y "icmp" -T fields -e icmp.seq
```

**ICMP Sequence Numaraları:**
```
1, 10, 24, 22, 27, 6, 12, 17, 11, 0, 14, 23, 15, 28, 20, 5, 3, 7, 4, 18, 19, 2, 25, 21, 9, 16, 26, 13, 8
```

**Gözlem:** Sequence'ler karışık sırada! Normal ICMP ping'de ardışık olmalı (0,1,2,3...)

---

### Adım 6: ICMP Data Analizi
```bash
# ICMP paketlerinin data kısmını incele
tshark -r Oldschool.pcap -Y "icmp.type == 8" -T fields -e icmp.seq -e data.data
```

**Seq -> Data Mapping:**
```
Seq 0:  6d (hex)
Seq 1:  69
Seq 2:  72
Seq 3:  73
...
Seq 28: 65
```

**Her ICMP paketi tek bir byte HEX data taşıyor!**

---

### Adım 7: Sequence Sırasına Göre Decode
```bash
# Sequence sırasına göre hex'i ASCII'ye çevir
python3 << 'EOF'
import subprocess

result = subprocess.run(['tshark', '-r', 'Oldschool.pcap', '-Y', 'icmp.type == 8', 
                        '-T', 'fields', '-e', 'icmp.seq', '-e', 'data.data'], 
                       capture_output=True, text=True)

packets = {}
for line in result.stdout.strip().split('\n'):
    parts = line.split('\t')
    if len(parts) >= 2:
        seq = int(parts[0])
        data = parts[1]
        packets[seq] = data

# Sequence sırasına göre
flag = ''
for i in range(max(packets.keys()) + 1):
    if i in packets:
        hex_data = packets[i].replace(':', '')
        char = chr(int(hex_data[:2], 16))
        flag += char
        print(f"Seq {i}: {hex_data[:2]} -> {char}")

print(f"\nSequence sırası: {flag}")
EOF
```

**Çıktı:**
```
Seq 0: 6d -> m
Seq 1: 69 -> i
Seq 2: 72 -> r
Seq 3: 73 -> s
...
Seq 28: 65 -> e

Sequence sırası: mirsnebsirars}dlltaia{ankiFge
```

**Analiz:** Karışık! `}` önce, `{` sonra geliyor - Flag formatı yanlış!

---

### Adım 8: Timestamp Sırasına Göre Decode

**"Ben karıştırdım, sen sırala" ipucunu hatırla!**
```bash
# Timestamp sırasına göre sırala
tshark -r Oldschool.pcap -Y "icmp.type == 8" -T fields \
  -e frame.time_epoch -e icmp.seq -e data.data | sort -n
```

**Timestamp Sırası:**
```
1704067200.0    Seq 26: 46
1704067200.5    Seq 16: 6c
1704067201.0    Seq 18: 61
1704067201.5    Seq 27: 67
1704067202.0    Seq 21: 7b
...
```
```bash
# Timestamp sırasına göre ASCII'ye çevir
python3 << 'EOF'
import subprocess

result = subprocess.run(['tshark', '-r', 'Oldschool.pcap', '-Y', 'icmp.type == 8', 
                        '-T', 'fields', '-e', 'frame.time_epoch', '-e', 'icmp.seq', '-e', 'data.data'], 
                       capture_output=True, text=True)

packets = []
for line in result.stdout.strip().split('\n'):
    parts = line.split('\t')
    if len(parts) >= 3:
        timestamp = float(parts[0])
        seq = int(parts[1])
        data = parts[2]
        packets.append((timestamp, seq, data))

# Timestamp'e göre sırala
packets.sort(key=lambda x: x[0])

# ASCII'ye çevir
flag = ''
print("Timestamp sırasına göre:")
for timestamp, seq, data in packets:
    hex_data = data.replace(':', '')
    char = chr(int(hex_data[:2], 16))
    flag += char
    print(f"Time: {timestamp}, Seq {seq}: {hex_data[:2]} -> {char}")

print(f"\n🚩 FLAG: {flag}")
EOF
```

**Decoding:**
```
Time: 1704067200.0, Seq 26: 46 → 'F'
Time: 1704067200.5, Seq 16: 6c → 'l'
Time: 1704067201.0, Seq 18: 61 → 'a'
Time: 1704067201.5, Seq 27: 67 → 'g'
Time: 1704067202.0, Seq 21: 7b → '{'
Time: 1704067202.5, Seq 6:  62 → 'b'
Time: 1704067203.0, Seq 5:  65 → 'e'
Time: 1704067203.5, Seq 23: 6e → 'n'
Time: 1704067204.0, Seq 24: 6b → 'k'
Time: 1704067204.5, Seq 10: 61 → 'a'
Time: 1704067205.0, Seq 2:  72 → 'r'
Time: 1704067205.5, Seq 8:  69 → 'i'
Time: 1704067206.0, Seq 7:  73 → 's'
Time: 1704067206.5, Seq 17: 74 → 't'
Time: 1704067207.0, Seq 1:  69 → 'i'
Time: 1704067207.5, Seq 9:  72 → 'r'
Time: 1704067208.0, Seq 14: 64 → 'd'
Time: 1704067208.5, Seq 19: 69 → 'i'
Time: 1704067209.0, Seq 0:  6d → 'm'
Time: 1704067209.5, Seq 12: 73 → 's'
Time: 1704067210.0, Seq 28: 65 → 'e'
Time: 1704067210.5, Seq 4:  6e → 'n'
Time: 1704067211.0, Seq 3:  73 → 's'
Time: 1704067211.5, Seq 25: 69 → 'i'
Time: 1704067212.0, Seq 11: 72 → 'r'
Time: 1704067212.5, Seq 22: 61 → 'a'
Time: 1704067213.0, Seq 15: 6c → 'l'
Time: 1704067213.5, Seq 20: 61 → 'a'
Time: 1704067214.0, Seq 13: 7d → '}'
```

---

## 🚩 Flag
```
Flag{benkaristirdimsensirala}
```

**Türkçe Anlamı:** "Ben karıştırdım, sen sırala" 😄

---

## 💡 Açıklama

Bu challenge'da **ICMP Covert Channel via Timing-based Steganography** zafiyeti bulunmaktadır.

**Saldırı Yöntemi:**
1. ICMP ping paketleri kullanılarak gizli veri taşınıyor
2. Her ICMP paketinin data kısmında tek bir ASCII karakter (hex formatında)
3. Sequence numaraları kasıtlı olarak karıştırılmış
4. Mesaj timestamp (gönderilme zamanı) sırasına göre decode edilmeli

**Yanıltma Teknikleri:**
- FTP trafiği ile dikkat dağıtma
- HTTP'de sahte flag'ler (`flag{fakeflag}`)
- FTP banner'da "bulamazki :)" mesajı (psikolojik oyun)
- Sequence numaralarını karıştırarak kolay çözümü engelleme

**Tespit Yöntemi:**
- ICMP sequence'lerinin ardışık olmaması
- Her ICMP paketinde sadece 1 byte data
- Timestamp'lere göre sıralandığında anlamlı mesaj
- "Old School" adı → Klasik timing-based steganography

**Güvenlik Önerileri:**
- ICMP trafiğini monitör et
- Anomali tespiti uygula (düzensiz sequence'ler)
- ICMP payload boyutlarını kontrol et
- Timing pattern analizi yap
- Firewall'da gereksiz ICMP trafiğini engelle

---

## 🛠️ Kullanılan Araçlar

- **Wireshark/tshark** - PCAP analizi
- **strings** - String extraction
- **Python** - Data processing ve automation
- **xxd** - Hex to ASCII dönüşümü
- **bash** - Scripting

---

## 📊 Saldırı Akış Şeması
```
🎯 Old School Challenge
       ↓
📦 PCAP dosyasını aç
   → 500+ paket, karışık trafik
       ↓
🔍 Protokol analizi
   → FTP, HTTP, ICMP, DNS, ARP
       ↓
🚫 FTP trafiği inceleme
   → "220 FTP Server - Flag? bulamazki :)"
   → Anomali: Komutlarla birlikte 220 response
   → Yanıltma amaçlı
       ↓
🚫 HTTP trafiği kontrol
   → flag{fakeflag} (sahte)
       ↓
🔍 ICMP paketleri keşfi
   → Sequence'ler karışık: 1,10,24,22,27...
   → Normal: 0,1,2,3,4... olmalı
       ↓
📡 ICMP Data analizi
   → Her pakette 1 byte hex data
   → Seq 0: 6d, Seq 1: 69, Seq 2: 72...
       ↓
🔢 Sequence sırasına göre decode
   → mirsnebsirars}dlltaia{ankiFge
   → Karışık! } önce, { sonra
       ↓
⏰ Timestamp sırasına göre decode
   → ICMP paketlerini gönderilme zamanına göre sırala
       ↓
💡 ASCII dönüşümü
   → 46→F, 6c→l, 61→a, 67→g, 7b→{
       ↓
🚩 Flag{benkaristirdimsensirala}
```
