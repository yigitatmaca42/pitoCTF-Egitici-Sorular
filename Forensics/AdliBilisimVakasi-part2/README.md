# 🎯 AdliBilisimVakasi-part2 - Forensics Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Forensics-Challenge-darkblue?style=for-the-badge" alt="Digital Forensics">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-C2 Detection-purple?style=for-the-badge" alt="C2 Detection">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Forensics  
**Seviye:** Orta  
**Puan:** 500

**Challenge Dosyası:** [📥 Google Drive - AdliBilisimVakasi](https://drive.google.com/file/d/1jccVg1lI2y1hmMIiCPCHq1WV9Fsfsfy9/view?usp=drivesdk)

**Challenge Açıklaması:** Zararlı dosyanın bağlantı kurduğu C2 sunucusunun ip ve portunu bulabilir misin?  

**Flag Formatı:** `Flag{ip:port}`

---

## 🔍 Analiz

**Hedef:** `monitorStock.exe` zararlı yazılımının bağlandığı C2 sunucusunun IP ve port bilgisini tespit etmek

**Strateji:** Sysmon event loglarındaki network connection kayıtlarını incelemek

**İpuçları:**
- 🎯 Sysmon **EventID 3** → Network Connection olaylarını kaydeder
- 🔍 `monitorStock.exe` önceki soruda tespit edilen zararlı dosya
- 💡 `DestinationIp` ve `DestinationPort` alanları C2 bilgisini içerir
- 🕵️ Sysmon log dosyası: `Microsoft-Windows-Sysmon%4Operational.evtx`

---

## ✅ Çözüm Adımları

### 📦 **Adım 1: python3-evtx Kur**
```bash
sudo apt install python3-evtx -y
```

**Analiz:**
- ✅ `evtx_dump.py` aracı Windows `.evtx` log dosyalarını parse eder
- 💡 Sysmon logları bu araç olmadan okunamaz

---

### 🔎 **Adım 2: Sysmon'da monitorStock İzlerini Ara**
```bash
evtx_dump.py "C/Windows/System32/winevt/logs/Microsoft-Windows-Sysmon%4Operational.evtx" \
| grep -i "monitorstock\|monitor" -A 10 -B 2
```

**Önemli Çıktılar:**
```xml
<Data Name="Image">C:\Users\khaled.allam\Downloads\monitorStock.exe</Data>
<Data Name="TargetFilename">C:\Users\khaled.allam\Downloads\monitorStock.exe</Data>
<Data Name="Contents">[ZoneTransfer] ZoneId=3 ReferrerUrl=http://app.finance.com/</Data>
<Data Name="Contents">[ZoneTransfer] ZoneId=3 HostUrl=http://app.finance.com/monitorStock.exe</Data>
```

**Analiz:**
- ✅ `monitorStock.exe` indirilmiş ve çalıştırılmış
- 🚨 Kaynak: `http://app.finance.com/monitorStock.exe` (part-1 ile örtüşüyor)
- 💡 Ayrıca `C:\Windows\Temp\monitorStock.exe` olarak kopyalanmış → **persistence**
- 🔑 Registry Run key eklenmiş: `HKCU\...\Run\FilePersistence` → kalıcılık sağlanmış

---

### 🔨 **Adım 3: Network Bağlantılarını Sorgula**
```bash
evtx_dump.py "C/Windows/System32/winevt/logs/Microsoft-Windows-Sysmon%4Operational.evtx" \
2>/dev/null | grep -i "DestinationIp\|DestinationPort\|Image" | grep -B1 -A1 "3\."
```

**Çıktı:**
```xml
<Data Name="Image">C:\Users\khaled.allam\Downloads\chartnexus-4.3-installer_pUe-Cs1.exe</Data>
<Data Name="DestinationIp">3.160.203.158</Data>
<Data Name="DestinationPort">443</Data>
--
<Data Name="Image">C:\Users\khaled.allam\Downloads\chartnexus-4.3-installer_pUe-Cs1.exe</Data>
<Data Name="DestinationIp">23.45.69.62</Data>
<Data Name="DestinationPort">80</Data>
--
<Data Name="Image">C:\Users\khaled.allam\Downloads\monitorStock.exe</Data>
<Data Name="DestinationIp">3.121.219.28</Data>
<Data Name="DestinationPort">8888</Data>
```

**Analiz:**
- ✅ `chartnexus` installer → 443 ve 80 portları → meşru HTTPS/HTTP trafiği
- 🚨 `monitorStock.exe` → **3.121.219.28:8888** → standart dışı port, C2 bağlantısı!
- 💡 Port 8888 → yaygın C2 framework portu (Metasploit, Cobalt Strike vb.)

---

## 🚩 **FLAG**
```
Flag{3.121.219.28:8888}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">📦<br><b>python3-evtx</b><br><sub>EVTX log parser</sub></td>
<td align="center">🔍<br><b>evtx_dump.py</b><br><sub>Event log dump</sub></td>
<td align="center">🐚<br><b>grep</b><br><sub>Pattern matching</sub></td>
<td align="center">🕵️<br><b>Sysmon</b><br><sub>Network connection logs</sub></td>
<td align="center">🌐<br><b>EventID 3</b><br><sub>Network Connection event</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**
```
🎯 AdliBilisimVakasi-part2 Challenge
       ↓
📦 python3-evtx kur
   → evtx_dump.py aracı kullanılabilir hale gelir
       ↓
🔎 Sysmon logunda monitorStock izlerini ara
   → İndirilmiş: http://app.finance.com/monitorStock.exe
   → C:\Windows\Temp\monitorStock.exe olarak kopyalanmış
   → Registry Run key ile persistence sağlanmış
       ↓
🔨 Network bağlantılarını sorgula
   → DestinationIp + DestinationPort filtrele
   → chartnexus → 443/80 (meşru)
   → monitorStock.exe → 3.121.219.28:8888 (şüpheli!)
       ↓
🚩 Flag{3.121.219.28:8888}
```
