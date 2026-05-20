# 🎯 Threads - Reverse Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Reverse-Challenge-darkblue?style=for-the-badge" alt="Reverse">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Static Analysis-purple?style=for-the-badge" alt="Static Analysis">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Reverse  
**Seviye:** Kolay  
**Puan:** 300

**Challenge Dosyası:** [📥 GitHub - threads.exe](https://github.com/cihangungor/pitoctf/blob/main/threads.exe)

---

## 🔍 Analiz

**Hedef:** Windows çalıştırılabilir dosyasını analiz ederek gizlenmiş flag'i bulmak

**Strateji:** Dosya tipi tespiti → String analizi → Flag extraction

**İpuçları:**
- 🎯 `.exe` dosyası → Windows PE32 çalıştırılabilir
- 🔍 Statik analiz → Çalıştırmaya gerek yok
- 💡 `strings` komutu → Gömülü metinler okunabilir
- 🔑 300 puan = Temel statik analiz yeterli

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Dosya Tipini Tespit Et**

```bash
file threads.exe
```

**Çıktı:**
```
threads.exe: PE32 executable for MS Windows 4.00 (console), Intel i386, 13 sections
```

**Analiz:**
- ✅ PE32 formatı → 32-bit Windows çalıştırılabilir dosyası
- 💡 Intel i386 mimarisi → x86 assembly
- 🎯 13 section → Birden fazla bölüm içeriyor
- 🔑 Console uygulaması → Terminal çıktısı veriyor

---

### 📊 **Adım 2: String Analizi ile Flag'i Bul**

```bash
strings threads.exe | grep -i "PITO\|flag\|pito\|CTF\|{"
```

**Çıktı:**
```
flag.txt
BayrakBende{b3n_h1z1m!}
Flag: 
_flag
X86_TUNE_PARTIAL_FLAG_REG_STALL
X86_TUNE_FUSE_CMP_AND_BRANCH_SOFLAGS
__ZL12encoded_flag
__Z11decode_flagB5cxx11v
__loader_flags__
```

**KRİTİK BULGULAR:**
- ✅ **FLAG BULUNDU!** 🎉
- 💡 `BayrakBende{b3n_h1z1m!}` → Flag doğrudan string olarak gömülmüş
- 🎯 `__ZL12encoded_flag` → C++ mangled isim, encoded flag değişkeni
- 🔑 `__Z11decode_flagB5cxx11v` → Decode fonksiyonu mevcut ama gerekli olmadı

---

## 🚩 **FLAG**
```
BayrakBende{b3n_h1z1m!}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>strings</b><br><sub>String extraction</sub></td>
<td align="center">🐚<br><b>bash</b><br><sub>Command line</sub></td>
<td align="center">🔧<br><b>grep</b><br><sub>Pattern matching</sub></td>
<td align="center">📁<br><b>file</b><br><sub>Dosya tipi tespiti</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 Threads Reverse Challenge
       ↓
📂 Dosyayı incele
   → threads.exe
   → PE32 Windows çalıştırılabilir
   → Intel i386, 13 section
       ↓
🔍 Statik analiz
   → strings komutu ile gömülü metinler
   → grep ile flag pattern arama
   → PITO | flag | CTF | { filtreleri
       ↓
🚩 Flag bulundu
   → BayrakBende{b3n_h1z1m!}
   → Direkt string olarak gömülmüş
       ↓
✅ FLAG: BayrakBende{b3n_h1z1m!}
```
