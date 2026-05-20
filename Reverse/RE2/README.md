# 🔓 RE2 - Reverse Challenge Write-up
<p align="center">
  <img src="https://img.shields.io/badge/Reverse-Challenge-darkblue?style=for-the-badge" alt="Reverse">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Static Analysis-purple?style=for-the-badge" alt="Static">
</p>

---

## 📋 Soru Bilgileri
  
**Kategori:** Reverse  
**Seviye:** Kolay  
**Puan:** 200

**Açıklama:** Doğru Serial Key'i bulabilir misin?  

**Challenge Dosyası:** [📥 Github - RE2.exe](https://github.com/cihangungor/pitoctf/blob/main/RE2.exe)  

**FLAG Formatı:** `FLAG{dogruSerialKey}`

---

## 🔍 Analiz

**Hedef:** Windows PE executable'ı analiz ederek hardcoded serial key'i bulmak

**Strateji:** File analizi → Strings extraction → Serial key context search → Flag

---

## ✅ Çözüm Adımları

### 🔧 **Adım 1: Dosya Tipi Kontrolü**

```bash
file RE2.exe
```

**Response:**
```
RE2.exe: PE32 executable for MS Windows 5.00 (GUI), Intel i386 (stripped to external PDB), 9 sections
```

**Analiz:**
- ✅ Windows PE32 executable (32-bit)
- ✅ **GUI application** (grafik arayüzlü)
- ✅ Intel i386 architecture
- ✅ Stripped to external PDB
- ✅ 9 sections

---

### 🔍 **Adım 2: İlk String Taraması**

```bash
strings RE2.exe | head -50
```

**Response:**
```
This program must be run under Win32
.text
`.data
.tls
.rdata
P.idata
@.didata
.edata
@.rsrc
@.reloc
 `Kp
 |<q
 DSq
...
```

**Analiz:**
- ✅ PE executable header'ları
- ✅ Section isimleri
- 💡 Anlamlı stringler için daha fazla tarama gerekli

---

### 💡 **Adım 3: Serial Key Araması**

```bash
strings RE2.exe | grep -i "serial\|key\|correct\|wrong\|valid"
```

**Response (önemli kısımlar):**
```
Enter Serial Key:
editSerialKey
1. Find the correct serial key.
2. Change it to a different key of your choice.
```

**Analiz:**
- ✅ `Enter Serial Key:` → Prompt mesajı bulundu
- ✅ `editSerialKey` → Input alanının adı
- ✅ Challenge açıklaması bulundu
- ❌ Henüz serial key'in kendisi yok
- 💡 Context araması gerekli

---

### 💎 **Adım 4: Alfanumerik String Araması**

```bash
strings RE2.exe | sort | uniq | grep -E "^[A-Z0-9]{5,}$"
```

**Response (çok fazla sonuç):**
```
010V0
0123456789ABCDEF
030V0
...
HBITMAP
HBRUSH
HFONT
...
```

**Analiz:**
- ✅ Çok fazla sonuç var
- ❌ Assembly code ve Windows API çağrıları
- 💡 Daha hedefli arama gerekli

---

### 🎯 **Adım 5: "Enter Serial Key" Context'i**

```bash
strings RE2.exe | grep -A5 -B5 "Enter Serial Key"
```

**Response:**
```
        TGroupBox       GroupBox1
Left
Width
Height
Caption
Enter Serial Key:
Color
        clBtnFace
ParentBackground
ParentColor
TabOrder
```

**Analiz:**
- ✅ GUI elemanları bulundu
- ❌ Serial key burada yok
- 💡 Input alanı context'ine bakmak gerekli

---

### 🔬 **Adım 6: "editSerialKey" Context'i - BINGO!**

```bash
strings RE2.exe | grep -A5 -B5 "editSerialKey"
```

**Response:**
```
ABC-123456
        GroupBox1
        GroupBox2
Label2
btnAbout
editSerialKey
Button1
Label1
btnAboutClick
Button1Click
TForm1
```

**✅ SERIAL KEY BULUNDU!**

**Analiz:**
- ✅ **Serial Key:** `ABC-123456`
- ✅ Input alanının (`editSerialKey`) hemen üstünde
- ✅ Hardcoded serial key kesin olarak doğrulandı!

---

## 🏁 **FLAG**

```
FLAG{ABC-123456}
```

---

## 🛠️ **Kullanılan Komutlar**

```bash
# Dosya tipi kontrolü
file RE2.exe

# İlk string taraması
strings RE2.exe | head -50

# Serial key araması
strings RE2.exe | grep -i "serial\|key\|correct\|wrong\|valid"

# Alfanumerik string araması
strings RE2.exe | sort | uniq | grep -E "^[A-Z0-9]{5,}$"

# Enter Serial Key context'i
strings RE2.exe | grep -A5 -B5 "Enter Serial Key"

# editSerialKey context'i (BINGO!)
strings RE2.exe | grep -A5 -B5 "editSerialKey"
```

---

## 💭 **Çözüm Akışı**

```
🔓 RE2 Reverse Challenge
            ↓
🔧 File type check
   → PE32 executable (GUI application)
            ↓
🔍 Initial strings extraction
   → PE headers ve assembly code
            ↓
💡 Serial key search
   → Enter Serial Key:, editSerialKey bulundu
            ↓
💎 Alfanumerik string search
   → Çok fazla sonuç (assembly/API calls)
            ↓
🎯 Enter Serial Key context
   → Sadece GUI elemanları
            ↓
🔬 editSerialKey context
   → ABC-123456 bulundu! ✅
            ↓
🏁 FLAG{ABC-123456}
```
