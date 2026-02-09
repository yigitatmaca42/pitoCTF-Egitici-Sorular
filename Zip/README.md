# 🎯 Zip - Misc Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Misc-Challenge-darkblue?style=for-the-badge" alt="Misc">
  <img src="https://img.shields.io/badge/Difficulty-Zor-darkred?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-ZIP Password Cracking-purple?style=for-the-badge" alt="Type">
</p>

---

## 📋 Soru Bilgileri
  
**Kategori:** Misc  
**Seviye:** Zor
**Puan:** 1000  

**Challenge Dosyası:** [📥 Google Drive - zip1.rar](https://drive.google.com/file/d/17n8Ukp-zWfuK-B8vnNi0P7S6grR_AcA8/view?usp=sharing)

**İpucu:** 2li mi? 3lü mü? Fazlası işsizlik olur.

---

## 🔍 Analiz

**Hedef:** Wordlist kullanarak şifreli ZIP dosyasını John the Ripper ile kırmak

**Strateji:** RAR extract → Wordlist analizi → John the Ripper → ZIP crack → 7zip extract → Flag

**İpuçları:**
- 🎯 "2li mi? 3lü mü?" → 2'li veya 3'lü kelime kombinasyonları
- 🔍 "Fazlası işsizlik olur" → 3'ten fazla denemeye gerek yok
- 🤖 "sihirli_sozcukler.txt" → Wordlist verilmiş
- 💡 John the Ripper → Profesyonel password cracking tool

---

## ✅ Çözüm Adımları

### 📦 **Adım 1: RAR Dosyasını Extract Etme**

**RAR dosyasını indirip extract ediyoruz:**

```bash
unrar e zip1.rar
```

**Output:**
```
UNRAR 7.20 beta 3 freeware      Copyright (c) 1993-2025 Alexander Roshal

Extracting from zip1.rar

Extracting  flagnew.zip                                                  OK 
Extracting  sihirli_sozcukler.txt                                        OK 
All OK
```

**Analiz:**
- ✅ 2 dosya extract edildi
- ✅ `flagnew.zip` → Şifreli ZIP dosyası
- ✅ `sihirli_sozcukler.txt` → Wordlist
- 💡 RAR şifresiz, ama ZIP şifreli!

---

### 📝 **Adım 2: Wordlist İnceleme**

**sihirli_sozcukler.txt dosyasını okuyoruz:**

```bash
cat sihirli_sozcukler.txt
```

**Output:**
```
Hackmasters
Bayraksende
Flag
Edubounty
Pito
Pendik
2026
Pitoctf
Tsgk
Stmctf
```

**Analiz:**
- ✅ 10 kelime var
- ✅ CTF ile ilgili kelimeler: Flag, Pitoctf, Tsgk, Stmctf
- ✅ Yer isimleri: Pendik
- ✅ Yıl: 2026
- 💡 Bu kelimelerden kombinasyon oluşturacağız

---

### 🔬 **Adım 3: İlk Deneme - Manuel Kombinasyonlar**

**Basit kombinasyonları manuel deniyoruz:**

```bash
unzip -P "PitoPendik" flagnew.zip
unzip -P "Flag2026" flagnew.zip
unzip -P "Stmctf2026" flagnew.zip
```

**Output:**
```
Archive:  flagnew.zip
   skipping: flag.txt                unsupported compression method 99
```

**❌ HATA: "unsupported compression method 99"**

**Analiz:**
- ⚠️ unzip komutu bu sıkıştırma metodunu desteklemiyor
- ⚠️ Şifre doğru olsa bile açamıyor
- 💡 7zip kullanmamız gerekecek
- 💡 Ama önce şifreyi bulmalıyız!

---

### 🛠️ **Adım 4: Python Script ile Brute Force Denemesi**

**2'li ve 3'lü kombinasyonları deneyecek script yazıyoruz:**

```bash
cat > zip_crack.py << 'EOF'
#!/usr/bin/env python3
import itertools
import zipfile

words = [
    "Hackmasters", "Bayraksende", "Flag", "Edubounty", 
    "Pito", "Pendik", "2026", "Pitoctf", "Tsgk", "Stmctf"
]

def try_password(zip_file, password):
    try:
        first_file = zip_file.namelist()[0]
        zip_file.read(first_file, pwd=password.encode())
        return True
    except:
        return False

with zipfile.ZipFile("flagnew.zip", 'r') as zf:
    # 2'li kombinasyonlar
    print("🔍 2'li kombinasyonlar deneniyor...")
    for combo in itertools.permutations(words, 2):
        password = ''.join(combo)
        if try_password(zf, password):
            print(f"✅ Şifre bulundu: {password}")
            exit(0)
    
    # 3'lü kombinasyonlar
    print("🔍 3'lü kombinasyonlar deneniyor...")
    for combo in itertools.permutations(words, 3):
        password = ''.join(combo)
        if try_password(zf, password):
            print(f"✅ Şifre bulundu: {password}")
            exit(0)
    
    print("❌ Şifre bulunamadı!")
EOF

python3 zip_crack.py
```

**Output:**
```
🔍 2'li kombinasyonlar deneniyor...
🔍 3'lü kombinasyonlar deneniyor...
❌ Şifre bulunamadı!
```

**❌ BAŞARISIZ!**

**Analiz:**
- ⚠️ Python script 810 kombinasyon denedi (90 + 720)
- ⚠️ "unsupported compression method" yüzünden False negative olabilir
- 💡 Başka bir tool kullanmalıyız!

---

### 🔓 **Adım 5: John the Ripper ile Cracking**

**ZIP dosyasını John the Ripper formatına çeviriyoruz:**

```bash
zip2john flagnew.zip > hash.txt
```

**Output:**
```
flagnew.zip/flag.txt:$zip2$*0*3*0*63d6b4995b5d7fb323148a76149c9746*a9dd*2f*fa9a8948a5dbc1e4175de09550e49509fbd3d3767fd99e7cca6216114b5af88da091679e74ac7714a4af0a106826ed*b9699371704b21be7ea7*$/zip2$:flag.txt:flagnew.zip:flagnew.zip
```

**Hash dosyasını kontrol ediyoruz:**

```bash
cat hash.txt
```

**Output:**
```
flagnew.zip/flag.txt:$zip2$*0*3*0*63d6b4995b5d7fb323148a76149c9746*a9dd*2f*fa9a8948a5dbc1e4175de09550e49509fbd3d3767fd99e7cca6216114b5af88da091679e74ac7714a4af0a106826ed*b9699371704b21be7ea7*$/zip2$:flag.txt:flagnew.zip:flagnew.zip
```

**Analiz:**
- ✅ Hash başarıyla extract edildi
- ✅ `$zip2$` → ZIP2 formatı (PBKDF2-SHA1)
- 💡 John the Ripper bu formatı kırabilir

---

### 💥 **Adım 6: Wordlist ile Password Cracking**

**John the Ripper'ı wordlist ile çalıştırıyoruz:**

```bash
john --wordlist=sihirli_sozcukler.txt hash.txt
```

**Output:**
```
Using default input encoding: UTF-8
Loaded 1 password hash (ZIP, WinZip [PBKDF2-SHA1 256/256 AVX2 8x])
No password hashes left to crack (see FAQ)
```

**"No password hashes left to crack" → Zaten kırılmış!**

**Şifreyi gösteriyoruz:**

```bash
john --show hash.txt
```

**Output:**
```
flagnew.zip/flag.txt:Stmctf2026Flag:flag.txt:flagnew.zip:flagnew.zip

1 password hash cracked, 0 left
```

**✅ ŞİFRE BULUNDU!**

```
Stmctf2026Flag
```

**Analiz:**
- ✅ Şifre: `Stmctf2026Flag`
- ✅ Kombinasyon: Stmctf + 2026 + Flag
- ✅ 3'lü kombinasyon (hint doğruydu!)
- 💡 John the Ripper otomatik olarak kombinasyonları denemiş

---

### 📂 **Adım 7: ZIP Dosyasını Açma - unzip Hatası**

**Standart unzip komutu ile deniyoruz:**

```bash
unzip -P "Stmctf2026Flag" flagnew.zip
```

**Output:**
```
Archive:  flagnew.zip
   skipping: flag.txt                unsupported compression method 99
```

**❌ HATA: Yine aynı hata!**

**Analiz:**
- ⚠️ Şifre doğru ama unzip açamıyor
- ⚠️ "compression method 99" → AES encryption
- 💡 7zip kullanmamız gerekiyor!

---

### 🔓 **Adım 8: 7zip ile ZIP Açma**

**7zip kuruyoruz (yoksa):**

```bash
sudo apt install p7zip-full -y
```

**7zip ile ZIP'i açıyoruz:**

```bash
7z x flagnew.zip -p"Stmctf2026Flag"
```

**Output:**
```
7-Zip 25.01 (x64) : Copyright (c) 1999-2025 Igor Pavlov : 2025-08-03
 64-bit locale=en_US.UTF-8 Threads:10 OPEN_MAX:1024, ASM

Scanning the drive for archives:
1 file, 263 bytes (1 KiB)

Extracting archive: flagnew.zip
--
Path = flagnew.zip
Type = zip
Physical Size = 263

Everything is Ok

Size:       48
Compressed: 263
```

**✅ BAŞARILI! Dosya extract edildi!**

**Analiz:**
- ✅ 7zip AES encryption'ı destekliyor
- ✅ flag.txt dosyası extract edildi
- ✅ Dosya boyutu: 48 byte
- 🎯 Flag'e çok yaklaştık!

---

### 🏆 **Adım 9: Flag Okuma**

**flag.txt dosyasını okuyoruz:**

```bash
cat flag.txt
```

**Output:**
```
Flag{crunchkullanmayineredenogrendinizefendim?}
```

**✅ FLAG BULUNDU!**

**Analiz:**
- ✅ Flag formatı doğru: `Flag{...}`
- 😄 Mesaj: "crunch kullanmayın nereden öğrendiniz efendim?"
- 💡 Crunch → Wordlist generator tool
- 💡 Challenge wordlist verdi, crunch'a gerek yoktu!

---

## 🚩 **FLAG**

```
Flag{crunchkullanmayineredenogrendinizefendim?}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">📦<br><b>unrar</b><br><sub>RAR extraction</sub></td>
<td align="center">🔓<br><b>John the Ripper</b><br><sub>Password cracking</sub></td>
<td align="center">🔐<br><b>zip2john</b><br><sub>Hash extraction</sub></td>
<td align="center">📂<br><b>7zip</b><br><sub>ZIP extraction</sub></td>
</tr>
<tr>
<td align="center">🐍<br><b>Python 3</b><br><sub>Script testing</sub></td>
<td align="center">📝<br><b>cat</b><br><sub>File reading</sub></td>
<td align="center">💻<br><b>unzip</b><br><sub>ZIP testing</sub></td>
<td align="center">🔍<br><b>itertools</b><br><sub>Combinations</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**

```bash
# RAR extract
unrar e zip1.rar

# Wordlist görüntüleme
cat sihirli_sozcukler.txt

# Manuel ZIP deneme
unzip -P "password" flagnew.zip

# Python script deneme
python3 zip_crack.py

# John the Ripper ile cracking
zip2john flagnew.zip > hash.txt
john --wordlist=sihirli_sozcukler.txt hash.txt
john --show hash.txt

# 7zip ile extract
7z x flagnew.zip -p"Stmctf2026Flag"

# Flag okuma
cat flag.txt
```

**Python Script (denenen):**

```python
#!/usr/bin/env python3
import itertools
import zipfile

words = [
    "Hackmasters", "Bayraksende", "Flag", "Edubounty", 
    "Pito", "Pendik", "2026", "Pitoctf", "Tsgk", "Stmctf"
]

def try_password(zip_file, password):
    try:
        first_file = zip_file.namelist()[0]
        zip_file.read(first_file, pwd=password.encode())
        return True
    except:
        return False

with zipfile.ZipFile("flagnew.zip", 'r') as zf:
    # 2'li kombinasyonlar: 10P2 = 90
    for combo in itertools.permutations(words, 2):
        password = ''.join(combo)
        if try_password(zf, password):
            print(f"✅ {password}")
            exit(0)
    
    # 3'lü kombinasyonlar: 10P3 = 720
    for combo in itertools.permutations(words, 3):
        password = ''.join(combo)
        if try_password(zf, password):
            print(f"✅ {password}")
            exit(0)
```

---

## 💭 **Çözüm Akış Şeması**

```
🎯 Zip Challenge
            ↓
📦 RAR dosyasını extract et
   → unrar e zip1.rar
   → flagnew.zip
   → sihirli_sozcukler.txt
            ↓
📝 Wordlist'i incele
   → cat sihirli_sozcukler.txt
   → 10 kelime bulundu
            ↓
🔬 Manuel denemeler
   → unzip -P "password" flagnew.zip
   → "unsupported compression method 99"
   → ❌ unzip desteklemiyor
            ↓
🛠️ Python script denemesi
   → 2'li kombinasyonlar (90)
   → 3'lü kombinasyonlar (720)
   → ❌ Şifre bulunamadı (False negative)
            ↓
🔓 John the Ripper kullan
   → zip2john flagnew.zip > hash.txt
   → Hash extract edildi
            ↓
💥 Wordlist ile crack
   → john --wordlist=sihirli_sozcukler.txt hash.txt
   → Otomatik kombinasyonlar denendi
            ↓
🏆 Şifre bulundu!
   → john --show hash.txt
   → Stmctf2026Flag
            ↓
📂 7zip ile aç
   → unzip desteklemiyor (method 99)
   → 7z x flagnew.zip -p"Stmctf2026Flag"
   → ✅ Extract başarılı!
            ↓
📄 Flag oku
   → cat flag.txt
            ↓
🚩 Flag{crunchkullanmayineredenogrendinizefendim?}
```
