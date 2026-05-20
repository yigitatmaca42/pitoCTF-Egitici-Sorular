# 🎯 Dörtgenler - Stego Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Stego-Challenge-darkblue?style=for-the-badge" alt="Stego">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-PDF JavaScript-purple?style=for-the-badge" alt="PDF JavaScript">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Stego  
**Seviye:** Orta  
**Puan:** 500

**Challenge Dosyası:** [📥 GitHub - Dortgens.pdf](https://github.com/cihangungor/pitoctf/blob/main/Dortgens.pdf)

---

## 🔍 Analiz

**Hedef:** PDF dosyasının içine gizlenmiş flag'i bulmak

**Strateji:** PDF inceleme → metadata analizi → stream decompress → JavaScript tespiti → XOR deobfuscation → flag

**İpuçları:**
- 🎯 PDF'in `Producer` metadata alanında şüpheli bir sayı var: `283548893274`
- 🔍 PDF içinde gizli **obfuscate edilmiş JavaScript** kodu bulunuyor
- 💡 JavaScript, kullanıcı girdisini `Producer` değeriyle XOR'layıp hardcoded `k` dizisiyle karşılaştırıyor
- 🔑 XOR tersine çevrilerek flag elde edilebilir

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: PDF Dosyasını İncele**

`Dortgens.pdf` dosyası terminalde incelendi:

```bash
file Dortgens.pdf
```

**Çıktı:**
```
Dortgens.pdf: PDF document, version 1.5
```

```bash
exiftool Dortgens.pdf
```

**Çıktı:**
```
ExifTool Version Number         : 13.44
File Name                       : Dortgens.pdf
PDF Version                     : 1.5
Page Count                      : 1
Author                          : 
Title                           : 
Creator                         : 
Producer                        : 283548893274
Create Date                     : 2024:05:28 13:48:25-04:00
PTEX Fullbanner                 : This is pdfTeX, Version 3.141592653-2.6-1.40.26
```

**Analiz:**
- ✅ PDF geçerli ve açılabilir durumda
- 💡 `Producer` alanı normalde yazılım adı içerir, burada `283548893274` gibi şüpheli bir sayı var
- 🎯 Bu sayı ileride XOR anahtarı olarak kullanılıyor olabilir

---

### 🧠 **Adım 2: binwalk ile İçeriği Tara**

```bash
binwalk Dortgens.pdf
```

**Çıktı:**
```
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PDF document, version: "1.5"
181           0xB5            Zlib compressed data, best compression
428           0x1AC           Zlib compressed data, best compression
770           0x302           Zlib compressed data, best compression
1007          0x3EF           Zlib compressed data, best compression
1272          0x4F8           Zlib compressed data, best compression
9232          0x2410          Zlib compressed data, best compression
11516         0x2CFC          Zlib compressed data, best compression
...
```

**Analiz:**
- ✅ PDF içinde birden fazla Zlib sıkıştırılmış stream bulunuyor
- 💡 Bu stream'ler decompress edilerek içerik incelenebilir
- 🎯 Stream'lerden birinde JavaScript kodu olabilir

---

### 🔑 **Adım 3: PDF Stream'lerini Decompress Et**

Python ile tüm stream'ler açıldı:

```python
import zlib, re

with open("Dortgens.pdf", "rb") as f:
    data = f.read()

streams = re.findall(rb'stream\r?\n(.*?)\r?\nendstream', data, re.DOTALL)
for i, s in enumerate(streams):
    try:
        dec = zlib.decompress(s)
        print(f"Stream {i}:\n{dec[:300]}\n")
    except:
        pass
```

**Kritik Çıktı (Stream 10):**
```
...
<< /S/JavaScript/JS(var _thereisdjs=true;
   (function(_0x18b13a,_0x4d582d){ ... }
   function update(){
     var _0x2923fd=this[_0x3d0e72(0x1cd)]('A')['value']
     ...
     k=[0x46,0x2d,0x62,0x11,0x6b,0x4c,0x72,0x5f,0x76,0x38,
        0x19,0x28,0x5f,0x31,0x36,0x63,0xf7,0xb1,0x69,0x2a,
        0x18,0x5e,0x36,0x1,0x37,0x3a,0x1c,0x5,0x11,0x56,
        0xe5,0x7b,0x64,0x2c,0x11,0x14,0x53,0x5a,0x35,0x17,
        0x41,0x62,0x3];
     ...
   })
```

**Analiz:**
- ✅ Stream 10 içinde gizlenmiş, obfuscate edilmiş JavaScript kodu tespit edildi
- 💡 PDF interaktif bir form içeriyor: kullanıcıdan `A` alanına input alıyor
- 🎯 `k` dizisi hardcoded hedef değerleri içeriyor → bunlar XOR ile şifrelenmiş flag karakterleri

---

### 🔢 **Adım 4: JavaScript Mantığını Deobfuscate Et**

Obfuscate edilmiş JavaScript temizlenip okunabilir hale getirildi:

```javascript
function update() {
    var input = this.getField('A').value;  // Kullanıcı girdisi
    var result = [];

    for (var i = 0; i < input.length; i++) {
        result.push(
            input.charCodeAt(i) ^ (parseInt(info.producer) % (0x75 + i))
        );
    }

    k = [0x46,0x2d,0x62,0x11,0x6b,0x4c,0x72,0x5f,0x76,0x38,
         0x19,0x28,0x5f,0x31,0x36,0x63,0xf7,0xb1,0x69,0x2a,
         0x18,0x5e,0x36,0x1,0x37,0x3a,0x1c,0x5,0x11,0x56,
         0xe5,0x7b,0x64,0x2c,0x11,0x14,0x53,0x5a,0x35,0x17,
         0x41,0x62,0x3];

    if (result.length != k.length) { app.alert('Flag is incorrect!'); return; }
    for (var i = 0; i < k.length; i++) {
        if (result[i] != k[i]) { app.alert('Flag is incorrect!'); return; }
    }
    app.alert('Flag is correct!');
}
```

**Analiz:**
- ✅ Şifreleme mantığı netleşti: `charCode XOR (producer % (0x75 + i))`
- 💡 `info.producer` = `283548893274` (exiftool'dan bulduğumuz değer!)
- 🎯 XOR tersine çevrilebilir: `k[i] XOR key` işlemi flag karakterini verir

---

### 🔄 **Adım 5: XOR'u Tersine Çevir → Flag'i Bul**

```python
producer = 283548893274
k = [0x46,0x2d,0x62,0x11,0x6b,0x4c,0x72,0x5f,0x76,0x38,
     0x19,0x28,0x5f,0x31,0x36,0x63,0xf7,0xb1,0x69,0x2a,
     0x18,0x5e,0x36,0x1,0x37,0x3a,0x1c,0x5,0x11,0x56,
     0xe5,0x7b,0x64,0x2c,0x11,0x14,0x53,0x5a,0x35,0x17,
     0x41,0x62,0x3]

flag = ""
for i, val in enumerate(k):
    key = producer % (0x75 + i)
    flag += chr(val ^ key)

print(flag)
```

**Çıktı:**
```
bcactf{InTerACtIv3_PdFs_W0W_cbd14436e6aea8}
```

---

## 🚩 **FLAG**
```
bcactf{InTerACtIv3_PdFs_W0W_cbd14436e6aea8}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>exiftool</b><br><sub>Metadata analizi</sub></td>
<td align="center">📦<br><b>binwalk</b><br><sub>Stream tespiti</sub></td>
<td align="center">🐍<br><b>Python / zlib</b><br><sub>Stream decompress + XOR</sub></td>
<td align="center">💻<br><b>strings / xxd</b><br><sub>Binary içerik okuma</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 Dörtgenler Stego Challenge
       ↓
📄 Dortgens.pdf (GitHub'dan indirildi)
   → file → "PDF document, version 1.5"
   → exiftool → Producer: 283548893274 (şüpheli!)
       ↓
📦 binwalk ile tarama
   → 12 adet Zlib stream tespit edildi
   → Stream'ler decompress edildi
       ↓
🧠 Stream 10 analizi
   → Obfuscate edilmiş JavaScript bulundu
   → update() fonksiyonu: input XOR producer % (0x75+i)
   → Hardcoded k dizisi tespit edildi ✅
       ↓
🔄 JavaScript deobfuscation
   → Şifreleme mantığı netleştirildi
   → Anahtar = Producer değeri (283548893274) ✅
       ↓
🔑 XOR tersine çevirme
   → k[i] XOR (producer % (0x75+i)) = flag karakteri
   → Python ile 43 karakter çözüldü ✅
       ↓
🚩 FLAG: bcactf{InTerACtIv3_PdFs_W0W_cbd14436e6aea8}
```
