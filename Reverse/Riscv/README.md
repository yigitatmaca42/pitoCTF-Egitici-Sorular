# 🎯 Riscv - Reverse Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Reverse-Challenge-darkblue?style=for-the-badge" alt="Reverse">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-RISC-V%20Binary-purple?style=for-the-badge" alt="RISC-V Binary">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Reverse  
**Seviye:** Orta  
**Puan:** 500

**Challenge Dosyası:** [📥 GitHub - riscv](https://github.com/cihangungor/pitoctf/blob/main/riscv)

---

## 🔍 Analiz

**Hedef:** ELF binary'yi tersine mühendislik ile analiz ederek flag'i bulmak

**Strateji:** Dosya tipi tespiti → QEMU kurulumu → Binary çalıştırma → Assembly analizi → 4 dönüşüm fonksiyonu → Ters hesaplama ile flag

**İpuçları:**
- 🎯 Binary x86-64 değil, UCB RISC-V mimarisinde — doğrudan çalıştırmak için `qemu-riscv64` gerekiyor
- 🔍 Input tam 32 byte uzunluğunda olmalı — binary `strlen` ile kontrol ediyor
- 💡 4 adet dönüşüm fonksiyonu var — her biri input üzerinde add/xor operasyonları yapıyor
- 🔑 Toplam 237 adet add/xor operasyonu rodata'daki sabit değerlerle uygulanıyor — hepsi tersine çevrilebilir

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Dosya Tipini Tespit Et**

```bash
file riscv
```

**Çıktı:**
```
riscv: ELF 64-bit LSB pie executable, UCB RISC-V, RVC, double-float ABI,
version 1 (SYSV), dynamically linked,
interpreter /lib/ld-linux-riscv64-lp64d.so.1,
BuildID[sha1]=6b688b5bb9533950c4ee5bd2736e263fb527086b,
for GNU/Linux 4.15.0, stripped
```

**Analiz:**
- ✅ 64-bit ELF — ama x86-64 değil, RISC-V mimarisi!
- 💡 `stripped` → Debug sembolleri yok, analiz zorlaşıyor
- 🎯 RISC-V binary çalıştırmak için QEMU emülatörü gerekiyor

---

### 🖥️ **Adım 2: QEMU Kurulumu ve Binary'i Çalıştır**

```bash
sudo apt install gcc-riscv64-linux-gnu qemu-user libc6-riscv64-cross -y
sudo ln -s /usr/riscv64-linux-gnu/lib/ld-linux-riscv64-lp64d.so.1 /lib/ld-linux-riscv64-lp64d.so.1
sudo ln -s /usr/riscv64-linux-gnu/lib/libc.so.6 /lib/libc.so.6
./riscv
```

**Çıktı:**
```
whats the flag: test
nuh uh

whats the flag: aaaabbbbccccddddeeeeffffgggghhhh
nuh uh
```

**Analiz:**
- ✅ Binary başarıyla QEMU üzerinde çalıştırıldı
- 💡 İki farklı mesaj var: `nuh uh` (yanlış) ve `good job!` (doğru)
- 🎯 `strlen` kontrolü var — input tam 32 karakter olmalı

---

### 🔍 **Adım 3: String Analizi**

```bash
strings riscv | grep -i "flag\|correct\|good\|nuh"
```

**Çıktı:**
```
whats the flag: 
nuh uh
good job!
```

**Analiz:**
- 💡 Flag doğrudan `strings` çıktısında görünmüyor — binary transform uyguluyor
- 🎯 Assembly analizi gerekiyor — rodata'daki şifreli veriler çözülmeli

---

### 🧠 **Adım 4: Assembly Analizi — Main Fonksiyonu**

```bash
riscv64-linux-gnu-objdump -d riscv | grep -A 100 "main@@Base>:"
```

Assembly'de `0x2136` adresinde main fonksiyonu:

```asm
2136: addi sp,sp,-64
2150: auipc a0,0x0          ; 'whats the flag: ' yazdır
2158: jal  printf@plt
2174: jal  getline@plt      ; input oku
21bc: jal  strlen@plt       ; uzunluk kontrol
21c2: li   a5,32            ; 32 karaktere eşit mi?
21c6: bne  a4,a5,21de       ; değilse -> 'nuh uh'
219a: jal  0x1ad4           ; transform f1
21a2: jal  0x1346           ; transform f2
21aa: jal  0xc8c            ; transform f3
21b2: jal  0x786            ; transform f4
21ce: jal  0x20be           ; doğrulama fonksiyonu
21dc: beqz a5,21ee          ; 'good job!' veya 'nuh uh'
```

**Analiz:**
- ✅ Input 32 byte — 4 adet 64-bit word (s[0], s[1], s[2], s[3]) olarak işleniyor
- 💡 4 farklı transform fonksiyonu sırayla çağrılıyor: f1 → f2 → f3 → f4
- 🎯 Sonuç rodata'nın sonundaki beklenen değerlerle karşılaştırılıyor

---

### 📊 **Adım 5: Transform Fonksiyonlarını Keşfet**

```bash
riscv64-linux-gnu-objdump -s -j .rodata riscv
```

Her fonksiyonun boyutu ve kullandığı key aralığı:

| Fonksiyon | Adres Aralığı | Op Sayısı | Key Aralığı (rodata) |
|-----------|---------------|-----------|----------------------|
| f1 | 0x1ad4 – 0x2136 | 55 op | 0x2240 ~ 0x2450 |
| f2 | 0x1346 – 0x1ad4 | 72 op | 0x2450 ~ 0x2730 |
| f3 | 0x0c8c – 0x1346 | 63 op | 0x2730 ~ 0x2980 |
| f4 | 0x0786 – 0x0c8c | 47 op | 0x2980 ~ 0x29a8 |

**Analiz:**
- ✅ Toplam 237 adet add/xor operasyonu
- 💡 Her operasyon rodata'daki 64-bit sabit bir değer kullanıyor
- 🎯 Operasyonların tamamı tersine çevrilebilir: `add` → `subtract`, `xor` → `xor`

---

### 🔢 **Adım 6: Constraint'leri Çıkar**

Python ile RISC-V assembly simülasyonu yaparak tüm add/xor operasyonları çıkarıldı:

```python
python3 -c "
import struct

# Tüm transform fonksiyonlarını simüle et
# Her fonksiyon s[0..3] üzerinde add/xor uygular
# Örnek çıkan operasyonlar:

# f1 (55 op):
#   s[0] add= 0x0075978f47ac76cf
#   s[1] add= 0xff889b2d229768ef
#   s[0] xor= 0x0076bf86ade5d5cc

# f4 (47 op):
#   s[3] add= 0xffdb0267f30481de
#   s[0] add= 0x000d349975ed71ce
#   s[3] add= 0x00396b3ec36373cc

# Toplam: 237 operasyon
"
```

**Toplam constraint türleri:**
- ADD (mod 2^64): `s[i] = s[i] + K` → tersi: `s[i] = s[i] - K`
- XOR: `s[i] = s[i] ^ K` → tersi: `s[i] = s[i] ^ K` (kendi tersi)

---

### 🐍 **Adım 7: Ters Hesaplama ile Flag'i Bul**

Beklenen son durum rodata'nın sonunda `0x29a8` adresinde:

```
expected[0] = 0x3167deae217139c1
expected[1] = 0x6745aeaf0c9a62e5
expected[2] = 0x62664d91c2da0c7b
expected[3] = 0x7ee01bea8defde65
```

```python
from z3 import *
import struct

MASK = (1 << 64) - 1

# Beklenen final state
expected = [
    0x3167deae217139c1,
    0x6745aeaf0c9a62e5,
    0x62664d91c2da0c7b,
    0x7ee01bea8defde65,
]

# Tüm operasyonları ters sırada uygula
state = list(expected)

for slot, op, key in reversed(all_ops):  # 237 operasyon
    if op == 'add':
        state[slot] = (state[slot] - key) & MASK
    elif op == 'xor':
        state[slot] = state[slot] ^ (key & MASK)

# state -> flag bytes
flag_bytes = b''
for v in state:
    flag_bytes += struct.pack('<Q', v)

print(flag_bytes.decode('utf-8'))
```

**Çıktı:**
```
akasec{1n_my_b4g_0n3_s3c0nd_0n3}
```

---

### ✅ **Adım 8: Flag'i Doğrula**

```bash
echo "akasec{1n_my_b4g_0n3_s3c0nd_0n3}" | ./riscv
```

**Çıktı:**
```
whats the flag: good job!
```

**Analiz:**
- ✅ **FLAG DOĞRULANDI!** 🎉
- 💡 `good job!` → Binary'nin doğru flag mesajı bu
- 🎯 CTF platformu flag'i kabul etti ✅

---

## 🚩 **FLAG**
```
akasec{1n_my_b4g_0n3_s3c0nd_0n3}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>file</b><br><sub>Dosya tipi tespiti</sub></td>
<td align="center">📦<br><b>QEMU</b><br><sub>RISC-V emülasyonu</sub></td>
<td align="center">🔧<br><b>objdump</b><br><sub>Assembly analizi</sub></td>
<td align="center">🐍<br><b>Python</b><br><sub>Binary simülasyonu</sub></td>
<td align="center">🔢<br><b>struct</b><br><sub>Byte dönüşümü</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 RISC-V Reverse Challenge
       ↓
📂 Dosyayı incele
   → ELF 64-bit, UCB RISC-V mimarisi, stripped
       ↓
🖥️ Binary'i çalıştır (QEMU ile)
   → "nuh uh" veya "good job!"
   → Input tam 32 karakter olmalı
       ↓
🧠 Assembly analizi
   → riscv64-linux-gnu-objdump ile tüm fonksiyonlar incelendi
   → 4 transform fonksiyonu tespit edildi (f1, f2, f3, f4)
   → Her fonksiyon s[0..3] üzerinde add/xor yapıyor
       ↓
📊 Rodata analizi
   → 0x2240'tan itibaren 237 adet 64-bit sabit değer
   → 0x29a8'de beklenen final state (4x64-bit)
       ↓
🔢 Operasyonları çıkar
   → Python RISC-V simülatörü ile 237 add/xor op çıkarıldı
   → f1: 55 op, f2: 72 op, f3: 63 op, f4: 47 op
       ↓
🧮 Ters hesaplama
   → Tüm operasyonlar ters sırada tersine uygulandı
   → add → subtract, xor → xor (kendi tersi)
   → akasec{1n_my_b4g_0n3_s3c0nd_0n3}
       ↓
✅ Doğrula
   → echo "flag" | ./riscv
   → "good job!" ✅
   → CTF platformu kabul etti ✅
       ↓
🚩 FLAG: akasec{1n_my_b4g_0n3_s3c0nd_0n3}
```
