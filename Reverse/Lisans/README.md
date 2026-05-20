# 🔓 Lisans - Reverse Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Reverse-Challenge-darkblue?style=for-the-badge" alt="Reverse">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-.NET%20Decompile-purple?style=for-the-badge&logo=dotnet" alt=".NET">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Reverse  
**Seviye:** Orta  
**Puan:** 400
**Açıklama:** "alicia" kullanıcısının serial keyini bulabilir misin?

**Flag Formatı:** `Flag{alicia'nın Serial Keyi}`

**Challenge Dosyası:** [📥 LisanslaBeni.exe](https://github.com/cihangungor/pitoctf/blob/main/LisanslaBeni.exe)


---

## 🔍 Analiz

### İlk Bakış

Dosya adı **LisanslaBeni.exe** - muhtemelen bir lisans/serial key kontrol programı. .NET executable olduğu için decompile etmek kolay olacak.

**Strateji:** Önce dosya türünü belirle → IL kodunu çıkar → Serial key algoritmasını bul → Alicia için hesapla

---

## ✅ Çözüm Adımları

### 📥 **1. Dosyayı İndirme**

```bash
cd ~/Desktop
wget https://github.com/cihangungor/pitoctf/raw/main/LisanslaBeni.exe
```

---

### 🔬 **2. Dosya Türünü Belirleme**

```bash
file LisanslaBeni.exe
```

**Çıktı:**
```
LisanslaBeni.exe: PE32 executable for MS Windows 6.00 (GUI), Intel i386 Mono/.Net assembly, 3 sections
```

> 💡 **Kritik Bilgi:** Bu bir **.NET executable**! Kolayca decompile edilebilir. Wine ile çalıştırmaya gerek yok, direkt IL kodunu okuyabiliriz.

---

### 🛠️ **3. monodis ile IL Kodunu Çıkarma**

.NET uygulamalarını decompile etmek için **monodis** kullanıyoruz:

```bash
monodis --output=crackme_il.txt LisanslaBeni.exe
```

> Not: Kali'de monodis genellikle yüklüdür. Yoksa `sudo apt install mono-utils`

---

### 📖 **4. IL Kodunu İnceleme**

```bash
cat crackme_il.txt | less
```

**Veya productId değerlerini direkt ara:**

```bash
cat crackme_il.txt | grep -A 5 "productId"
```

---

### 🔎 **5. btnCheck_Click Metodunu Bulma**

IL kodunda **satır 108-175** arasında serial key kontrol algoritması var:

**İsim İşleme Kısmı:**
```assembly
IL_0000:  ldarg.0 
IL_0001:  ldfld class [System.Windows.Forms]System.Windows.Forms.TextBox CrackMe18.Form1::txtName
IL_0006:  callvirt instance string class [System.Windows.Forms]System.Windows.Forms.Control::get_Text()
IL_000b:  ldc.i4.0 
IL_000c:  ldc.i4.4 
IL_000d:  callvirt instance string string::Substring(int32, int32)
IL_0012:  callvirt instance string string::ToUpper()
IL_0017:  stloc.0
```

**Ne Yapıyor:**
1. Kullanıcı adını text box'tan al
2. İlk 4 karakterini al (`.Substring(0, 4)`)
3. Büyük harfe çevir (`.ToUpper()`)

**Serial Key Oluşturma:**
```assembly
IL_0066:  ldloc.2           // productId1'i yükle
IL_0067:  ldloc.0           // İsmin ilk 4 harfi (UPPERCASE)
IL_0068:  ldloc.3           // productId2'yi yükle
IL_0069:  ldloc.s 4         // productId3'ü yükle
IL_006b:  call string string::Concat(string, string, string, string)
IL_0070:  stloc.s 5         // Sonuç = doğru serial key
IL_0072:  ldloc.1           // Kullanıcının girdiği serial key
IL_0075:  callvirt instance bool string::Equals(string)  // Karşılaştır
```

**🎯 Serial Key Algoritması Bulundu:**
```
Serial Key = productId1 + İsmin İlk 4 Harfi (UPPERCASE) + productId2 + productId3
```

---

### 🔢 **6. ProductId Değerlerini Bulma**

Algoritma **productId1, productId2, productId3** label'larının Text değerlerini kullanıyor. Bu değerleri bulmak için **InitializeComponent** metodunda **set_Text** çağrılarını arıyoruz.

**Adım 1: Tüm set_Text çağrılarını listele**

```bash
cat crackme_il.txt | grep -B 2 "set_Text" | less
```

Bu komut bize `set_Text` çağrılarından 2 satır öncesini de gösterir. Hemen önünde `ldstr` ile yüklenen string değerini görürüz.

**Adım 2: productId label'larını bul**

```bash
cat crackme_il.txt | grep -n "productId[1-3]"
```

**Çıktı:**
```
82:    .field  private  class [System.Windows.Forms]System.Windows.Forms.Label productId1
83:    .field  private  class [System.Windows.Forms]System.Windows.Forms.Label productId2
84:    .field  private  class [System.Windows.Forms]System.Windows.Forms.Label productId3
...
```

**Adım 3: Her productId'nin Text atamasını manuel incele**

```bash
cat crackme_il.txt | sed -n '369,380p'
```

**productId1'in Text değeri (satır 377):**
```assembly
IL_01a3:  ldstr "productId1"
IL_01a8:  callvirt ... set_Name(string)
...
IL_01d3:  ldstr "X398"
IL_01d8:  callvirt instance void class [System.Windows.Forms]System.Windows.Forms.Control::set_Text(string)
```

**Aynı şekilde productId2 ve productId3 için:**

```bash
# productId2 için
cat crackme_il.txt | sed -n '580,595p' | grep -B 2 "set_Text"

# productId3 için
cat crackme_il.txt | sed -n '640,655p' | grep -B 2 "set_Text"
```

**📊 Bulunan Değerler:**

| Label | Satır | Text Değeri |
|-------|-------|-------------|
| productId1 | 377 | `X398` |
| productId2 | 588 | `33CE` |
| productId3 | 645 | `A639` |

> Not: productId4 (`34FE`) da var ama algoritma sadece ilk 3'ünü kullanıyor.

---

### 🧮 **7. "alicia" için Serial Key Hesaplama**

**Algoritma:**
```
Serial Key = productId1 + İsmin İlk 4 Harfi (UPPERCASE) + productId2 + productId3
```

**"alicia" kullanıcısı için:**

1. `productId1` = **X398**
2. İsmin ilk 4 harfi = **"alic"** → `.ToUpper()` → **"ALIC"**
3. `productId2` = **33CE**
4. `productId3` = **A639**

**Terminal'de hesaplayalım:**

```bash
python3 -c "print('X398' + 'ALIC' + '33CE' + 'A639')"
```

**Çıktı:**
```
X398ALIC33CEA639
```

---

### ✅ **8. Flag Formatına Çevirme**

Flag formatı: `Flag{alicia'nın Serial Keyi}`

**Serial Key:** `X398ALIC33CEA639`

---

## 🚩 **FLAG**

```
Flag{X398ALIC33CEA639}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>file</b><br><sub>Dosya türü belirleme</sub></td>
<td align="center">🔓<br><b>monodis</b><br><sub>.NET IL decompiler</sub></td>
<td align="center">📝<br><b>grep</b><br><sub>String arama</sub></td>
<td align="center">🐍<br><b>Python3</b><br><sub>Serial key hesaplama</sub></td>
</tr>
</table>

**Terminalde Kullanılan Komutlar:**
```bash
# Dosya türünü kontrol et
file LisanslaBeni.exe

# IL kodunu çıkar
monodis --output=crackme_il.txt LisanslaBeni.exe

# IL kodunu incele ve algoritmayi bul
cat crackme_il.txt | less

# productId label'larını bul
cat crackme_il.txt | grep -n "productId[1-3]"

# set_Text çağrılarını incele
cat crackme_il.txt | grep -B 2 "set_Text" | less

# Belirli satırları kontrol et (productId değerleri için)
cat crackme_il.txt | sed -n '369,380p'
cat crackme_il.txt | sed -n '580,595p'
cat crackme_il.txt | sed -n '640,655p'

# Serial key'i hesapla
python3 -c "print('X398' + 'ALIC' + '33CE' + 'A639')"
```

---

## 💭 **Çözüm Akış Şeması**

```
📥 Challenge Dosyası: LisanslaBeni.exe
            ↓
🔍 Dosya Türü Kontrolü: file komutu
            ↓
💡 .NET Assembly tespit edildi
            ↓
🛠️ monodis ile IL Kodunu Çıkar
            ↓
📖 btnCheck_Click Metodunu İncele
            ↓
🔎 Serial Key Algoritmasını Bul:
   Serial = productId1 + Name[0:4].ToUpper() + productId2 + productId3
            ↓
🔢 IL Kodunda set_Text Çağrılarını Ara
            ↓
📝 ProductId Değerlerini Topla:
   productId1 = X398
   productId2 = 33CE
   productId3 = A639
            ↓
🧮 "alicia" için Hesapla:
   X398 + ALIC + 33CE + A639
            ↓
✅ Serial Key: X398ALIC33CEA639
            ↓
🚩 FLAG: Flag{X398ALIC33CEA639}
```
