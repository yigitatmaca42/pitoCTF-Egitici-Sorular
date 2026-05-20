# 🎯 Pingin Kaç? - Forensics Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Forensics-Challenge-darkblue?style=for-the-badge" alt="Forensics">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Network%20Analysis-purple?style=for-the-badge" alt="Network Analysis">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Forensics  
**Seviye:** Kolay  
**Puan:** 300

**Challenge Dosyası:** [📥 Google Drive - Pinginkac.pcapng](https://drive.google.com/file/d/1OX2l1Ux6a3_NECDvNptPsroh1CBSZRi3/view?usp=drivesdk)

---

## 🔍 Analiz

**Hedef:** PCAP dosyasındaki ICMP ping trafiğini analiz ederek gizli veriyi bulmak

**Strateji:** Dosya tipi tespiti → Trafik analizi → ICMP TTL değerlerini çıkar → ASCII dönüşümü → Flag

**İpuçları:**
- 🎯 Soru adı "Pingin Kaç?" → ICMP ping paketleri şüpheli
- 🔍 Her ping request'inde TTL değeri farklı → Rastgele değil, gizli veri olabilir
- 💡 TTL değerleri ASCII karakter aralığında (33-125)
- 🌐 Network steganography — veri TTL alanına gizlenmiş

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Dosya Tipini Tespit Et**

```bash
file Pinginkac.pcapng
```

**Çıktı:**
```
Pinginkac.pcapng: pcapng capture file - version 1.0
```

**Analiz:**
- ✅ PCAP dosyası → Network trafik kaydı
- 💡 Wireshark veya tshark ile analiz edilebilir
- 🎯 İçinde ne var görmemiz lazım

---

### 🌐 **Adım 2: Trafik Genel Görünümü**

```bash
tshark -r Pinginkac.pcapng | head -50
```

**Çıktı:**
```
 1  0.000000  192.168.1.254 → 239.255.255.250  SSDP 371 NOTIFY * HTTP/1.1
 2  0.121112  192.168.1.254 → 239.255.255.250  SSDP 371 NOTIFY * HTTP/1.1
...
10  2.058867  192.168.1.233 → 192.168.1.26     ICMP 74 Echo (ping) request  id=0x0001, seq=39/9984,  ttl=78
11  2.058895  192.168.1.26  → 192.168.1.233    ICMP 74 Echo (ping) reply    id=0x0001, seq=39/9984,  ttl=64
12  2.605400  192.168.1.233 → 192.168.1.26     ICMP 74 Echo (ping) request  id=0x0001, seq=40/10240, ttl=72
13  2.605425  192.168.1.26  → 192.168.1.233    ICMP 74 Echo (ping) reply    id=0x0001, seq=40/10240, ttl=64
14  3.148258  192.168.1.233 → 192.168.1.26     ICMP 74 Echo (ping) request  id=0x0001, seq=41/10496, ttl=52
...
```

**Analiz:**
- ✅ SSDP ve ICMP trafiği var
- 🎯 ICMP ping request'lerinde TTL değerleri dikkat çekiyor
- 💡 Reply paketlerinde TTL hep `64` — sabit
- 🔍 Request paketlerinde TTL sürekli değişiyor: `78, 72, 52, 67...` → **Şüpheli!**
- 🎯 Soru adı "Pingin Kaç?" → TTL değerlerine odaklanacağız

---

### 📊 **Adım 3: ICMP Request TTL Değerlerini Çıkar**

```bash
tshark -r Pinginkac.pcapng -Y "icmp.type==8" -T fields -e icmp.seq -e ip.ttl
```

**Çıktı:**
```
39      78
40      72
41      52
42      67
43      75
44      123
45      65
46      110
47      52
48      108
49      121
50      90
51      33
52      110
53      71
54      95
55      112
56      67
57      52
58      112
59      83
60      95
61      83
62      51
63      51
64      109
65      115
66      95
67      84
68      48
69      95
70      98
71      51
72      95
73      70
74      117
75      78
76      33
77      125
```

**Analiz:**
- ✅ `icmp.type==8` → Sadece request paketleri filtrelendi
- 💡 TTL değerleri: `78, 72, 52, 67, 75...`
- 🎯 Bu değerler ASCII aralığında (33-125) → Karakter olabilir!
- 🔍 ASCII tablosuna bakarsak: `78=N`, `72=H`, `52=4`, `67=C`, `75=K` → **NH4CK** flag prefix'i!

---

### 🐍 **Adım 4: TTL Değerlerini ASCII'ye Çevir**

```bash
tshark -r Pinginkac.pcapng -Y "icmp.type==8" -T fields -e ip.ttl | python3 -c "
import sys
ttls = [int(x) for x in sys.stdin.read().split()]
print(''.join(chr(t) for t in ttls))
"
```

**Çıktı:**
```
NH4CK{An4lyZ!nG_pC4pS_S33ms_T0_b3_FuN!}
```

**Analiz:**
- 🎉 **FLAG BULUNDU!**
- ✅ Her TTL değeri bir ASCII karaktere karşılık geliyor
- 💡 Saldırgan veriyi ICMP paketlerinin TTL alanına gizlemiş
- 🎯 Bu teknik **network steganography** olarak bilinir

---

### 🚩 **Adım 5: Flag**

**TTL → ASCII Dönüşüm Tablosu (İlk 5):**

| Seq | TTL | ASCII |
|-----|-----|-------|
| 39  | 78  | N     |
| 40  | 72  | H     |
| 41  | 52  | 4     |
| 42  | 67  | C     |
| 43  | 75  | K     |

**FLAG:**
```
NH4CK{An4lyZ!nG_pC4pS_S33ms_T0_b3_FuN!}
```

---

## 🚩 **FLAG**
```
NH4CK{An4lyZ!nG_pC4pS_S33ms_T0_b3_FuN!}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🦈<br><b>tshark</b><br><sub>PCAP analizi</sub></td>
<td align="center">🐍<br><b>Python</b><br><sub>ASCII dönüşümü</sub></td>
<td align="center">🔍<br><b>grep</b><br><sub>Paket filtreleme</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 Pingin Kaç? Challenge
       ↓
📂 Dosyayı incele
   → file Pinginkac.pcapng
   → pcapng capture file
       ↓
🌐 Trafiği analiz et
   → tshark ile genel görünüm
   → SSDP + ICMP ping trafiği var
   → Request TTL değerleri değişiyor: 78, 72, 52...
   → Reply TTL hep sabit: 64
       ↓
📊 TTL değerlerini çıkar
   → tshark -Y "icmp.type==8" -T fields -e ip.ttl
   → 39 farklı TTL değeri
   → Hepsi ASCII aralığında (33-125)
       ↓
🐍 ASCII'ye çevir
   → python3 ile chr() dönüşümü
   → NH4CK{An4lyZ!nG_pC4pS_S33ms_T0_b3_FuN!}
       ↓
🚩 FLAG
   → NH4CK{An4lyZ!nG_pC4pS_S33ms_T0_b3_FuN!}
```
