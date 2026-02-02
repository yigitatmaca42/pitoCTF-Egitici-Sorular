# 📄 Boş - Crypto Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Crypto-Challenge-darkblue?style=for-the-badge&logo=key" alt="Crypto">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Whitespace_Stego-purple?style=for-the-badge&logo=file" alt="Whitespace">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Crypto  
**Seviye:** Orta  
**Puan:** 300  

**Challenge Dosyası:** [📥 GitHub - bombos.docx](https://github.com/cihangungor/pitoctf/blob/main/bombos.docx)

**Firstblood:** +50 puan

---

## 🔍 Analiz

**Hedef:** Word dosyasındaki gizli whitespace steganography'yi çözmek

**Strateji:** Docx unzip → XML analiz → Whitespace pattern extraction → Decode

**İpuçları:**
- 📄 Dosya adı: "Boş" 
- 👁️ Görünür text: "Not Visible"
- 🔍 Gizli data: Whitespace karakterlerde

---

## ✅ Çözüm Adımları

### 📥 **1. Dosyayı İndirme ve İlk Analiz**

```bash
file bombos.docx
```

**Çıktı:**
```
bombos.docx: Microsoft Word 2007+
```

Word dosyasını açtığımızda:
```
Boş
Not Visible
[Görünüşte boş sayfa]
```

> 🤔 "Not Visible" → Gizli bir şey var!

---

### 📦 **2. Docx Yapısını İnceleme**

**Docx = ZIP arşivi!**

```bash
unzip -l bombos.docx
```

**Çıktı:**
```
Archive:  bombos.docx
  Length      Date    Time    Name
---------  ---------- -----   ----
     1322  1980-01-01 00:00   [Content_Types].xml
      593  1980-01-01 00:00   _rels/.rels
    18120  1980-01-01 00:00   word/document.xml  ← İçerik burada!
...
```

---

### 🔍 **3. Document.xml İnceleme**

```bash
unzip bombos.docx
cat word/document.xml
```

**XML içeriğinde pattern:**
```xml
<w:p><w:r><w:t xml:space="preserve">   </w:t></w:r><w:r><w:tab /></w:r>...</w:p>
<w:p><w:r><w:tab /></w:r></w:p>
<w:p><w:r><w:t xml:space="preserve">     </w:t></w:r><w:r><w:tab /></w:r>...</w:p>
```

> 💡 **Space ve Tab kombinasyonları!** Bu Whitespace Steganography!

---

### 📝 **4. Whitespace Pattern Çıkarma**

**Python script ile extract:**

```python
import xml.etree.ElementTree as ET

tree = ET.parse('word/document.xml')
root = tree.getroot()

ns = {'w': 'http://schemas.openxmlformats.org/wordprocessingml/2006/main'}

# Tüm paragrafları işle
for para in root.findall('.//w:p', ns):
    line = ""
    for elem in para.iter():
        if elem.tag.endswith('}t'):
            if elem.text:
                line += elem.text.replace(' ', 'S')
        elif elem.tag.endswith('}tab'):
            line += 'T'
    
    if line and line != 'T':  # Separator değilse
        print(line)
```

**Çıkan pattern'ler:**
```
SSSTSSSTTS
SSSSSTTSTTSS
SSSSSTTSSSST
SSSSSTTSSTTT
SSSSSTTTTSTT
...
```

---

### 🔓 **5. Whitespace Decode**

**Pattern analizi:**
- Her satır 5 Space (SSSSS) ile başlıyor → Padding
- Geri kalan kısım: Space (S) ve Tab (T) kombinasyonu
- Her pattern bir karakter encode ediyor!

**Decoding kuralı:**
- İlk 5 S'yi çıkar (padding)
- T = 1, S = 0 (binary)
- 7-bit ASCII olarak decode et

**Decode scripti:**

```python
patterns = [
    "SSSTSSSTTS",
    "SSSSSTTSTTSS",
    "SSSSSTTSSSST",
    # ... tüm pattern'ler
]

decoded = ""
for pattern in patterns:
    # İlk 5 S'yi çıkar
    if pattern.startswith("SSSSS"):
        data = pattern[5:]
    elif pattern.startswith("SSS"):
        data = pattern[3:]
    else:
        data = pattern
    
    # Binary'ye çevir (T=1, S=0)
    binary = data.replace('T', '1').replace('S', '0')
    
    # 7-bit ASCII
    binary = binary.ljust(7, '0')[:7]
    decoded += chr(int(binary, 2))

print(decoded)
```

**Çıktı:**
```
Flag{whitespaceNeredenOgrendinKral}
```

---

## 🚩 **FLAG**

```
Flag{whitespaceNeredenOgrendinKral}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>file</b><br><sub>Dosya tipi</sub></td>
<td align="center">📦<br><b>unzip</b><br><sub>Docx açma</sub></td>
<td align="center">🐍<br><b>Python</b><br><sub>XML parsing</sub></td>
<td align="center">📝<br><b>xml.etree</b><br><sub>XML analiz</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**
```bash
# Dosya analizi
file bombos.docx
unzip -l bombos.docx

# İçerik çıkarma
unzip bombos.docx
cat word/document.xml

# Python ile decode
python3 decode_whitespace.py
```

---

## 💭 **Çözüm Akışı**

```
📄 Crypto Challenge: "Boş"
            ↓
📥 bombos.docx indirildi
            ↓
👁️ Görünür: "Boş" + "Not Visible"
            ↓
🔍 Docx = ZIP → Unzip
            ↓
📝 word/document.xml analiz
            ↓
🔤 Space + Tab pattern'leri bulundu
            ↓
💡 Whitespace Steganography!
            ↓
🔓 Pattern decode (T=1, S=0)
            ↓
📊 7-bit ASCII decode
            ↓
🚩 Flag{whitespaceNeredenOgrendinKral}
```
