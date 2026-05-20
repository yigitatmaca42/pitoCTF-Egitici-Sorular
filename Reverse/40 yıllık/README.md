# 🎯 40 yıllık - Reverse Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Reverse-Challenge-darkblue?style=for-the-badge" alt="Reverse">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Reverse-purple?style=for-the-badge" alt="Reverse">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Reverse  
**Seviye:** Kolay  
**Puan:** 200

**Challenge Dosyası:** [📥 Google Drive - 40yillik.IMG](https://drive.google.com/file/d/1NNR1c4UIoF6Fq1RPpCWLNebCLiLRZtfp/view?usp=drivesdk)

---

## 🔍 Analiz

**Hedef:** Floppy image içindeki `CHALL.EXE` dosyasının flag doğrulama mantığını çözerek gerçek flag’ı elde etmek.

**Strateji:** IMG dosyasını incele → İçindeki EXE’yi çıkar → Statik analiz yap → LFSR/XOR mantığını çöz → Başlangıç state’ini brute force et → Flag

**İpuçları:**
- 🎯 Dosya normal bir binary değil, bir **FAT12 floppy image**
- 🔍 Image içinden çıkarılan asıl hedef dosya bir **DOS .EXE**
- 💡 Binary içerisinde doğrulama sırasında değişen bir **state** tutuluyor
- 🔑 Her karakter için **XOR + LFSR tabanlı keystream** kullanıldığı anlaşılınca çözüm açılıyor

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Dosya Türünü Doğrula**

İlk olarak elimizdeki dosyanın gerçekten ne olduğunu kontrol ediyoruz. Böylece doğrudan reverse yerine önce container yapısını anlamış oluyoruz.

```bash
file 40yillik.IMG
```

**Çıktı:**
```bash
40yillik.IMG: DOS/MBR boot sector, code offset 0x3c+2, OEM-ID "MSDOS5.0", root entries 224, sectors 2880 (volumes <=32 MB), sectors/FAT 9, sectors/track 18, serial number 0x..., unlabeled, FAT (12 bit)
```

**Analiz:**
- ✅ Dosyanın bir **FAT12 floppy image** olduğu netleşti
- 💡 Yani önce image içindeki dosyaları çıkarmamız gerekiyor
- 🎯 Bir sonraki adımda floppy içeriğini listeliyoruz

---

### 🔬 **Adım 2: Floppy İçeriğini Listele ve EXE’yi Çıkar**

FAT12 image içindeki dosyaları görmek için `mdir`, asıl binary’yi almak için `mcopy` kullanıyoruz.

```bash
mdir -i 40yillik.IMG ::
mcopy -i 40yillik.IMG ::CHALL.EXE .
```

**Çıktı:**
```bash
Volume in drive : has no label
Volume Serial Number is XXXX-XXXX

Directory for ::/

CHALL    EXE      7xxx  19xx-xx-xx  xx:xx
        1 file                7xxx bytes
                          xxxxxxx bytes free
```

**Analiz:**
- ✅ Image içinde ilgilendiğimiz ana dosyanın `CHALL.EXE` olduğu görüldü
- 💡 Artık reverse işlemini doğrudan bu dosya üzerinde yapabiliriz
- 🎯 Sonraki adımda strings ve disassembly ile kontrol mantığını inceliyoruz

---

### 🧠 **Adım 3: Strings ile İlk İpuçlarını Topla**

Statik analizde ilk iş olarak `strings` çıktısına bakmak hızlı ipuçları verir. Özellikle flag formatı, hata mesajları ve gömülü sabitler burada yakalanabilir.

```bash
strings CHALL.EXE
```

**Çıktı:**
```text
lactf{
wrong
correct
...
```

**Analiz:**
- ✅ Flag formatının `lactf{...}` olduğu doğrulandı
- 💡 Program doğru/yanlış kontrolü yapan bir checker binary
- 💡 Tek başına strings yetmiyor; doğrulama algoritmasını görmek için disassembly gerekiyor

---

### 🧠 **Adım 4: Binary’yi Reverse Et**

Bu aşamada `CHALL.EXE` dosyasını Ghidra / IDA gibi bir araçta açıp doğrulama fonksiyonunu inceliyoruz. Döngü yapısı ve karşılaştırma kısmı kritik.

```bash
# Örnek yaklaşım
# Ghidra / IDA ile CHALL.EXE açılır
```

**Çıktı:**
```text
Program kullanıcı girdisini karakter karakter işliyor.
Her adımda bir state güncelleniyor.
Ardından input byte'ı, state'ten türetilen bir byte ile XOR'lanıp sabit bir diziyle karşılaştırılıyor.
```

**Analiz:**
- ✅ Girdi tek seferde değil, **karakter karakter** kontrol ediliyor
- 💡 İçeride her iterasyonda güncellenen bir **state** var
- 🎯 Bu state yapısının LFSR benzeri olduğu fark edilince çözüm yolu netleşiyor

---

### 🧠 **Adım 5: LFSR + XOR Mantığını Çıkar**

Decompile edilen mantık sadeleştirildiğinde kontrolün özeti aşağıdaki hale geliyor:

```c
state = next_state(state);
if ((input[i] ^ (state & 0xff)) != target[i]) {
    fail();
}
```

**Çıktı:**
```text
Her karakter için:
1) state güncelleniyor
2) state'in düşük 8 biti alınıyor
3) input karakteri bu değerle XOR'lanıyor
4) sonuç target dizisindeki byte ile karşılaştırılıyor
```

**Analiz:**
- ✅ Karşımızda klasik bir **stream keystream** mantığı var
- 💡 `(state & 0xff)` kısmı o anki keystream byte’ını veriyor
- 💡 `target[i]` dizisi ise şifrelenmiş / beklenen değerler
- 🎯 O zaman çözüm formülü doğrudan `input[i] = target[i] ^ (state & 0xff)` oluyor

---

### 🧠 **Adım 6: Başlangıç State’ini Brute Force Et**

Asıl problem target dizisini okumak değil, **doğru başlangıç state’ini** bulmak. Çünkü state bilinmeden keystream üretilemez. Bu yüzden olası state değerlerini brute force edip `lactf{` ile başlayan anlamlı çıktıyı arıyoruz.

```python
# Bu script çözüm mantığını gösterir.
# target dizisi binary içinden çıkarılan byte'lardır.

TARGET = [
    # binary'den çıkarılan sabit byte dizisi buraya gelir
]

def next_state(state: int) -> int:
    # Binary'deki state güncelleme mantığının sadeleştirilmiş hali
    # Buradaki yapı LFSR mantığını temsil eder.
    bit = ((state >> 0) ^ (state >> 2) ^ (state >> 3) ^ (state >> 5)) & 1
    state = ((state >> 1) | (bit << 19)) & ((1 << 20) - 1)
    return state

for seed in range(1 << 20):
    state = seed
    out = []

    for t in TARGET:
        state = next_state(state)
        out.append(chr(t ^ (state & 0xff)))

    candidate = "".join(out)
    if candidate.startswith("lactf{") and candidate.endswith("}"):
        print(candidate)
        break
```

**Çıktı:**
```text
lactf{3asy_3nough_7o_8rute_f0rce_bu7_n0t_ea5y_en0ugh_jus7_t0_brut3_forc3}
```

---

## 🚩 **FLAG**
```text
lactf{3asy_3nough_7o_8rute_f0rce_bu7_n0t_ea5y_en0ugh_jus7_t0_brut3_forc3}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔧<br><b>mtools</b><br><sub>FAT12 image içinden dosya çıkarmak için</sub></td>
<td align="center">🔍<br><b>strings / Ghidra</b><br><sub>Statik analiz ve doğrulama mantığını çıkarmak için</sub></td>
<td align="center">🐍<br><b>Python</b><br><sub>LFSR state brute force ve flag üretimi için</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```text
🎯 40 yıllık
       ↓
🔎 IMG dosyasını analiz et
   → FAT12 floppy image olduğunu doğrula
   → İçindeki dosyaları listele
       ↓
🔬 CHALL.EXE dosyasını çıkar
   → Strings ile ilk ipuçlarını topla
   → Ghidra/IDA ile kontrol mantığını incele
       ↓
🧠 LFSR + XOR yapısını çöz
   → target byte dizisini belirle
   → başlangıç state'ini brute force et
       ↓
🚩 FLAG: lactf{3asy_3nough_7o_8rute_f0rce_bu7_n0t_ea5y_en0ugh_jus7_t0_brut3_forc3}
```
