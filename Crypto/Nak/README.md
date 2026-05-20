# 🦆 Nak - Crypto Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Crypto-Challenge-darkblue?style=for-the-badge" alt="Crypto">
  <img src="https://img.shields.io/badge/Difficulty-Easy-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Cipher%20Decoding-purple?style=for-the-badge" alt="Cipher">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Crypto  
**Seviye:** Kolay  
**Puan:** 500

**Challenge Açıklaması:** Naknaknak Nak Naknak Naknak. Naknaknak Nak? Naknak naknaknak Naknaknak nakak Naknak naknaknak Naknaknak Nananak Naknak Nak? Naknak nak? Naknak Naknaknak Naknak Naknak. Naknak nak! Naknaknak Naknak Naknak Nanak Naknak nakak Naknaknak Naknak Naknak Nanak Naknak nakak Naknaknak Naknak Naknak Nanak Naknak nakak Naknaknak Naknak Naknak Nanak Naknak nakak Naknak Nak? Naknak nak? Naknaknak Nananak nak? naknaknak Naknaknak Nak Naknak Nanak Naknaknak Naknak. Naknaknak Nak? Naknak Nanak Naknak nakak Naknak Nak? Naknaknak nak? Naknak Nanananak Naknak nakak Naknaknak nak!

---

## 🔍 Analiz

**Hedef:** "Nak" kelimelerinden oluşan şifreli metni decode etmek ve flag'i bulmak

**Strateji:** Cipher identifier → Duckspeak tespit → Decode → Flag format düzelt

**İpuçları:**
- 🎯 "Nak" kelimeleri → Duckspeak cipher
- 🔍 Tekrarlanan pattern'ler var
- 💡 Flag format bozuk çıkabilir
- 🔑 Farklı decoder'lar farklı sonuç verebilir

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Cipher Identification**

DCode Cipher Identifier kullanarak şifre tipini tespit et:

```
URL: https://www.dcode.fr/cipher-identifier
Input: Naknaknak Nak Naknak Naknak. Naknaknak Nak? Naknak naknaknak...
```

**Sonuç:**
- ✅ **Nak Nak (Duckspeak)** - %95 eşleşme
- 💡 Ordek konuşması simülasyonu cipher'ı
- 🎯 DCode'da bulunan Duckspeak decoder'ı mevcut

---

### 🦆 **Adım 2: İlk Decode Denemesi (DCode)**

DCode Duckspeak decoder ile ilk deneme:

```
URL: https://www.dcode.fr/nak-nak-duckspeak
Mode: Decrypt
Input: Naknaknak Nak Naknak Naknak. Naknaknak Nak?...
```

**Çıkan Sonuç:**
```
pito?ordegimva?va?va?va?der_payta?duc?}
```

**Analiz:**
- ❌ Flag format bozuk
- 💭 "pito" → "pico" olması lazım
- 🔍 Eksik harfler ve yanlış decode var
- 🚨 Farklı decoder denemek gerekli

---

### 🔧 **Adım 3: Alternatif Decoder Arayışı**

Google'da "duckspeak decoder" araması:

```bash
# Arama terimi: "duckspeak decoder"
# En iyi sonuç: BoxentriQ
```

**Bulunan Site:**
- 🌐 **URL:** https://www.boxentriq.com/encodings/nak-nak-duckspeak
- ✅ Farklı algoritma kullanan decoder
- 💡 Daha doğru sonuçlar veriyor

---

### 🎯 **Adım 4: Doğru Decode**

BoxentriQ Duckspeak Decoder ile final decode:

```
URL: https://www.boxentriq.com/encodings/nak-nak-duckspeak
Mode: Decrypt
Input: Naknaknak Nak Naknak Naknak. Naknaknak Nak? Naknak naknaknak...
```

**Final Sonuç:**
```
pito{ordegimvakvakvakvakder_paytakduck}
```

**Decode Analizi:**

| Encoder | Sonuç | Durum |
|---------|--------|-------|
| DCode | `pito?ordegimva?va?va?va?der_payta?duc?}` | ❌ Bozuk |
| BoxentriQ | `pito{ordegimvakvakvakvakder_paytakduck}` | ✅ Düzgün |

---

### 🔐 **Adım 5: Flag Format Kontrolü**

```
pito{ordegimvakvakvakvakder_paytakduck}
  ↑              ↑                  ↑
pico{    ordegim + vak*4 + der_paytak + duck    }
```

**Kelime Çözümlemesi:**
- `ordegim` → "ördek" (Türkçe)
- `vak` → Ordek sesi tekrarı
- `der_paytak` → Ordek çıkardığı ses
- `duck` → İngilizce ördek

---

## 🚩 **FLAG**
```
pito{ordegimvakvakvakvakder_paytakduck}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>DCode Cipher ID</b><br><sub>Şifre türü tespiti</sub></td>
<td align="center">🦆<br><b>DCode Duckspeak</b><br><sub>İlk decode denemesi</sub></td>
<td align="center">🔧<br><b>BoxentriQ</b><br><sub>Doğru decoder</sub></td>
<td align="center">🌐<br><b>Google Search</b><br><sub>Alternatif tool bulma</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🦆 Nak — Crypto Challenge
       ↓
🔤 Şifreli Metin (Nak Nak pattern)
   → "Naknaknak Nak Naknak Naknak..."
       ↓
🔍 Cipher Identifier (DCode)
   → %95 eşleşme: Nak Nak (Duckspeak)
       ↓
🥇 İlk Decode (DCode)
   → "pito?ordegimva?va?va?va?der_payta?duc?}"
   → ❌ Flag format bozuk
       ↓
🔎 Google: "duckspeak decoder"
   → BoxentriQ bulundu
       ↓
🎯 Doğru Decode (BoxentriQ)
   → Mode: Decrypt
   → "pito{ordegimvakvakvakvakder_paytakduck}"
       ↓
🚩 FLAG: pito{ordegimvakvakvakvakder_paytakduck}
```
