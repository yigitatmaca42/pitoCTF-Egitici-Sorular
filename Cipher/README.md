# 🔐 Cipher - Crypto Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Crypto-Challenge-darkblue?style=for-the-badge&logo=lock" alt="Crypto">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Multi%20Layer%20Decoding-purple?style=for-the-badge&logo=layers" alt="Multi Layer">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Crypto  
**Seviye:** Kolay  
**Puan:** 100  
**Açıklama:** Şifrelenmiş bir metin var, çöz ve flag'i bul!

**Challenge Dosyası:** [📥 Google Drive - Cipher.txt](https://drive.google.com/file/d/1yCERax79WJhH6iO_h8NOx6lhv-Nq78oM/view?usp=drivesdk)

**Flag Formatı:** `Flag{çözülmüşMetin}`

---

## 🔍 Analiz

**Hedef:** Multi-layer encoding → Key/IV/Cipher extraction → DES Decryption

**Strateji:** Base64 decode → Hex decode → Base64 decode → Parse credentials → CyberChef DES-CBC decrypt

---

## ✅ Çözüm Adımları

### 📥 **1. Dosyayı İndirme**

Google Drive'dan `Cipher.txt` dosyasını indiriyoruz.

**İçerik:**

```
TmpFZ016SWdOVFlnTXpVZ05HWWdOVFFnTmpjZ016TWdOR1VnTm1FZ05UVWdNekFnTkdRZ04yRWdORGtnTnpnZ05HUWdORFFnTkRFZ056Z2dOR1FnTm1FZ05HUWdNekFnTmpFZ05UZ2dOVGtnTnpnZ05HUWdObUVnTkdRZ016QWdOR1VnTlRRZ05Ua2dNek1nTkdZZ05EUWdObUlnTnpnZ05HUWdObUVnTkdRZ016QWdOR1VnTlRRZ05XRWdObUVnTmpFZ05UZ2dORElnTm1ZZ05XRWdOVGdnTkRrZ016UWdOV0VnTm1FZ05URWdNelFnTldFZ05EY2dORFVnTXpNZ05XRWdObUVnTlRVZ056Y2dOR1FnTXpJZ05Ua2dOMkVnTldFZ05UY2dOV0VnTm1RZ05HUWdORGNnTkRVZ04yRWdOR1VnTjJFZ05XRWdObUVnTkdRZ04yRWdORGtnTnpnZ05HVWdObUVnTldFZ05tUWdOVGtnTm1FZ05UVWdNelVnTlRrZ04yRWdORFVnTXpBZ05HWWdOVFFnTlRFZ016TWdOR1lnTlRRZ05XRWdObVFnTkdRZ04yRWdOak1nTnpjZ05HWWdOVFFnTkRJZ05tTWdOVGtnTm1RZ05XRWdOamtnTkdVZ04yRWdORFVnTnpnZ05XRWdORFFnTkdVZ05qa2dOV0VnTm1RZ05Ua2dOMkVnTkdRZ05tRWdOVEVnTXpVZ05HVWdOVFFnTkRrZ016VWdOV0VnTm1RZ05XRWdObUVnTkdRZ05EUWdOVEVnTXpVZ05HVWdOMkVnTldFZ05qZ2dOVGtnTlRjZ05EVWdNekVnTlRrZ04yRWdOak1nTjJFZ05HUWdOamNnTTJRZ00yUT0=
```

---

**Adım Adım Terminalde Çözme:**

**Birinci Katman - Base64 Decode:**
```bash
echo "TmpFZ016..." | base64 -d
# Çıktı: NjEgMzIgNTYgMzUgNGYgNTQg...
```

**İkinci Katman - Base64 Decode:**
```bash
echo "NjEgMzIgNTYg..." | base64 -d
# Çıktı: 61 32 56 35 4f 54 67 33...
```

**Üçüncü Katman - Hex Decode:**
```bash
echo "61 32 56 35 4f 54 67..." | xxd -r -p
# Çıktı: a2V5OTg3NjU0MzIxMDAxMjM0...
```

**Dördüncü Katman - Base64 Decode:**
```bash
echo "a2V5OTg3..." | base64 -d
# Çıktı: key987654321001234iv1234...
```

**Final Çıktı:**

```
key987654321001234iv123456789123456cipher8f48da7f503f3eff0a376c32166fb59c1494796f37090ebfb711d3bff3249529ffc04976aaa5c732
```

> 🚀 **BINGO!** 3 katmanlı encoding'i çözdük ve Key, IV, Cipher parametrelerini elde ettik!

---

### 🔍 **3. Verileri Ayrıştırma**

Çıktıyı incelediğimizde **3 ayrı parametre** görüyoruz:

**Parametreler:**

| Parametre | Değer |
|-----------|-------|
| **key** | `987654321001234` |
| **iv** | `123456789123456` |
| **cipher** | `8f48da7f503f3eff0a376c32166fb59c1494796f37090ebfb711d3bff3249529ffc04976aaa5c732` |

> 💡 **Analiz:** Key ve IV var → Bu bir **block cipher** algoritması kullanıyor!

---

### 🔐 **4. Şifreleme Algoritmasını Belirleme**

**IV (Initialization Vector)** kullanan başlıca algoritmalar:

1. ✅ **AES-CBC** (Advanced Encryption Standard)
2. ✅ **DES-CBC** (Data Encryption Standard)
3. ✅ **3DES-CBC** (Triple DES)
4. ✅ **Blowfish-CBC**

> 🎯 **Strateji:** CyberChef'te sırayla deneyeceğiz!

---

### 🌐 **5. CyberChef ile Deneme - AES**

**Site:** https://gchq.github.io/CyberChef/

**İlk Deneme: AES Decrypt**

**Recipe:**

1. **Operation:** AES Decrypt
2. **Key:** `987654321001234` (Hex formatında)
3. **IV:** `123456789123456` (Hex formatında)
4. **Mode:** CBC
5. **Input:** `8f48da7f503f3eff0a376c32166fb59c1494796f37090ebfb711d3bff3249529ffc04976aaa5c732` (Hex)

**Sonuç:** ❌ **Çalışmadı!**

---

### 🔓 **6. CyberChef ile Deneme - DES**

**İkinci Deneme: DES Decrypt**

**Recipe:**

1. **Operation:** DES Decrypt
2. **Key:** `987654321001234` (Hex formatında)
3. **IV:** `123456789123456` (Hex formatında)
4. **Mode:** CBC
5. **Input:** `8f48da7f503f3eff0a376c32166fb59c1494796f37090ebfb711d3bff3249529ffc04976aaa5c732` (Hex)
6. **Output:** Raw

**Sonuç:** ✅ **BAŞARILI!**

**Decrypt Edilen Metin:**

```
dessifrelemeguvensizderlerdidogruymus
```

> 🎉 **FLAG BULUNDU!**

---

## 🚩 **FLAG**

```
Flag{dessifrelemeguvensizderlerdidogruymus}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔓<br><b>Base64</b><br><sub>Multi-layer decode</sub></td>
<td align="center">🔢<br><b>xxd</b><br><sub>Hex decoder</sub></td>
<td align="center">🌐<br><b>CyberChef</b><br><sub>DES decryption</sub></td>
</tr>
</table>

**Kullanılan Siteler:**
- 🔓 **Terminal:** Base64 ve Hex decode komutları
- 🌐 **CyberChef:** https://gchq.github.io/CyberChef/
- 📥 **Challenge File:** Google Drive

---

## 💻 **Kullanılan Komutlar**

```bash
# 1. İlk Base64 decode
echo "TmpFZ016SWdOVFlnTXpVZ05HWWdOVFFnTmpjZ016TWdOR1VnTm1FZ05UVWdNekFnTkdRZ04yRWdORGtnTnpnZ05HUWdORFFnTkRFZ056Z2dOR1FnTm1FZ05HUWdNekFnTmpFZ05UZ2dOVGtnTnpnZ05HUWdObUVnTkdRZ016QWdOR1VnTlRRZ05Ua2dNek1nTkdZZ05EUWdObUlnTnpnZ05HUWdObUVnTkdRZ016QWdOR1VnTlRRZ05XRWdObUVnTmpFZ05UZ2dORElnTm1ZZ05XRWdOVGdnTkRrZ016UWdOV0VnTm1FZ05URWdNelFnTldFZ05EY2dORFVnTXpNZ05XRWdObUVnTlRVZ056Y2dOR1FnTXpJZ05Ua2dOMkVnTldFZ05UY2dOV0VnTm1RZ05HUWdORGNnTkRVZ04yRWdOR1VnTjJFZ05XRWdObUVnTkdRZ04yRWdORGtnTnpnZ05HVWdObUVnTldFZ05tUWdOVGtnTm1FZ05UVWdNelVnTlRrZ04yRWdORFVnTXpBZ05HWWdOVFFnTlRFZ016TWdOR1lnTlRRZ05XRWdObVFnTkdRZ04yRWdOak1nTnpjZ05HWWdOVFFnTkRJZ05tTWdOVGtnTm1RZ05XRWdOamtnTkdVZ04yRWdORFVnTnpnZ05XRWdORFFnTkdVZ05qa2dOV0VnTm1RZ05Ua2dOMkVnTkdRZ05tRWdOVEVnTXpVZ05HVWdOVFFnTkRrZ016VWdOV0VnTm1RZ05XRWdObUVnTkdRZ05EUWdOVEVnTXpVZ05HVWdOMkVnTldFZ05qZ2dOVGtnTlRjZ05EVWdNekVnTlRrZ04yRWdOak1nTjJFZ05HUWdOamNnTTJRZ00yUT0=" | base64 -d

# 2. İkinci Base64 decode
echo "NjEgMzIgNTYgMzUgNGYgNTQgNjcgMzMgNGUgNmEgNTUgMzAgNGQgN2EgNDkgNzggNGQgNDQgNDEgNzggNGQgNmEgNGQgMzAgNjEgNTggNTkgNzggNGQgNmEgNGQgMzAgNGUgNTQgNTkgMzMgNGYgNDQgNmIgNzggNGQgNmEgNGQgMzAgNGUgNTQgNWEgNmEgNjEgNTggNDIgNmYgNWEgNTggNDkgMzQgNWEgNmEgNTEgMzQgNWEgNDcgNDUgMzMgNWEgNmEgNTUgNzcgNGQgMzIgNTkgN2EgNWEgNTcgNWEgNmQgNGQgNDcgNDUgN2EgNGUgN2EgNWEgNmEgNGQgN2EgNDkgNzggNGUgNmEgNWEgNmQgNTkgNmEgNTUgMzUgNTkgN2EgNDUgMzAgNGYgNTQgNTEgMzMgNGYgNTQgNWEgNmQgNGQgN2EgNjMgNzcgNGYgNTQgNDIgNmMgNTkgNmQgNWEgNjkgNGUgN2EgNDUgNzggNWEgNDQgNGUgNjkgNWEgNmQgNTkgN2EgNGQgNmEgNTEgMzUgNGUgNTQgNDkgMzUgNWEgNmQgNWEgNmEgNGQgNDQgNTEgMzUgNGUgN2EgNWEgNjggNTkgNTcgNDUgMzEgNTkgN2EgNjMgN2EgNGQgNjcgM2QgM2Q=" | base64 -d

# 3. Hex decode
echo "61 32 56 35 4f 54 67 33 4e 6a 55 30 4d 7a 49 78 4d 44 41 78 4d 6a 4d 30 61 58 59 78 4d 6a 4d 30 4e 54 59 33 4f 44 6b 78 4d 6a 4d 30 4e 54 5a 6a 61 58 42 6f 5a 58 49 34 5a 6a 51 34 5a 47 45 33 5a 6a 55 77 4d 32 59 7a 5a 57 5a 6d 4d 47 45 7a 4e 7a 5a 6a 4d 7a 49 78 4e 6a 5a 6d 59 6a 55 35 59 7a 45 30 4f 54 51 33 4f 54 5a 6d 4d 7a 63 77 4f 54 42 6c 59 6d 5a 69 4e 7a 45 78 5a 44 4e 69 5a 6d 59 7a 4d 6a 51 35 4e 54 49 35 5a 6d 5a 6a 4d 44 51 35 4e 7a 5a 68 59 57 45 31 59 7a 63 7a 4d 67 3d 3d" | xxd -r -p

# 4. Üçüncü Base64 decode
echo "a2V5OTg3NjU0MzIxMDAxMjM0aXYxMjM0NTY3ODkxMjM0NTZjaXBoZXI4ZjQ4ZGE3ZjUwM2YzZWZmMGEzNzZjMzIxNjZmYjU5YzE0OTQ3OTZmMzcwOTBlYmZiNzExZDNiZmYzMjQ5NTI5ZmZjMDQ5NzZhYWE1YzczMg==" | base64 -d

---

## 💭 **Çözüm Akış Şeması**

```
🔐 Crypto Challenge: "Cipher"
                    ↓
        📥 Cipher.txt dosyasını indir
                    ↓
        🔓 1. Katman: Base64 Decode
                    ↓
        🔢 2. Katman: Hex String bulundu
                    ↓
        🔓 2. Adım: Hex Decode (xxd -r -p)
                    ↓
        🔓 3. Katman: Base64 Decode
                    ↓
        📝 Key, IV, Cipher parametreleri çıktı
                    ↓
        key: 987654321001234
        iv:  123456789123456
        cipher: 8f48da7f503f3eff0a376c32166fb59c1494796f37090ebfb711d3bff3249529ffc04976aaa5c732
                    ↓
        🌐 CyberChef'e Git
                    ↓
        ❌ AES Decrypt → Çalışmadı
                    ↓
        ✅ DES Decrypt → BAŞARILI!
                    ↓
        📝 Mode: CBC, Input: Hex, Output: Raw
                    ↓
        🔓 Decrypted: dessifrelemeguvensizderlerdidogruymus
                    ↓
        📋 Flag Formatına Çevir
                    ↓
        🚩 Flag{dessifrelemeguvensizderlerdidogruymus}
```
