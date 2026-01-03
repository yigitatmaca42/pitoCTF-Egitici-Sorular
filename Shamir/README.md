# 🔐 Shamir - Cryptography Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Crypto-Challenge-darkblue?style=for-the-badge&logo=lock" alt="Crypto">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Secret%20Sharing-purple?style=for-the-badge&logo=key" alt="Secret Sharing">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Cryptography  
**Seviye:** Orta  
**Açıklama:** Shamir Secret Sharing ile parçalanmış şifreyi birleştir.

**Challenge Dosyası:** [📥 GitHub - Shamir.txt](https://github.com/yigitatmaca42/pitoCTF-Egitici-Sorular/blob/main/Shamir/Shamir.txt)

**Verilen Parçalar:**
- P1 (Share 1)
- P2 (Share 2)
- P3 (Share 3)

---

## 🔍 Analiz

### Shamir Secret Sharing Nedir?

**Shamir Secret Sharing**, bir sırrı birden fazla parçaya bölen ve bu parçaların belirli sayıda bir araya gelmesiyle sırrı yeniden oluşturan kriptografik bir yöntemdir.

| Özellik | Açıklama |
|---------|----------|
| 🔑 **Threshold** | Minimum kaç parça gerekli |
| 🧩 **Shares** | Toplam parça sayısı |
| 🔐 **Secret** | Gizli mesaj/flag |
| 📊 **Polynomial** | Matematiksel temel (Lagrange interpolasyonu) |

**Örnek:** (3,5) şeması = 5 parçadan en az 3'ü gerekli

---

## ✅ Çözüm Adımları

### 📥 **1. Parçaları Belirleme**

Elimizde 3 adet **hex string** var:

**P1:**
```
8010ba0d6ed38ef563074c3ee80a44f7fe680e82015a8d35f7f2245f66ec9c889b4e31a0c3e97bceeb6f28695f7a494918e0ca079677f07fff8eb570c17a4cb1db0477b84e9c68b9f02b21b33850f33bbd18f886b65c1f3bb015ddbe2723e64abfe8595e181d69d3f8ca3b7cc01c875ea25b97ef1e171c4f3f887e5752541270ae461cc610b3eb422c34df84e7b9a567f7933ee4b6969d19273d212a3ee92f8509679a4b40b6823c007e6d5c6241959e86bc8f989754649cd3008bdbb5bf030c9e802adf54d3afce4edef9bb709c7db4c2ac1f96f3e05cd220534b5647f35888e0e3d2435abdb1d7f32413bb630b3e8b0502e774dda8ac2bd4c2623ac433f79bd12
```

**P2:**
```
80264e325aa037314746964303cf6fee98d64e1e03d613fb8f327f5241850adbd06e1f959bdb6e5bd35874188e3fa4740a1948befcacb8949350574825ba4519793a6a617048fb2f5bdd9bc3267a61051484ec16e83ff7baaafac81a3aa4fb2077da312ee4f00c705b8f626334ff3045e41f451858988a3549e314f8a70f0879f5a30fbcd5fcc1645575186af8a434876304bb1ebc360533389143f7d918682307736bac713b63338482ef1cf80ac415f213625231ef3d3bdd70f811c8cc7515cf83a74ea25c31264a9a5dbe0615c5959e181bf8effa1698ece11cb5e9c794d381311ba1900f0c550f33b61fd49959d9b4ba73588b14906fddb625bd13f7149a95a
```

**P3:**
```
8036f43f3473b9c42441da7debc52b1966be409c028c9ece78c05b0d276996534b202e355832159538375c71d145ed3d12f982b96adb48eb6cdee238e4c009a8a23e1dd93ed49396abf6ba701e2a923ea99c14905e63e8811aef15a41d871d6ac8326870fced65a3a345591ff4e3b71b4644d2f7468f966a71bb698ff6cb198958d5105ac65f2c367a31c4fe1c0d97b09717867a09209e0a1cac64ede1c144d60854f7c7321de22f82ec8470991b57db729feb8aa0eb5ab7081070aa7b33755952238b81f5cf9dc80724a26575d9bba15ae4027e1f9a490acfd25183adb4ca1b62a2ca92c9a2bee2fa27a634b4b26402b298975c509c3f240f344037d1a4e44142b
```

> **Not:** Her parça `80` ile başlıyor - bu Shamir share formatının bir göstergesi.

---

### 🌐 **2. Online Tool Kullanımı**

Shamir Secret Sharing'i çözmek için **Ian Coleman'ın Shamir Tool**'unu kullanıyoruz.

**Site:** https://iancoleman.io/shamir/

**Adımlar:**

1. Siteye gidiyoruz
2. **"Combine Shares"** sekmesine tıklıyoruz
3. Her bir parçayı sırayla giriyoruz

---

### 🔓 **3. Parçaları Birleştirme**

**Share 1 Input:**
```
8010ba0d6ed38ef563074c3ee80a44f7fe680e82015a8d35f7f2245f66ec9c889b4e31a0c3e97bceeb6f28695f7a494918e0ca079677f07fff8eb570c17a4cb1db0477b84e9c68b9f02b21b33850f33bbd18f886b65c1f3bb015ddbe2723e64abfe8595e181d69d3f8ca3b7cc01c875ea25b97ef1e171c4f3f887e5752541270ae461cc610b3eb422c34df84e7b9a567f7933ee4b6969d19273d212a3ee92f8509679a4b40b6823c007e6d5c6241959e86bc8f989754649cd3008bdbb5bf030c9e802adf54d3afce4edef9bb709c7db4c2ac1f96f3e05cd220534b5647f35888e0e3d2435abdb1d7f32413bb630b3e8b0502e774dda8ac2bd4c2623ac433f79bd12
```

**Share 2 Input:**
```
80264e325aa037314746964303cf6fee98d64e1e03d613fb8f327f5241850adbd06e1f959bdb6e5bd35874188e3fa4740a1948befcacb8949350574825ba4519793a6a617048fb2f5bdd9bc3267a61051484ec16e83ff7baaafac81a3aa4fb2077da312ee4f00c705b8f626334ff3045e41f451858988a3549e314f8a70f0879f5a30fbcd5fcc1645575186af8a434876304bb1ebc360533389143f7d918682307736bac713b63338482ef1cf80ac415f213625231ef3d3bdd70f811c8cc7515cf83a74ea25c31264a9a5dbe0615c5959e181bf8effa1698ece11cb5e9c794d381311ba1900f0c550f33b61fd49959d9b4ba73588b14906fddb625bd13f7149a95a
```

**Share 3 Input:**
```
8036f43f3473b9c42441da7debc52b1966be409c028c9ece78c05b0d276996534b202e355832159538375c71d145ed3d12f982b96adb48eb6cdee238e4c009a8a23e1dd93ed49396abf6ba701e2a923ea99c14905e63e8811aef15a41d871d6ac8326870fced65a3a345591ff4e3b71b4644d2f7468f966a71bb698ff6cb198958d5105ac65f2c367a31c4fe1c0d97b09717867a09209e0a1cac64ede1c144d60854f7c7321de22f82ec8470991b57db729feb8aa0eb5ab7081070aa7b33755952238b81f5cf9dc80724a26575d9bba15ae4027e1f9a490acfd25183adb4ca1b62a2ca92c9a2bee2fa27a634b4b26402b298975c509c3f240f344037d1a4e44142b
```

> **Not:** Tüm 3 parçayı da giriyoruz. Minimum threshold karşılandığında secret otomatik olarak hesaplanacak.

---

### 🎯 **4. Secret Elde Etme**

Parçaları girdikten sonra **"Combine"** veya otomatik hesaplama ile sonuç çıkıyor.

**Recovered Secret (Hex):**
```
6374667b643662373235323963363137376438663634386165383566363234613234643666316564636535636132396264376363306238383865313137613132333839327d
```

---

### 📝 **5. Hex to ASCII Dönüşümü**

Elde ettiğimiz hex değerini ASCII'ye çeviriyoruz.

**Online:** https://www.rapidtables.com/convert/number/hex-to-ascii.html

veya

**Terminal:**
```bash
echo "6374667b643662373235323963363137376438663634386165383566363234613234643666316564636535636132396264376363306238383865313137613132333839327d" | xxd -r -p
```

**Çıktı:**
```
ctf{d6b72529c6177d8f648ae85f624a24d6f1edce5ca29bd7cc0b888e117a123892}
```

---

## 🚩 **FLAG**

```
ctf{d6b72529c6177d8f648ae85f624a24d6f1edce5ca29bd7cc0b888e117a123892}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔐<br><b>Shamir Tool</b><br><sub>Secret combining</sub></td>
<td align="center">🔓<br><b>Hex to ASCII</b><br><sub>Format dönüştürme</sub></td>
<td align="center">💻<br><b>xxd</b><br><sub>Terminal hex decoder</sub></td>
</tr>
</table>

**Kullanılan Siteler:**
- 🔐 **Shamir Secret Sharing:** https://iancoleman.io/shamir/
- 🔓 **Hex to ASCII:** https://www.rapidtables.com/convert/number/hex-to-ascii.html

---

## 💻 **Kullanılan Komutlar**

```bash
# Hex'i ASCII'ye çevirme
echo "6374667b643662373235323963363137376438663634386165383566363234613234643666316564636535636132396264376363306238383865313137613132333839327d" | xxd -r -p
```

---

## 💭 **Çözüm Akış Şeması**

```
🔐 Shamir Challenge: 3 Share verildi
                    ↓
        📥 P1, P2, P3 Parçalarını Belirleme
                    ↓
        🌐 Ian Coleman Shamir Tool'a Git
                    ↓
        📝 "Combine Shares" Sekmesi
                    ↓
        ✍️ Share 1, Share 2, Share 3'ü Gir
                    ↓
        🔓 Combine → Recovered Secret (Hex)
                    ↓
        6374667b643662373235...
                    ↓
        📝 Hex to ASCII Dönüşüm
                    ↓
        🚩 FLAG: ctf{d6b72529c6177d8f648ae85f624a24d6f1edce5ca29bd7cc0b888e117a123892}
```
