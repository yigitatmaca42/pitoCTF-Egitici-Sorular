# 👋 Hoşgeldin - OSINT Challenge

<p align="center">
  <img src="https://img.shields.io/badge/OSINT-Challenge-darkblue?style=for-the-badge&logo=search" alt="OSINT">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-User%20Research-purple?style=for-the-badge&logo=user" alt="User Research">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** OSINT 
**Seviye:** Kolay  
**Puan:** 100  
**Açıklama:** Hoşgeldin. Flag buralarda bir yerde.

**Challenge URL:** [🌐 pitoCTF Platform](https://ctf.pitoctf.online/)

---

## 🔍 Analiz

### Challenge Açıklaması

Challenge başlığı: **"Hoşgeldin"**  
Açıklama: **"Hoşgeldin. Flag buralarda bir yerde."**

> 💡 **İpucu:** Bu açıklama bize flag'in platformun içinde, muhtemelen kullanıcı profillerinde veya genel erişilebilir bir yerde olduğunu söylüyor.

---

## ✅ Çözüm Adımları

### 🎯 **1. Challenge'ı Anlama**

**Anahtar Kelimeler:**
- "Hoşgeldin" → Yeni kullanıcılar için
- "Buralarda bir yerde" → Platform içinde arama yapmalıyız

**Hedef:** CTF platformunda flag'i bulmak

---

### 👥 **2. Users Sayfasını Keşfetme**

İlk adım olarak platformdaki **kullanıcıları** inceliyoruz.

**Adımlar:**

1. CTF platformunda **"Users"** sekmesine gidiyoruz
2. URL: `https://ctf.pitoctf.online/users`

**Users Listesi:**

| User | Website | Affiliation | Country |
|------|---------|-------------|---------|
| **admin** | 🔗 | Farklı Logları Ara, Gizli Hedefi Odaklan... | - |

> 🎯 **Dikkat:** **admin** kullanıcısının **Affiliation** alanı tamamlanmamış görünüyor!

---

### 🔍 **3. Admin Profilini İnceleme**

**admin** kullanıcısına tıklıyoruz:

**URL:** `https://ctf.pitoctf.online/users/3`

**Profil Bilgileri:**

**Username:**
```
admin
```

**Affiliation:**
```
Farklı Logları Ara, Gizli Hedefi Odaklan, Süphelen, Bilgiyi Ustaca Leak et, Doğruyu Ulaşınca Kap.
```

> 💡 **İpucu Bulundu!** Bu metin bir akrostiş (acrostic) olabilir!

**Website:**  [🔗Link ](https://www.youtube.com/shorts/M7QUm_iDNsI)

**No solves yet** - Admin henüz çözmemiş

---

### 🧩 **4. Akrostiş (Acrostic) Analizi**

Admin'in Affiliation metnindeki **kelimelerin baş harflerini** alıyoruz:

```
Farklı Logları Ara, Gizli Hedefi Odaklan, Süphelen, Bilgiyi Ustaca Leak et, Doğruyu Ulaşınca Kap.
```

**Baş Harfler:**
- **F**arklı
- **L**ogları
- **A**ra
- **G**izli
- **H**edefi
- **O**daklan
- **Ş**üphelen (Ş)
- **B**ilgiyi
- **U**staca
- **L**eak
- **D**oğruyu
- **U**laşınca
- **K**ap

**Akrostiş Sonucu:**
```
F-L-A-G-H-O-Ş-B-U-L-D-U-K
```

---

### 🚩 **5. Flag Formatını Düzenleme**

Akrostiş'ten elde ettiğimiz kelime: **HOŞBULDUK**

CTF flag formatı: `FLAG{...}` 

---

## 🚩 **FLAG**

```
FLAG{HOŞBULDUK}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">👥<br><b>Users List</b><br><sub>Kullanıcı keşfi</sub></td>
<td align="center">🔍<br><b>Profile Analysis</b><br><sub>Profil inceleme</sub></td>
<td align="center">🧩<br><b>Acrostic Cipher</b><br><sub>Baş harf şifresi</sub></td>
</tr>
</table>

**Kullanılan Teknikler:**
- 👥 **User Enumeration:** Kullanıcı listesini tarama
- 🔍 **Profile Investigation:** Profil bilgilerini inceleme
- 🧩 **Acrostic Analysis:** Kelimelerin baş harflerinden mesaj çıkarma

---

## 💭 **Çözüm Akış Şeması**

```
👋 OSINT Challenge: "Hoşgeldin"
                    ↓
        👥 Users Sayfasına Git
                    ↓
        🔍 Admin Kullanıcısını Bul
                    ↓
        📝 Affiliation Alanını İncele
                    ↓
        "Farklı Logları Ara, Gizli Hedefi Odaklan..."
                    ↓
        🧩 Baş Harfleri Al (Acrostic)
                    ↓
        F-L-A-G-H-O-Ş-B-U-L-D-U-K
                    ↓
        📝 Flag Formatına Çevir
                    ↓
        🚩 FLAG{HOŞBULDUK}
```
