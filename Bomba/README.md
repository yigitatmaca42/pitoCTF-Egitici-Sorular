# 💣 Bomba - Reverse Engineering Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Reverse Engineering-Challenge-blue?style=for-the-badge&logo=hackthebox" alt="Reverse">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Tool-Ghidra-purple?style=for-the-badge&logo=ghost" alt="Ghidra">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Reverse Engineering  
**Seviye:** Orta  
**Açıklama:** Bombayı çöz. Flag{SırasıylaçözdüğünŞifrelerAralarınaAltTireKoy}

**Challenge Dosyası:** [📥 GitHub - bomb](https://github.com/cihangungor/pitoctf/blob/main/bomb)

---

## 🔍 Analiz

Bu challenge'da **3 aşamalı** bomba çözme senaryosu var. Her phase için doğru şifreyi bulmamız gerekiyor.

**Aşamalar:**
- phase_1
- phase_2
- phase_3

---

## ✅ Çözüm Adımları

### 🛠️ **1. Ghidra Kurulumu ve Projeyi Açma**

**Ghidra'yı açıyoruz ve proje oluşturuyoruz:**

1. **File → New Project → Non-Shared Project**
2. Projemize isim veriyoruz (örneğin: "Sorular")
3. **Finish** diyoruz

**Bomb dosyasını import ediyoruz:**

1. **File → Import File**
2. Bomb dosyasının bulunduğu dizine gidip seçiyoruz
3. **Open** diyoruz

**Analiz başlatıyoruz:**

1. Ekrana **"Analyze?"** uyarısı geliyor → **Yes** diyoruz
2. **Analysis Options** penceresinde:
   - **Select All**
   - **Apply**
   - **Analyze**

---

## 🔎 **2. Keşif Aşaması**

### **Gizli Stringleri Bulma**

**Window → Defined Strings**

Burada gizli stringleri görüyoruz. Önemli yazılar buluyoruz:
- `phase_1`
- `phase_2`
- `phase_3`

> **Not:** Terminal'den `strings bomb` komutu ile de görülebilir.

### **Main Fonksiyonunu İnceleme**

**Symbol Tree → Functions → double click "main"**

1. Main'e geliyoruz ve aşağı doğru iniyoruz
2. Hangi fonksiyonlar çağrılıyor diye bakıyoruz
3. `phase_1`, `phase_2`, `phase_3` fonksiyonlarının çağrıldığını görüyoruz

---

## 🔓 **3. Main Kısmını Decompile Etme**

**Window → Decompiler**

Sağ tarafta decompile kısmı geliyor ve C benzeri kod görüyoruz:
```c
phase_1();
phase_2();
phase_3();
```

---

## 🧩 **4. Phase Decompile İşlemleri**

### **Phase 1 Decompile**

1. **Symbol Tree**'den `phase_1`'i seçiyoruz
2. Sağ taraftaki decompile kısmına bakıyoruz
   - (Eğer yanlışlıkla kapattıysanız: **phase_1 + Ctrl + E** yaparak geri getirin)

**10. satırda şu kodu görüyoruz:**
```c
builtin_strncpy(local_17,"9749d0",7);
```

✅ **phase_1 = `9749d0`**

---

### **Phase 2 Decompile**

1. **Symbol Tree**'den `phase_2`'yi seçiyoruz
2. Sağ taraftaki decompile kısmına bakıyoruz
   - (Eğer yanlışlıkla kapattıysanız: **phase_2 + Ctrl + E** yaparak geri getirin)

**14., 15., 16. ve 17. satırlarda şu kodları görüyoruz:**
```c
local_38[0] = 0xd2;
local_38[1] = 0x17a2;
local_38[2] = 0x253;
local_38[3] = 0x2373;
```

İlk başta anlamıyoruz fakat **mouse imlecini bu hex verilerinin üzerine getirip 1 saniye beklediğimizde** hangi rakama denk olduklarını görüyoruz.

**Hex → Decimal Dönüşümü:**

| Hex | Decimal |
|-----|---------|
| 0xd2 | 210 |
| 0x17a2 | 6050 |
| 0x253 | 595 |
| 0x2373 | 9075 |

✅ **phase_2 = `210 6050 595 9075`**

---

### **Phase 3 Decompile**

1. **Symbol Tree**'den `phase_3`'ü seçiyoruz
2. Sağ taraftaki decompile kısmına bakıyoruz
   - (Eğer yanlışlıkla kapattıysanız: **phase_3 + Ctrl + E** yaparak geri getirin)

**10. satırda şu kodu görüyoruz:**
```c
builtin_strncpy(local_19,"550d9479",9);
```

✅ **phase_3 = `550d9479`**

---

## 🔗 **5. Phase Birleştirme**

Tüm şifreleri sırayla birleştiriyoruz:

```
phase_1 = 9749d0
phase_2 = 210 6050 595 9075
phase_3 = 550d9479
```

**Birleşik Şifre:**
```
9749d0
210 6050 595 9075
550d9479
```

---

## 🧪 **6. Test**

### **Terminal'de Test Etme**

**1. Uygulamayı çalıştırılabilir yapma:**
```bash
chmod +x bomb
```

**2. Şifreleri otomatik girme:**
```bash
printf "9749d0\n210 6050 595 9075\n550d9479\n" | ./bomb
```

**3. Çıktı:**
```
Tebrikler!! Bombayı başarıyla çözdünüz.

Yukarıdaki anahtar değerleri forma girerek bayrağa ulaşabilirsiniz.
```

---

## 🚩 **FLAG**

Soruda bize **"Flag{SırasıylaçözdüğünŞifrelerAralarınaAltTireKoy}"** diyor.

**Şifreler:**
- phase_1: `9749d0`
- phase_2: `210 6050 595 9075`
- phase_3: `550d9479`

**Flag Formatı:**
```
Flag{9749d0_210_6050_595_9075_550d9479}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">👻<br><b>Ghidra</b><br><sub>Reverse engineering tool</sub></td>
<td align="center">🔍<br><b>Decompiler</b><br><sub>Kod analizi</sub></td>
<td align="center">🖥️<br><b>Terminal</b><br><sub>Test ortamı</sub></td>
</tr>
</table>

**Ghidra Komutları:**
- **Window → Defined Strings:** Gizli stringleri görme
- **Symbol Tree → Functions:** Fonksiyon listesi
- **Window → Decompiler:** C koduna dönüştürme
- **Ctrl + E:** Decompiler penceresini açma

**Terminal Komutları:**
- `chmod +x bomb` - Çalıştırılabilir yapma
- `strings bomb` - String'leri görüntüleme
- `printf "...\n" | ./bomb` - Otomatik input girme

---

## 💭 **Çözüm Akış Şeması**

```
🛠️ Ghidra Kurulum → 📂 Bomb Import → 🔬 Analyze
                            ↓
              🔍 Defined Strings: phase_1, phase_2, phase_3
                            ↓
              📝 Main Decompile: Fonksiyon çağrıları
                            ↓
              🔓 Phase 1 Decompile: builtin_strncpy → 9749d0
                            ↓
              🔓 Phase 2 Decompile: Hex array → 210 6050 595 9075
                            ↓
              🔓 Phase 3 Decompile: builtin_strncpy → 550d9479
                            ↓
              🔗 Şifre Birleştirme
                            ↓
              🧪 Terminal Test: printf | ./bomb
                            ↓
              ✅ Başarılı Çıktı
                            ↓
              🚩 FLAG: Flag{9749d0_210_6050_595_9075_550d9479}
```

---


😁😁 **Tebrikler! Bombayı Başarıyla Çözdünüz!** 😁😁
