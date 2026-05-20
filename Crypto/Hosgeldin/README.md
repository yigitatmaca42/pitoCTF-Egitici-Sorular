# 🎯 Hosgeldin - Crypto Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Crypto-Challenge-darkblue?style=for-the-badge" alt="Crypto">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-green?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Chained Encoding-purple?style=for-the-badge" alt="Chained Encoding">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Crypto  
**Seviye:** Kolay  
**Puan:** 50

**Şifreli Metin:**
```
JFsopokvvz8JgQKAHEf3VCV6pZtzSQyJRmBrJdjcsG3xqFK47m5eAPCsDMmqc2TQPfoDHcFBEdTd6R2GD1LrLm5vktvvcJPURjK99PSt5pHPmQXYTLqB3FdtfHKH9j2EKXzgxaeuUfdKbP3qhvD76vkTKM8iK7yPNua6RiZMR2hRzKXDqXPjCwwiVb9ZVfNmQFMqFaeUiU7t83iZnxF3opvVSZUno7TKkG2d5DyCYVhhV5Ze9n23vAq78BmQFBAtMA4ue9fiFZdt8RCaUS32PphaPVnnTKNPd6rcgfwBiFWfCAiBckKZaTSv4d
```

---

## 🔍 Analiz

**Hedef:** Zincirleme encoding katmanlarını manuel olarak çözerek flag'e ulaşmak

**Strateji:** Encoding tespiti → Adım adım decode → Flag extraction

**İpuçları:**
- 🎯 "Hoşgeldin" → Giriş seviyesi, temel encoding
- 🔍 Sadece alfanumerik karakterler var → Base58 veya Base64 olabilir
- 💡 Base64'te `+`, `/`, `=` karakterleri bulunur → Bunlar yok
- 🔑 50 puan = Kolay → Zincirleme encoding

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Encoding Tespiti**

Şifreli metni incelediğimizde:
- ✅ Büyük harf, küçük harf, rakam var
- ❌ `+`, `/`, `=` gibi Base64'e özgü karakterler **yok**

Bu durumda akla ilk gelen **Base58** encoding. Base58; Bitcoin adreslerinde kullanılan, karıştırılabilecek karakterleri (`0`, `O`, `I`, `l`) dışarıda bırakan bir alfanumerik encoding yöntemidir.

---

### 🔓 **Adım 2: Base58 → Layer 1**

```bash
echo "JFsopokvvz8JgQKAHEf3VCV6pZtzSQyJRmBrJdjcsG3xqFK47m5eAPCsDMmqc2TQ
PfoDHcFBEdTd6R2GD1LrLm5vktvvcJPURjK99PSt5pHPmQXYTLqB3FdtfHKH9j2EKXzgx
aeuUfdKbP3qhvD76vkTKM8iK7yPNua6RiZMR2hRzKXDqXPjCwwiVb9ZVfNmQFMqFaeUiU
7t83iZnxF3opvVSZUno7TKkG2d5DyCYVhhV5Ze9n23vAq78BmQFBAtMA4ue9fiFZdt8RCa
US32PphaPVnnTKNPd6rcgfwBiFWfCAiBckKZaTSv4d" | base58 -d
```

**Çıktı:**
```
2v4xchhoHwfChwQHmZ3vBECjg8v68rNjYDBR7UR7orot4CiLrDRx2Qni5J3xJyVMZgZhoNj1iPfvGLpVD2n9wxR75ZEZEUHUjinwWCcKuyvUsUsb29LFu6rUVHWPiRi6MsNaXYksqx6mTwPnQ4QajwqmUJZ6p3XqyEVLCPzVejoqccJ86ZtvkYr5Sreq5NMQhiToocYhLh1zqBb1NYzg9hhX6LNwDLtetAwxbn
```

**Analiz:** Yine alfanumerik → Tekrar **Base58**!

---

### 🔓 **Adım 3: Base58 → Layer 2**

```bash
echo "2v4xchhoHwfChwQHmZ3vBECjg8v68rNjYDBR7UR7orot4CiLrDRx2Qni5J3xJyVM
ZgZhoNj1iPfvGLpVD2n9wxR75ZEZEUHUjinwWCcKuyvUsUsb29LFu6rUVHWPiRi6MsNaXY
ksqx6mTwPnQ4QajwqmUJZ6p3XqyEVLCPzVejoqccJ86ZtvkYr5Sreq5NMQhiToocYhLh1z
qBb1NYzg9hhX6LNwDLtetAwxbn" | base58 -d
```

**Çıktı:**
```
UTNCV1JIaE1hVUZHWWswNWIxaEhTMWxyUVRscmRIZFhRWGxOVG05aFdraFNja28wUVVwdFZtRnhha1JHWjNoTFRUSnpVSGRYWjNWQmVuZHlhRzVWVEdoSGJsYzROMUJOWW5WaFprSnZSMWhuYTBRM1ozVkdWRW80VEdGdw==
```

**Analiz:** Sonda `==` padding var → **Base64**!

---

### 🔓 **Adım 4: Base64 → Layer 3**

```bash
echo "UTNCV1JIaE1hVUZHWWswNWIxaEhTMWxyUVRscmRIZFhRWGxOVG05aFdraFNja28w
UVVwdFZtRnhha1JHWjNoTFRUSnpVSGRYWjNWQmVuZHlhRzVWVEdoSGJsYzROMUJOWW5WaF
prSnZSMWhuYTBRM1ozVkdWRW80VEdGdw==" | base64 -d
```

**Çıktı:**
```
Q3BWRHhMaUFGYk05b1hHS1lrQTlrdHdXQXlNTm9hWkhScko0QUptVmFxakRGZ3hLTTJzUHdXZ3VBendyaG5VTGhHblc4N1BNYnVhZkJvR1hna0Q3Z3VGVEo4TGFw
```

**Analiz:** Yine Base64'e benziyor ama `=` yok → **Base64** tekrar!

---

### 🔓 **Adım 5: Base64 → Layer 4**

```bash
echo "Q3BWRHhMaUFGYk05b1hHS1lrQTlrdHdXQXlNTm9hWkhScko0QUptVmFxakRGZ3hL
TTJzUHdXZ3VBendyaG5VTGhHblc4N1BNYnVhZkJvR1hna0Q3Z3VGVEo4TGFw" | base64 -d
```

**Çıktı:**
```
CpVDxLiAFbM9oXGKYkA9ktwWAyMNoaZHRrJ4AJmVaqjDFgxKM2sPwWguAzwrhnULhGnW87PMbuafBoGXgkD7guFTJ8Lap
```

**Analiz:** Özel karakterler var ama Base64 değil (invalid). **Base58** dene!

---

### 🔓 **Adım 6: Base58 → Layer 5**

```bash
echo "CpVDxLiAFbM9oXGKYkA9ktwWAyMNoaZHRrJ4AJmVaqjDFgxKM2sPwWguAzwrhnUL
hGnW87PMbuafBoGXgkD7guFTJ8Lap" | base58 -d
```

**Çıktı:**
```
ZVo5MTV5WWZLaEJCWU5RUVBmTWNrMUtRVmQ3Y1ZaNkRnS0hWV1JoWmM3V2dlYkJvaQ==
```

**Analiz:** `==` padding → **Base64**!

---

### 🔓 **Adım 7: Base64 → Layer 6**

```bash
echo "ZVo5MTV5WWZLaEJCWU5RUVBmTWNrMUtRVmQ3Y1ZaNkRnS0hWV1JoWmM3V2dlYkJv
aQ==" | base64 -d
```

**Çıktı:**
```
eZ915yYfKhBBYNQQPfMck1KQVd7cVZ6DgKHVWRhZc7WgebBoi
```

**Analiz:** Base64 invalid → **Base58** dene!

---

### 🔓 **Adım 8: Base58 → Layer 7**

```bash
echo "eZ915yYfKhBBYNQQPfMck1KQVd7cVZ6DgKHVWRhZc7WgebBoi" | base58 -d
```

**Çıktı:**
```
UElUT3tnZWxnZWxheWFnaW5hbGlzc2lufQ==
```

**Analiz:** `==` padding → Son bir **Base64**!

---

### 🚩 **Adım 9: Base64 → FLAG!**

```bash
echo "UElUT3tnZWxnZWxheWFnaW5hbGlzc2lufQ==" | base64 -d
```

**Çıktı:**
```
PITO{gelgelayaginalissin}
```

**ANALİZ:**
- ✅ **FLAG BULUNDU!** 🎉🎉🎉
- 🚩 **Flag:** `PITO{gelgelayaginalissin}`

---

## 🚩 **FLAG**
```
PITO{gelgelayaginalissin}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔢<br><b>base58</b><br><sub>Base58 decode</sub></td>
<td align="center">🔡<br><b>base64</b><br><sub>Base64 decode</sub></td>
<td align="center">🐚<br><b>bash</b><br><sub>Command line</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 Hoşgeldin Challenge
       ↓
🔍 Encoding tespiti
   → Sadece alfanumerik → Base64 değil
   → Base58 olabilir!
       ↓
Layer 0 → Base58 decode
   → 2v4xchho...Awxbn
       ↓
Layer 1 → Base58 decode
   → UTNCV1JI...Gdw==  (== padding → Base64!)
       ↓
Layer 2 → Base64 decode
   → Q3BWRHhM...TGFw
       ↓
Layer 3 → Base64 decode
   → CpVDxLiA...Lap  (Base64 invalid → Base58!)
       ↓
Layer 4 → Base58 decode
   → ZVo5MTV5...aQ==  (== padding → Base64!)
       ↓
Layer 5 → Base64 decode
   → eZ915yYf...Boi  (Base64 invalid → Base58!)
       ↓
Layer 6 → Base58 decode
   → UElUT3tn...fQ==  (== padding → Base64!)
       ↓
Layer 7 → Base64 decode
   → PITO{gelgelayaginalissin}
       ↓
✅ FLAG: PITO{gelgelayaginalissin}
```
