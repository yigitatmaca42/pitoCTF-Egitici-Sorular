# 🔐 Business Logic - Web Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Web-Challenge-darkblue?style=for-the-badge" alt="Web">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Authentication%20Bypass-purple?style=for-the-badge&logo=shield" alt="Auth Bypass">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Web  
**Seviye:** Kolay  
**Puan:** 300
**Açıklama:** Şifreye brute atmanıza ve agresif saldırılara gerek yok. Faydası da olmaz! Siteyi gptye yazdırdım. Ama acemi yapay zeka bazı şeyleri düşünememiş.

**Challenge URL:** http://64.226.74.243:5001

---

## 🔍 Analiz

### Challenge Özeti

**Hedef:** Authentication bypass ile admin paneline erişim

**İpucu Analizi:**
- "Brute force gerekmez" → Mantık hatası var
- "GPT ile yazılmış" → Kod hataları olabilir
- "Bazı şeyleri düşünememiş" → Business logic vulnerability

**Strateji:** Register → Username manipulation → Admin access → Flag

---

## ✅ Çözüm Adımları

### 🌐 **1. Siteyi Keşfetme**

**Ana URL:** http://64.226.74.243:5001

**Adımlar:**

1. Siteye gidiyoruz
2. **Giriş Yap** sayfası açılıyor
3. **Kayıt Ol** butonu görünüyor

---

### 📝 **2. Normal Kullanıcı ile Kayıt**

**Kayıt URL'si:** http://64.226.74.243:5001/register

**İlk Deneme:**

**Kullanıcı Adı:** `yigit42`  
**Şifre:** `424242`

**Adımlar:**

1. Register formunu dolduruyoruz
2. **Giriş Yap** butonuna tıklıyoruz
3. Başarıyla giriş yapıyoruz

---

### 🚫 **3. Dashboard - Erişim Engellendi**

**Dashboard URL:** http://64.226.74.243:5001/dashboard

**Gördüğümüz Mesaj:**
```
🎯 Dashboard

Kullanıcı Adı: yigit42
Yetki Seviyesi: 🔵 Normal Kullanıcı

⛔ Erişim Engellendi

Flag'i görebilmek için root veya robot kullanıcısı olmanız gerekiyor.

Sadece admin kullanıcıları flag'e erişebilir.
```

> 💡 **Kritik Bilgi:** Flag'e erişmek için **root** veya **robot** kullanıcısı olmalıyız!

---

### 🧩 **4. Business Logic Analizi**

**Sorun Tespit Edildi:**

Challenge açıklamasında **"GPT ile yazılmış, bazı şeyleri düşünememiş"** deniliyor.

**Olası Zafiyetler:**

1. ❌ Username validation yok
2. ❌ Case-insensitive kontrol
3. ❌ Reserved username kontrolü eksik

**Hipotez:** Eğer kullanıcı adını **"RoBoT"** veya **"RoOt"** gibi mixed-case versiyonlarla kaydedersek, sistem bizi admin olarak tanıyabilir!

---

### 🎯 **5. Username Manipulation - Robot**

**İkinci Kayıt Denemesi:**

**Çıkış Yap** → **Kayıt Ol** → Yeni hesap oluştur

**Kullanıcı Adı:** `RoBoT` (Mixed case)  
**Şifre:** `123456`

**Adımlar:**

1. http://64.226.74.243:5001/register adresine gidiyoruz
2. Username: `RoBoT` yazıyoruz
3. Şifre giriyoruz
4. **Giriş Yap** butonuna tıklıyoruz

---

### ✅ **6. Admin Paneline Erişim Başarılı!**

**Dashboard Değişimi:**
```
🎯 Dashboard

Kullanıcı Adı: RoBoT
Yetki Seviyesi: 👑 Admin

🎉 Tebrikler!

Admin yetkisi ile flag'e erişim sağladınız!

Flag{c4s3_1ns3ns1t1v3_us3rn4m3s_4r3_d4ng3r0us_goodby2025}
```

> 🚩 **FLAG BULUNDU!**

---

### 🔄 **7. Alternatif Çözümler**

**Case-Insensitive Exploitation ile çalışan kullanıcı adları:**

#### **Robot Varyasyonları:**
```
robot
Robot
ROBOT
RoBoT
rObOt
RoBOT
r0b0t
R0B0T
R0b0T
```

#### **Root Varyasyonları:**
```
root
Root
ROOT
RoOt
rOoT
RoOT
r00t
R00T
Ro0t
```

> 💡 Herhangi bir varyasyon ile kayıt olup admin yetkisi alabilirsiniz!

---

## 🚩 **FLAG**
```
Flag{c4s3_1ns3ns1t1v3_us3rn4m3s_4r3_d4ng3r0us_goodby2025}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🌐<br><b>Web Browser</b><br><sub>Site erişimi</sub></td>
<td align="center">🔍<br><b>Developer Tools</b><br><sub>İnceleme</sub></td>
<td align="center">🧠<br><b>Logic Analysis</b><br><sub>Zafiyet tespiti</sub></td>
</tr>
</table>

**Kullanılan Teknikler:**
- 🔍 **Reconnaissance:** Site keşfi
- 🧪 **Testing:** Normal kullanıcı testi
- 🎯 **Exploitation:** Username manipulation
- 🔓 **Privilege Escalation:** Admin yetkisi elde etme

---

## 💭 **Çözüm Akış Şeması**
```
🔐 Web Challenge: "Business Logic"
                    ↓
        🌐 http://64.226.74.243:5001
                    ↓
        📝 Normal kullanıcı ile kayıt (yigit42)
                    ↓
        🔓 Giriş yap → Dashboard
                    ↓
        ⛔ "root veya robot olmalısınız" mesajı
                    ↓
        💡 İpucu: GPT kodu, validation eksik
                    ↓
        🧩 Case-insensitive zafiyet keşfi
                    ↓
        📝 Yeni kayıt: Username = RoBoT
                    ↓
        🔓 Giriş yap
                    ↓
        👑 Admin yetkisi ile dashboard
                    ↓
        🚩 Flag{c4s3_1ns3ns1t1v3_us3rn4m3s_4r3_d4ng3r0us_goodby2025}
```
