# 👶 Babyrev - Reverse Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Reverse-Challenge-darkblue?style=for-the-badge&logo=binary" alt="Reverse">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-String_Analysis-purple?style=for-the-badge&logo=code" alt="String">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Reverse  
**Seviye:** Kolay  
**Puan:** 200

**Challenge Dosyası:** [📥 Google Drive - Babyrev](https://drive.google.com/file/d/1GWnQcBj6o40grxUHagEbBsmjkzYPm4_N/view?usp=drivesdk)


---

## 🔍 Analiz

**Hedef:** Binary dosyasını analiz edip flag'i bulmak

**Strateji:** File analiz → Strings extraction → Flag

**İpuçları:**
- 👶 **Babyrev:** Başlangıç seviyesi reverse challenge
- 📝 **String analizi:** Flag direkt strings içinde
- ✅ **Not stripped:** Debug sembolleri var - kolay analiz!

---

## ✅ Çözüm Adımları

### 📥 **1. Dosyayı İndirme ve Analiz**

**Dosyayı Google Drive'dan indiriyoruz.**

**Dosya tipini kontrol:**

```bash
file Babyrev
```

**Çıktı:**
```
Babyrev: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), 
dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, 
BuildID[sha1]=0b4b6e2fc29b1a4d555524f0ee9a62cb5fdb3e60, 
for GNU/Linux 3.2.0, not stripped
```

**Analiz:**
- 🐧 **ELF 64-bit:** Linux executable
- 🔗 **Dynamically linked:** Harici kütüphaneler kullanıyor
- ✅ **Not stripped:** Debug sembolleri var → Kolay analiz!

---

### 🔤 **2. String Analizi**

**Strings komutu ile binary içindeki tüm string'leri çıkarıyoruz:**

```bash
strings Babyrev
```

**Önemli çıktılar:**

```bash
/lib64/ld-linux-x86-64.so.2
libc.so.6
puts
strcmp
Usage:
        ./easyrev <flag>
hctf{It_h4s_b33N_345Y}          # ← 🚩 FLAG!
You have found the flag!
Not the right one :(
```

> 🎯 **FLAG BULUNDU:** `hctf{It_h4s_b33N_345Y}`

---

### 💡 **3. Program Mantığı**

**Strings'ten program akışını anlıyoruz:**

**Kullanım:**
```
./easyrev <flag>
```

**Program ne yapıyor:**
- 📥 Kullanıcıdan input alıyor
- 🔍 `strcmp()` ile kontrol ediyor
- ✅ Doğruysa: "You have found the flag!"
- ❌ Yanlışsa: "Not the right one :("

**Doğru flag:**
```
hctf{It_h4s_b33N_345Y}
```

---

### 🧪 **4. Flag Doğrulama**

**Bulduğumuz flag'i test ediyoruz:**

```bash
chmod +x Babyrev
./Babyrev hctf{It_h4s_b33N_345Y}
```

**Çıktı:**
```
You have found the flag!
```

> ✅ **FLAG DOĞRULANDI!** Program doğru flag'i kabul etti!

---

## 🚩 **FLAG**

```
hctf{It_h4s_b33N_345Y}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>file</b><br><sub>Dosya tipi analizi</sub></td>
<td align="center">🔤<br><b>strings</b><br><sub>String extraction</sub></td>
<td align="center">🐧<br><b>Linux Terminal</b><br><sub>Komut çalıştırma</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**
```bash
# Dosya analizi
file Babyrev

# String çıkarma
strings Babyrev

# (Opsiyonel) Çalıştırma
chmod +x Babyrev
./Babyrev hctf{It_h4s_b33N_345Y}
```

---

## 💭 **Çözüm Akışı**

```
👶 "Babyrev" Challenge
            ↓
📥 Babyrev dosyası indirildi
            ↓
🔍 file Babyrev
   → ELF 64-bit, not stripped
            ↓
🔤 strings Babyrev
            ↓
📝 String'ler arasında tarama
            ↓
🎯 "hctf{It_h4s_b33N_345Y}" bulundu
            ↓
💡 strcmp() ile kontrol ediliyor
            ↓
🚩 hctf{It_h4s_b33N_345Y}
```
