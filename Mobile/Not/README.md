# 🎯 Not - Mobile Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Mobile-Challenge-darkblue?style=for-the-badge" alt="Mobile">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-APK_Reverse-purple?style=for-the-badge" alt="APK Reverse">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Mobile  
**Seviye:** Kolay  
**Puan:** 200
 
**Challenge Dosyası** [📥 Google Drive - not.apk](https://drive.google.com/file/d/1K1TtfG_-GNWkeaYWGQH36kg8ImYNSe8-/view?usp=sharing)

---

## 🔍 Analiz

**Hedef:** APK içindeki korunan secret değerini okuyup flag'i elde etmek.

**Strateji:** APK inceleme → Content Provider keşfi → Kriptografik akışın anlaşılması → 4 haneli PIN brute-force → Flag

**İpuçları:**
- 🎯 Uygulama içinde `SecretDataProvider` isimli bir provider bulunuyor.
- 🔍 `content://com.mobilehackinglab.securenotes.secretprovider` URI'si kritik nokta.
- 💡 `assets/config.properties` içinde `encryptedSecret`, `salt`, `iv` ve `iterationCount` alanları yer alıyor.
- 🔑 PIN doğrulanmıyor; provider tarafında doğrudan denenebiliyor.

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: APK İçeriğini İnceleme**

İlk olarak APK'nın içeriğini açıp dikkat çeken dosyalara baktım.

```bash
file not.apk
unzip -l not.apk | head
unzip -p not.apk assets/config.properties
```

**Çıktı:**
```text
not.apk: Android package (APK), with APK Signing Block

encryptedSecret=bTjBHijMAVQX+CoyFbDPJXRUSHcTyzGaie3OgVqvK5w=
salt=m2UvPXkvte7fygEeMr0WUg==
iv=L15Je6YfY5owgIckR9R3DQ==
iterationCount=10000
```

**Analiz:**
- ✅ APK içinde `assets/config.properties` dosyası var.
- 💡 Buradaki alanlar doğrudan bir şifre çözme akışına işaret ediyor.
- 🎯 Sonraki adımda uygulamanın bu değerleri nasıl kullandığını anlamak gerekiyor.

---

### 🔬 **Adım 2: Kod/Yapı Analizi**

Statik incelemede `strings` çıktısında çok kritik sınıf ve sabitler görünüyor.

```bash
strings classes3.dex | grep -E "SecretDataProvider|PBKDF2|AES|secretprovider|pin="
```

**Çıktı:**
```text
com/mobilehackinglab/securenotes/SecretDataProvider
content://com.mobilehackinglab.securenotes.secretprovider
PBKDF2WithHmacSHA1
AES/CBC/PKCS5Padding
config.properties
decryptSecret
generateKeyFromPin
pin=
[ERROR: Incorrect PIN]
```

**Analiz:**
- ✅ Uygulama `SecretDataProvider` üzerinden secret veriyi döndürüyor.
- 💡 Şifreleme tarafında `PBKDF2WithHmacSHA1` ile anahtar türetilip `AES/CBC/PKCS5Padding` ile çözme yapılıyor.
- 🎯 Buradan sonra problem, doğru PIN'i bulup provider'a sorgu atmaya dönüyor.

---

### 🧠 **Adım 3: Zafiyeti Anlama**

Asıl problem, `SecretDataProvider`'ın dışarıya açık olması. Uygulama PIN'i kendi içinde güvenli şekilde doğrulamak yerine provider sorgusuna taşıyor. Böylece 4 haneli PIN alanı dışarıdan brute-force edilebiliyor.

Mantık şu şekilde özetlenebilir:

```text
Kullanıcı PIN girer
   ↓
MainActivity → content://com.mobilehackinglab.securenotes.secretprovider
   ↓
Provider, PIN ile key üretir
   ↓
config.properties içindeki encryptedSecret çözülürse secret döner
```

**Analiz:**
- ✅ Provider dış erişime açık olduğu için uygulama arayüzüne bağlı kalmadan sorgu yapılabiliyor.
- 💡 PIN sadece 4 haneli olduğundan deneme alanı çok küçük: `0000-9999`.
- 🎯 En mantıklı çözüm brute-force.

---

### ⚙️ **Adım 4: PIN Brute-Force**

Bu aşamada provider'a tüm 4 haneli PIN'leri deneyen küçük bir script veya ADB tabanlı bir yaklaşım kullanılabilir.

Örnek yaklaşım:

```bash
adb shell content query \
  --uri content://com.mobilehackinglab.securenotes.secretprovider \
  --where "pin=2580"
```

**Çıktı:**
```text
Row: 0 Secret=CTF{D1d_y0u_gu3ss_1t!1?}
```

**Analiz:**
- ✅ Doğru PIN: `2580`
- 💡 Provider başarılı çözmede `Secret` alanını döndürüyor.
- 🎯 Böylece flag doğrudan elde ediliyor.

---

### 🚩 **Adım 5: Flag'i Elde Et**

Sonuç olarak brute-force ile doğru PIN bulununca provider secret veriyi döndürüyor.

```text
PIN  : 2580
FLAG : CTF{D1d_y0u_gu3ss_1t!1?}
```

---

## 🚩 **FLAG**
```text
CTF{D1d_y0u_gu3ss_1t!1?}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">📦<br><b>unzip</b><br><sub>APK içeriğini çıkarmak için</sub></td>
<td align="center">🔎<br><b>strings</b><br><sub>DEX içindeki kritik sabitleri görmek için</sub></td>
<td align="center">📱<br><b>adb</b><br><sub>Content Provider sorgulamak için</sub></td>
<td align="center">🐍<br><b>Python / brute-force</b><br><sub>PIN denemeleri için</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```text
🎯 Not
   ↓
🔎 APK içeriğini incele
   → assets/config.properties bulundu
   → encryptedSecret / salt / iv / iterationCount görüldü
   ↓
🔬 DEX sabitlerini incele
   → SecretDataProvider bulundu
   → PBKDF2WithHmacSHA1 + AES/CBC/PKCS5Padding tespit edildi
   ↓
🧠 Zafiyeti anla
   → Exported Content Provider
   → 4 haneli PIN dışarıdan brute-force edilebilir
   ↓
⚙️ PIN brute-force
   → Doğru PIN: 2580
   ↓
🚩 FLAG: CTF{D1d_y0u_gu3ss_1t!1?}
```
