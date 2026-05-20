# 🎯 HackliEkmek - Stego Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Stego-Challenge-darkblue?style=for-the-badge" alt="Stego">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Steganography-purple?style=for-the-badge" alt="Stego">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Stego  
**Seviye:** Orta  
**Puan:** 300  

**Challenge Dosyası:** [📥 Google Drive - IMG-20260401-WA0011.jpg](https://drive.google.com/file/d/1YJXcF-w_PBAZt5uRTzCwCW0-QHk4fOdD/view)

---

## 🔍 Analiz

**Hedef:** Görsel içerisindeki flagi bulmak  

**Strateji:** AperiSolve → Şifre kısmı → Flag deneme  

**İpuçları:**
- 🎯 AperiSolve kullanıldı
- 🔍 Password kısmında flag göründü
- 💡 Fake yazısına aldanma

---

## ✅ Çözüm Adımları

### 🔎 Adım 1: AperiSolve Analizi

Görsel AperiSolve'a yüklendi.

```bash
# aperisolve kullanıldı
```

**Çıktı:**
```
Fakeflagdenemeyinbeyler
Flag{haberimizy0kmusg1b1cekAbi}
```

**Analiz:**
- ✅ Flag direkt görünüyor
- 💡 Fake yazısı aslında kandırmaca
- 🎯 Flag denenmeli

---

### 🚩 Adım 2: Flag'i Elde Et

Flag denendi ve doğru olduğu görüldü.

```bash
Flag{haberimizy0kmusg1b1cekAbi}
```

---

## 🚩 **FLAG**
```
Flag{haberimizy0kmusg1b1cekAbi}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🌐<br><b>AperiSolve</b><br><sub>Stego analiz</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 HackliEkmek
       ↓
🔎 AperiSolve
   → Password kısmı
   → Flag bulundu
       ↓
🚩 FLAG: Flag{haberimizy0kmusg1b1cekAbi}
```
