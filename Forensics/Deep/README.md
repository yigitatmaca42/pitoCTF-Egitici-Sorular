# 🔍 Deep - Forensics Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Forensics-Challenge-darkblue?style=for-the-badge&logo=searchengin" alt="Forensics">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Windows Forensics-purple?style=for-the-badge&logo=windows" alt="Windows">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Forensics  
**Seviye:** Orta  
**Puan:** 400

**Açıklama:** Saldırıya uğradık. Lütfen yardım et.

**Challenge Dosyası:** [📥 Deep.zip](https://drive.google.com/file/d/1wzJv11RyrqwLP6UI3rrGMOLlKEchuj1N/view?usp=sharing)


---

## 🔍 Analiz

**Hedef:** Windows sistem imajını analiz edip saldırganın bıraktığı XOR şifrelenmiş FLAG'i bulmak

**Strateji:** Windows disk analizi → PowerShell history → XOR decryption → Flag

**İpuçları:**
- 💻 **Windows Forensics:** Sistem dosyaları ve kullanıcı aktiviteleri
- 📜 **PowerShell History:** ConsoleHost_history.txt kritik!
- 🔐 **XOR Encryption:** Basit şifreleme yöntemi
- 🎯 **Format:** 0xL4ugh{...} formatında flag

---

## ✅ Çözüm Adımları

### 📥 **1. Dosya Analizi**

**ZIP dosyasını indirip kontrol ediyoruz:**

```bash
file Deep.zip
```

**Çıktı:**
```
Deep.zip: Zip archive data, at least v1.0 to extract, compression method=store
```

> ✅ **ZIP arşivi** doğrulandı!

---

### 📦 **2. Dosyayı Extract Etme**

**ZIP dosyasını çıkarıyoruz:**

```bash
mkdir Deep
unzip Deep.zip -d Deep
cd Deep
ls -la
```

**Çıktı:**
```
drwxrwxr-x  7 kali kali 4096 Jan 12 17:52  .
drwxr-xr-x 14 kali kali 4096 Jan 12 17:52  ..
drwxrwxr-x  2 kali kali 4096 Feb 13  2023  PerfLogs
drwxrwxr-x 23 kali kali 4096 Feb 13  2023 'Program Files'
drwxrwxr-x 18 kali kali 4096 Feb 13  2023 'Program Files (x86)'
drwxrwxr-x  5 kali kali 4096 Feb 13  2023  Users
drwxrwxr-x 79 kali kali 4096 Feb 13  2023  Windows
```

**Analiz:**
- 📁 **Windows dizin yapısı:** Bu bir Windows sistem imajı!
- 👤 **Users klasörü:** Kullanıcı hesapları ve aktiviteler burada
- 🪟 **Windows klasörü:** Sistem dosyaları ve loglar

> 🎯 Bu klasik bir Windows C:\ disk imajı!

---

### 👥 **3. Kullanıcı Hesaplarını Tespit Etme**

**Users klasöründeki hesapları listeliyoruz:**

```bash
ls -la Users/
```

**Çıktı:**
```
drwxrwxr-x 12 kali kali 4096 Feb 13  2023 Default
-rw-rw-r--  1 kali kali   84 Feb 13  2023 desktop.ini
drwxrwxr-x 16 kali kali 4096 Feb 13  2023 developer    ← 🎯 Hedef!
drwxrwxr-x 10 kali kali 4096 Feb 13  2023 Public
```

**Kullanıcılar:**
```
1. Default   → Sistem varsayılan profili
2. developer → 🚨 Ana kullanıcı hesabı (saldırı hedefi)
3. Public    → Genel paylaşım klasörü
```

> 🎯 **developer** kullanıcısı saldırıya uğramış!

---

### 🖥️ **4. Desktop İncelemesi**

**developer kullanıcısının masaüstüne bakıyoruz:**

```bash
ls -la Users/developer/Desktop/
```

**Çıktı:**
```
-rw-rw-r--  1 kali kali   282 Feb 13  2023  desktop.ini
-rw-rw-r--  1 kali kali 59613 Feb 13  2023  free-palestine-arabic-big-palestine-flag-free-gaza-brock-zophia.jpg
-rw-rw-r--  1 kali kali  2348 Feb 13  2023 'Microsoft Edge.lnk'
```

**Analiz:**
- 🖼️ **free-palestine-arabic-big-palestine-flag-free-gaza-brock-zophia.jpg** ← Şüpheli!
- 🔗 **Microsoft Edge.lnk** → Tarayıcı kısayolu

> 🤔 Bu resim dosyası neden burada? Muhtemelen dikkat dağıtmak için!

---

### 📜 **5. Son Kullanılan Dosyalar**

**Recent (son kullanılan) dosyaları kontrol ediyoruz:**

```bash
find Users/developer/AppData/Roaming/Microsoft/Windows/Recent/ -type f 2>/dev/null
```

**Önemli çıktılar:**
```
Users/developer/AppData/Roaming/Microsoft/Windows/Recent/AutomaticDestinations/1b4dd67f29cb1962.automaticDestinations-ms
Users/developer/AppData/Roaming/Microsoft/Windows/Recent/AutomaticDestinations/5f7b5f1e01b83767.automaticDestinations-ms
Users/developer/AppData/Roaming/Microsoft/Windows/Recent/free-palestine-arabic-big-palestine-flag-free-gaza-brock-zophia.lnk
Users/developer/AppData/Roaming/Microsoft/Windows/Recent/The Internet.lnk
```

> 💡 Resim dosyası recent listesinde de var!

---

### 🔍 **6. PowerShell Geçmişi - ALTIN MADENI!**

**PowerShell komut geçmişini arıyoruz:**

```bash
find Users/developer/ -name "ConsoleHost_history.txt" 2>/dev/null
```

**Çıktı:**
```
Users/developer/AppData/Roaming/Microsoft/Windows/PowerShell/PSReadLine/ConsoleHost_history.txt
```

> 🏆 **BULUNDU!** Bu dosya saldırganın tüm komutlarını içerir!

---

### 📖 **7. PowerShell History İncelemesi**

**Geçmiş dosyasını okuyoruz:**

```bash
cat Users/developer/AppData/Roaming/Microsoft/Windows/PowerShell/PSReadLine/ConsoleHost_history.txt
```

**Kritik içerik:**
```powershell
hheeeey
while ($true){$w=@("www.google.com","www.microsoft.com","www.apple.com","www.amazon.com","www.facebook.com");$r=Get-Random -InputObject $w;Write-Host "Pinging $r...";Test-Connection $r -Count 1;Start-Sleep -Seconds 1}
cd ..
while ($true){$r=(New-Object System.Net.WebClient).DownloadString("https://www.random.org/domains/");Write-Host "Pinging $r...";Test-Connection $r -Count 1;Start-Sleep -Seconds 1}
while ($true){$r=(Get-Random -Minimum 0 -Maximum 255).ToString()+"."+(Get-Random -Minimum 0 -Maximum 255).ToString()+"."+(Get-Random -Minimum 0 -Maximum 255).ToString()+"."+(Get-Random -Minimum 0 -Maximum 255).ToString();Write-Host "Pinging $r...";Test-Connection $r -Count 1;Start-Sleep -Seconds 1}
Get-ChildItem -Path . -Directory | ForEach-Object {Write-Host $_.FullName}

[Text.Encoding]::ASCII.GetString([Text.Encoding]::ASCII.GetBytes([System.Text.Encoding]::ASCII.GetBytes("6~J2san}_6sYGT5YA6IBY7HP5317A16T{") -BXOR [System.Text.Encoding]::ASCII.GetBytes("0x06")))

while ($true){$w=@("www.google.com","www.microsoft.com",...
```

**Saldırganın Aktiviteleri:**

1. ✅ **"hheeeey"** → Sisteme giriş mesajı
2. 🔁 **Rastgele ping komutları** → Ağ trafiği yaratarak gizlenme çabası
3. 🌐 **www.random.org'dan domain çekme** → Daha fazla gürültü
4. 🎲 **Rastgele IP'lere ping** → DDoS benzeri davranış
5. 🚩 **XOR şifrelenmiş FLAG!** → İşte aradığımız!

> 🎯 **KRITIK BULGU:**
> ```powershell
> [Text.Encoding]::ASCII.GetString([Text.Encoding]::ASCII.GetBytes(
>     [System.Text.Encoding]::ASCII.GetBytes("6~J2san}_6sYGT5YA6IBY7HP5317A16T{") 
>     -BXOR 
>     [System.Text.Encoding]::ASCII.GetBytes("0x06")
> ))
> ```

---

### 🔐 **8. XOR Şifreleme Analizi**

**Şifrelenmiş veri:**
```
Encrypted: 6~J2san}_6sYGT5YA6IBY7HP5317A16T{
Key:       0x06 (Hex) = 6 (Decimal)
```

**XOR nasıl çalışır:**
```
XOR (Exclusive OR) işlemi:
- Aynı değerler → 0
- Farklı değerler → 1

Örnek:
'6' (ASCII 54) XOR 0x06 = 54 XOR 6 = 48 = '0'
'~' (ASCII 126) XOR 0x06 = 126 XOR 6 = 120 = 'x'
```

---

### 🐍 **9. Python ile XOR Decryption**

**FLAG'i çözmek için Python scripti:**

```python
# XOR ile şifrelenmiş flag'i çöz
encrypted = "6~J2san}_6sYGT5YA6IBY7HP5317A16T{"
key = 0x06  # Hex 0x06 = decimal 6

decrypted = ""
for char in encrypted:
    # Her karakteri XOR işlemiyle çöz
    decrypted += chr(ord(char) ^ key)

print("🚩 FLAG BULUNDU:")
print(decrypted)
```

**Çalıştırma:**
```bash
python3 xor_decrypt.py
```

**Çıktı:**
```
🚩 FLAG BULUNDU:
0xL4ugh{Y0u_AR3_G0OD_1NV3571G70R}
```

---

## 🚩 **FLAG**

```
0xL4ugh{Y0u_AR3_G0OD_1NV3571G70R}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">📄<br><b>file</b><br><sub>Dosya tipi analizi</sub></td>
<td align="center">📦<br><b>unzip</b><br><sub>ZIP extraction</sub></td>
<td align="center">🔍<br><b>find</b><br><sub>Dosya arama</sub></td>
<td align="center">🐍<br><b>Python</b><br><sub>XOR decryption</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**

```bash
# Dosya analizi
file Deep.zip

# ZIP extraction
unzip Deep.zip -d Deep

# Kullanıcı hesapları
ls -la Deep/Users/

# Desktop incelemesi
ls -la Deep/Users/developer/Desktop/

# PowerShell geçmişini bulma
find Deep/Users/developer/ -name "ConsoleHost_history.txt"

# PowerShell geçmişini okuma
cat Deep/Users/developer/AppData/Roaming/Microsoft/Windows/PowerShell/PSReadLine/ConsoleHost_history.txt

# XOR decryption
python3 xor_decrypt.py
```

**Python decryption scripti:**

```python
#!/usr/bin/env python3

# XOR Decryption Script
encrypted = "6~J2san}_6sYGT5YA6IBY7HP5317A16T{"
key = 0x06

decrypted = ""
for char in encrypted:
    decrypted += chr(ord(char) ^ key)

print("=" * 50)
print("🔐 XOR DECRYPTION")
print("=" * 50)
print(f"Encrypted: {encrypted}")
print(f"Key:       0x{key:02x} ({key})")
print(f"Decrypted: {decrypted}")
print("=" * 50)
```

---

## 🕵️ **Saldırı Analizi**

### **Saldırganın Kullandığı Teknikler:**

1. **🎭 Dikkat Dağıtma:**
   - Desktop'a alakasız resim dosyası bırakmış
   - "free-palestine-arabic-big-palestine-flag..." → Yanıltmaca

2. **🌊 Gürültü Yaratma:**
   - Sürekli rastgele ping komutları
   - Google, Microsoft, Amazon gibi sitelere ping
   - Random.org'dan domain çekme
   - Rastgele IP'lere ping atma
   - **Amaç:** Gerçek aktiviteleri loglar arasında gizlemek

3. **🔐 XOR Şifreleme:**
   - Basit ama etkili şifreleme
   - Key: 0x06 (6 decimal)
   - PowerShell native fonksiyonları kullanmış

4. **📜 İz Bırakma:**
   - PowerShell history dosyasını temizlememiş
   - **Hata:** ConsoleHost_history.txt'yi silmeyi unutmuş!

---

## 💭 **Çözüm Akışı**

```
🔍 "Deep" Forensics Challenge
            ↓
📥 Deep.zip indirildi
            ↓
📄 file Deep.zip → ZIP archive
            ↓
📦 unzip → Windows disk imajı
            ↓
👤 Users/developer/ → Hedef kullanıcı
            ↓
🖥️ Desktop → Şüpheli resim (yanıltmaca)
            ↓
📜 PowerShell history arandı
   → ConsoleHost_history.txt bulundu!
            ↓
📖 History incelendi
   → XOR şifreli komut tespit edildi
   → "6~J2san}_6sYGT5YA6IBY7HP5317A16T{"
   → Key: 0x06
            ↓
🔐 XOR analizi
   → '6' XOR 0x06 = '0'
   → '~' XOR 0x06 = 'x'
   → 'J' XOR 0x06 = 'L'
   → ...
            ↓
🐍 Python decryption
            ↓
🚩 FLAG: 0xL4ugh{Y0u_AR3_G0OD_1NV3571G70R}
```
