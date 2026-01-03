# 🔧 GecersizKoduPathle - Reverse Engineering Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Reverse Engineering-Challenge-darkblue?style=for-the-badge&logo=hackthebox" alt="Reverse">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type String Analysis-purple?style=for-the-badge&logo=code" alt="String Analysis">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Reverse Engineering  
**Seviye:** Orta  
**Açıklama:** Bozmuşlar dosyayı. Kaynak kod görünmüyor.

**Challenge Dosyası:** [📥 GitHub - Gecersizkodupatchle](https://github.com/cihangungor/pitoctf/blob/main/Gecersizkodupatchle)

---

## 🔍 Analiz

### İkili (ELF) Dosyası İncelemesi

Dosyayı incelediğimizde şunları tespit ediyoruz:

**Kontrol Mekanizması:**
- `strncmp` fonksiyonu ile **3'er baytlık** parçalar halinde kontrol yapıyor
- Toplam uzunluk **0xC (12 karakter)** olarak kontrol ediliyor

**String Karşılaştırması:**
```c
strncmp(input, "Itz", 3)
strncmp(input+3, "_0n", 3)
strncmp(input+6, "Ly_", 3)
strncmp(input+9, "UD2", 3)
```

---

## ✅ Çözüm Adımları

### 🔍 **1. .rodata Bölümü İncelemesi**

`.rodata` (read-only data) içinde karşılaştırılan parçaları buluyoruz:

**Bulunan String Parçaları:**

| Offset | Parça | Uzunluk |
|--------|-------|---------|
| 0 | `Itz` | 3 byte |
| 3 | `_0n` | 3 byte |
| 6 | `Ly_` | 3 byte |
| 9 | `UD2` | 3 byte |

**Toplam:** 12 karakter (0xC)

---

### 🧩 **2. String Birleştirme**

Parçaları sırasıyla birleştirdiğimizde:

```
Itz + _0n + Ly_ + UD2 = Itz_0nLy_UD2
```

---

### 🔐 **3. Flag Formatı**

Program `printf("> HTB{%s}\n", input)` ile flag'i yazdırıyor.

**Çıktı Formatı:**
```
HTB{Itz_0nLy_UD2}
```

---

## 🧪 **4. Doğrulama (Test)**

### Terminal'de Test

**Komut 1: Çalıştırılabilir yapma**
```bash
chmod +x Gecersizkodupatchle
```

**Komut 2: Çalıştırma**
```bash
./Gecersizkodupatchle Itz_0nLy_UD2
```

**Çıktı:**
```
> HTB{Itz_0nLy_UD2}
```

✅ Flag doğrulandı!

---

## 🚩 **FLAG**

```
HTB{Itz_0nLy_UD2}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">👻<br><b>Ghidra</b><br><sub>Decompiler</sub></td>
<td align="center">🔍<br><b>strings</b><br><sub>String extraction</sub></td>
<td align="center">📊<br><b>readelf</b><br><sub>.rodata analizi</sub></td>
<td align="center">🖥️<br><b>Terminal</b><br><sub>Test ortamı</sub></td>
</tr>
</table>

**Terminal Komutları:**
```bash
# Dosya türü kontrolü
file Gecersizkodupatchle

# String'leri görüntüleme
strings Gecersizkodupatchle

# .rodata bölümünü inceleme
readelf -x .rodata Gecersizkodupatchle

# Çalıştırma
chmod +x Gecersizkodupatchle
./Gecersizkodupatchle Itz_0nLy_UD2
```

---

## 💭 **Çözüm Akış Şeması**

```
📥 Dosya İndirme: Gecersizkodupatchle (ELF)
            ↓
🔍 İkili Analizi: strncmp fonksiyonu tespit
            ↓
📊 Kontrol Mekanizması:
   - 3'er baytlık parçalar
   - Toplam 0xC (12) karakter
            ↓
🔎 .rodata İncelemesi:
   - "Itz" (offset 0)
   - "_0n" (offset 3)
   - "Ly_" (offset 6)
   - "UD2" (offset 9)
            ↓
🧩 String Birleştirme: Itz_0nLy_UD2
            ↓
🔐 Flag Formatı: printf("> HTB{%s}\n")
            ↓
🧪 Test: ./Gecersizkodupatchle Itz_0nLy_UD2
            ↓
✅ Çıktı: > HTB{Itz_0nLy_UD2}
            ↓
🚩 FLAG: HTB{Itz_0nLy_UD2}
```
