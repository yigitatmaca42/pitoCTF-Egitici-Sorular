# 🎯 Keygen - Reverse Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Reverse-Challenge-darkblue?style=for-the-badge" alt="Reverse">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Serial Key-purple?style=for-the-badge" alt="Serial Key">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Reverse  
**Seviye:** Orta  
**Puan:** 300

**Challenge Dosyası:** [📥 GitHub - keygenyusuf.exe](https://github.com/cihangungor/pitoctf/blob/main/keygenyusuf.exe)

---

## 🔍 Analiz

**Hedef:** `yusuf` kullanıcısına özel serial key'i bulmak için Windows çalıştırılabilir dosyasını tersine mühendislik ile analiz etmek

**Strateji:** Dosya tipi tespiti → String analizi → Assembly analizi → Algoritma tespiti → Serial hesaplama

**İpuçları:**
- 🎯 PE32 Windows çalıştırılabilir → Tersine mühendislik gerekli
- 🔍 Name + Serial input → Kullanıcıya özel hesaplama
- 💡 Assembly'de matematiksel işlem → Tersine çevrilebilir
- 🔑 300 puan = Orta seviye reverse challenge

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Dosya Tipini Tespit Et**

```bash
file keygenyusuf.exe
```

**Çıktı:**
```
keygenyusuf.exe: PE32 executable for MS Windows 4.00 (console),
Intel i386 (stripped to external PDB), 5 sections
```

**Analiz:**
- ✅ PE32 formatı → 32-bit Windows çalıştırılabilir
- 💡 Stripped → Debug sembolleri yok, daha zor analiz
- 🎯 Intel i386 → x86 assembly

---

### 📊 **Adım 2: String Analizi**

```bash
strings keygenyusuf.exe | grep -i "yusuf\|serial\|key\|flag\|valid\|correct\|wrong\|{"
```

**Çıktı:**
```
Serial:
False Serial.
Right Serial.
Now make a KeyGen
CUNG
CUNG5
```

**Analiz:**
- ✅ "Right Serial." → Doğru serial mesajı
- 💡 "CUNG", "CUNG5" → Serial prefix olabilir
- 🎯 Name + Serial girişi var

---

### 🔍 **Adım 3: Wine ile Çalıştır**

```bash
printf "yusuf\nCUNG5\n" | wine keygenyusuf.exe
```

**Çıktı:**
```
Name:
Serial:
False Serial.
```

**Analiz:**
- ❌ CUNG5 yanlış
- 💡 Serial sayısal bir değer olabilir
- 🎯 Assembly analizi gerekiyor

---

### 🧠 **Adım 4: Main Fonksiyonunu Analiz Et**

```bash
objdump -d keygenyusuf.exe | grep -A 200 "^00401390 <_main>" | head -100
```

**Kritik Assembly Satırları:**
```asm
401442:  call  length()          ; name uzunluğunu al
401447:  add   $0xca,%eax        ; uzunluk + 202 (0xca)
40144c:  xor   $0x3d8d40f,%eax   ; sonucu XOR 0x3d8d40f ile
401451:  mov   %eax,-0x30(%ebp)  ; beklenen serial değeri
...
401486:  read serial (integer)   ; kullanıcıdan serial al
40148e:  cmp   serial == beklenen ; karşılaştır
401491:  je    correct            ; eşitse doğru
```

**Analiz:**
- ✅ Algoritma tespit edildi!
- 💡 `serial = (len(name) + 0xca) XOR 0x3d8d40f`
- 🎯 "yusuf" → uzunluk = 5

---

### 💥 **Adım 5: Serial Key Hesapla**

```bash
python3 -c "
name = 'yusuf'
length = len(name)      # 5
result = length + 0xca  # 5 + 202 = 207
result = result ^ 0x3d8d40f  # XOR
print(f'Serial: {result}')
"
```

**Çıktı:**
```
Serial: 64541888
```

---

### 🚀 **Adım 6: Serial Key Doğrula**

```bash
printf "yusuf\n64541888\n" | wine keygenyusuf.exe
```

**Çıktı:**
```
Name:
Serial:
Right Serial.
Now make a KeyGen
```

**KRİTİK BULGULAR:**
- ✅ **SERIAL KEY DOĞRULANDI!** 🎉
- 💡 `serial = (len("yusuf") + 0xca) XOR 0x3d8d40f`
- 🎯 `= (5 + 202) XOR 64541711 = 64541888`

---

## 🚩 **FLAG**
```
FLAG{64541888}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>strings</b><br><sub>String extraction</sub></td>
<td align="center">🔧<br><b>objdump</b><br><sub>Assembly analizi</sub></td>
<td align="center">🐍<br><b>Python</b><br><sub>Serial hesaplama</sub></td>
<td align="center">🍷<br><b>Wine</b><br><sub>Windows exe çalıştırma</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 Keygen Reverse Challenge
       ↓
📂 Dosyayı incele
   → PE32 Windows çalıştırılabilir
   → Stripped, 5 section
       ↓
🔍 String analizi
   → "Serial:", "Right Serial.", "False Serial."
   → CUNG, CUNG5 → yanlış tahmin
       ↓
🧠 Assembly analizi
   → objdump ile _main fonksiyonu
   → Algoritma: (len(name) + 0xca) XOR 0x3d8d40f
       ↓
🐍 Serial hesapla
   → name = "yusuf", len = 5
   → (5 + 202) XOR 64541711 = 64541888
       ↓
✅ Doğrula
   → printf "yusuf\n64541888\n" | wine keygenyusuf.exe
   → "Right Serial." ✅
       ↓
🚩 FLAG: FLAG{64541888}
```
