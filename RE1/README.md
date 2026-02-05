# 🔓 RE1 - Reverse Engineering Challenge Write-up
<p align="center">
  <img src="https://img.shields.io/badge/Reverse-Challenge-darkblue?style=for-the-badge" alt="Reverse">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Static Analysis-purple?style=for-the-badge" alt="Static">
</p>

---

## 📋 Soru Bilgileri
  
**Kategori:** Reverse  
**Seviye:** Kolay  
**Puan:** 300  

**Açıklama:** Kullanıcı adı ve şifreyi bulabilir misin? 

**Challenge Dosyası:** [📥 Github - RE1.exe](https://github.com/cihangungor/pitoctf/blob/main/RE1.exe)  

**FLAG Formatı:** `FLAG{kullaniciadi_şifre}`

---

## 🔍 Analiz

**Hedef:** Windows PE executable'ı analiz ederek hardcoded username ve password'ü bulmak

**Strateji:** File analizi → Strings extraction → Hardcoded credentials → Flag

---

## ✅ Çözüm Adımları

### 🔧 **Adım 1: Dosya Tipi Kontrolü**

```bash
file RE1.exe
```

**Response:**
```
RE1.exe: PE32 executable for MS Windows 6.00 (console), Intel i386, 9 sections
```

**Analiz:**
- ✅ Windows PE32 executable (32-bit)
- ✅ Console application
- ✅ Intel i386 architecture
- ✅ 9 sections var

---

### 🔍 **Adım 2: İlk String Taraması**

```bash
strings RE1.exe | head -50
```

**Response:**
```
!This program cannot be run in DOS mode.
Rich
.textbss
.text
`.rdata
@.data
.idata
@.msvcjmcH
.00cfg
@.rsrc
@.reloc
SVWQ
SVWQ
Y_^[
...
```

**Analiz:**
- ✅ PE executable header'ları
- ✅ Section isimleri ve assembly code
- 💡 Anlamlı stringler için daha fazla tarama gerekli

---

### 💡 **Adım 3: User/Pass Araması**

```bash
strings RE1.exe | grep -i "user\|pass"
```

**Response:**
```
password
username
Username:  
password: 
user not registered! :( 
C:\Users\Can\Desktop\Visual Studio Projects\CrackMe\Debug\CrackMe.pdb
USER32.dll
__setusermatherr
```

**Analiz:**
- ✅ `Username:` ve `password:` prompt mesajları bulundu
- ✅ `user not registered!` → Hatalı giriş mesajı
- ✅ `CrackMe.pdb` → Debug mode, CrackMe challenge'ı
- ❌ Henüz hardcoded credentials yok

---

### 💎 **Adım 4: Alfanumerik String Araması**

```bash
strings RE1.exe | sort | uniq | grep -E "^[A-Za-z0-9]{3,}"
```

**Response (önemli kısımlar):**
```
...
4ACE00F
...
Nick
...
Username:  
password: 
user not registered! :( 
...
```

**Analiz:**
- ✅ `Nick` → İsim formatında
- ✅ `4ACE00F` → Şifre formatında
- 💡 Context kontrolü gerekli

---

### 🔬 **Adım 5: Context Doğrulaması - BINGO!**

```bash
strings RE1.exe | grep -A2 -B2 "Nick"
```

**Response:**
```
string too long
bad cast
Nick
4ACE00F
Username:  
```

**✅ HARDCODED CREDENTIALS BULUNDU!**

**Analiz:**
- ✅ **Username:** `Nick`
- ✅ **Password:** `4ACE00F`
- ✅ Sıralama: `Nick` → `4ACE00F` → `Username:`
- ✅ Hardcoded credentials kesin olarak doğrulandı!

---

## 🏁 **FLAG**

```
FLAG{Nick_4ACE00F}
```

---

## 🛠️ **Kullanılan Komutlar**

```bash
# Dosya tipi kontrolü
file RE1.exe

# İlk string taraması
strings RE1.exe | head -50

# User/pass araması
strings RE1.exe | grep -i "user\|pass"

# Alfanumerik string araması
strings RE1.exe | sort | uniq | grep -E "^[A-Za-z0-9]{3,}"

# Context doğrulaması
strings RE1.exe | grep -A2 -B2 "Nick"
```

---

## 💭 **Çözüm Akışı**

```
🔓 RE1 Reverse Engineering Challenge
            ↓
🔧 File type check
   → PE32 executable (32-bit Windows)
            ↓
🔍 Initial strings extraction
   → PE headers ve assembly code
            ↓
💡 User/pass search
   → Username:, password:, user not registered!
            ↓
💎 Alfanumerik string search
   → Nick, 4ACE00F bulundu
            ↓
🔬 Context verification
   → Nick → 4ACE00F → Username:
   → Hardcoded credentials confirmed! ✅
            ↓
🏁 FLAG{Nick_4ACE00F}
```
