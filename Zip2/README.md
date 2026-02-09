# 🗜️ Zip2 - Misc Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Misc-Challenge-darkblue?style=for-the-badge" alt="Misc">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Nested ZIP-purple?style=for-the-badge" alt="ZIP">
</p>

---

## 📋 Soru Bilgileri
  
**Kategori:** Misc  
**Seviye:** Kolay  
**Puan:** 1000

**Challenge Dosyası:** [📥 Google Drive - level1.zip](https://drive.google.com/file/d/1UGCWXfuvvVA-DadY771FRflMkXpQH1Kv/view?usp=drivesdk)

---

## 🔍 Analiz

**Hedef:** 500+ iç içe şifreli ZIP dosyasını açarak flag'e ulaşmak

**Strateji:** ZIP analizi → Şifre pattern tespiti → Otomasyon → Flag

**İpuçları:**
- 🗜️ "Zip içinde zip" → Nested ZIP dosyaları
- 🔢 500 level → Manuel açmak imkansız
- 🔑 PNG'lerde şifreler → Exiftool kullanımı
- 🤖 Otomasyon gerekli → Python script

---

## ✅ Çözüm Adımları

### 📥 **Adım 1: İlk ZIP'i İndirme ve Açma**

**level1.zip dosyasını indiriyoruz:**

```bash
cd ~/Desktop
mkdir zip2
cd zip2
# level1.zip dosyasını buraya at
```

**İlk ZIP'i açıyoruz:**

```bash
unzip level1.zip
```

**Çıktı:**

```
Archive:  level1.zip
 extracting: level2.zip         
  inflating: resim1.png
```

**Analiz:**
- ✅ `level2.zip` - Bir sonraki level
- ✅ `resim1.png` - Muhtemelen şifre içeriyor

---

### 🔎 **Adım 2: PNG Metadata Analizi**

**resim1.png'nin metadata'sını inceliyoruz:**

```bash
exiftool resim1.png
```

**Çıktı:**

```
ExifTool Version Number         : 13.44
File Name                       : resim1.png
Directory                       : .
File Size                       : 168 bytes
File Modification Date/Time     : 2026:02:05 23:37:19-05:00
File Type                       : PNG
Image Width                     : 50
Image Height                    : 50
Comment                         : p3z3UY8v3l
```

**🎯 ÖNEMLİ BULGU!**
- ✅ **Comment:** `p3z3UY8v3l`
- 💡 Bu level2.zip'in şifresi olmalı!

---

### 🔓 **Adım 3: Pattern Tespiti**

**level2.zip'i şifre ile açıyoruz:**

```bash
unzip level2.zip
```

**Şifre girildiğinde:** `p3z3UY8v3l`

**Çıktı:**

```
Archive:  level2.zip
 extracting: level3.zip              
  inflating: resim2.png
```

**Pattern anlaşıldı:**

```
resim1.png Comment → level2.zip şifresi
resim2.png Comment → level3.zip şifresi
resim3.png Comment → level4.zip şifresi
...
resimN.png Comment → levelN+1.zip şifresi
```

**Analiz:**
- ✅ Her PNG'nin Comment'i bir sonraki ZIP'in şifresi
- ✅ 500+ level var
- ❌ Manuel açmak imkansız
- 💡 **Otomasyon gerekli!**

---

### 🐍 **Adım 4: Python Automation Script Yazma**

**Otomatik ZIP açıcı script:**

```python
#!/usr/bin/env python3
import os
import zipfile
import subprocess
import re

def get_png_comment(png_file):
    """PNG dosyasındaki Comment alanını exiftool ile çıkart"""
    try:
        result = subprocess.run(['exiftool', png_file], 
                              capture_output=True, text=True)
        for line in result.stdout.split('\n'):
            if 'Comment' in line:
                comment = line.split(':')[1].strip()
                print(f"  [+] {png_file} Comment: {comment}")
                return comment
    except Exception as e:
        print(f"  [-] Hata: {e}")
    return None

def get_highest_numbered_file(directory, extension):
    """En büyük numaralı dosyayı bul (örn: resim5.png)"""
    files = [f for f in os.listdir(directory) if f.endswith(extension)]
    if not files:
        return None
    
    # Dosya adlarından sayıları çıkart
    numbered_files = []
    for f in files:
        match = re.search(r'(\d+)', f)
        if match:
            number = int(match.group(1))
            numbered_files.append((number, f))
    
    if not numbered_files:
        return files[0]  # Numarasız varsa ilkini al
    
    # En büyük numaralıyı döndür
    numbered_files.sort(reverse=True)
    return numbered_files[0][1]

def extract_zip_with_password(zip_file, password, extract_dir):
    """Şifreli zip dosyasını çıkart"""
    try:
        with zipfile.ZipFile(zip_file, 'r') as zip_ref:
            zip_ref.extractall(path=extract_dir, pwd=password.encode())
        print(f"  [+] {zip_file} çıkartıldı (şifre: {password})")
        return True
    except Exception as e:
        print(f"  [-] {zip_file} çıkartılamadı: {e}")
        return False

def main():
    work_dir = "/home/kali/Desktop/zip2"
    os.chdir(work_dir)
    
    print("[*] Zip2 Challenge Otomatik Çözücü")
    print(f"[*] Çalışma dizini: {work_dir}\n")
    
    level = 1
    
    while True:
        print(f"\n{'='*60}")
        print(f"[*] Level {level} işleniyor...")
        print(f"{'='*60}")
        
        # En büyük numaralı PNG dosyasını bul
        png_file = get_highest_numbered_file(".", ".png")
        if not png_file:
            print("[!] PNG dosyası bulunamadı!")
            break
        
        print(f"[*] PNG dosyası: {png_file}")
        
        # PNG'den şifreyi al
        password = get_png_comment(png_file)
        if not password:
            print("[!] Şifre bulunamadı!")
            break
        
        # En büyük numaralı ZIP dosyasını bul
        zip_file = get_highest_numbered_file(".", ".zip")
        if not zip_file:
            print("[!] ZIP dosyası bulunamadı!")
            print("\n[✓] TÜM LEVEL'LAR TAMAMLANDI!")
            break
        
        print(f"[*] ZIP dosyası: {zip_file}")
        
        # ZIP'i şifre ile çıkart
        if not extract_zip_with_password(zip_file, password, "."):
            print("[!] ZIP çıkartma başarısız!")
            break
        
        # Eski ZIP'i sil (karışıklık olmasın)
        try:
            os.remove(zip_file)
            print(f"  [+] {zip_file} silindi")
        except:
            pass
        
        level += 1
        
        # Sonraki level'da ZIP var mı kontrol et
        next_zip = get_highest_numbered_file(".", ".zip")
        if not next_zip:
            print("\n[✓] TÜM LEVEL'LAR TAMAMLANDI! Artık ZIP dosyası yok.")
            break
    
    print("\n[*] İşlem tamamlandı!")
    print("[*] Kalan dosyalar:")
    for f in os.listdir("."):
        if f.endswith((".png", ".zip", ".txt", ".flag")):
            print(f"  - {f}")

if __name__ == "__main__":
    main()
```

**Script özellikler:**
- ✅ En büyük numaralı PNG'yi bulur
- ✅ Comment'ten şifreyi çıkartır
- ✅ En büyük numaralı ZIP'i açar
- ✅ Eski ZIP'i siler
- ✅ ZIP kalmayınca durur

---

### 🚀 **Adım 5: Script'i Çalıştırma**

**Script'i kaydedip çalıştırıyoruz:**

```bash
nano oto_zip_acici.py
# Script'i yapıştır, Ctrl+S, Ctrl+X, Enter
```

**Çalıştırma:**

```bash
python3 oto_zip_acici.py
```

**Çıktı:**

```
[*] Zip2 Challenge Otomatik Çözücü
[*] Çalışma dizini: /home/kali/Desktop/zip2

============================================================
[*] Level 1 işleniyor...
============================================================
[*] PNG dosyası: resim2.png
  [+] resim2.png Comment: UEbUf7UCDe
[*] ZIP dosyası: level3.zip
  [+] level3.zip çıkartıldı (şifre: UEbUf7UCDe)
  [+] level3.zip silindi

============================================================
[*] Level 2 işleniyor...
============================================================
[*] PNG dosyası: resim3.png
  [+] resim3.png Comment: Na4TMlfvKp
[*] ZIP dosyası: level4.zip
  [+] level4.zip çıkartıldı (şifre: Na4TMlfvKp)
  [+] level4.zip silindi

... (498 level daha) ...

============================================================
[*] Level 399 işleniyor...
============================================================
[*] PNG dosyası: resim500.png
  [+] resim500.png Comment: 
[!] Şifre bulunamadı!

[*] İşlem tamamlandı!
[*] Kalan dosyalar:
  - flag.txt
  - resim500.png
  - (499 PNG daha...)
```

**Analiz:**
- ✅ **500 level başarıyla tamamlandı!**
- ✅ `flag.txt` dosyası ortaya çıktı
- ⏱️ Yaklaşık 30sn-1dk sürdü

---

### 🏆 **Adım 6: Flag Elde Etme**

**flag.txt dosyasını okuyoruz:**

```bash
cat flag.txt
```

**Çıktı:**

```
Flag{Cekirge3ziplardemislersi500kerezipladi}
```

**🎉 FLAG BULUNDU!**

---

## 🚩 **FLAG**

```
Flag{Cekirge3ziplardemislersi500kerezipladi}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🗜️<br><b>unzip</b><br><sub>ZIP extraction</sub></td>
<td align="center">🔍<br><b>exiftool</b><br><sub>Metadata reader</sub></td>
<td align="center">🐍<br><b>Python</b><br><sub>Automation</sub></td>
<td align="center">📦<br><b>zipfile</b><br><sub>Python module</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**

```bash
# Manuel ZIP açma
unzip level1.zip
unzip level2.zip

# PNG metadata okuma
exiftool resim1.png

# Script çalıştırma
python3 oto_zip_acici.py

# Flag okuma
cat flag.txt
```

---

## 💭 **Çözüm Akış Şeması**

```
🗜️ Zip2 Challenge
            ↓
📥 level1.zip indirildi
            ↓
🔓 level1.zip açıldı
   → level2.zip
   → resim1.png
            ↓
🔍 resim1.png metadata
   → Comment: p3z3UY8v3l
            ↓
💡 Pattern tespit edildi
   → resimN.png → levelN+1.zip şifresi
            ↓
🤖 Otomasyon gerekli
   → 500+ level var
            ↓
🐍 Python script yazıldı
   1. En büyük PNG bul
   2. Comment'ten şifre al
   3. En büyük ZIP aç
   4. Eski ZIP'i sil
   5. ZIP varsa tekrarla
            ↓
🚀 Script çalıştırıldı
   → 500 level tamamlandı
   → ~1 dakika sürdü
            ↓
🏆 flag.txt bulundu
            ↓
🚩 Flag{Cekirge3ziplardemislersi500kerezipladi}
```
