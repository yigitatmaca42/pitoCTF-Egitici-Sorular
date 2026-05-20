# 🎯 Kolaysin - Blockchain Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Blockchain-Challenge-darkblue?style=for-the-badge" alt="Blockchain">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-ECDSA-purple?style=for-the-badge" alt="ECDSA">
</p>

---

## 📋 Soru Bilgileri
  
**Kategori:** Blockchain  
**Seviye:** Kolay  
**Puan:** 200

**Challenge Dosyası:** [📥 Google Drive - blockchain.txt](https://drive.google.com/file/d/1lo_4YEO-JRx9D5JbdwqM40ARUqKxBmKM/view?usp=drivesdk)

---

## 🔍 Analiz

**Hedef:** ECDSA nonce reuse saldırısı ile private key'i bulmak

**Strateji:** İki imzada aynı `r` değeri → Aynı nonce kullanılmış → Private key hesaplanabilir

**İpuçları:**
- 🎯 İki imzada `r` değeri aynı → Nonce reuse
- 🔍 Matematiksel formül ile private key hesaplanabilir
- 💡 Flag format: `Flag{privatekey}`

---

## ✅ Çözüm Adımları

### 📦 **Adım 1: Challenge Dosyasını İndirme**
```bash
cat blockchain.txt
```

**Çıktı:**
```
# ECDSA Signature Challenge
## Curve: secp256k1

Signature 1:
Message: "Transfer 1 BTC to Alice"
z: 0x1c061b1bb5897de74400a2f72de6b3849c4cf3c095948c11287f7b5946cc4b31
r: 0x5ffc3d812f59da0f09673a2a7ca492cafc6894dc7d56e12b8c898a798640177
s: 0x3f9f552cf65fd57649a03504ee9722418471ab9706026199f6b5ccbb30e3aa24

Signature 2:
Message: "Transfer 2 BTC to Bob"
z: 0x14bb5259a53e3d3fad6b90685c5c9a8deaf10b66882b74838f4b4280fd52e9e6
r: 0x5ffc3d812f59da0f09673a2a7ca492cafc6894dc7d56e12b8c898a798640177
s: 0x9b3496eaaa39fbf62410fb2b4517b50982d7a14c45805461d224bf829d11b023
```

**Analiz:**
- ⚠️ İki imzada `r` değeri aynı → Nonce reuse!
- ✅ `z` ve `s` değerleri farklı
- 💡 Private key geri hesaplanabilir

---

### 🔎 **Adım 2: ECDSA Nonce Reuse Attack**

**Matematiksel Formül:**
```
k = (z1 - z2) / (s1 - s2) mod n
private_key = (s1 × k - z1) / r mod n
```

**Python Script:**
```python
n = 0xfffffffffffffffffffffffffffffffebaaedce6af48a03bbfd25e8cd0364141
z1 = 0x1c061b1bb5897de74400a2f72de6b3849c4cf3c095948c11287f7b5946cc4b31
r = 0x5ffc3d812f59da0f09673a2a7ca492cafc6894dc7d56e12b8c898a798640177
s1 = 0x3f9f552cf65fd57649a03504ee9722418471ab9706026199f6b5ccbb30e3aa24
z2 = 0x14bb5259a53e3d3fad6b90685c5c9a8deaf10b66882b74838f4b4280fd52e9e6
s2 = 0x9b3496eaaa39fbf62410fb2b4517b50982d7a14c45805461d224bf829d11b023

s_diff = (s1 - s2) % n
z_diff = (z1 - z2) % n
k = (z_diff * pow(s_diff, -1, n)) % n
numerator = (s1 * k - z1) % n
private_key = (numerator * pow(r, -1, n)) % n

print(hex(private_key)[2:])
```

---

### 🚀 **Adım 3: Script Çalıştırma**
```bash
python3 << 'EOF'
n = 0xfffffffffffffffffffffffffffffffebaaedce6af48a03bbfd25e8cd0364141
z1 = 0x1c061b1bb5897de74400a2f72de6b3849c4cf3c095948c11287f7b5946cc4b31
r = 0x5ffc3d812f59da0f09673a2a7ca492cafc6894dc7d56e12b8c898a798640177
s1 = 0x3f9f552cf65fd57649a03504ee9722418471ab9706026199f6b5ccbb30e3aa24
z2 = 0x14bb5259a53e3d3fad6b90685c5c9a8deaf10b66882b74838f4b4280fd52e9e6
s2 = 0x9b3496eaaa39fbf62410fb2b4517b50982d7a14c45805461d224bf829d11b023

s_diff = (s1 - s2) % n
z_diff = (z1 - z2) % n
k = (z_diff * pow(s_diff, -1, n)) % n
numerator = (s1 * k - z1) % n
private_key = (numerator * pow(r, -1, n)) % n

print(hex(private_key)[2:])
EOF
```

**Çıktı:**
```
c3960bed0ce86704c7de9f7ba9acfe5534c1d7a3c8c0bff409ae094f4cd97fa9
```

---

## 🚩 **FLAG**
```
Flag{c3960bed0ce86704c7de9f7ba9acfe5534c1d7a3c8c0bff409ae094f4cd97fa9}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🐍<br><b>Python3</b><br><sub>Script execution</sub></td>
<td align="center">🔢<br><b>Modular Arithmetic</b><br><sub>Mathematical operations</sub></td>
<td align="center">🔐<br><b>ECDSA</b><br><sub>Cryptography analysis</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**
```
🎯 Kolaysin Challenge
       ↓
📦 blockchain.txt indir
   → 2 ECDSA imzası var
   → secp256k1 curve
       ↓
🔍 İmzaları analiz et
   → r değeri aynı!
   → Nonce reuse tespit edildi
       ↓
🧮 Matematiksel formül uygula
   → k = (z1-z2)/(s1-s2) mod n
   → pk = (s1×k-z1)/r mod n
       ↓
🐍 Python script çalıştır
   → Private key hesaplandı
       ↓
🚩 Flag{c3960bed0ce86704c7de9f7ba9acfe5534c1d7a3c8c0bff409ae094f4cd97fa9}
```

---

## 📚 **Öğrenilen Kavramlar**

- **ECDSA Nonce Reuse:** Aynı nonce iki farklı imzada kullanılırsa private key ele geçirilebilir
- **secp256k1:** Bitcoin'de kullanılan elliptic curve
- **Modular Arithmetic:** Kriptografide modüler aritmetik işlemler
- **Critical Security:** Asla aynı nonce'u tekrar kullanma!
