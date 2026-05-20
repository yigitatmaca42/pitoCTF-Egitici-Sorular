# 🎯 :) - Crypto Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Crypto-Challenge-darkblue?style=for-the-badge" alt="Crypto">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Base100 Encoding-blue?style=for-the-badge" alt="Encoding">
</p>

---

## 📋 Soru Bilgileri
  
**Kategori:** Crypto  
**Seviye:** Orta  
**Puan:** 300

**Challenge:** 
```
🐹👘👰👩👘👢👲👝👣👘👞👦👡👠👖👜👤👦👡👠👴
```

---

## 🔍 Analiz

**Hedef:** Emoji encoding'i çözmek

**Strateji:** Emoji pattern analizi → Base100 tespiti → Decode → Flag

**İpuçları:**
- 🎯 Sadece emoji'lerden oluşan mesaj
- 🔍 "emoji encoding" araması
- 💡 dCode.fr sitesi

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Emoji Encoding Araştırması**

Google'da **"emoji encoding"** araması yapıldığında dCode.fr sitesi çıkıyor:
```
https://www.dcode.fr/base100-emoji-encoding
```

**Analiz:**
- ✅ Base100 (Base💯) Emoji Encoding
- ✅ Her byte bir emoji ile temsil edilir
- 💡 U+1F3F7 - U+1F4F6 arası emojiler kullanılır

---

### 🌐 **Adım 2: dCode.fr Sitesinde Decode**

**Site:** https://www.dcode.fr/base100-emoji-encoding

1. Emoji string'i decode kutusuna yapıştır:
```
   🐹👘👰👩👘👢👲👝👣👘👞👦👡👠👖👜👤👦👡👠👴
```

2. **"Decode"** butonuna tıkla

3. **Sonuç:**
```
   Bayrak{flagoji_emoji}
```

---

### 🐍 **Adım 3: Manuel Python ile Decode (Alternatif)**
```python
#!/usr/bin/env python3

# Base100 Emoji Encoding Decoder
emojis = "🐹👘👰👩👘👢👲👝👣👘👞👦👡👠👖👜👤👦👡👠👴"

# Base100 başlangıç noktası
BASE100_START = 0x1F3F7

decoded_bytes = []

for emoji in emojis:
    unicode_value = ord(emoji)
    byte_value = unicode_value - BASE100_START
    decoded_bytes.append(byte_value)

# Bytes'ı string'e çevir
result = bytes(decoded_bytes).decode('ascii')
print(f"Flag: {result}")
```

**Çalıştırma:**
```bash
python3 base100_decode.py
```

**Çıktı:**
```
Flag: Bayrak{flagoji_emoji}
```

---

### 📊 **Adım 4: Decode Detayları**

| Emoji | Unicode | Byte | ASCII |
|-------|---------|------|-------|
| 🐹 | U+1F439 | 0x42 | B |
| 👘 | U+1F458 | 0x61 | a |
| 👰 | U+1F470 | 0x79 | y |
| 👩 | U+1F469 | 0x72 | r |
| 👘 | U+1F458 | 0x61 | a |
| 👢 | U+1F462 | 0x6B | k |
| 👲 | U+1F472 | 0x7B | { |
| 👝 | U+1F45D | 0x66 | f |
| 👣 | U+1F463 | 0x6C | l |
| 👘 | U+1F458 | 0x61 | a |
| 👞 | U+1F45E | 0x67 | g |
| 👦 | U+1F466 | 0x6F | o |
| 👡 | U+1F461 | 0x6A | j |
| 👠 | U+1F460 | 0x69 | i |
| 👖 | U+1F456 | 0x5F | _ |
| 👜 | U+1F45C | 0x65 | e |
| 👤 | U+1F464 | 0x6D | m |
| 👦 | U+1F466 | 0x6F | o |
| 👡 | U+1F461 | 0x6A | j |
| 👠 | U+1F460 | 0x69 | i |
| 👴 | U+1F474 | 0x7D | } |

**Sonuç:** `Bayrak{flagoji_emoji}`

---

## 🚩 **FLAG**
```
Bayrak{flagoji_emoji}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🌐<br><b>dCode.fr</b><br><sub>Online decoder</sub></td>
<td align="center">🐍<br><b>Python3</b><br><sub>Manual decode</sub></td>
<td align="center">💯<br><b>Base100</b><br><sub>Emoji encoding</sub></td>
</tr>
</table>

**Kullanılan Siteler:**
- https://www.dcode.fr/base100-emoji-encoding

---

## 💭 **Çözüm Akış Şeması**
```
🎯 :) Challenge
       ↓
🎨 Emoji string analiz
   → Sadece emoji'lerden oluşuyor
   → 21 emoji var
       ↓
🔍 Google: "emoji encoding"
   → dCode.fr sitesi bulundu
   → Base100 Emoji Encoding
       ↓
🌐 dCode.fr'de decode
   → Emoji string yapıştır
   → Decode butonuna tıkla
       ↓
📝 Sonuç alındı
   → Bayrak{flagoji_emoji}
       ↓
🚩 Bayrak{flagoji_emoji}
```
