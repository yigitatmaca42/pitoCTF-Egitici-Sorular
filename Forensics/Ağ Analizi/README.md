# 🎯 Ağ Analizi - Forensics Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Forensics-Challenge-darkblue?style=for-the-badge" alt="Forensics">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Network Analysis-purple?style=for-the-badge" alt="Network Analysis">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Forensics  
**Seviye:** Kolay  
**Puan:** 100

**Challenge Dosyası:** [📥 Google Drive - Ağ Analizi](https://drive.google.com/file/d/1IEeDGi1gS5afd1u9GXffaKRdFy1lTVy5/view?usp=drivesdk)

---

## 🔍 Analiz

**Hedef:** Pcap dosyasındaki ağ trafiğini inceleyerek gizlenmiş flag'i bulmak

**Strateji:** Protocol analysis → Traffic inspection → String extraction → Flag extraction

**İpuçları:**
- 🎯 `.pcapng` dosyası → Ağ trafiği kaydı
- 🔍 Birden fazla protokol → FTP, HTTP, SMTP incelenecek
- 💡 Dosya transferi olabilir → İçerikler kontrol edilmeli
- 🔑 100 puan = Kolay challenge → Temel analiz yeterli

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Protokol Hiyerarşisini İncele**

```bash
tshark -r AgAnalizi101.pcapng -q -z io,phs
```

**Çıktı:**
```
===================================================================
Protocol Hierarchy Statistics
Filter: 

frame                                    frames:177 bytes:16627
  eth                                    frames:177 bytes:16627
    ip                                   frames:148 bytes:13797
      udp                                frames:31 bytes:4310
        dns                              frames:6 bytes:568
        dhcp                             frames:2 bytes:684
        nbns                             frames:20 bytes:2200
        nbdgm                            frames:3 bytes:858
          smb                            frames:3 bytes:858
      icmp                               frames:8 bytes:784
      tcp                                frames:109 bytes:8703
        ftp                              frames:42 bytes:3629
        http                             frames:2 bytes:591
          data-text-lines                frames:1 bytes:387
        smtp                             frames:3 bytes:279
    ipv6                                 frames:23 bytes:2578
    arp                                  frames:6 bytes:252
===================================================================
```

**Analiz:**
- ✅ Toplam 177 frame, 16627 byte
- 💡 FTP trafiği mevcut: 42 frame → Dosya transferi olabilir
- 🎯 HTTP trafiği mevcut: 2 frame + 1 data-text-lines
- 🔍 SMTP trafiği mevcut: 3 frame → E-posta trafiği

---

### 📊 **Adım 2: TCP Konuşmalarını İncele**

```bash
tshark -r AgAnalizi101.pcapng -q -z conv,tcp
```

**Çıktı:**
```
================================================================================
TCP Conversations
                                                           |       <-      | |       ->      | |     Total     |
192.168.64.4:42332  <-> 192.168.64.3:21       31 2,757B    34 2,406B    65 5,163B   103.376s   188.05s
192.168.64.4:52488  <-> 192.168.64.3:80        5   659B     6   542B    11 1,201B   313.581s     0.14s
192.168.64.4:36906  <-> 192.168.64.3:25        4   343B     5   348B     9   691B   362.572s     3.98s
192.168.64.4:47668  <-> 192.168.64.3:9978      3   206B     3   206B     6   412B   150.375s     4.19s
192.168.64.4:42746  <-> 192.168.64.3:24456     3   206B     3   206B     6   412B   154.567s     0.00s
192.168.64.4:54450  <-> 192.168.64.3:35100     3   206B     3   206B     6   412B   157.942s     0.00s
192.168.64.4:39378  <-> 192.168.64.3:44983     3   206B     3   206B     6   412B   162.476s   128.95s
================================================================================
```

**Analiz:**
- ✅ `192.168.64.4:42332 ↔ 192.168.64.3:21` → **FTP bağlantısı** (port 21), 188 saniye sürmüş, en büyük trafik!
- ✅ `192.168.64.4:52488 ↔ 192.168.64.3:80` → **HTTP bağlantısı** (port 80)
- ✅ `192.168.64.4:36906 ↔ 192.168.64.3:25` → **SMTP bağlantısı** (port 25)
- 🎯 FTP + HTTP kombinasyonu → Dosya yüklenip web'den erişilmiş olabilir

---

### 🔍 **Adım 3: String Arama ile Flag Bul**

```bash
strings AgAnalizi101.pcapng | grep -i "flag\|CTF\|PITO\|pito\|{" | head -20
```

**Çıktı:**
```
STOR NotFlag.txt
STOR NotFlag.txt
GET /NotFlag.txt HTTP/1.1
flag{You_Found_Me_#3019}
/NotFlag.txt
/NotFlag.txt    200
```

**KRİTİK BULGULAR:**
- ✅ **FLAG BULUNDU!** 🎉
- 💡 FTP ile `NotFlag.txt` adlı dosya yüklenmiş (`STOR` komutu)
- 🎯 HTTP ile aynı dosyaya `GET` isteği yapılmış
- 🚩 Dosya içeriği: `flag{You_Found_Me_#3019}`
- ✅ HTTP 200 response → Dosyaya başarıyla erişilmiş

---

## 🚩 **FLAG**
```
flag{You_Found_Me_#3019}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🦈<br><b>tshark</b><br><sub>Network analysis</sub></td>
<td align="center">🔍<br><b>strings</b><br><sub>String extraction</sub></td>
<td align="center">🐚<br><b>bash</b><br><sub>Command line</sub></td>
<td align="center">🔧<br><b>grep</b><br><sub>Pattern matching</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 Ağ Analizi Challenge
       ↓
📂 Dosyayı incele
   → AgAnalizi101.pcapng
   → 177 frame, 16627 byte
       ↓
📊 Protokol hiyerarşisi
   → FTP: 42 frame ✅ (en büyük trafik)
   → HTTP: 2 frame ✅
   → SMTP: 3 frame
   → DNS, DHCP, ICMP...
       ↓
🔗 TCP konuşmaları
   → 192.168.64.4 ↔ 192.168.64.3:21 (FTP) - 188sn
   → 192.168.64.4 ↔ 192.168.64.3:80 (HTTP)
   → FTP + HTTP → Dosya transferi!
       ↓
🔍 String arama
   → strings | grep flag
   → STOR NotFlag.txt (FTP upload)
   → GET /NotFlag.txt HTTP/1.1
   → flag{You_Found_Me_#3019} ✅
       ↓
✅ FLAG: flag{You_Found_Me_#3019}
```
