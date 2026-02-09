# 🎯 App - Forensics Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Forensics-Challenge-darkgreen?style=for-the-badge" alt="Forensics">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Git Forensics-blue?style=for-the-badge" alt="Git">
</p>

---

## 📋 Soru Bilgileri
  
**Kategori:** Forensics  
**Seviye:** Orta  
**Puan:** 500

**Challenge Dosyası:** [📥 Google Drive - App.zip](https://drive.google.com/file/d/1Jmg_rhDV9AA_jXywrIh6iff718_xkCtz/view?usp=drivesdk)

---

## 🔍 Analiz

**Hedef:** Git repository'sindeki silinen dosyaları ve gizli bilgileri bulmak

**Strateji:** Zip extract → Git repo analizi → Commit history → Silinen dosyaları bulma → Flag

**İpuçları:**
- 🎯 `.git` klasörü → Git repository mevcut
- 🔍 "accidentally pushed secrets" → Commit history'de flag olabilir
- 💡 Git her şeyi hatırlar, silinmiş bile olsa

---

## ✅ Çözüm Adımları

### 📦 **Adım 1: Zip Dosyasını Extract Etme**

```bash
unzip App.zip
```

**Çıktı:**
```
Archive:  App.zip
   creating: .git/
 extracting: .git/HEAD               
  inflating: .git/index              
   creating: .git/hooks/
  ...
  inflating: app.py
```

**Analiz:**
- ✅ `.git` klasörü mevcut → Git repository
- ✅ `app.py` dosyası var
- 💡 Git history incelenebilir

---

### 🔎 **Adım 2: Uygulamayı Çalıştırma**

```bash
python3 app.py
```

**Çıktı:**
```
Welcome to the application!
Version: 2.0
All systems operational.
```

**Analiz:**
- ✅ Uygulama çalışıyor ama flag yok
- 💡 Git history'ye bakmamız gerekiyor

---

### 📜 **Adım 3: Git Commit History İnceleme**

```bash
git log
```

**Çıktı:**
```
commit 123fcf7ea486bc36d1fb170025c3f7e4bacca92a (HEAD -> master)
Author: Junior Dev <developer@company.com>
Date:   Sat Feb 7 07:44:00 2026 +0000
    Remove config file - accidentally pushed secrets!

commit 6adbb3f867d46a434652f76393943f0e40eb95a8
Author: Junior Dev <developer@company.com>
Date:   Sat Feb 7 07:43:57 2026 +0000
    Update application to v2.0

commit ed2ce4f9ed74b2316b4c7aa57848c902ae1c4c6d
Author: Junior Dev <developer@company.com>
Date:   Sat Feb 7 07:43:50 2026 +0000
    Add configuration

commit 20a53d342998c3355ff1f950be69407524c26ca0
Author: Junior Dev <developer@company.com>
Date:   Sat Feb 7 07:43:41 2026 +0000
    Initial commit
```

**Analiz:**
- ⚠️ "accidentally pushed secrets!" → Şüpheli commit!
- ✅ `ed2ce4f` → "Add configuration" commit'i
- ✅ Sonra `123fcf7` → "Remove config file" commit'i
- 💡 Config dosyası eklenmiş sonra silinmiş

---

### 🔍 **Adım 4: Silinen Config Dosyasını İnceleme**

```bash
git show ed2ce4f
```

**Çıktı:**
```
commit ed2ce4f9ed74b2316b4c7aa57848c902ae1c4c6d
Author: Junior Dev <developer@company.com>
Date:   Sat Feb 7 07:43:50 2026 +0000
    Add configuration

diff --git a/config.py b/config.py
new file mode 100644
index 0000000..65586f6
--- /dev/null
+++ b/config.py
@@ -0,0 +1,10 @@
+# Configuration File
+# WARNING: Do not commit this file!
+
+DATABASE_URL = "postgresql://localhost/mydb"
+API_KEY = "sk_live_12345abcde"
+SECRET_TOKEN = "super_secret_token_xyz"
+
+# Git{gecmisibirakipgidemesingitunutmaz}
+
+ADMIN_PASSWORD = "admin123"
```

**✅ FLAG BULUNDU!**

---

## 🚩 **FLAG**

```
Git{gecmisibirakipgidemesingitunutmaz}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">📦<br><b>unzip</b><br><sub>Archive extraction</sub></td>
<td align="center">🐍<br><b>python3</b><br><sub>Run application</sub></td>
<td align="center">🌳<br><b>git log</b><br><sub>Commit history</sub></td>
<td align="center">🔍<br><b>git show</b><br><sub>Commit details</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**
```bash
# Extract zip
unzip App.zip

# Uygulamayı çalıştır
python3 app.py

# Commit history
git log

# Commit detayları
git show ed2ce4f
```

---

## 💭 **Çözüm Akış Şeması**

```
🎯 App Challenge
       ↓
📦 App.zip extract
   → .git klasörü bulundu
   → app.py dosyası var
       ↓
🐍 app.py çalıştır
   → Version 2.0
   → Flag yok
       ↓
📜 Git log kontrol
   → 4 commit var
   → "accidentally pushed secrets!"
       ↓
🔍 ed2ce4f commit'i incele
   → config.py eklendi
   → Ardından silindi
       ↓
🎯 git show ed2ce4f
   → config.py içeriği görüntülendi
   → Comment'te flag bulundu
       ↓
🚩 Git{gecmisibirakipgidemesingitunutmaz}
```
