# 🎯 ElfGozler - Reverse Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Reverse Engineering-Challenge-darkblue??style=for-the-badge" alt="Reverse Engineering">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-XOR-purple?style=for-the-badge" alt="XOR">
</p>

---

## 📋 Soru Bilgileri
  
**Kategori:** Reverse   
**Seviye:** Orta  
**Puan:** 500

**Challenge Dosyası:** [📥 Google Drive - elfgoz](https://drive.google.com/file/d/1j0dnWlGTp_dIQnvPYyhPrkxMRjX2uOob/view?usp=drive_link)

---

## 🔍 Analiz

**Hedef:** ELF binary'yi reverse engineering yaparak şifreli flag'i çözmek

**Strateji:** Binary'yi disassemble edip XOR şifreleme mantığını bulmak

**İpuçları:**
- 🎯 Stripped ELF binary → Semboller yok
- 🔍 strings komutu ipucu verebilir
- 💡 XOR encryption kullanılmış
- 🔑 Flag format: `Flag{...}`

---

## ✅ Çözüm Adımları

### 📦 **Adım 1: Binary'yi İnceleme**
```bash
file elfgoz
```

**Çıktı:**
```
elfgoz: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), 
dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, 
BuildID[sha1]=d22df9f3c5535a7b6a6b7c13e97342102c92fc2c, 
for GNU/Linux 3.2.0, stripped
```

**Analiz:**
- ✅ 64-bit ELF executable
- ⚠️ Stripped (semboller kaldırılmış)
- 💡 Dinamik olarak linklenmiş

---

### 🔎 **Adım 2: Strings Analizi**
```bash
strings elfgoz
```

**Önemli Çıktılar:**
```
SZ{jk|
puts
__stack_chk_fail
unlink
mkstemp
/tmp/.s_H
_XXXXXX
BBBBf
BBBBBBBBBBBBBBBB
```

**Analiz:**
- 🎯 `SZ{jk|` → Şifreli veri gibi görünüyor
- 🔑 `BBBBBBBBBBBBBBBB` → 16 byte 'B' karakteri (0x42)
- 💡 XOR key olabilir!

---

### 🔨 **Adım 3: Objdump ile Disassemble**
```bash
objdump -d elfgoz | grep -B 5 -A 20 "pxor"
```

**Assembly Code:**
```asm
1244:  66 0f 6f 05 c4 0d 00 00    movdqa 0xdc4(%rip),%xmm0
124c:  66 0f ef 05 bc 2d 00 00    pxor   0x2dbc(%rip),%xmm0
1254:  81 35 c2 2d 00 00 42 42    xorl   $0x42424242,0x2dc2(%rip)
       42 42
125e:  66 81 35 bd 2d 00 00 42    xorw   $0x4242,0x2dbd(%rip)
       42
1267:  80 35 b8 2d 00 00 42       xorb   $0x42,0x2db8(%rip)
```

**Analiz:**
- ✅ `pxor` instruction → XOR işlemi yapıyor
- 🔑 0x42424242 (`BBBB`) ile XOR yapılıyor
- 💡 .rodata section'dan veri alıyor

---

### 📊 **Adım 4: Data Section Analizi**
```bash
objdump -s -j .data elfgoz
```

**Çıktı:**
```
Contents of section .data:
 4010 042e2325 39313630 20272331 2b2e2320  ..#%9160 '#1+.# 
 4020 372e232c 26373f                      7.#,&7?
```

**Hex Data:**
```
04 2e 23 25 39 31 36 30 20 27 23 31 2b 2e 23 20 37 2e 23 2c 26 37 3f
```

---

### 🧮 **Adım 5: XOR Decryption**

**Python Script:**
```python
# .data section'daki şifreli veri
data_hex = "042e232539313630202723312b2e2320372e232c26373f"
data_bytes = bytes.fromhex(data_hex)

# XOR key: 0x42 ('B' karakteri)
key = 0x42

# Decrypt
decoded = ""
for byte in data_bytes:
    decoded += chr(byte ^ key)

print(decoded)
```

**Detaylı Decode Tablosu:**
```
[0]  0x04 XOR 0x42 = 0x46 ('F')
[1]  0x2e XOR 0x42 = 0x6c ('l')
[2]  0x23 XOR 0x42 = 0x61 ('a')
[3]  0x25 XOR 0x42 = 0x67 ('g')
[4]  0x39 XOR 0x42 = 0x7b ('{')
[5]  0x31 XOR 0x42 = 0x73 ('s')
[6]  0x36 XOR 0x42 = 0x74 ('t')
[7]  0x30 XOR 0x42 = 0x72 ('r')
[8]  0x20 XOR 0x42 = 0x62 ('b')
[9]  0x27 XOR 0x42 = 0x65 ('e')
[10] 0x23 XOR 0x42 = 0x61 ('a')
[11] 0x31 XOR 0x42 = 0x73 ('s')
[12] 0x2b XOR 0x42 = 0x69 ('i')
[13] 0x2e XOR 0x42 = 0x6c ('l')
[14] 0x23 XOR 0x42 = 0x61 ('a')
[15] 0x20 XOR 0x42 = 0x62 ('b')
[16] 0x37 XOR 0x42 = 0x75 ('u')
[17] 0x2e XOR 0x42 = 0x6c ('l')
[18] 0x23 XOR 0x42 = 0x61 ('a')
[19] 0x2c XOR 0x42 = 0x6e ('n')
[20] 0x26 XOR 0x42 = 0x64 ('d')
[21] 0x37 XOR 0x42 = 0x75 ('u')
[22] 0x3f XOR 0x42 = 0x7d ('}')
```

---

### 🚀 **Adım 6: Script Çalıştırma**
```bash
python3 << 'EOF'
# Şifreli veri (.data section)
data_hex = "042e232539313630202723312b2e2320372e232c26373f"
data_bytes = bytes.fromhex(data_hex)

# XOR key: 0x42
key = 0x42

# Decrypt
decoded = ""
for byte in data_bytes:
    decoded += chr(byte ^ key)

print(f"🎯 FLAG: {decoded}")
EOF
```

**Çıktı:**
```
🎯 FLAG: Flag{strbeasilabulandu}
```

---

## 🚩 **FLAG**
```
Flag{strbeasilabulandu}
```

**Anlamı:** "string başarısıyla bulundu" (Türkçe wordplay)

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔧<br><b>file</b><br><sub>Binary type detection</sub></td>
<td align="center">📝<br><b>strings</b><br><sub>String extraction</sub></td>
<td align="center">⚙️<br><b>objdump</b><br><sub>Disassembly</sub></td>
<td align="center">🐍<br><b>Python3</b><br><sub>Decryption script</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**
```
🎯 ElfGozler Challenge
       ↓
📦 elfgoz binary indir
   → 64-bit ELF
   → stripped (sembol yok)
       ↓
📝 strings komutu çalıştır
   → "SZ{jk|" şifreli veri
   → "BBBBBBBBBBBBBBBB" XOR key?
       ↓
🔨 objdump ile disassemble et
   → pxor instruction bulundu
   → 0x42 ile XOR yapılıyor
       ↓
📊 .data section'ı incele
   → Şifreli flag verisi bulundu
   → Hex: 042e232539313630...
       ↓
🧮 XOR decrypt uygula
   → Her byte ^ 0x42
   → Flag decode edildi
       ↓
🚩 Flag{strbeasilabulandu}
```
