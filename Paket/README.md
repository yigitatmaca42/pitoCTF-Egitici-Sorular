# 📦 Paket - Misc Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Misc-Challenge-darkblue?style=for-the-badge&logo=packet" alt="Misc">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-PCAP Analysis-purple?style=for-the-badge&logo=wireshark" alt="PCAP">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Misc  
**Seviye:** Orta  
**Puan:** 300  

**Challenge Dosyası:** [📥 Paket.pcap](https://drive.google.com/file/d/1QVTEVUtYZlXlOB-etJJ9ZQpPc_gqpPMM/view?usp=drivesdk)

**Firstblood:** +50 puan

---

## 🔍 Analiz

**Hedef:** PCAP dosyasını analiz edip içindeki gizlenmiş PNG imajından FLAG'i çıkarmak

**Strateji:** PCAP analizi → Hex değerleri çıkarma → PNG reconstruction → Flag

**İpuçları:**
- 📦 **PCAP:** Network packet capture dosyası
- 🖼️ **PNG Magic Bytes:** `0x89 0x50 0x4e 0x47`
- 🔤 **Hex Encoding:** Data hex formatında saklanmış
- 🎯 **Format:** 0xL4ugh{...} formatında flag

---

## ✅ Çözüm Adımları

### 📥 **1. Dosya Analizi**

**PCAP dosyasını kontrol ediyoruz:**

```bash
file Paket.pcap
```

**Çıktı:**
```
Paket.pcap: pcap capture file, microsecond ts (little-endian) - version 2.4 (Raw IPv4, capture length 65535)
```

**Analiz:**
- ✅ Geçerli PCAP dosyası
- 📊 Microsecond timestamp
- 🌐 Raw IPv4 paketleri
- 📏 Max capture: 65535 bytes

**Dosya boyutu:**
```bash
ls -lh Paket.pcap
```
```
-rw-r--r-- 1 kali kali 578K Jan 12 17:00 Paket.pcap
```

> 📦 578 KB network capture verisi!

---

### 🔍 **2. String Analizi - İlk İpucu**

**PCAP içindeki okunabilir string'leri çıkarıyoruz:**

```bash
strings Paket.pcap | head -50
```

**Kritik Çıktı:**
```
Hello From 0xL4ugh, Hope you enjoy the CTF ..., MMOX
0x89
0x50
0x4e
0x47
0xd
0xa
0x1a
0xa
0x0
0x0
0x0
0xd
0x49
0x48
0x44
0x52
...
```

**Analiz:**
- 💬 **Mesaj:** "Hello From 0xL4ugh, Hope you enjoy the CTF"
- 🖼️ **PNG Magic Bytes tespit edildi:**
  - `0x89` = PNG signature byte 1
  - `0x50` = 'P'
  - `0x4e` = 'N'
  - `0x47` = 'G'

> 🎯 **PNG dosyası hex formatında PCAP içinde gizlenmiş!**

---

### 🔢 **3. Hex Değerleri Sayma**

**Kaç hex değer var kontrol ediyoruz:**

```python
import re

with open('Paket.pcap', 'rb') as f:
    content = f.read()

text_content = content.decode('latin-1')
hex_pattern = r'0x[0-9a-fA-F]{1,2}'
matches = re.findall(hex_pattern, text_content)

print(f"Toplam {len(matches)} hex değer bulundu")
```

**Çıktı:**
```
Toplam 9889 hex değer bulundu
```

> 💡 9889 hex değer = 9889 byte PNG dosyası!

---

### 🎨 **4. PNG Header Doğrulama**

**İlk 4 byte'ı kontrol ediyoruz:**

```python
header = [matches[0], matches[1], matches[2], matches[3]]
print(f"İlk 4 byte: {header}")

if header == ['0x89', '0x50', '0x4e', '0x47']:
    print("✅ PNG header doğrulandı!")
```

**Çıktı:**
```
İlk 4 byte: ['0x89', '0x50', '0x4e', '0x47']
✅ PNG header doğrulandı!
```

**PNG File Format:**
```
Byte 0-3:  89 50 4E 47  (.PNG)     → PNG signature
Byte 4-7:  0D 0A 1A 0A             → DOS line ending + EOF + Unix line ending
Byte 8-11: 00 00 00 0D             → IHDR chunk length
Byte 12+:  IHDR chunk data         → Image header
...
```

---

### 🔄 **5. Hex to Binary Conversion**

**Hex değerlerini PNG dosyasına dönüştürüyoruz:**

```python
import re

# PCAP dosyasından hex değerleri çıkar
with open('Paket.pcap', 'rb') as f:
    content = f.read()

text_content = content.decode('latin-1')
hex_pattern = r'0x[0-9a-fA-F]{1,2}'
matches = re.findall(hex_pattern, text_content)

# Hex'i byte'a dönüştür
byte_data = bytearray()
for hex_val in matches:
    byte_val = int(hex_val, 16)
    byte_data.append(byte_val)

# PNG dosyası olarak kaydet
with open('extracted_image.png', 'wb') as f:
    f.write(byte_data)

print(f"✅ {len(byte_data)} byte PNG dosyası oluşturuldu!")
```

**Çıktı:**
```
✅ 9889 byte PNG dosyası oluşturuldu!
Dosya boyutu: 9889 bytes (9.66 KB)
```

---

### ✅ **6. PNG Dosyası Doğrulama**

**Oluşturulan dosyayı kontrol ediyoruz:**

```bash
file extracted_image.png
```

**Çıktı:**
```
extracted_image.png: PNG image data, 1152 x 648, 8-bit/color RGB, non-interlaced
```

**PNG Özellikleri:**
- ✅ **Format:** PNG (geçerli)
- 📏 **Boyut:** 1152 x 648 piksel
- 🎨 **Renk:** 8-bit RGB
- 📐 **Tip:** Non-interlaced

---

### 🖼️ **7. PNG İçeriğini Görüntüleme**

**Resmi açıyoruz:**

```bash
xdg-open extracted_image.png
# veya
eog extracted_image.png
# veya Python ile
from PIL import Image
img = Image.open('extracted_image.png')
img.show()
```

**Resimde ne var:**
```
╔════════════════════════════════════════════════╗
║                                                ║
║                                                ║
║   0xL4ugh{By735_3verywh3r3_WE3333}           ║
║                                                ║
║                                                ║
╚════════════════════════════════════════════════╝
```

> 🚩 **FLAG bulundu!** Siyah arka planda beyaz yazı ile!

---

### 🎯 **8. Flag Doğrulama**

**Bulunan FLAG:**
```
0xL4ugh{By735_3verywh3r3_WE3333}
```

**Analiz:**
- ✅ Format doğru: `0xL4ugh{...}`
- 🔤 İçerik: "Bytes everywhere WE3333" (leet speak)
- 📊 Karakter sayısı: 36

**Leet Speak Çevirisi:**
```
By735_3verywh3r3_WE3333
↓
Bytes Everywhere WE3333
```

> 💡 Challenge ismi "Paket" (packet), içeriğinde byte'lar var → "Bytes Everywhere"!

---

## 🚩 **FLAG**

```
0xL4ugh{By735_3verywh3r3_WE3333}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">📄<br><b>file</b><br><sub>Dosya tipi analizi</sub></td>
<td align="center">📝<br><b>strings</b><br><sub>String extraction</sub></td>
<td align="center">🐍<br><b>Python</b><br><sub>Hex parsing</sub></td>
<td align="center">🖼️<br><b>PIL/ImageMagick</b><br><sub>Image viewing</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**

```bash
# Dosya analizi
file Paket.pcap
ls -lh Paket.pcap

# String extraction
strings Paket.pcap | head -50
strings Paket.pcap | grep -i "0x"

# Python hex extraction ve PNG oluşturma
python3 extract_png.py

# PNG doğrulama
file extracted_image.png

# PNG görüntüleme
xdg-open extracted_image.png
eog extracted_image.png
```

**Python extraction scripti:**

```python
#!/usr/bin/env python3
"""
PCAP'ten Hex değerlerini çıkarıp PNG'ye dönüştürme
"""
import re
import os

def extract_png_from_pcap(pcap_file, output_file):
    """PCAP dosyasından hex değerlerini çıkar ve PNG oluştur"""
    
    # PCAP dosyasını oku
    with open(pcap_file, 'rb') as f:
        content = f.read()
    
    # Latin-1 encoding ile string'e çevir
    text_content = content.decode('latin-1')
    
    # Hex pattern: 0x ile başlayan 1-2 haneli hex değerler
    hex_pattern = r'0x[0-9a-fA-F]{1,2}'
    matches = re.findall(hex_pattern, text_content)
    
    print(f"[+] Toplam {len(matches)} hex değer bulundu")
    
    # PNG header kontrolü
    if len(matches) >= 4:
        header = matches[:4]
        if header == ['0x89', '0x50', '0x4e', '0x47']:
            print("[✓] PNG header doğrulandı!")
        else:
            print("[!] PNG header bulunamadı!")
            return False
    
    # Hex'i byte'a dönüştür
    byte_data = bytearray()
    for hex_val in matches:
        byte_val = int(hex_val, 16)
        byte_data.append(byte_val)
    
    # PNG dosyası olarak kaydet
    with open(output_file, 'wb') as f:
        f.write(byte_data)
    
    file_size = os.path.getsize(output_file)
    print(f"[✓] {len(byte_data)} byte PNG dosyası oluşturuldu: {output_file}")
    print(f"[i] Dosya boyutu: {file_size} bytes ({file_size/1024:.2f} KB)")
    
    return True

if __name__ == "__main__":
    # PCAP'ten PNG çıkar
    if extract_png_from_pcap('Paket.pcap', 'extracted_image.png'):
        print("[✓] İşlem başarılı!")
        print("[i] Resmi görüntülemek için: xdg-open extracted_image.png")
    else:
        print("[!] İşlem başarısız!")
```

**Çalıştırma:**
```bash
python3 extract_png.py
```

---

## 💭 **Çözüm Akışı**

```
📦 "Paket" PCAP Challenge
            ↓
📥 Paket.pcap indirildi (578 KB)
            ↓
📄 file Paket.pcap → PCAP capture file
            ↓
📝 strings Paket.pcap
   → "Hello From 0xL4ugh, Hope you enjoy the CTF"
   → 0x89, 0x50, 0x4e, 0x47 tespit edildi
            ↓
🖼️ PNG Magic Bytes tanındı!
   89 50 4E 47 = .PNG signature
            ↓
🔢 Regex ile hex extraction
   Pattern: 0x[0-9a-fA-F]{1,2}
   → 9889 hex değer bulundu
            ↓
🔄 Hex → Binary conversion
   for each 0xXX: int(0xXX, 16) → byte
            ↓
💾 PNG dosyası oluşturuldu
   extracted_image.png (9889 bytes)
            ↓
✅ file extracted_image.png
   → PNG image data, 1152 x 648
            ↓
🖼️ Resim görüntülendi
            ↓
🚩 FLAG: 0xL4ugh{By735_3verywh3r3_WE3333}
```
