# 🎯 AdliBilisimVakasi-part1 - Forensics Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Forensics-Challenge-darkblue?style=for-the-badge" alt="Digital Forensics">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Browser History-purple?style=for-the-badge" alt="Browser History">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Forensics  
**Seviye:** Orta  
**Puan:** 500

**Challenge Açıklaması:** Kullanıcı bir yerden bir exe indirmiş. İndirilen domaini ve exe adını bulabilir mis

**Challenge Dosyası:** [📥 Google Drive - AdliBilisimVakasi](https://drive.google.com/file/d/1jccVg1lI2y1hmMIiCPCHq1WV9Fsfsfy9/view?usp=drivesdk)
in? 
 
**Flag Formatı:** `Flag{domain_indirilenExeAdi}` — Örn: `Flag{evil.com_backdoor.exe}`

---

## 🔍 Analiz

**Hedef:** Kullanıcının hangi domain'den hangi exe'yi indirdiğini tespit etmek

**Strateji:** Windows forensics artifact'larını inceleyerek tarayıcı indirme geçmişini ortaya çıkarmak

**İpuçları:**
- 🎯 ZIP içinde Windows kullanıcı profilleri ve sistem dosyaları mevcut
- 🔍 Microsoft Edge browser history SQLite DB dosyaları var
- 💡 `downloads` tablosu indirilen dosyaları ve kaynak URL'leri tutar
- 🕵️ Prefetch dosyaları hangi exe'lerin çalıştırıldığını gösterir

---

## ✅ Çözüm Adımları

### 📦 **Adım 1: ZIP'i Aç ve İçeriği İncele**
```bash
unzip AdliBilisimVakasi.zip
cd AdliBilisimVakasi
ls C/Users/
```

**Çıktı:**
```
Default  adam.lee  khaled.allam  omar.ahmed
```

**Analiz:**
- ✅ 3 kullanıcı profili mevcut: `khaled.allam`, `omar.ahmed`, `adam.lee`
- 💡 Her kullanıcının Edge browser history'si ayrı incelenecek

---

### 🔎 **Adım 2: PowerShell History'yi İncele**
```bash
cat "C/Users/khaled.allam/AppData/Roaming/Microsoft/Windows/PowerShell/PSReadline/ConsoleHost_history.txt"
```

**Çıktı:**
```
python
python3
.\Sysmon64.exe -i .\sysmonconfig-export.xml
./kape.exe --tsource C:\ --tdest '\\vmware-host\Shared Folders\yep' --target KapeTriage --zip output
```

**Analiz:**
- ✅ `khaled.allam` kullanıcısı Sysmon kurmuş → sistemi izleme amacı var
- 💡 KAPE ile forensic triage yapılmış

---

### 🔨 **Adım 3: Tüm Kullanıcıların Edge History'sini Sorgula**
```bash
for db in C/Users/*/AppData/Local/Microsoft/Edge/User\ Data/Default/History; do
  echo "=== $db ===";
  sqlite3 "$db" "SELECT url, title FROM urls;" 2>/dev/null;
done
```

**Önemli Çıktılar (`khaled.allam`):**
```
http://app.finance.com/  |  Directory listing for /
https://www.tradingview.com/desktop/  |  TradingView Desktop Application
https://chartnexus.en.softonic.com/download  |  Download ChartNexus
```

**Analiz:**
- 🚨 `http://app.finance.com/` → **"Directory listing for /"** dönüyor (açık dizin listeleme — şüpheli!)
- 💡 Kullanıcı borsa takip uygulamaları araştırmış

---

### 📊 **Adım 4: Downloads Tablosunu Sorgula**
```bash
sqlite3 "C/Users/khaled.allam/AppData/Local/Microsoft/Edge/User Data/Default/History" \
"SELECT target_path, tab_url, referrer FROM downloads;"
```

**Çıktı:**
```
C:\Users\a1l4m\Downloads\python-3.13.0-amd64.exe      | https://www.python.org/downloads/ | https://www.bing.com/
C:\Users\khaled.allam\Downloads\monitorStock.exe       | http://app.finance.com/           |
C:\Users\khaled.allam\Downloads\TradingView.msix       | https://tvd-packages.tradingview.com/...  |
C:\Users\khaled.allam\Downloads\chartnexus-4.3-installer_pUe-Cs1.exe | https://en.softonic.com/... |
```

**Analiz:**
- 🚨 `monitorStock.exe` → kaynak: `http://app.finance.com/` — referrer **YOK** (doğrudan indirme)
- ⚠️ `app.finance.com` gerçek bir finans sitesi değil; dizin listeleme açık olan şüpheli domain
- ✅ Diğer indirmeler (python, TradingView, ChartNexus) meşru kaynaklara işaret ediyor

---

### 🔧 **Adım 5: Prefetch ile Çalıştırılmayı Doğrula**
```bash
ls C/Windows/prefetch/ | grep -i "monitor"
```

**Çıktı:**
```
MONITORSTOCK.EXE-0F7B9B9D.pf
MONITORSTOCK.EXE-4D4C5193.pf
```

**Analiz:**
- ✅ `monitorStock.exe` prefetch'te **2 kez** görünüyor → indirilmekle kalmamış, **çalıştırılmış!**
- 🚨 Farklı hash'lere sahip iki prefetch kaydı → farklı konumlardan çalıştırılmış olabilir

---

### 🚀 **Adım 6: Şüpheli Domain'i Doğrula**
```bash
sqlite3 "C/Users/khaled.allam/AppData/Local/Microsoft/Edge/User Data/Default/History" \
"SELECT url, title, last_visit_time FROM urls WHERE url LIKE '%app.finance.com%';"
```

**Çıktı:**
```
http://app.finance.com/   |   Directory listing for /   |   13383376621572019
https://app.finance.com/  |   Directory listing for /   |   13383376621572019
```

**Analiz:**
- 🚨 Hem HTTP hem HTTPS üzerinden ziyaret edilmiş
- 🔑 Başlık "Directory listing for /" → sunucuda dizin listeleme açık, dosyalar düz erişilebilir
- ✅ `monitorStock.exe` bu domain'den indirilmiş → **şüpheli executable**

---

## 🚩 **FLAG**
```
Flag{app.finance.com_monitorStock.exe}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">📦<br><b>unzip</b><br><sub>ZIP extraction</sub></td>
<td align="center">🗄️<br><b>sqlite3</b><br><sub>Browser history query</sub></td>
<td align="center">🔍<br><b>strings</b><br><sub>Artifact analysis</sub></td>
<td align="center">📁<br><b>prefetch</b><br><sub>Execution evidence</sub></td>
<td align="center">🐚<br><b>bash</b><br><sub>Shell scripting</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**
```
🎯 AdliBilisimVakasi-part1 Challenge
       ↓
📦 ZIP'i aç
   → 3 kullanıcı profili: khaled.allam, omar.ahmed, adam.lee
   → Windows forensic artifact'ları mevcut
       ↓
🔎 PowerShell history oku
   → Sysmon kurulumu
   → KAPE triage yapılmış
       ↓
🗄️ Edge Browser History sorgula (SQLite)
   → khaled.allam ziyaret etmiş: app.finance.com
   → "Directory listing for /" → şüpheli açık dizin
       ↓
📊 Downloads tablosunu sorgula
   → monitorStock.exe → http://app.finance.com/
   → Referrer YOK → doğrudan şüpheli kaynaktan indirilmiş
       ↓
📁 Prefetch dosyalarını kontrol et
   → MONITORSTOCK.EXE 2 kez prefetch'te
   → Sadece indirilmemiş, ÇALIŞTIRILMIŞ
       ↓
🚩 Flag{app.finance.com_monitorStock.exe}
```
