# 🎯 Z3 - Reverse Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Reverse-Challenge-darkblue?style=for-the-badge" alt="Reverse">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-SMT%20Solving-purple?style=for-the-badge" alt="SMT Solving">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Reverse  
**Seviye:** Orta  
**Puan:** 500

**Challenge Dosyası:** [📥 GitHub - Z3](https://github.com/cihangungor/pitoctf/blob/main/Z3)



---

## 🔍 Analiz

**Hedef:** ELF binary'yi tersine mühendislik ile analiz ederek flag'i bulmak

**Strateji:** Dosya tipi tespiti → Çalıştırma → Assembly analizi → Fonksiyon pointer tablosu → Z3 SMT solver ile constraint çözme

**İpuçları:**
- 🎯 Binary her çalıştığında `rand()` ile rastgele bir constraint fonksiyonu seçiyor
- 🔍 60 adet constraint fonksiyonu var — hepsi aynı anda doğrulanmıyor
- 💡 "MAYBE correct flag" → `bwctf{` prefix kontrolü geçti demek
- 🔑 Z3 SMT solver ile tüm constraint'ler aynı anda çözülebilir

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Dosya Tipini Tespit Et**

```bash
file Z3
```

**Çıktı:**
```
Z3: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked,
interpreter /lib64/ld-linux-x86-64.so.2, stripped
```

**Analiz:**
- ✅ 64-bit ELF Linux executable
- 💡 `stripped` → Debug sembolleri yok, analiz zorlaşıyor
- 🎯 Dinamik analiz + assembly okuma gerekiyor

---

### 🖥️ **Adım 2: Binary'i Çalıştır**

```bash
chmod +x Z3
./Z3
```

**Çıktı:**
```
Enter the flag: test
Wrong flag
```

**Farklı bir girişle:**
```bash
./Z3
```
```
Enter the flag: bwctf{test}
MAYBE correct flag
```

**Analiz:**
- ✅ Flag doğrulama programı
- 💡 İki farklı mesaj var: `Wrong flag` ve `MAYBE correct flag`
- 🎯 `MAYBE correct flag` → prefix kontrolü geçti ama tam doğru değil

---

### 🔍 **Adım 3: String Analizi**

```bash
strings Z3 | grep -v "^[[:space:]]*$" | head -20
```

**Çıktı:**
```
Enter the flag: 
MAYBE correct flag
Wrong flag
```

**Analiz:**
- 💡 Flag formatı assembly'den okunmalı
- 🎯 Static strings yeterli değil — assembly analizi gerekiyor

---

### 🧠 **Adım 4: Assembly Analizi — Flag Formatı**

```bash
objdump -d Z3 > z3_asm.txt
cat z3_asm.txt
```

Assembly'de `0x401210` adresinde flag prefix kontrolü:

```asm
401210:  cmpb $0x62,(%rdi)     ; f[0] == 'b'
401215:  cmpb $0x77,0x1(%rdi)  ; f[1] == 'w'
40121b:  cmpb $0x63,0x2(%rdi)  ; f[2] == 'c'
401221:  cmpb $0x74,0x3(%rdi)  ; f[3] == 't'
401227:  cmpb $0x66,0x4(%rdi)  ; f[4] == 'f'
40122d:  cmpb $0x7b,0x5(%rdi)  ; f[5] == '{'
401233:  cmpb $0x7d,0x2f(%rdi) ; f[47] == '}'
```

**Analiz:**
- ✅ Flag formatı: `bwctf{...}`
- 💡 Toplam 48 karakter (0x2f+1=48)
- 🎯 `0x401240`'ta tire pozisyonları kontrol ediliyor

---

### 📊 **Adım 5: Fonksiyon Pointer Tablosunu Keşfet**

```bash
ltrace ./Z3 <<< "test" 2>&1
```

**Çıktı:**
```
time(nil)    = 1773055939
srand(1773055939) = <void>
rand()       = 2113797782
puts("Wrong flag")
```

**Analiz:**
- 🎉 **Binary her çalışmada `rand()` ile rastgele bir constraint seçiyor!**
- 💡 `srand(time(nil))` → Her seferinde farklı seed
- 🎯 `.rodata`'da fonksiyon pointer tablosu var

```bash
objdump -s -j .rodata Z3 | grep -A 5 "402040"
```

**Çıktı:**
```
402040 00101240 00000000 00084012 40000000  ...@......@.@...
402050 00000570 12400000 ...
```

Her entry **9 byte**: 1 byte `flag_off` + 8 byte fonksiyon pointer.

```bash
python3 -c "
import struct
flag_offs = []
func_ptrs = []
with open('Z3', 'rb') as f:
    data = f.read()
offset = 0x2040
while offset < 0x225c:
    flag_off = data[offset]
    ptr = struct.unpack_from('<Q', data, offset+1)[0]
    if 0x401200 <= ptr <= 0x401800:
        flag_offs.append(flag_off)
        func_ptrs.append(hex(ptr))
    offset += 9
print(f'Toplam fonksiyon: {len(flag_offs)}')
"
```

**Çıktı:**
```
Toplam fonksiyon: 60
```

**Analiz:**
- ✅ 60 adet constraint fonksiyonu var
- 💡 Her fonksiyon `rdi = flag + flag_off` adresindeki karakterleri kontrol ediyor
- 🎯 Binary her seferinde sadece birini test ediyor → Z3 ile hepsini birden çözmeliyiz

---

### 🔢 **Adım 6: Constraint'leri Çıkar**

Assembly'deki her fonksiyonu analiz ederek constraint'leri çıkardık. Örnekler:

```
func=0x401240 flag_off=8  → f[11]=='-', f[17]=='-', f[23]=='-', f[29]=='-', f[35]=='-', f[41]=='-'
func=0x4012f0 flag_off=13 → f[13] * f[27] == 0xd59
func=0x401360 flag_off=9  → f[20] ^ f[19] == 0x66
func=0x401540 flag_off=0  → f[14] + f[15] == 0x84
func=0x401710 flag_off=6  → f[7] * f[21] == 0x1773
```

**Toplam constraint türleri:**
- Eşitlik (`sete`): `f[i] == f[j]` veya `f[i] op f[j] == k`
- Küçüktür (`setl`): `f[i] < f[j]`
- Büyüktür (`setg`): `f[i] > f[j]`
- Operasyonlar: `XOR`, `ADD`, `MUL`

---

### 🐍 **Adım 7: Z3 SMT Solver ile Çöz**

```python
from z3 import *

f = [BitVec(f'f{i}', 8) for i in range(48)]
s = Solver()

# Yazdırılabilir karakterler
for i in range(48):
    s.add(f[i] >= 0x20, f[i] <= 0x7e)

# Flag formatı
s.add(f[0]==ord('b'), f[1]==ord('w'), f[2]==ord('c'))
s.add(f[3]==ord('t'), f[4]==ord('f'), f[5]==ord('{'))
s.add(f[47]==ord('}'))

# Tire pozisyonları
s.add(f[11]==ord('-'), f[17]==ord('-'), f[23]==ord('-'))
s.add(f[29]==ord('-'), f[35]==ord('-'), f[41]==ord('-'))

# Constraint'ler (60 fonksiyondan çıkarıldı)
s.add(f[13] * f[27] == 0xd59)   # mul
s.add(f[20] ^ f[19] == 0x66)    # xor
s.add(f[14] + f[15] == 0x84)    # add
s.add(f[7]  * f[21] == 0x1773)  # mul
s.add(f[25] == f[34])            # eq
# ... (toplam 60 constraint)

if s.check() == sat:
    m = s.model()
    flag = ''.join(chr(m[f[i]].as_long()) for i in range(48))
    print('FLAG:', flag)
```

**Çıktı:**
```
FLAG: bwctf{WE1C0-M3T0B-1U3W4-T3RCT-FH0P3-Y0UH4-VEFUN}
```

---

### ✅ **Adım 8: Flag'i Doğrula**

```bash
echo "bwctf{WE1C0-M3T0B-1U3W4-T3RCT-FH0P3-Y0UH4-VEFUN}" | ./Z3
```

**Çıktı:**
```
Enter the flag: MAYBE correct flag
```

**Analiz:**
- ✅ **FLAG DOĞRULANDI!** 🎉
- 💡 `MAYBE correct flag` → Binary'nin doğru flag mesajı bu (rastgele constraint seçimi yüzünden "kesin doğru" diyemiyor)
- 🎯 CTF platformu flag'i kabul etti ✅

---

## 🚩 **FLAG**
```
bwctf{WE1C0-M3T0B-1U3W4-T3RCT-FH0P3-Y0UH4-VEFUN}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>file</b><br><sub>Dosya tipi tespiti</sub></td>
<td align="center">🔧<br><b>objdump</b><br><sub>Assembly analizi</sub></td>
<td align="center">🕵️<br><b>ltrace</b><br><sub>Library call takibi</sub></td>
<td align="center">🐍<br><b>Python</b><br><sub>Constraint çıkarma</sub></td>
<td align="center">🧮<br><b>Z3</b><br><sub>SMT solver</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 Z3 Reverse Challenge
       ↓
📂 Dosyayı incele
   → ELF 64-bit, stripped
       ↓
🖥️ Binary'i çalıştır
   → "Wrong flag" veya "MAYBE correct flag"
   → İki farklı mesaj var
       ↓
🧠 Assembly analizi
   → objdump ile tüm fonksiyonlar incelendi
   → Flag formatı: bwctf{...} (48 karakter)
   → 60 adet constraint fonksiyonu tespit edildi
       ↓
📊 Fonksiyon pointer tablosu
   → .rodata: 0x402040'tan başlıyor
   → Her entry 9 byte: 1 byte flag_off + 8 byte ptr
   → ltrace: rand() ile rastgele seçim yapılıyor
       ↓
🔢 Constraint'leri çıkar
   → rdi = flag + flag_off
   → XOR, ADD, MUL, EQ, LT, GT operasyonları
   → Toplam 60 constraint
       ↓
🧮 Z3 SMT solver
   → pip install z3-solver
   → Tüm constraint'ler aynı anda çözüldü
   → bwctf{WE1C0-M3T0B-1U3W4-T3RCT-FH0P3-Y0UH4-VEFUN}
       ↓
✅ Doğrula
   → echo "flag" | ./Z3
   → "MAYBE correct flag" ✅
   → CTF platformu kabul etti ✅
       ↓
🚩 FLAG: bwctf{WE1C0-M3T0B-1U3W4-T3RCT-FH0P3-Y0UH4-VEFUN}
```
