# 🕵️ Bulamazlar - OSINT Challenge

<p align="center">
  <img src="https://img.shields.io/badge/OSINT-Challenge-darkblue?style=for-the-badge&logo=search" alt="OSINT">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Telegram%20OSINT-purple?style=for-the-badge&logo=telegram" alt="Telegram OSINT">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** OSINT 
**Seviye:** Kolay  
**Puan:** 100  
**Açıklama:** +zRmmBnNhTzIzNjE0

**Firstblood Bonus:** +50 puan

---

## 🔍 Analiz

### Challenge Özeti

**Hedef:** Verilen string'i kullanarak flag'i bulmak

**Verilen:**
```
+zRmmBnNhTzIzNjE0
```

**Strateji:** Username OSINT aracı kullanarak bu string'in hangi platformlarda kullanıldığını bulmak

---

## ✅ Çözüm Adımları

### 🌐 **1. WhatsMyName Tool Kullanımı**

İlk adım olarak **WhatsMyName** OSINT aracını kullanıyoruz.

**Site:** https://whatsmyname.me/

---

### 🔎 **2. Username Araması**

**Adımlar:**

1. https://whatsmyname.me/ sitesine gidiyoruz
2. Arama kutusuna verilen string'i yapıştırıyoruz:

```
+zRmmBnNhTzIzNjE0
```

3. **Search** butonuna tıklıyoruz

---

### 📊 **3. Arama Sonuçlarını İnceleme**

WhatsMyName taraması sonucunda:

> 🎯 **HEDEF BULUNDU:** Telegram kanalı/grubu mevcut!

---

### 📱 **4. Telegram Grubunu Açma**

```
https://t.me/+zRmmBnNhTzIzNjE0
```

**Adımlar:**

1. Link'e tıklıyoruz
2. Telegram açılıyor (web veya uygulama)
3. Kanal/Grup bilgileri görünüyor

**Grup Bilgileri:**

**Grup Adı:**
```
Kimse Bulamaz
```

**Aboneler:**
```
918 subscribers
```

**Yukarıdaki Mesajları İncelerken:**
```
SiberVatan{kapsonlunu_giy_ve_arkana_yaslan}
```

---

> 🚩 **FLAG BULUNDU!** Telegram grubunun eski mesajlarında flag var!

---

## 🚩 **FLAG**

```
SiberVatan{kapsonlunu_giy_ve_arkana_yaslan}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>WhatsMyName</b><br><sub>Username OSINT</sub></td>
<td align="center">📱<br><b>Telegram</b><br><sub>Messaging platform</sub></td>
<td align="center">🌐<br><b>Web Browser</b><br><sub>Platform erişimi</sub></td>
</tr>
</table>

**Kullanılan Siteler:**
- 🔍 **WhatsMyName.me:** https://whatsmyname.me/
- 📱 **Telegram Group:** https://t.me/+zRmmBnNhTzIzNjE0
- 🎯 **Challenge Site:** https://ctf.pitoctf.online/

---

## 💻 **Kullanılan Linkler**

### **Challenge'da Verilen:**
```
+zRmmBnNhTzIzNjE0
```

### **WhatsMyName Bulunan:**
```
https://t.me/%2BzRmmBnNhTzIzNjE0
```

### **Direkt Telegram Linki:**
```
https://t.me/+zRmmBnNhTzIzNjE0
```

---

## 💭 **Çözüm Akış Şeması**

```
🕵️ OSINT Challenge: "Bulamazlar"
                    ↓
        📝 String Veriliyor: +zRmmBnNhTzIzNjE0
                    ↓
        🌐 WhatsMyName.me'ye Git
                    ↓
        🔎 String'i Ara
                    ↓
        ⏳ 703 Platform Taranıyor...
                    ↓
        ✅ Telegram'da 100% Match!
                    ↓
        📱 Telegram Linkine Tıkla
                    ↓
        👥 "Kimse Bulamaz" Grubu Açıldı
                    ↓
        📜 Mesajları Yukarı Doğru Oku
                    ↓
        🚩 SiberVatan{kapsonlunu_giy_ve_arkana_yaslan}
```
