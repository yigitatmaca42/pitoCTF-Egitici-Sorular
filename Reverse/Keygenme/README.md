# 🎯 Keygenme - Reverse Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Reverse-Engineering-darkblue?style=for-the-badge" alt="Reverse">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Anti--Debug Bypass-purple?style=for-the-badge" alt="Type">
</p>

---

## 📋 Soru Bilgileri
  
**Kategori:** Reverse  
**Seviye:** Orta
**Puan:** 400

**Challenge Dosyası:** [📥 Google Drive - keygenvarmi](https://drive.google.com/file/d/1wAhB2EC_dU6pBfD8ALYF9RIPEghnESDS/view?usp=drivesdk)

---

## 🔍 Analiz

**Hedef:** Anti-debugging korumasını bypass edip key validation algoritmasını tersine çevirmek

**Strateji:** Binary analizi → Anti-debug tespiti → ptrace bypass → Disassembly → Key extraction → Flag

**İpuçları:**
- 🎯 "stripped" → Debug sembolleri silinmiş
- 🔍 "ptrace" → Anti-debugging mekanizması
- 🤖 "dynamically linked" → Sistem kütüphaneleri kullanılıyor
- 💡 64-bit ELF → x86-64 assembly bilgisi gerekli

---

## ✅ Çözüm Adımları

### 📁 **Adım 1: Binary'yi İndirme ve Temel Analiz**

**Dosyayı indirdikten sonra file komutuyla inceliyoruz:**

```bash
file keygenvarmi
```

**Output:**
```
keygenvarmi: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=78ea8f7de5c95806876ee876276577dd1249197e, for GNU/Linux 3.2.0, stripped
```

**Analiz:**
- ✅ ELF 64-bit executable
- ✅ x86-64 architecture
- ✅ Dynamically linked (system libraries kullanıyor)
- ⚠️ **stripped** → Debug sembolleri yok!
- 💡 PIE (Position Independent Executable)

---

### 🎮 **Adım 2: Binary'yi Çalıştırma - İlk Denemeler**

**Çalıştırma izni veriyoruz:**

```bash
chmod +x keygenvarmi
```

**Binary'yi çalıştırıyoruz:**

```bash
./keygenvarmi
```

**Output:**
```
License key: 42
Invalid key!
```

**İkinci deneme:**

```bash
./keygenvarmi
```

**Input:**
```
License key: test123
Invalid key!
```

**Analiz:**
- ✅ Program kullanıcıdan input alıyor
- ✅ "Invalid key!" mesajı veriyor
- 💡 Key validation mekanizması var
- 🎯 Doğru key'i bulmamız gerekiyor

---

### 🔍 **Adım 3: String Analizi**

**Binary içindeki string'leri inceliyoruz:**

```bash
strings keygenvarmi
```

**Output:**
```
/lib64/ld-linux-x86-64.so.2
v'ew
mgUa
fgets
stdin
puts
__strcpy_chk
__stack_chk_fail
__printf_chk
exit
strlen
usleep
ptrace
strcspn
__libc_start_main
__cxa_finalize
libc.so.6
...
Debugger detected!
9*3$"
.#%9
0#!)
0#/#;#
'0')
-)      ',&+/
0#!).'0+/?
7!!'11cb
'0'b+1b6*'b$.#%xH
,4#.+&b)';cH
+!',1'b)';xb
...
```

**❗ ÖNEMLİ BULGULAR:**

**Analiz:**
- ⚠️ **"Debugger detected!"** → Anti-debugging var!
- ⚠️ **ptrace** fonksiyonu → Debugger kontrolü yapıyor
- 💡 Şifreli string'ler var (9*3$", .#%9, vb.)
- ✅ fgets, strlen, strcspn → Input işleme
- 🎯 Key validation algoritması karmaşık

---

### 🐛 **Adım 4: ltrace ile Fonksiyon Çağrılarını İnceleme**

**ltrace ile dinamik analiz deniyoruz:**

```bash
ltrace ./keygenvarmi
```

**Input:**
```
License key: 13
```

**Output:**
```
__printf_chk(2, "%s", "License key: ")     = 13
fgets("13\n", 100, 0x7ff1208158e0)         = 0x7ffd2300c7e0
strcspn("13\n", "\n")                      = 2
ptrace(0, 0, 1, 0)                         = -1
puts("Debugger detected!"Debugger detected!
)                 = 19
exit(1 <no return ...>
+++ exited (status 1) +++
```

**❌ PROBLEM: Debugger Detected!**

**Analiz:**
- ⚠️ `ptrace(0, 0, 1, 0)` çağrısı yapılıyor
- ⚠️ Return değeri -1 → Debugger tespit edildi
- ⚠️ Program exit(1) ile sonlanıyor
- 💡 ptrace'i bypass etmemiz gerekiyor!

---

### 🔓 **Adım 5: Anti-Debug Bypass - ptrace Override**

**ptrace bypass kütüphanesi oluşturuyoruz:**

```bash
cat > bypass_ptrace.c << 'EOF'
long ptrace(int request, int pid, void *addr, void *data) {
    return 0;
}
EOF
```

**Kütüphaneyi compile ediyoruz:**

```bash
gcc -shared -fPIC bypass_ptrace.c -o bypass_ptrace.so
```

**Binary'yi bypass kütüphanesi ile çalıştırıyoruz:**

```bash
LD_PRELOAD=./bypass_ptrace.so ./keygenvarmi
```

**Input:**
```
License key: 13
```

**Output:**
```
Invalid key!
```

**✅ BAŞARILI! Debugger kontrolü bypass edildi!**

**Analiz:**
- ✅ LD_PRELOAD ile ptrace override edildi
- ✅ "Debugger detected!" mesajı çıkmadı
- ✅ Program normal çalışıyor
- 💡 Artık key validation algoritmasını analiz edebiliriz

---

### 🔬 **Adım 6: Radare2 ile Disassembly**

**Radare2 ile binary'yi açıyoruz:**

```bash
r2 -A ./keygenvarmi
```

**Output:**
```
WARN: Relocs has not been applied. Please use `-e bin.relocs.apply=true` or `-e bin.cache=true` next time
INFO: Analyze all flags starting with sym. and entry0 (aa)
INFO: Analyze imports (af@@@i)
INFO: Analyze entrypoint (af@ entry0)
INFO: Analyze symbols (af@@@s)
...
```

**Fonksiyonları listeliyoruz:**

```
[0x000012a0]> afl
```

**Output:**
```
0x000010e0    1     10 sym.imp.puts
0x000010f0    1     10 sym.imp.strlen
0x00001100    1     10 sym.imp.__stack_chk_fail
0x00001110    1     10 sym.imp.strcspn
0x00001120    1     10 sym.imp.fgets
0x00001130    1     10 sym.imp.ptrace
0x00001140    1     10 sym.imp.__strcpy_chk
0x00001150    1     10 sym.imp.__printf_chk
0x00001160    1     10 sym.imp.exit
0x00001170    1     10 sym.imp.usleep
0x000012a0    1     37 entry0
0x00001180    8    276 main
0x00001380    5     60 entry.init0
0x00001340    5     54 entry.fini0
0x000010d0    1     10 fcn.000010d0
0x000012d0    4     34 fcn.000012d0
0x000013f0    5     41 fcn.000013f0
0x00001570   24    294 fcn.00001570
0x000016a0    8    111 fcn.000016a0
```

**Analiz:**
- ✅ main fonksiyonu: 0x00001180
- ✅ Şüpheli fonksiyon: fcn.00001570 (294 byte!)
- 💡 fcn.00001570 muhtemelen validation fonksiyonu

---

### 📊 **Adım 7: Main Fonksiyonunu İnceleme**

**Main fonksiyonunu disassemble ediyoruz:**

```
[0x000012a0]> pdf @main
```

**Kritik kısımlar:**

```assembly
│           0x00001202      488b15872e..   mov rdx, qword [obj.stdin]
│           0x00001209      be64000000     mov esi, 0x64           ; 'd' ; int size
│           0x0000120e      4889df         mov rdi, rbx            ; char *s
│           0x00001211      e80affffff     call sym.imp.fgets
│           0x00001216      4885c0         test rax, rax
│       ┌─< 0x00001219      7431           je 0x124c
│       │   0x0000121b      4889df         mov rdi, rbx
│       │   0x0000121e      488d35f50d..   lea rsi, [0x0000201a]   ; "\n"
│       │   0x00001225      e8e6feffff     call sym.imp.strcspn
│       │   0x0000122a      4889df         mov rdi, rbx
│       │   0x0000122d      c6040400       mov byte [rsp + rax], 0
│       │   0x00001231      e83a030000     call fcn.00001570       ; ← KEY VALIDATION!
│       │   0x00001236      85c0           test eax, eax
│      ┌──< 0x00001238      7538           jne 0x1272
```

**Analiz:**
- ✅ fgets ile input alınıyor
- ✅ strcspn ile newline temizleniyor
- ✅ **fcn.00001570 çağrılıyor** → Validation!
- ✅ Return değeri test ediliyor
- 💡 eax = 0 ise Invalid, eax ≠ 0 ise Success

---

### 🎯 **Adım 8: Validation Fonksiyonunu Analiz Etme**

**fcn.00001570 fonksiyonunu inceliyoruz:**

```
[0x000012a0]> s fcn.00001570
[0x00001570]> pdf
```

**Kritik assembly kodu:**

```assembly
│           0x00001597      e894fbffff     call sym.imp.ptrace
│           0x0000159c      4885c0         test rax, rax
│       ┌─< 0x0000159f      0f88db000000   js 0x1680               ; Jump to "Debugger detected!"
│       │   0x000015a5      bfa0860100     mov edi, 0x186a0
│       │   0x000015aa      e8c1fbffff     call sym.imp.usleep
│       │   0x000015af      ba64000000     mov edx, 0x64
│       │   0x000015b4      4889e7         mov rdi, rsp
│       │   0x000015b7      4889de         mov rsi, rbx
│       │   0x000015ba      e881fbffff     call sym.imp.__strcpy_chk
│       │   0x000015bf      4889df         mov rdi, rbx
│       │   0x000015c2      e829fbffff     call sym.imp.strlen
│       │   0x000015c7      4889c2         mov rdx, rax
│       │   0x000015ca      31c0           xor eax, eax
│       │   0x000015cc      4883fa0f       cmp rdx, 0xf            ; ← LENGTH CHECK: 15!
│      ┌──< 0x000015d0      741e           je 0x15f0
```

**✅ İLK BULGU: Key uzunluğu 15 karakter olmalı!**

**Karakter kontrolü kısmı:**

```assembly
│ ╎╎╎╎╎└──> 0x000015f0      803b6e         cmp byte [rbx], 0x6e    ; 'n'
│ ────────< 0x000015f3      75dd           jne 0x15d2
│ ╎╎╎╎╎ │   0x000015f5      807b016f       cmp byte [rbx + 1], 0x6f    ; 'o'
│ ────────< 0x000015f9      75d7           jne 0x15d2
│ ╎╎╎╎╎ │   0x000015fb      807b0263       cmp byte [rbx + 2], 0x63    ; 'c'
│ ────────< 0x000015ff      75d1           jne 0x15d2
│ ╎╎╎╎╎ │   0x00001601      807b0372       cmp byte [rbx + 3], 0x72    ; 'r'
│ ────────< 0x00001605      75cb           jne 0x15d2
│ ╎╎╎╎╎ │   0x00001607      807b0461       cmp byte [rbx + 4], 0x61    ; 'a'
│ ────────< 0x0000160b      75c5           jne 0x15d2
│ ╎╎╎╎╎ │   0x0000160d      807b0563       cmp byte [rbx + 5], 0x63    ; 'c'
│ ────────< 0x00001611      75bf           jne 0x15d2
│ ╎╎╎╎╎ │   0x00001613      807b066b       cmp byte [rbx + 6], 0x6b    ; 'k'
│ ────────< 0x00001617      75b9           jne 0x15d2
│ ╎╎╎╎╎ │   0x00001619      807b0779       cmp byte [rbx + 7], 0x79    ; 'y'
│ ────────< 0x0000161d      75b3           jne 0x15d2
│ ╎╎╎╎╎ │   0x0000161f      31c0           xor eax, eax
│ ╎╎╎╎╎ │   0x00001621      807b0865       cmp byte [rbx + 8], 0x65    ; 'e'
│ ────────< 0x00001625      75ab           jne 0x15d2
│ ╎╎╎╎╎ │   0x00001627      807b0973       cmp byte [rbx + 9], 0x73    ; 's'
│ ────────< 0x0000162b      75a5           jne 0x15d2
│ ╎╎╎╎╎ │   0x0000162d      807b0a6d       cmp byte [rbx + 0xa], 0x6d  ; 'm'
│ ────────< 0x00001631      759f           jne 0x15d2
│ ╎╎╎╎╎ │   0x00001633      807b0b6f       cmp byte [rbx + 0xb], 0x6f  ; 'o'
│ └───────< 0x00001637      7599           jne 0x15d2
│  ╎╎╎╎ │   0x00001639      807b0c6e       cmp byte [rbx + 0xc], 0x6e  ; 'n'
│  └──────< 0x0000163d      7593           jne 0x15d2
│   ╎╎╎ │   0x0000163f      807b0d65       cmp byte [rbx + 0xd], 0x65  ; 'e'
│   └─────< 0x00001643      758d           jne 0x15d2
│    ╎╎ │   0x00001645      807b0e79       cmp byte [rbx + 0xe], 0x79  ; 'y'
│    └────< 0x00001649      7587           jne 0x15d2
```

**✅ KEY BULUNDU!**

**Analiz:**
- ✅ Her byte tek tek kontrol ediliyor
- ✅ Hex değerleri: `6e 6f 63 72 61 63 6b 79 65 73 6d 6f 6e 65 79`
- 💡 ASCII'ye çevirelim!

---

### 🔤 **Adım 9: Hex'ten ASCII'ye Çevirme**

**Hex değerleri ASCII'ye çeviriyoruz:**

| Hex | ASCII |
|-----|-------|
| 0x6e | n |
| 0x6f | o |
| 0x63 | c |
| 0x72 | r |
| 0x61 | a |
| 0x63 | c |
| 0x6b | k |
| 0x79 | y |
| 0x65 | e |
| 0x73 | s |
| 0x6d | m |
| 0x6f | o |
| 0x6e | n |
| 0x65 | e |
| 0x79 | y |

**✅ KEY: `nocrackyesmoney`**

**Ek validation (checksum kontrolü):**

```assembly
│     ╎ │   ; CODE XREF from fcn.00001570 @ 0x166d(x)
│     ╎┌──> 0x00001658      0fbe4413ff     movsx eax, byte [rbx + rdx - 1]
│     ╎╎│   0x0000165d      0fafc2         imul eax, edx
│     ╎╎│   0x00001660      4883c201       add rdx, 1
│     ╎╎│   0x00001664      83f037         xor eax, 0x37
│     ╎╎│   0x00001667      01c1           add ecx, eax
│     ╎╎│   0x00001669      4883fa10       cmp rdx, 0x10
│     ╎└──< 0x0000166d      75e9           jne 0x1658
│     ╎ │   0x0000166f      31c0           xor eax, eax
│     ╎ │   0x00001671      81f97f330000   cmp ecx, 0x337f         ; Checksum = 0x337f
│     ╎ │   0x00001677      0f94c0         sete al
```

**Analiz:**
- ✅ Her karakter pozisyonuyla çarpılıyor
- ✅ XOR 0x37 yapılıyor
- ✅ Toplam 0x337f olmalı
- 💡 "nocrackyesmoney" bu checksum'ı sağlıyor!

---

### 🏆 **Adım 10: Final Test - Flag Elde Etme**

**Bypass kütüphanesi ile binary'yi çalıştırıyoruz:**

```bash
LD_PRELOAD=./bypass_ptrace.so ./keygenvarmi
```

**Input:**
```
License key: nocrackyesmoney
```

**Output:**
```
Success! Here is the flag:
Flag{CrackAramayaGerekYokKendimCracklerim}
```

**✅ FLAG BULUNDU!**

---

## 🚩 **FLAG**

```
Flag{CrackAramayaGerekYokKendimCracklerim}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">📁<br><b>file</b><br><sub>Binary analizi</sub></td>
<td align="center">🔍<br><b>strings</b><br><sub>String extraction</sub></td>
<td align="center">🐛<br><b>ltrace</b><br><sub>Library tracing</sub></td>
<td align="center">⚙️<br><b>gcc</b><br><sub>Bypass compile</sub></td>
</tr>
<tr>
<td align="center">🔬<br><b>radare2</b><br><sub>Disassembly</sub></td>
<td align="center">🔓<br><b>LD_PRELOAD</b><br><sub>Anti-debug bypass</sub></td>
<td align="center">💻<br><b>Assembly</b><br><sub>x86-64 reading</sub></td>
<td align="center">🎯<br><b>Reverse Eng.</b><br><sub>Logic analysis</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**

```bash
# Binary analizi
file keygenvarmi
chmod +x keygenvarmi
./keygenvarmi

# String analizi
strings keygenvarmi

# Dinamik analiz
ltrace ./keygenvarmi

# ptrace bypass kütüphanesi
cat > bypass_ptrace.c << 'EOF'
long ptrace(int request, int pid, void *addr, void *data) {
    return 0;
}
EOF

gcc -shared -fPIC bypass_ptrace.c -o bypass_ptrace.so

# Bypass ile çalıştırma
LD_PRELOAD=./bypass_ptrace.so ./keygenvarmi

# Radare2 analizi
r2 -A ./keygenvarmi
afl                    # Fonksiyonları listele
pdf @main              # main'i disassemble et
s fcn.00001570         # Validation fonksiyonuna git
pdf                    # Disassemble et

# Final test
echo "nocrackyesmoney" | LD_PRELOAD=./bypass_ptrace.so ./keygenvarmi
```

---

## 💭 **Çözüm Akış Şeması**

```
🎯 Keygenme Challenge
            ↓
📁 Binary'yi indir ve analiz et
   → file komutu
   → ELF 64-bit, stripped
            ↓
🎮 İlk çalıştırma denemeleri
   → ./keygenvarmi
   → "Invalid key!" mesajı
            ↓
🔍 String analizi
   → strings keygenvarmi
   → "Debugger detected!" bulundu!
   → ptrace fonksiyonu tespit edildi
            ↓
🐛 ltrace ile dinamik analiz
   → ltrace ./keygenvarmi
   → ptrace(0,0,1,0) = -1
   → exit(1) - Program sonlanıyor
            ↓
🔓 Anti-debug bypass
   → bypass_ptrace.c oluştur
   → gcc ile compile et
   → LD_PRELOAD ile override et
   → ✅ Debugger kontrolü bypass edildi!
            ↓
🔬 Radare2 ile disassembly
   → r2 -A ./keygenvarmi
   → afl - Fonksiyonları listele
   → main: 0x00001180
   → fcn.00001570: 294 byte (şüpheli!)
            ↓
📊 Main fonksiyonu analizi
   → pdf @main
   → fgets ile input alınıyor
   → fcn.00001570 çağrılıyor
   → ✅ Validation fonksiyonu bulundu!
            ↓
🎯 Validation fonksiyonu inceleme
   → s fcn.00001570
   → pdf
   → cmp rdx, 0xf
   → ✅ Key uzunluğu: 15 karakter!
            ↓
🔤 Byte-by-byte karşılaştırma
   → cmp byte [rbx], 0x6e
   → cmp byte [rbx+1], 0x6f
   → ... (15 karakterlik kontrol)
   → ✅ Hex değerleri bulundu!
            ↓
🔡 Hex'ten ASCII'ye çevirme
   → 6e 6f 63 72 61 63 6b 79 65 73 6d 6f 6e 65 79
   → n  o  c  r  a  c  k  y  e  s  m  o  n  e  y
   → ✅ KEY: nocrackyesmoney
            ↓
🏆 Final test
   → LD_PRELOAD=./bypass_ptrace.so ./keygenvarmi
   → Input: nocrackyesmoney
   → ✅ Success!
            ↓
🚩 Flag{CrackAramayaGerekYokKendimCracklerim}
```
