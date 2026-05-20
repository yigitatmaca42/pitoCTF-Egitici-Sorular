# 🎯 Saatin var mı? - Reverse Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Reverse Engineering-Challenge-darkblue?style=for-the-badge" alt="Reverse Engineering">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Time Based-purple?style=for-the-badge" alt="Time Based">
</p>

---

## 📋 Soru Bilgileri
  
**Kategori:** Reverse   
**Seviye:** Orta  
**Puan:** 500

**Challenge Dosyası:** [📥 Google Drive - time](https://drive.google.com/file/d/1rwA80PFoukf21h1ESM90yky2ZPo8Fi6C/view?usp=sharing)

---

## 🔍 Analiz

**Hedef:** Zaman tabanlı kontrol yapan binary'yi reverse engineering yaparak flag'i çıkarmak

**Strateji:** Binary'yi disassemble edip zaman kontrolünü bypass etmek

**İpuçları:**
- 🎯 Program `localtime` fonksiyonu kullanıyor
- 🔍 "Wrong time." mesajı → Zaman kontrolü var
- 💡 Belirli dakikalarda flag görünüyor
- 🕐 Binary patch ile zaman kontrolünü bypass edebiliriz

---

## ✅ Çözüm Adımları

### 📦 **Adım 1: Binary'yi İnceleme**
```bash
file time
chmod +x time
./time
```

**Çıktı:**
```
time: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), 
dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, 
for GNU/Linux 3.2.0, stripped

Wrong time.
```

**Analiz:**
- ✅ 64-bit ELF executable
- ⚠️ Stripped (semboller yok)
- 💡 "Wrong time" → Zaman kontrolü yapılıyor

---

### 🔎 **Adım 2: Strings Analizi**
```bash
strings time
```

**Önemli Çıktılar:**
```
localtime
Wrong time.
1##6)#!}
4#)+f
#/#//+}?
```

**Analiz:**
- 🎯 `localtime` → Sistem saatini kontrol ediyor
- 🔑 Şifreli veri parçaları görünüyor

---

### 🔨 **Adım 3: Objdump ile Zaman Kontrolünü Bulma**
```bash
objdump -d time | grep -B 5 -A 15 "localtime"
```

**Assembly Code:**
```asm
1356: call   1120 <localtime@plt>
135b: mov    0x4(%rax),%edx       ; Dakika değerini al
135e: xor    %eax,%eax
1360: cmp    $0x3a,%edx           ; Dakika kontrolü
1363: ja     1379
1365: movabs $0x401004010040100,%rax  ; Bitmask
136f: bt     %rdx,%rax            ; Bit test
1373: setb   %al
```

**Analiz:**
- ✅ 0x135b adresinde dakika değeri alınıyor
- 🔑 Bitmask ile belirli dakikalar kontrol ediliyor

---

### 📊 **Adım 4: Geçerli Dakikaları Bulma**
```bash
python3 << 'EOF'
bitmask = 0x401004010040100

print("Geçerli dakikalar:")
for minute in range(59):
    if (bitmask >> minute) & 1:
        print(f"  Dakika {minute}: {minute:02d}")
EOF
```

**Çıktı:**
```
Geçerli dakikalar:
  Dakika 8: 08
  Dakika 18: 18
  Dakika 28: 28
  Dakika 38: 38
  Dakika 48: 48
  Dakika 58: 58
```

**Analiz:**
- ✅ Sadece bu dakikalarda flag görünüyor
- 💡 Binary'yi patch ederek bypass edebiliriz

---

### 🔧 **Adım 5: Binary Patch (Dakikayı Zorla Ayarla)**
```bash
python3 << 'EOF'
with open('time', 'rb') as f:
    data = bytearray(f.read())

# 135b: 8b 50 04 -> mov 0x4(%rax),%edx
# Patch: ba 08 00 00 00 -> mov $0x8,%edx

offset = 0x135b
data[offset] = 0xba    # mov $imm,%edx
data[offset+1] = 0x08  # dakika = 8
data[offset+2] = 0x00
data[offset+3] = 0x00
data[offset+4] = 0x00

with open('time_patched', 'wb') as f:
    f.write(data)

import os
os.chmod('time_patched', 0o755)
print("✅ time_patched oluşturuldu!")
EOF
```

**Çıktı:**
```
Mevcut bytes: 8b500431c0
Yeni bytes: ba08000000
✅ time_patched oluşturuldu!
```

---

### 🚀 **Adım 6: Patch'li Binary'yi Çalıştırma**
```bash
strace -f -e trace=write ./time_patched 2>&1 | grep "Flag"
```

**Çıktı:**
```
[pid 327267] write(4, "Flag{saatkac?vakittamammi?}", 27) = 27
```

---

## 🚩 **FLAG**
```
Flag{saatkac?vakittamammi?}
```

**Anlamı:** "saat kaç? vakit tam mı?" (Türkçe)

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔧<br><b>file</b><br><sub>Binary type detection</sub></td>
<td align="center">📝<br><b>strings</b><br><sub>String extraction</sub></td>
<td align="center">⚙️<br><b>objdump</b><br><sub>Disassembly</sub></td>
<td align="center">🐍<br><b>Python3</b><br><sub>Binary patching</sub></td>
<td align="center">🔍<br><b>strace</b><br><sub>System call tracing</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**
```
🎯 Saatin var mı? Challenge
       ↓
📦 time binary indir
   → 64-bit ELF
   → stripped
   → "./time" → "Wrong time."
       ↓
📝 strings komutu çalıştır
   → "localtime" fonksiyonu var
   → "Wrong time." mesajı
   → Şifreli veri parçaları
       ↓
🔨 objdump ile disassemble et
   → 0x135b: dakika kontrolü
   → Bitmask: 0x401004010040100
   → Belirli dakikalar: 8,18,28,38,48,58
       ↓
🔧 Binary'yi patch et
   → 0x135b offset'i değiştir
   → Dakikayı 8'e zorla ayarla
   → time_patched oluştur
       ↓
🚀 Patch'li binary çalıştır
   → strace ile flag'i yakala
   → write() system call'unda flag görünür
       ↓
🚩 Flag{saatkac?vakittamammi?}
```
