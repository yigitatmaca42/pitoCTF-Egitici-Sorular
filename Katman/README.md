# 🖼️ Katman - Stego Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Stego-Challenge-darkblue?style=for-the-badge&logo=image" alt="Stego">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-GIMP%20Layers-purple?style=for-the-badge&logo=gimp" alt="GIMP">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Stego  
**Seviye:** Kolay  
**Puan:** 100  

**Challenge Dosyası:** [📥 Google Drive - KatmanKatman](https://drive.google.com/file/d/1jeORbeAljPQ-_I2QcfCoJJ05RdJPlPA4/view?usp=drivesdk)

**Firstblood:** +50 puan

---

## 🔍 Analiz

**Hedef:** GIMP dosyasındaki katmanları inceleyerek flag bulmak

**Strateji:** File type check → GIMP ile aç → Layer'ları kontrol et

---

## ✅ Çözüm Adımları

### 📥 **1. Dosyayı İndirme**

Google Drive'dan `KatmanKatman` dosyasını indiriyoruz.

---

### 🔍 **2. Dosya Tipini Belirleme**

**Komut:**

```bash
file KatmanKatman
```

**Çıktı:**

```
KatmanKatman: GIMP XCF image data, version 011, 1024 x 1024, RGB Color
```

> 🎯 **GIMP XCF dosyası!** GIMP ile açmamız gerekiyor.

---

### 🎨 **3. GIMP Kurulumu**

**Kali Linux:**

```bash
sudo apt update
sudo apt install gimp
```

**Windows/MacOS:**

[Gimp İndirme](https://www.gimp.org/downloads/)

---

### 🖼️ **4. Dosyayı GIMP ile Açma**

**Adımlar:**

1. GIMP'i başlat
2. **File → Open**
3. `KatmanKatman` dosyasını seç
4. Dosya açılıyor

---

### 👁️ **5. Layer'ları (Katmanları) İnceleme**

Sağ tarafta **Layers** panelinde birden fazla katman görünüyor:

```
- Pasted Layer #9
- Pasted Layer #8
- Pasted Layer #7
- Pasted Layer #6
- Pasted Layer #5
- Pasted Layer #4
- Pasted Layer #3
- Pasted Layer #2
- Pasted Layer #1
- Pasted Layer 
```

**Her katmanın yanındaki göz ikonuna** tıklayarak katmanları tek tek kapatıyoruz.

---

### 🎯 **6. Flag'i Bulma**

**3. Layer'da** flag görünüyor:

```
Flag{9a64bc4a390cb0ce31452820ee562c3F}
```

> ✅ Flag, üstteki katmanların altında gizlenmişti!

---

## 🚩 **FLAG**

```
Flag{9a64bc4a390cb0ce31452820ee562c3F}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>file</b><br><sub>Dosya tipi tespiti</sub></td>
<td align="center">🎨<br><b>GIMP</b><br><sub>Image editor</sub></td>
<td align="center">👁️<br><b>Layers Panel</b><br><sub>Katman yönetimi</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**
```bash
# Dosya tipini öğrenme
file KatmanKatman

# GIMP kurulumu
sudo apt install gimp

# GIMP'i başlatma
gimp
```

---

## 💭 **Çözüm Akış Şeması**

```
🖼️ Stego Challenge: "Katman"
                    ↓
        📥 KatmanKatman dosyasını indir
                    ↓
        🔍 file KatmanKatman → GIMP XCF
                    ↓
        🎨 GIMP'i kur ve başlat
                    ↓
        📂 File → Open → KatmanKatman
                    ↓
        👁️ Layers panelini aç
                    ↓
        🔄 Her katmanı tek tek kapat/aç
                    ↓
        🎯 3. katmanda flag göründü!
                    ↓
        🚩 Flag{9a64bc4a390cb0ce31452820ee562c3F}
```
