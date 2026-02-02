# 🌐 Internet - OSINT Challenge

<p align="center">
  <img src="https://img.shields.io/badge/OSINT-Challenge-darkblue?style=for-the-badge&logo=search" alt="OSINT">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Video Analysis-purple?style=for-the-badge&logo=youtube" alt="Video">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** OSINT  
**Seviye:** Orta  
**Puan:** 300  

**Açıklama:** Turuncu bir balık ve mavi bir kedinin internet hakkında konuştuklarını duydum. Anneleri zaman tünelinde şifresini paylaşmış. Şifreyi bulabilir misin?

**Firstblood:** +50 puan

**Flag Formatı:** `Flag{şifre}`

---

## 🔍 Analiz

**Hedef:** Video içeriğinde gizli şifreyi bulmak

**Strateji:** Karakter analizi → Google Search → Video bulma → Şifre keşfi

**İpuçları:**
- 🐠 **Turuncu balık** → Darwin
- 🐱 **Mavi kedi** → Gumball
- 👩 **Anneleri** → Nicole
- 🕰️ **Zaman tüneli** → Video içeriği

---

## ✅ Çözüm Adımları

### 🎭 **1. Karakter Analizi**

**Soruda geçen karakterler:**
- Turuncu balık = **Darwin**
- Mavi kedi = **Gumball**
- Anneleri = **Nicole**

> 💡 Bu karakterler **"The Amazing World of Gumball"** çizgi filminden!

---

### 🔎 **2. Google Araması**

Google'da aratıyoruz:

```
gumball darwin internet
```

**Sonuçlar:**
- İnternet
- İnternet Ağı 

---

### 📺 **3. Doğru Video Seçimi**

İki sonuç çıkıyor, açıklamalara bakıyoruz:

**İnternet Ağı bölümünün açıklaması:**
```
Gumball ve Darwin'i zor bir görev bekliyor. 
Nicole'a bilgisayarı nasıl kullanması gerektiğini öğretiyorlar.
```

> ✅ Bu doğru video! Nicole bilgisayar kullanmayı öğreniyor ve şifre giriyor.

---

### 🎬 **4. Video İnceleme**

Videoyu açıp hızlandırarak izlemeye başlıyoruz.

**07:30 saniyesinde:**
- Nicole şifresini giriyor
- Darwin: "Şifreni zaman tünelinde paylaştın" diyor
- 🔐 **Şifre görünüyor:** `7eR3$@`

---

### 🏆 **5. Flag Oluşturma**

Bulduğumuz şifreyi flag formatına uygun şekilde yazıyoruz:

```
Flag{7eR3$@}
```

---

## 🚩 **FLAG**

```
Flag{7eR3$@}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>Google</b><br><sub>Arama motoru</sub></td>
<td align="center">📺<br><b>YouTube</b><br><sub>Video platformu</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akışı**

```
🌐 "Internet" Challenge
        ↓
🎭 Karakter analizi
   (Gumball + Darwin + Nicole)
        ↓
🔎 Google: "gumball darwin internet"
        ↓
📺 2 sonuç: "İnternet Ağı" bulundu
        ↓
📝 Açıklamada Nicole'un bilgisayar
   öğrendiği bölüm olduğu anlaşıldı
        ↓
🎬 Video 07:30'da şifre girişi
        ↓
🔐 Şifre: 7eR3$@
        ↓
🚩 Flag{7eR3$@}
```
