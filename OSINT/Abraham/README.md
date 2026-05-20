# 🔍 Abraham - OSINT Challenge

<p align="center">
  <img src="https://img.shields.io/badge/OSINT-Challenge-darkblue?style=for-the-badge&logo=searchengineland" alt="OSINT">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Bonus-+2%20Points-success?style=for-the-badge&logo=star" alt="Bonus">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** OSINT  
**Seviye:** Orta  
**Puan:** 300  
**Soru No:** 84  
**Bonus:** +2 Puan  
**Açıklama:** Abraham Blackcoal gizli bir mesaj ele geçirmiş. Ama hala çözememiş. Biz de Abraham'ı bulmayı başaramadık. Bulabilir misin? Şifreyi ondan önce çöz.

**İpuçları:**
- Doğru hesabın profil resminde Kurtlar Vadisi repliği var
- Bu soruda aranan kişi "Emir Rüya" olsaydı adını "Order Dream" yapardım
- Abraham gizli mesajı çözebilmek için soru sorulan bir platforma üye olmuş
- **Q** harfi ile başlayan bir platform

---

## 🔍 Analiz

### İsim Çevirisi

**Abraham Blackcoal** adını araştırdığımızda ne bir hesap ne de başka bir şey bulamıyoruz. Ancak ipucunda verilen formata göre ismi çeviri ile çevirdiğimizde **İbrahim Karakömür** adını buluyoruz.

| İngilizce | Türkçe |
|-----------|--------|
| Abraham | İbrahim |
| Blackcoal | Karakömür |

### Platform Tespiti

İpuçlarından:
- Soru sorulan bir yer
- **Q** ile başlıyor
- ✅ Platform: **Quora**

---

## ✅ Çözüm Adımları

### 🔎 **1. Quora'da Kullanıcı Arama**

**Quora** sitesinde `İbrahim Karakömür` diye aratınca çıkmıyor. İsmi İngilizce karakterlere göre tekrar tasarlıyoruz: **Ibrahim Karakomur** ve böylece hesap çıkıyor.

**Hesap Arama Link:** https://www.quora.com/search?q=Ibrahim%20Karakomur  
**Hesap Link:** https://www.quora.com/profile/Ibrahim-Karak%C3%B6m%C3%BCr

---

### 📝 **2. Hesap İncelemesi ve Drive Linki**

Hesaba bakınca bir soru var ve soruda drive linki var. Linkten bir **txt** dosyası indiriyoruz.

**Drive Link:** https://drive.google.com/file/d/1qMRyo5dhGBWACUPWvNagUQMTcdH80VbX/view  
**Dosya:** `Gizlicok.txt`

---

### 🔧 **3. GCode Tespiti**

Txt içinde garip **X, Y, Z** koordinatları var. Bu kodları incelediğimizde bunların **GCode** olduğunu anlıyoruz ve **online GCode viewer** kullanarak bakıyoruz.

**Online GCode:** https://ncviewer.com/

**GCode formatı:**
```gcode
G21
G90
G0 Z5
G0 X10 Y10
G1 Z-1 F100
G1 X10 Y20
...
```

---

### 📊 **4. GCode Visualization**

Txt'deki kodu sol taraftaki kutuya yapıştırıp **plot** diyoruz ve ekrana bir yazı geliyor. Bakış açısını değiştirdiğimiz zaman yazı şöyle okunuyor:

**GCode Yazı:**
```
PITO{01000111 01000111 01001111 01010011 01010011 01001001 01001001 01010100 01010100 01010100 01010100 01010100}
```

---

### 🔓 **5. Binary Decode**

Bu **binary** şifrelemeyi hemen **ASCII** karşılığına decode ediyoruz.

**Binary Decode Link:** https://www.dcode.fr/ascii-code

| Binary | ASCII |
|--------|-------|
| 01000111 | G |
| 01000111 | G |
| 01001111 | O |
| 01010011 | S |
| 01010011 | S |
| 01001001 | I |
| 01001001 | I |
| 01010100 | T |
| 01010100 | T |
| 01010100 | T |
| 01010100 | T |
| 01010100 | T |

**Çıktı BIN/8:** `GGOSSIITTTTT`

---

## 🚩 **FLAG**

```
PITO{GGOSSIITTTTT}
```

**ANCAK BU FLAG NORMALDE HATALI, DOĞRUSU ŞU:**
```
PITO{GGOSINT}
```

**BOT BUNU ŞİFRELEYİP GCODE'A YAZARKEN BİNARY'DE HATA YAPMIŞ**

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>Quora</b><br><sub>Sosyal medya arama</sub></td>
<td align="center">☁️<br><b>Google Drive</b><br><sub>Dosya indirme</sub></td>
<td align="center">📊<br><b>NC Viewer</b><br><sub>GCode görselleştirme</sub></td>
<td align="center">🔐<br><b>dCode.fr</b><br><sub>Binary to ASCII</sub></td>
</tr>
</table>

**Kullanılan Siteler:**

- **QUORA ARAMA KISMI:** https://www.quora.com/search?q=Ibrahim%20Karakomur
- **QUORA HESAP LİNKİ:** https://www.quora.com/profile/Ibrahim-Karak%C3%B6m%C3%BCr
- **HESABIN PAYLAŞTIĞI DRİVE LİNK:** https://drive.google.com/file/d/1qMRyo5dhGBWACUPWvNagUQMTcdH80VbX/view
- **ONLİNE GCODE:** https://ncviewer.com/
- **BİNARY'İ ASCII KARŞILIĞI DECODE ETME:** https://www.dcode.fr/ascii-code

---

## 💭 **Çözüm Akış Şeması**

```
📝 Abraham Blackcoal → 🔄 İsim Çevirisi (İbrahim Karakömür)
                              ↓
                  🔍 Platform İpucu: Q ile başlıyor → Quora
                              ↓
              🌐 Hesap Arama: Ibrahim Karakomur (İngilizce karakter)
                              ↓
              ✅ Hesap Bulma + Profil Doğrulama (Kurtlar Vadisi)
                              ↓
              📥 Drive Linki → Gizlicok.txt İndirme
                              ↓
              🔧 İçerik Analizi: X, Y, Z koordinatları → GCode
                              ↓
              📊 NC Viewer: Plot → Görsel Metin
                              ↓
              🔐 Binary Decode: ASCII Karşılıkları
                              ↓
              🚩 FLAG: PITO{GGOSSIITTTTT} (Hatalı)
                              ↓
              ✅ Doğru FLAG: PITO{GGOSINT}
```

---
