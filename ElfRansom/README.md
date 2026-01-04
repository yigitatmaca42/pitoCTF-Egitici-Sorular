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

**Challenge Dosyası:** [📥 Google Drive - ransomlandikisifreRANSOM.rar](https://drive.google.com/file/d/1YifN_JxzuWowcYel3af9qs_JrLOTuBLC/view?usp=drivesdk)

**Flag Formatı:** `pitoCTF{...}`

**İpuçları:**
- 💡 Çivi çiviyi söker :)
- 💡 Terzi kendi söküğünü diker

**Verilen Dosyalar:**
- EasyRansom (Ransomware programı)
- resim.png.encrypted (Şifrelenmiş resim)
- ozelnot.txt.encrypted (Şifrelenmiş not)

---

## 🔍 Ne Yapacağız?

Elimizde bir **ransomware** (fidye yazılımı) var. Bu program dosyaları şifreler. AMA... aynı program şifrelenmiş dosyaları geri açabilir! İşte yapacağımız bu: Ransom'u şifrelenmiş dosyalarda çalıştırıp geri açacağız.

---

## ✅ Çözüm Adımları

### 📥 **Adım 1: RAR Dosyasını Açma**

RAR dosyasını indirip açıyoruz:

```bash
unrar x ransomlandiksifreRANSOM.rar -pRANSOM  
```

İçinden çıkan dosyalar:
- `EasyRansom` (program)
- `resim.png.encrypted` (şifrelenmiş resim)
- `ozelnot.txt.encrypted` (şifrelenmiş not)

---

### 🔧 **Adım 2: Programı Çalıştırmayı Deneme**

İlk deneme:

```bash
chmod +x EasyRansom
./EasyRansom
```

**Hata alıyoruz!** Program çalışmıyor.

---

### 🔧 **Adım 3: Özel Komutla Çalıştırma**

Bu özel komutla çalıştırıyoruz:

```bash
/lib64/ld-linux-x86-64.so.2 ./EasyRansom
```

**Çıktı:**
```
Usage: ./EasyRansom <file>
```

Program çalıştı! Ama dosya ismi bekliyor.

---

### 💡 **Adım 4: Anahtar Fikir**

İpucu: **"Çivi çiviyi söker"** ne demek?

**Cevap:** Şifreleyen program, şifreyi de açabilir!

Yani bu ransom programını, şifrelenmiş dosyalarda çalıştırırsak dosyalar açılacak!

---

### 🔓 **Adım 5: Şifrelenmiş Dosyaları Açma**

Şifrelenmiş dosyalarda ransom'u çalıştırıyoruz:

```bash
/lib64/ld-linux-x86-64.so.2 ./EasyRansom resim.png.encrypted
/lib64/ld-linux-x86-64.so.2 ./EasyRansom ozelnot.txt.encrypted
```

**Çıktı:**
```
ozelnot.txt.encrypted.encrypted
resim.png.encrypted.encrypted
```

**Başarılı!** Dosyalar açıldı:
- `resim.png.encrypted.encrypted` → `resimde flag parçası var`
- `ozelnot.txt.encrypted.encrypted` → `flag burada değil. Ama doğru yoldasın`

---

### 🖼️ **Adım 6: Resmi Açma**

Şifresi açılan resmi görüntülüyoruz:

```bash
open resim.png.encrypted.encrypted
# veya
xdg-open resim.png.encrypted.encrypted
```

**Ne görüyoruz?**

Resimde flag'in ilk yarısı yazıyor:
```
pitoCTF{paintShow_
```

Ama tamamı yok! Devamı nerede?

---

### 🔍 **Adım 7: Gizli Veriyi Bulma**

Resmin gizli verilerine bakalım:

```bash
exiftool resim.png
```

**Çıktı:**
```
...
Comment: and_metadata_birarada_RansomDecrypted}
...
```

**Bulundu!** Flag'in ikinci yarısı Comment kısmında!


---

### 🚩 **Adım 8: Flag'i Tamamlama**

İki parçayı birleştiriyoruz:

**Resimden:** `pitoCTF{paintShow_`  
**Comment'ten:** `and_metadata_birarada_RansomDecrypted}`

**Tam Flag:**
```
pitoCTF{paintShow_and_metadata_birarada_RansomDecrypted}
```

---

## 🚩 **FLAG**

```
pitoCTF{paintShow_and_metadata_birarada_RansomDecrypted}
```

---

## 💻 **Tüm Komutlar Sırayla**

```bash
# 1. RAR dosyasını aç
unrar x ransomlandikisifreRANSOM.rar -pRANSOM

# 2. Programa yetki ver
chmod +x EasyRansom

# 3. Şifrelenmiş dosyaları aç
/lib64/ld-linux-x86-64.so.2 ./EasyRansom resim.png.encrypted
/lib64/ld-linux-x86-64.so.2 ./EasyRansom ozelnot.txt.encrypted

# 4. Resmi aç
open resim.png.encrypted.encrypted
# veya
xdg-open resim.png.encrypted.encrypted

# 5. Gizli veriyi bul
exiftool resim.png.encrypted.encrypted

```

---

## 💭 **Basit Akış Şeması**

```
📥 RAR'ı indir ve aç
       ↓
🔧 EasyRansom programını özel komutla çalıştır
       ↓
🔓 resim.png.encrypted ve ozelnot.txt.encrypted'da ransom'u çalıştır
       ↓
✅ Dosyalar açıldı!
       ↓
🖼️ resim.png.encrypted.encrypted'yi aç → İlk yarı flag
       ↓
🔍 exiftool ile gizli veriyi bul → İkinci yarı
       ↓
🚩 FLAG tamam!
```
