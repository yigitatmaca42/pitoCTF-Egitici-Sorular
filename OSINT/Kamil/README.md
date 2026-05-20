# 🔍 Kamil - OSINT Challenge

<p align="center">
  <img src="https://img.shields.io/badge/OSINT-Challenge-darkblue?style=for-the-badge&logo=search" alt="OSINT">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Username%20Research-purple?style=for-the-badge&logo=user-circle" alt="Username Research">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** OSINT  
**Seviye:** Orta  
**Puan:** 300
**Açıklama:** Kamil Bayramoğlu'nun çalınan şifresini bulabilir misin?


**Flag Formatı:** `Flag{şifre}`  

---

## 💡 Hint'ler

### **Hint 1:**
```
Aradığın Kamil bakkalmış. Yumurtayı az pişmiş severmiş. :)
```

### **Hint 2:**
```
Gerçek bir kişi aramıyoruz. Sahte hesap da değil. O bir karakter!
```

> 🎯 **İpucu Analizi:**
> - "Bakkal" → Bir dükkan sahibi karakteri
> - "Yumurtayı az pişmiş" → Çizgi film karakteri olabilir
> - "Karakter" → Gerçek kişi değil, animasyon/dizi karakteri

---

## 🔍 Analiz

### Challenge Özeti

**Hedef:** Kamil Bayramoğlu adlı karakterin şifresini bulmak

**Verilen Bilgiler:**
- İsim: Kamil Bayramoğlu
- Meslek: Bakkal
- Özellik: Yumurtayı az pişmiş seviyor
- Gerçek değil, bir karakter

---

## ✅ Çözüm Adımları

### 🌐 **1. Username OSINT Aracı Kullanımı**

İlk adım olarak **WhatsMyName** adlı OSINT aracını kullanıyoruz.

**Site:** https://whatsmyname.me/

---

### 🔎 **2. Username Araması**

**Adımlar:**

1. https://whatsmyname.me/ sitesine gidiyoruz
2. Arama kutucuğuna aşağıdaki yazıyı yazıyoruz:

```
rafadan tayfa kamil bayramoğlu şifre
```
---

### 📺 **3. Arama Sonuçlarını İnceleme**

WhatsMyName sonuçlarında şu platformları görüyoruz:

**YouTube Sonuçları:**

1. **Dijital Tayfa - İnternet Kapanı** → ✅ **Hedef video!**
   - Visit Link → Tıklıyoruz

---

### 🎬 **4. YouTube Videosunu İnceleme**

**Video:** Dijital Tayfa - İnternet Kapanı

**Challenge URL:** https://www.youtube.com/watch?v=KV8S4alhIv8

**Kanal:** TRT Çocuk

---

### 🎯 **5. Şifreyi Bulduk!**

**Kritik Zaman Aralığı: 7:33 - 7:41**

Videoda **7:33** ve **7:41** saniyeler arasında **Kamil'in şifresi ekranda görünüyor:**

**Ekranda Görünen Şifre:**
```
Kamil şifre :
G0lkrali!!!????paketservisindogruadrasi+-*/2011 Rafadandunzirtbirtzort
```

> 🎯 **ŞİFRE BULUNDU!**

---

### 📝 **6. Flag Formatına Çevirme**

Challenge'ın istediği format: `Flag{şifre}`

```
Flag{G0lkrali!!!????paketservisindogruadrasi+-*/2011Rafadandunzirtbirtzort}  
```

---

## 🚩 **FLAG**

```
Flag{G0lkrali!!!????paketservisindogruadrasi+-*/2011Rafadandunzirtbirtzort}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>WhatsMyName</b><br><sub>Username OSINT</sub></td>
<td align="center">📺<br><b>YouTube</b><br><sub>Video analizi</sub></td>
<td align="center">🎬<br><b>Video Player</b><br><sub>Zaman damgası</sub></td>
</tr>
</table>

**Kullanılan Siteler:**
- 🔍 **WhatsMyName.me:** https://whatsmyname.me/
- 📺 **YouTube Video:** https://www.youtube.com/watch?v=KV8S4alhIv8
- 🎯 **Challenge Site:** https://ctf.pitoctf.online/

---

## 💭 **Çözüm Akış Şeması**

```
🔍 OSINT Challenge: "Kamil"
                    ↓
        📝 Hint'leri Oku: "Bakkal, Yumurtayı az pişmiş, Karakter"
                    ↓
        🎯 Rafadan Tayfa karakteri olduğunu anla
                    ↓
        🌐 WhatsMyName.me'ye Git
                    ↓
        🔎 "rafadan tayfa kamil bayramoğlu şifre" ara
                    ↓
        📺 YouTube Sonuçlarını İncele
                    ↓
        🎬 "Dijital Tayfa - İnternet Kapanı" videosunu aç
                    ↓
        ⏱️ 7:33 - 7:41 saniyeler arası izle
                    ↓
        📝 Ekranda şifreyi gör
                    ↓
        G0lkrali!!!????paketservisindogruadrasi+-*/2011Rafadandunzirtbirtzort
                    ↓
        📋 Flag Formatına Çevir
                    ↓
        🚩 Flag{G0lkrali!!!????paketservisindogruadrasi+-*/2011Rafadandunzirtbirtzort}
```
