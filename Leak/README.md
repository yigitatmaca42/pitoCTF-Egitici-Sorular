# 📱 Leak - Miscellaneous Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Misc-Challenge-darkblue?style=for-the-badge&logo=android" alt="Misc">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Android%20Backup-purple?style=for-the-badge&logo=file" alt="Android Backup">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Miscellaneous  
**Seviye:** Kolay  
**Açıklama:** Bu ne dosyası?

**Challenge Dosyası:** [📥 Google Drive - LeakedFile.zip](https://drive.google.com/file/d/1SDZAkXcb6XNT6YFl5ix_aggAf-b9bOra/view?usp=drivesdk)

**Verilen:**
- LeakedFile.zip (Android backup dosyası)

---

## 🔍 Analiz

### Android Backup (.ab) Dosyası Nedir?

**Android Backup**, Android cihazlardan alınan yedekleme dosyalarıdır. Uygulamalar, veriler, resimler ve diğer kullanıcı verilerini içerir.

| Özellik | Açıklama |
|---------|----------|
| 📱 **Format** | .ab (Android Backup) |
| 🗜️ **Sıkıştırma** | Genellikle zlib ile sıkıştırılmış |
| 📦 **İçerik** | Apps, shared data, pictures vb. |
| 🔓 **Çıkarma** | tar formatına dönüştürülerek açılır |

**Örnek:** Android telefon yedeği = Tüm uygulamalar + kullanıcı verileri

---

## ✅ Çözüm Adımları

### 📥 **1. Dosya İndirme ve Ayıklama**

Soruyu indirdiğimizde zip olarak geliyor ve masaüstüne atıyoruz:

```bash
cd Desktop
```

Dosyayı zipden çıkarıyoruz:

```bash
unzip LeakedFile.zip
```

**Çıktı:**
```
Archive: LeakedFile.zip
  inflating: content.bin
```

---

### 📂 **2. Düzenli Çalışma Ortamı Oluşturma**

Düzgün işlem yapmak için bir klasör oluşturuyoruz:

```bash
mkdir Leak
mv content.bin Leak
cd Leak
```

> **Not:** Artık tüm işlemlerimizi Leak klasörü içinde yapacağız.

---

### 🔍 **3. Dosya Türü Tespiti**

Dosyanın gerçek türünü öğrenelim:

```bash
file content.bin
```

**Çıktı:**
```
content.bin: Android Backup, version 5, Compressed, Not-Encrypted
```

> **Önemli:** Dosyamız **`.ab`** (Android Backup) uzantılı! Şifrelenmemiş ve sıkıştırılmış.

---

### 🔓 **4. Android Backup'ı TAR'a Dönüştürme**

Android backup dosyasını `.tar` formatına çeviriyoruz:

```bash
dd if=content.bin bs=24 skip=1 | openssl zlib -d > backup.tar
```

**Komut Açıklaması:**
- `dd if=content.bin bs=24 skip=1` = İlk 24 byte'ı atla (Android backup header)
- `openssl zlib -d` = zlib ile sıkıştırmayı aç
- `> backup.tar` = Çıktıyı tar dosyası olarak kaydet

**Çıktı:**
```
backup.tar dosyası oluşturuldu
```

---

### 📦 **5. TAR Dosyasını Açma**

TAR dosyasını extract ediyoruz:

```bash
tar -xvf backup.tar
```

**İçerik:**
```
apps/     = 573.1KB (oyun ve program dosyaları)
shared/   = 4.2MB (resim, video, ses dosyaları)
```

> **Not:** İki klasör var: `apps` (uygulamalar) ve `shared` (paylaşılan veriler)

---

### 🖼️ **6. Shared Klasörünü İnceleme**

`shared` klasörüne giriyoruz:

```bash
cd shared
ls -la
```

**İçerik:**
```
0/  (klasör)
```

`0` klasörüne giriyoruz:

```bash
cd 0
ls -la
```

**İçerik:**
```
DCIM/
Pictures/
Downloads/
(diğer telefon veri klasörleri...)
```

---

### 📸 **7. Pictures Klasörünü İnceleme**

Sadece `Pictures` klasörü dolu, ona giriyoruz:

```bash
cd Pictures
ls -lh
```

**İçerik:**
```
IMAG00001.jpg = 471.4KB (kedi fotoğrafı)
IMAG00002.jpg = 487.1KB (kedi fotoğrafı)
IMAG00003.jpg = 407.9KB (kedi fotoğrafı)
IMAG00004.jpg = 2MB (yüzü sansürlü adam fotoğrafı)
IMAG00005.jpg = 283.4KB (kedi fotoğrafı)
IMAG00006.jpg = 511.0KB (kedi fotoğrafı)
```

> **Dikkat:** `IMAG00004.jpg` diğerlerinden farklı - 2MB boyutunda ve yüzü sansürlü adam var!

---

### 🔍 **8. Şüpheli Fotoğrafı İnceleme**

`IMAG00004.jpg` dosyasını açıyoruz ve yakından inceliyoruz:

```bash
open IMAG00004.jpg
# veya
xdg-open IMAG00004.jpg
```

**Gözlem:**
- Adam elinde bir dosya/kağıt tutuyor
- Dosyanın sağ alt köşesine dikkatle bakıldığında...
- **FLAG görünüyor!**

> **Not:** Steganografi araçlarına (lsb, steghide, binwalk, exiftool, zsteg vb.) gerek yok - sadece göz ile inceleme yeterli!

---

### 🚩 **9. Flag Bulundu**

Fotoğraftaki dosyanın sağ alt köşesinde flag yazıyor:

```
HTB{ThisBackupIsUnprotected}
```

---

## 🚩 **FLAG**

```
HTB{ThisBackupIsUnprotected}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">📂<br><b>unzip</b><br><sub>Zip açma</sub></td>
<td align="center">🔍<br><b>file</b><br><sub>Dosya tipi</sub></td>
<td align="center">🔓<br><b>dd + openssl</b><br><sub>Backup decode</sub></td>
<td align="center">📦<br><b>tar</b><br><sub>Archive açma</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**
- 📂 **unzip** - ZIP dosyasını açma
- 🔍 **file** - Dosya türünü belirleme
- 🔓 **dd** - Binary veri işleme
- 🔐 **openssl zlib** - Sıkıştırmayı açma
- 📦 **tar** - TAR arşivini çıkarma

---

## 💻 **Kullanılan Komutlar**

```bash
# Masaüstüne geçiş
cd Desktop

# ZIP dosyasını açma
unzip LeakedFile.zip

# Çalışma klasörü oluşturma
mkdir Leak
mv content.bin Leak
cd Leak

# Dosya türünü kontrol etme
file content.bin

# Android backup'ı TAR'a çevirme
dd if=content.bin bs=24 skip=1 | openssl zlib -d > backup.tar

# TAR dosyasını açma
tar -xvf backup.tar

# Klasörleri gezinme
cd shared/0/Pictures

# Dosyaları listeleme
ls -lh

# Fotoğrafı açma
open IMAG00004.jpg
# veya
xdg-open IMAG00004.jpg
```

---

## 💭 **Çözüm Akış Şeması**

```
📱 Leak Challenge: LeakedFile.zip verildi
                    ↓
        📥 ZIP'i İndir ve Aç
                    ↓
        📂 content.bin Dosyası Çıktı
                    ↓
        🔍 file content.bin → Android Backup!
                    ↓
        🔓 dd + openssl ile TAR'a Çevir
                    ↓
        📦 backup.tar Oluşturuldu
                    ↓
        📂 tar -xvf backup.tar → İçeriği Aç
                    ↓
        📁 apps/ ve shared/ Klasörleri
                    ↓
        🔍 shared/0/Pictures/ → 6 Fotoğraf
                    ↓
        📸 IMAG00004.jpg Şüpheli!
                    ↓
        👁️ Fotoğrafı Yakından İncele
                    ↓
        📄 Adam Elinde Dosya Tutuyor
                    ↓
        🔍 Dosyanın Sağ Alt Köşesi
                    ↓
        🚩 FLAG: HTB{ThisBackupIsUnprotected}
```
