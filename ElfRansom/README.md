# 🦠 ElfRansom - Malware Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Malware-Challenge-darkblue?style=for-the-badge&logo=virus" alt="Malware">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Ransomware-purple?style=for-the-badge&logo=lock" alt="Ransomware">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Malware  
**Seviye:** Kolay  
**Açıklama:** Birisi bu ransomu kullanarak dosyalarımı şifreledi...😡

**Challenge Dosyası:** [📥 Google Drive - ElfRansom.zip](https://drive.google.com/file/d/1YifN_JxzuWowcYel3af9qs_JrLOTuBLC/view?usp=drivesdk)

**Flag Formatı:** `pitoctf{...}`

**İpuçları:**
- 💡 Dosyaların şifresini çözmek için kaynak kodlarda boğulmana gerek yok
- 💡 Çivi çiviyi söker :)
- 💡 Bu kütüphaneyle çalıştır. Belki terzi kendi söküğünü diker

**Verilen:**
- easyransom (ELF executable)
- Şifrelenmiş dosyalar (encrypted files)

---

## 🔍 Analiz

### Ransomware Nedir?

**Ransomware**, dosyaları şifreleyen ve genellikle fidye talep eden kötü amaçlı yazılımlardır. Bu challenge'da ransomware'in kendisini şifrelenmiş dosyaları **çözmek** için kullanacağız.

| Özellik | Açıklama |
|---------|----------|
| 🦠 **Ransomware** | Dosya şifreleme malware'i |
| 🔐 **Encryption** | Dosyaları şifreleme |
| 🔓 **Decryption** | Aynı programla şifre çözme |
| 🖼️ **Steganography** | Resimde gizli veri |

**Önemli İpucu:** "Çivi çiviyi söker" = Ransom kendisi dosyaları açabilir!

---

## ✅ Çözüm Adımları

### 📥 **1. Dosyaları İndirme ve İlk Deneme**

Dosyaları indirip ransomu çalıştıralım:

```bash
chmod +x easyransom
./easyransom
```

**Hata Çıktısı:**
```
error while loading shared libraries: ...
```

> **Problem:** Ransom dosyası çalışmıyor - loader hatası alıyoruz.

---

### 🔧 **2. Loader ile Çalıştırma**

Hata mesajını analiz ettiğimizde, ransom'un özel bir loader'a ihtiyacı olduğu anlaşılıyor:

```bash
/lib64/ld-linux-x86-64.so.2 ./easyransom
```

**Yeni Hata:**
```
Usage: ./easyransom <file>
```

> **İlerleme:** Program çalışıyor ama şifrelenecek dosya bekliyor!

---

### 📄 **3. Test Dosyası ile Deneme**

Bir test dosyası oluşturup ransomu deneyelim:

```bash
echo "test" > test.txt
/lib64/ld-linux-x86-64.so.2 ./easyransom test.txt
```

**Çıktı:**
```
File encrypted successfully!
```

> **Başarılı!** Ransom dosyayı şifreledi.

---

### 💡 **4. Anahtar İpucu: "Çivi Çiviyi Söker"**

**İpucu Analizi:**
- "Çivi çiviyi söker" = Ransom şifreleyen, ransom şifreyi açabilir
- "Terzi kendi söküğünü diker" = Ransom kendi şifrelediğini çözebilir

**Deney:** Şifrelenmiş dosyaları ransom ile tekrar çalıştıralım!

---

### 🔓 **5. Şifrelenmiş Dosyaları Decode Etme**

Challenge ile verilen şifrelenmiş dosyalarda ransomu çalıştırıyoruz:

```bash
/lib64/ld-linux-x86-64.so.2 ./easyransom encrypted_image.jpg.enc
/lib64/ld-linux-x86-64.so.2 ./easyransom encrypted_note.txt.enc
```

**Sonuç:**
```
File decrypted successfully!
```

> **Başarılı!** Dosyalar şifresi çözülüyor!

Şifre çözüldükten sonra:
- `encrypted_image.jpg.enc` → `encrypted_image.jpg`
- `encrypted_note.txt.enc` → `encrypted_note.txt`

---

### 🖼️ **6. Resmi İnceleme**

Şifresi çözülen resmi açıyoruz:

```bash
open encrypted_image.jpg
# veya
xdg-open encrypted_image.jpg
```

**Gözlem:**
- Resim açılıyor
- Flag'in **ilk yarısı** resimde görünüyor:
```
pitoctf{paintShow_and_metadata_birarada_
```

> **Yarım Flag!** Diğer yarısı nerede?

---

### 🔍 **7. Metadata ve Steganography Analizi**

İpucu: "metadata birarada" - flag'in diğer yarısı metadata'da olabilir!

**Exiftool ile metadata inceleme:**

```bash
exiftool encrypted_image.jpg
```

**Çıktı:**
```
...
Comment: RansomDecrypted}
...
```

> **İkinci Yarı Bulundu!** Comment alanında flag'in devamı var.

**Alternatif: Steghide ile arama:**

```bash
steghide extract -sf encrypted_image.jpg
```

Şifre sorulursa boş bırakıp Enter'a basın.

---

### 🚩 **8. Flag'i Birleştirme**

İki parçayı birleştiriyoruz:

**Resimden:** `pitoctf{paintShow_and_metadata_birarada_`  
**Metadata'dan:** `RansomDecrypted}`

**Tam Flag:**
```
pitoctf{paintShow_and_metadata_birarada_RansomDecrypted}
```

---

## 🚩 **FLAG**

```
pitoctf{paintShow_and_metadata_birarada_RansomDecrypted}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔧<br><b>ld-linux</b><br><sub>ELF loader</sub></td>
<td align="center">🔍<br><b>exiftool</b><br><sub>Metadata analizi</sub></td>
<td align="center">🖼️<br><b>steghide</b><br><sub>Steganography</sub></td>
<td align="center">💻<br><b>chmod</b><br><sub>Yetki verme</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**
- 🔧 **ld-linux-x86-64.so.2** - ELF dynamic loader
- 🔍 **exiftool** - EXIF metadata okuma
- 🖼️ **steghide** - Steganografi analizi
- 💻 **chmod** - Dosya yetkisi verme

---

## 💻 **Kullanılan Komutlar**

```bash
# Ransom'a çalıştırma yetkisi verme
chmod +x easyransom

# İlk deneme (başarısız)
./easyransom

# Loader ile çalıştırma
/lib64/ld-linux-x86-64.so.2 ./easyransom

# Test dosyası oluşturma
echo "test" > test.txt

# Test dosyasını şifreleme
/lib64/ld-linux-x86-64.so.2 ./easyransom test.txt

# Şifrelenmiş dosyaları decode etme
/lib64/ld-linux-x86-64.so.2 ./easyransom encrypted_image.jpg.enc
/lib64/ld-linux-x86-64.so.2 ./easyransom encrypted_note.txt.enc

# Resmi açma
open encrypted_image.jpg

# Metadata inceleme
exiftool encrypted_image.jpg

# Steghide ile analiz
steghide extract -sf encrypted_image.jpg

# Strings ile metin arama
strings encrypted_image.jpg | grep -i "ransom"
```

---

## 💭 **Çözüm Akış Şeması**

```
🦠 ElfRansom Challenge: Ransom + Encrypted files
                    ↓
        📥 Dosyaları İndir
                    ↓
        🔧 ./easyransom → Loader Error
                    ↓
        🔧 /lib64/ld-linux-x86-64.so.2 ile Çalıştır
                    ↓
        📄 Test dosyası ile deneme → Başarılı!
                    ↓
        💡 İpucu: "Çivi çiviyi söker"
                    ↓
        🔓 Şifrelenmiş dosyalarda ransom çalıştır
                    ↓
        ✅ Dosyalar decode edildi!
                    ↓
        🖼️ Resmi aç → Flag'in İlk Yarısı
                    ↓
        🔍 exiftool ile metadata kontrol
                    ↓
        📝 Comment: RansomDecrypted}
                    ↓
        🚩 FLAG: pitoctf{paintShow_and_metadata_birarada_RansomDecrypted}
```
