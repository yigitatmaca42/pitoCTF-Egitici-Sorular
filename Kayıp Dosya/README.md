# 📁 Kayıp Dosya - OSINT Challenge

<p align="center">
  <img src="https://img.shields.io/badge/OSINT-Challenge-darkblue?style=for-the-badge&logo=search" alt="OSINT">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-File Search-purple?style=for-the-badge&logo=file" alt="File Recovery">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** OSINT  
**Seviye:** Orta  
**Puan:** 500  

***Açıklama:**  Windows ders için kullandığım bir slayt dosyamı virüs var diye silmiş.
                Dosyamı bulamıyorum. Elimde bir tek bu var.
                Onun da ne işe yarayacağını bilmiyorum:
                5d0eb7ae3407537696bc690fd06cf6e6c185558bc8322dc7cfd04922b0c9d752

**Firstblood:** +50 puan

---

## 🔍 Analiz

**Hedef:** Verilen hash değerini kullanarak silinmiş dosyayı bulmak

**İlk Gözlemler:**
- 🔑 Verilen Hash Değeri → Dosya Kimliği
- 📝 "virüs var diye silmiş" → VirusTotal'de bakılabilir
- 🎯 Dosya tipi: Bilinmiyor, araştırılması gerek

**Strateji:**
- Hash'in ne olduğunu anlamak
- Hash ile dosya aramak
- Dosyayı bulup içeriğini incelemek

---

## ✅ Çözüm Adımları

### 🔎 **1. Hash Analizi**

**Verilen hash:**
```
5d0eb7ae3407537696bc690fd06cf6e6c185558bc8322dc7cfd04922b0c9d752
```

**Hash tipi:**
- 64 karakter → SHA256 hash
- Dosya bütünlüğünü doğrulayan özet değer

> 💡 Bu hash'i nerede arayabiliriz? → **VirusTotal!**

---

### 🦠 **2. VirusTotal Araması**

**VirusTotal'e gidiyoruz:**
```
https://www.virustotal.com
```

**Search kutusuna hash'i yapıştırıyoruz:**
```
5d0eb7ae3407537696bc690fd06cf6e6c185558bc8322dc7cfd04922b0c9d752
```

**Sonuç:**
- ✅ Dosya bulundu!
- 📄 **Dosya adı:** `teknikresim.pptx`
- 📊 **Dosya tipi:** PowerPoint Presentation
- 💬 **Yorumlar bölümünde:**
  - 👤 Kullanıcı: `flagbizdensorulur`
  - 🔗 Google Drive linki paylaşılmış!

---

### 📥 **3. Dosyayı İndirme**

**VirusTotal'den Google Drive linki:**
```
https://docs.google.com/presentation/d/1jHGy8iCkKAoIzpKpHcaRUYhg1rB8q0Oq/edit
```

**Dosyayı indiriyoruz:**
- Google Drive → Download as → Microsoft PowerPoint (.pptx)
- Dosya: `teknikresim.pptx`

---

### 📦 **4. PPTX Extraction**

**PPTX dosyası aslında bir ZIP arşivi!**

```bash
cd Desktop
unzip teknikresim.pptx
```

**Çıktı:**
```
Archive:  teknikresim.pptx
  inflating: [Content_Types].xml     
  inflating: _rels/.rels             
  inflating: docProps/core.xml       
  inflating: docProps/app.xml        
  inflating: ppt/presentation.xml    
  inflating: ppt/_rels/presentation.xml.rels  
  inflating: ppt/theme/theme1.xml    
  inflating: ppt/tableStyles.xml     
  inflating: ppt/presProps.xml       
  inflating: ppt/viewProps.xml       
  inflating: ppt/slideMasters/slideMaster1.xml  
  inflating: ppt/notesMasters/notesMaster1.xml  
  inflating: ppt/slides/slide1.xml   
  inflating: ppt/slides/slide2.xml   
  inflating: ppt/slides/slide3.xml   
  ...
  inflating: ppt/slides/slide48.xml  
  inflating: ppt/slides/_rels/slide1.xml.rels  
  ...
  inflating: ppt/media/image1.png    
  inflating: ppt/media/image2.png    
  inflating: ppt/media/image3.png    
  ...
  inflating: ppt/media/image50.gif   
  inflating: ppt/media/image51.jpeg  ← 🎯 ŞÜPHELİ!
```

**Analiz:**
- 48 slayt var
- Media klasöründe 51 görsel var
- `image51.jpeg` → En son eklenen dosya!

---

### 📂 **5. Klasör Yapısını İnceleme**

**Desktop'ta oluşan dosyalar:**
```
📁 Desktop/
├── 📄 [Content_Types].xml
├── 📁 docProps/
├── 📁 _rels/
└── 📁 ppt/
    ├── 📄 presentation.xml
    ├── 📁 slides/ (48 slayt XML dosyası)
    ├── 📁 slideLayouts/
    ├── 📁 theme/
    └── 📁 media/ ← BURAYA GİRİYORUZ!
    ...
```

**Media klasörüne giriyoruz:**

```bash
cd ppt/media
ls -la
```

---

### 🖼️ **6. Flag Görseli Bulma**

**İçindeki dosyalar:**
```
image1.png
image2.png
image3.png
...
image44.gif
image45.jpeg
image46.png
image47.gif
image48.png
image49.gif
image50.gif
image51.jpeg  ← 🚩 EN SON GÖRSEL!
```

**image51.jpeg'i açıyoruz:**

```bash
xdg-open image51.jpeg
# veya
eog image51.jpeg
```

**Görselde yazıyor:**
```
flag{slaytimiBuldugunicinTesekkurlerVirusFalanYok}
```

> 🎉 **FLAG BULUNDU!**

---

## 🚩 **FLAG**

```
flag{slaytimiBuldugunicinTesekkurlerVirusFalanYok}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🦠<br><b>VirusTotal</b><br><sub>Hash lookup</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**
```bash
# Desktop'a git
cd Desktop

# PPTX'i unzip et
unzip teknikresim.pptx

# Media klasörüne git
cd ppt/media

# Görselleri listele
ls -la

# Flag görselini aç
xdg-open image51.jpeg
# veya
eog image51.jpeg
```

---

## 💭 **Çözüm Akışı**

```
📁 "Kayıp Dosya" Challenge
            ↓
🔑 Hash verildi: 5d0eb7ae...
            ↓
🦠 VirusTotal'de arama
            ↓
📄 teknikresim.pptx bulundu
            ↓
💬 Yorumlarda ipucu: "flagbizdensorulur"
            ↓
📥 Google Drive'dan indirme
            ↓
📦 unzip teknikresim.pptx
            ↓
📂 ppt/media klasörüne girme
            ↓
🖼️ image51.jpeg'i açma
            ↓
👀 Görselde flag görünüyor!
            ↓
🚩 flag{slaytimiBuldugunicinTesekkurlerVirusFalanYok}
```
