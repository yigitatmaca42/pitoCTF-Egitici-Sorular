# 🎯 Telefon - Stego Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Stego-Challenge-darkblue?style=for-the-badge" alt="Stego">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Android%20Boot%20Image-purple?style=for-the-badge" alt="Android Boot Image">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Stego  
**Seviye:** Orta  
**Puan:** 500

**Challenge Dosyası:** [📥 Google Drive - tel](https://drive.google.com/file/d/1By1sxQD4p5HrI3qgRsswhk_AG9ancKM0/view?usp=drivesdk)

---

## 🔍 Analiz

**Hedef:** `tel` dosyası içindeki gizli flag'i bulmak

**Strateji:** Dosya tipi tespiti → Android boot image parse → Ramdisk extract → PNG stego analizi → Flag okuma

**İpuçları:**
- 🎯 `tel` dosyası bir Android boot image
- 🔍 İçinde kernel + ramdisk bulunuyor
- 💡 Ramdisk içindeki PNG dosyalarından birinde flag gizli
- 🔑 Flag bitmap font ile görsel olarak PNG'ye işlenmiş

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Dosya Tipini Tespit Et**

Drive linkinden `tel` dosyası indirildi. Terminal'de incelendi:

```bash
file tel
```

**Çıktı:**
```
tel: Android bootimg, kernel, ramdisk, page size: 2048, cmdline (console=ttyHSL0,115200,n8 androidboot.hardware=mako lpj=67677 user_debug=31)
```

**Analiz:**
- ✅ Dosya bir **Android Boot Image**
- 💡 İçinde kernel ve ramdisk bölümleri bulunuyor
- 🎯 `binwalk` ile detaylı analiz yapılacak

---

### 🧠 **Adım 2: binwalk ile İçeriği Analiz Et**

```bash
binwalk tel
```

**Çıktı:**
```
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             Android bootimg, kernel size: 6009416 bytes...
2048          0x800           Linux kernel ARM boot executable zImage (little-endian)
18399         0x47DF          gzip compressed data, maximum compression...
6012928       0x5BC000        gzip compressed data, from Unix...
```

**Analiz:**
- ✅ İki adet gzip verisi tespit edildi
- 💡 `0x5BC000` offsetindeki gzip → **ramdisk** (Android ramdisk'leri cpio arşividir)
- 🎯 `binwalk -e` ile extract edilecek

---

### 🗜️ **Adım 3: Dosyaları Extract Et**

```bash
binwalk -e tel
ls _tel.extracted/
```

**Çıktı:**
```
47DF  5BC000  5BC000.gz
```

`5BC000.gz` dosyasının tipi kontrol edildi:

```bash
file _tel.extracted/5BC000
```

**Çıktı:**
```
_tel.extracted/5BC000: ASCII cpio archive (SVR4 with no CRC)
```

**Analiz:**
- ✅ Ramdisk başarıyla extract edildi
- 💡 Dosya bir **cpio arşivi** → Android ramdisk standardı
- 🎯 `cpio` ile içeriği çıkarılacak

---

### 📂 **Adım 4: Ramdisk'i Aç**

```bash
mkdir ramdisk && cd ramdisk && cpio -i < ../_tel.extracted/5BC000
ls
```

**Çıktı:**
```
charger   default.prop  file_contexts  fstab.mako  init
init.environ.rc  init.mako.rc  init.mako.usb.rc  init.rc
init.trace.rc  init.usb.rc  init.zygote32.rc  proc
property_contexts  res  sbin  seapp_contexts  selinux_version
sepolicy  service_contexts  sys  system  ueventd.mako.rc  ueventd.rc
```

**Analiz:**
- ✅ Standart Android ramdisk yapısı çıktı
- 💡 `res/images/charger/` dizini dikkat çekici → charger animasyonu PNG dosyaları
- 🎯 PNG dosyaları incelenecek

---

### 🖼️ **Adım 5: Flag'i Oku**

Ramdisk extract edildikten sonra direkt `res/images/charger/` klasörüne gidildi. İçinde iki PNG dosyası bulunuyor:

```
battery_fail.png  battery_scale.png
```

`battery_fail.png` dosyası açıldığında görselin **sağ tarafında** flag açıkça okunabilir şekilde yazıyor:

```
bctf{gr33n_r0b0t_ph0N3}
```

**Analiz:**
- ✅ Flag, `battery_fail.png` içine bitmap font kullanılarak **görsel olarak gizlenmiş**
- 💡 Pil görselinin sağ kenarında düz metin olarak yazılmış, terminale gerek kalmadan gözle okunabiliyor
- 🎯 Gizleme yöntemi: standart steganografi araçları değil, **doğrudan piksel manipülasyonu**

---

## 🚩 **FLAG**
```
bctf{gr33n_r0b0t_ph0N3}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>binwalk</b><br><sub>Boot image analizi & extract</sub></td>
<td align="center">📦<br><b>cpio</b><br><sub>Ramdisk arşivi açma</sub></td>
<td align="center">🖼️<br><b>zsteg</b><br><sub>PNG stego analizi</sub></td>
<td align="center">📋<br><b>exiftool</b><br><sub>Metadata inceleme</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 Telefon Stego Challenge
       ↓
📄 tel dosyası (Drive'dan indirildi)
   → file → "Android bootimg"
   → binwalk → kernel + ramdisk tespit edildi
       ↓
🗜️ binwalk -e
   → _tel.extracted/5BC000.gz → cpio arşivi
       ↓
📂 cpio -i
   → Android ramdisk dosya sistemi açıldı
   → res/images/charger/ dizini bulundu
       ↓
🖼️ res/images/charger/ klasörüne gidildi
   → battery_fail.png açıldı
   → Sağ tarafta flag görsel olarak yazılmış ✅
       ↓
🚩 FLAG: bctf{gr33n_r0b0t_ph0N3}
```
