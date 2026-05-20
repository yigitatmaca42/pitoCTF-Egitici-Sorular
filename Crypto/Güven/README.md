# 🎯 Güven - Crypto Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Crypto-Challenge-darkblue?style=for-the-badge" alt="Crypto">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-RSA Wiener-purple?style=for-the-badge" alt="RSA Wiener">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Crypto  
**Seviye:** Orta  
**Puan:** 500

**Challenge Açıklaması:** Bence bu şifreleme çok güvenli oldu.

**Verilen Değerler:**
```
n = 689061037339483636851744871564868379980061151991904073814057216873412583484720768694905841053416938972235588548525570270575285633894975913717130070544407480547826227398039831409929129742007101671851757453656032161443946817685708282221883187089692065998793742064551244403369599965441075497085384181772038720949

e = 98161001623245946455371459972270637048947096740867123960987426843075734419854169415217693040603943985614577854750928453684840929755254248201161248375350238628917413291201125030514500977409961838501076015838508082749034318410808298025858181711613372870289482890074072555265382600388541381732534018133370862587
```

**Challenge Dosyası:** [📥 Google Drive - cokguvenli](https://drive.google.com/file/d/1gIlMncHZNJW-tYsxGYixdfvUqSpaD4aS/view?usp=drivesdk)

---

## 🔍 Analiz

**Hedef:** RSA ile şifrelenmiş dosyayı çözmek

**Strateji:** Dosya içeriği → e analizi → Wiener's Attack tespiti → n faktorizasyonu → Hex dönüşümü → dcode.fr ile decrypt

**İpuçları:**
- 🎯 `e` değeri standart değil, `n` ile neredeyse aynı büyüklükte
- 🔍 Büyük `e` → küçük `d` → **Wiener's Attack** zafiyeti
- 💡 `n` factordb.com'da daha önce faktörize edilmiş
- 🔑 p ve q bulununca RSA decrypt edilebilir

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Şifreli Dosyayı İncele**

Drive linkinden `cokguvenli` dosyası indirildi. Terminal'de incelendi:

```bash
cat cokguvenli
```

**Çıktı:**
```
tʁ���Q����gb7]]�����G]~U[7�%̍9�zV��A[��]�g*��e�<...
```

```bash
file cokguvenli
```

**Çıktı:**
```
cokguvenli: data
```

**Analiz:**
- ✅ Dosya binary formatta, okunabilir değil
- 💡 RSA ile şifrelenmiş ciphertext (c değeri) olduğu anlaşıldı
- 🎯 Önce `e` değerini analiz etmek gerekiyor

---

### 🧠 **Adım 2: e Değerini Analiz Et**

Verilen `e` değerine bakıldı:

```
e = 98161001623245946455371459972270637048...862587
```

**Kritik Bulgular:**
```
n → 309 basamak (≈1026 bit)
e → 309 basamak (≈1024 bit)
```

**Analiz:**
- ✅ Standart RSA'da `e = 65537` kullanılır (küçük ve sabit)
- 💡 Bu `e` değeri `n` ile neredeyse aynı büyüklükte → **alışılmadık!**
- 🎯 `e` büyük olunca `d` küçük kalmak zorunda → **Wiener's Attack** uygulanabilir

---

### 🔑 **Adım 3: dcode.fr ile Wiener's Attack Dene**

**dcode.fr/rsa-cipher** sitesine gidildi.

- **E** kutusuna `e` değeri girildi
- **N** kutusuna `n` değeri girildi
- **CALCULATE/DECRYPT** butonuna basıldı

**Çıktı:**
```
✅ P,Q computed with N,E (Wiener's attack)
✅ D computed with P,Q,E

p = 23382692671581814681147482973935631627...383831
q = 29468848905367295280655884817633431891...679379
d = 653994415707479966122460879586443902420890730143890608823697555250579490695 03
```

**Analiz:**
- ✅ Wiener's Attack başarılı! p ve q otomatik bulundu
- 💡 Ama C (ciphertext) değeri henüz girilmedi → dosyadan çıkarılması gerekiyor

---

### 🔢 **Adım 4: n'i factordb.com ile Doğrula**

Ek doğrulama için **factordb.com** sitesine gidildi, `n` değeri arama kutusuna yapıştırıldı:

```
http://factordb.com/index.php?query=689061037339...
```

**Çıktı:**
```
Status: FF (Fully Factored)
n = 2338269267...31  ·  2946884890...79
```

Her faktörün üzerine tıklanarak tam değerler alındı:

```
p = 23382692671581814681147482973935631627665647364166311968381601793328093890106781554729959383140453326622761892739721372265697969595635573251067121224383831

q = 29468848905367295280655884817633431891433303833944997890434617505601421238696194085380181344342650644888169861836368327919791052354262892280583075696679379
```

**Analiz:**
- ✅ Wiener's Attack ile bulunan p ve q değerleri doğrulandı
- 🎯 Şimdi ciphertext'i integer'a çevirmek gerekiyor

---

### 🔄 **Adım 5: Ciphertext'i Hex'e Çevir**

`cokguvenli` dosyasının içeriği hex olarak okundu:

```bash
xxd cokguvenli
```

**Hex Çıktısı:**
```
027401ca8188b58051f88ed8e4136762375d5d8dadf88fd1475d7e55125b379d
25cc8d39df7a56ebc0415bf1bf5d8a672a98e865b43cd9553af6cf975ad94647
55313d5309bb55f1dd307f9eb0c4a1ab3eaba4806d1f275230d53801becd4823
8c45cf4fd9052c42776f5a79c0fe63c3ae7092de1d26586209804ad6ab6f767ffc
```

---

### 🔄 **Adım 6: Hex → Integer Dönüşümü**

**rapidtables.com/convert/number/hex-to-decimal.html** sitesine gidildi.

Hex değeri (büyük harfle) yapıştırıldı:
```
027401CA8188B58051F88ED8E4136762375D5D8DADF88FD1475D7E55125B379D25CC8D39DF7A56EBC0415BF1BF5D8A672A98E865B43CD9553AF6CF975AD9464755313D5309BB55F1DD307F9EB0C4A1AB3EABA4806D1F275230D53801BECD48238C45CF4FD9052C42776F5A79C0FE63C3AE7092DE1D26586209804AD6AB6F767FFC
```

**Decimal Çıktısı:**
```
c = 441001510077083440712098978980133930415086107290453312932779721137710693129669898774537962879522006041519477907847531444975796042514212299155087533072902229706427765901890350700252954929903001909850453303487994374982644931473474420223319182460327997419996588889034403777436157228265528747769729921745312710652
```

---

### 🚀 **Adım 7: dcode.fr ile Decrypt Et**

**dcode.fr/rsa-cipher** sitesine geri dönüldü ve tüm değerler girildi:

| Alan | Değer |
|------|-------|
| **C** | `441001510077...652` |
| **E** | `98161001623...587` |
| **N** | `68906103733...949` |
| **P** | `23382692671...831` |
| **Q** | `29468848905...379` |

**Display** olarak **"Plaintext as Character string"** seçildi → **CALCULATE/DECRYPT** butonuna basıldı.

**Çıktı:**
```
Well done! Here is your flag: INTIGRITI{0r_n07_50_53cur3_m4yb3}
```

---

## 🚩 **FLAG**
```
INTIGRITI{0r_n07_50_53cur3_m4yb3}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔐<br><b>dcode.fr/rsa-cipher</b><br><sub>Wiener's Attack + Decrypt</sub></td>
<td align="center">🔢<br><b>factordb.com</b><br><sub>n faktorizasyonu doğrulama</sub></td>
<td align="center">🔄<br><b>rapidtables.com</b><br><sub>Hex → Integer dönüşümü</sub></td>
<td align="center">💻<br><b>xxd</b><br><sub>Binary → Hex okuma</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 Güven Crypto Challenge
       ↓
📄 cokguvenli dosyası (Drive'dan indirildi)
   → file → "data" (binary)
   → xxd → hex içerik görüldü
       ↓
🧠 e analizi
   → e ≈ 1024 bit (n ile aynı büyüklükte!)
   → Standart e = 65537 olmalıydı
   → Büyük e → küçük d → Wiener's Attack!
       ↓
🔐 dcode.fr/rsa-cipher
   → E ve N girildi
   → Wiener's Attack otomatik çalıştı
   → p ve q bulundu ✅
       ↓
🔢 factordb.com
   → n sorgulandı → FF (Fully Factored)
   → p ve q tam değerleri alındı ✅
       ↓
🔄 Hex → Integer
   → xxd ile dosya hex'e çevrildi
   → rapidtables.com ile decimal'e çevrildi
   → c değeri elde edildi ✅
       ↓
🔐 dcode.fr/rsa-cipher
   → C, E, N, P, Q girildi
   → CALCULATE/DECRYPT basıldı
       ↓
🚩 FLAG: INTIGRITI{0r_n07_50_53cur3_m4yb3}
```
