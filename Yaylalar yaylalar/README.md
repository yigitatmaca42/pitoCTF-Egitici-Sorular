# 🔍 Yaylalar Yaylalar - OSINT Challenge

<p align="center">
  <img src="https://img.shields.io/badge/OSINT-Challenge-darkblue?style=for-the-badge&logo=searchengineland" alt="OSINT">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Social%20Media-purple?style=for-the-badge&logo=twitter" alt="Category">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** OSINT  
**Seviye:** Orta  
**Açıklama:** Flag u.yaylacii hesabındaymış. Ama hesabını bulamadık. Sen bul flag senin olsun

**İpuçları:**
- Flag uçakla, biletle veya uçuşla ilgili herhangi bir yerde değil. Kişiye odaklan.
- İnstadan bulabileceğini buldun. Direksiyonu başka yöne çevir.

---

## 🔍 Analiz

### İlk Değerlendirme

Soruda verilen bilgileri analiz ediyoruz:

| İpucu | Detay |
|-------|-------|
| 👤 **Kullanıcı Adı** | `u.yaylacii` |
| 📱 **Platform Tipi** | Sosyal medya hesabı |
| 🎯 **Hedef** | Flag bu hesapta saklanmış |
| 🔍 **Strateji** | Çoklu platform araştırması gerekli |

---

## ✅ Çözüm Adımları

### 🔎 **1. Kullanıcı Adı Sorgulaması**

İlk olarak **namechk.com** sitesine gidip kullanıcı adını sorguluyoruz.

**Site:** https://namechk.com/

**Arama:** `u.yaylacii`

**Sonuç Gösterimi:**
- 🟢 **Yeşil** = Kullanıcı adı alınmamış (müsait)
- 🔴 **Kırmızı** = Kullanıcı adı alınmış (kayıtlı)

**Tespit:** X (Twitter) hesabı **kırmızı** (alınmış) ✅

---

### 🐦 **2. X (Twitter) Hesabı Bulma**

OSINT sorularında **X (Twitter)** sıkça kullanıldığı için ilk buraya bakıyoruz.

**Hesap:** `https://x.com/u.yaylacii`

**Hesapta Bulunanlar:**

| Tarih | İçerik | Ek |
|-------|--------|-----|
| 📅 **28 Mayıs** | "Bi şeyler var." | 📎 `kıbrıs_kikos.jpeg` (9.9 MB) |

✅ **Dosya indirildi**

---

### 🖼️ **3. Dosya Analizi - Steganografi**

İndirdiğimiz dosyayı inceleyince **steghide** ile veri gizlendiğini anlıyoruz.

**Test Komutu:**
```bash
steghide extract -sf kıbrıs_kikos.jpeg
```

**Çıktı:**
```
Enter passphrase: 
```

❌ **Sonuç:** Dosya **şifreli**! Parolaya ihtiyaç var.

---

### 📸 **4. Instagram'da Parola Araştırması**

İpucu: **"İnstadan bulabileceğini buldun. Direksiyonu başka yöne çevir"**

Bu ipucundan anlıyoruz ki parolayı Instagram'da bulacağız.

**Arama:** `u.yaylaci` 

⚠️ **DİKKAT:** Kullanıcı adının **son "i" harfi yok!**

**Hesap Bulundu:** `instagram.com/u.yaylaci`

**Hesap İçeriği:**
- 📷 **Galata Kulesi** fotoğrafı
- 💬 **Yorumlar:**
  - 1. Yorum: `Password`
  - 2. Alt Yorum: `Kıbrıs`

✅ **Parola bulundu:** `kıbrıs`

---

### 🔓 **5. Steghide ile Gizli Veriyi Çıkarma**

Elimizde hem dosya hem şifre var. Terminalde komutu çalıştırıyoruz:

```bash
steghide extract -sf kıbrıs_kikos.jpeg -p kıbrıs
```

**Komut Parametreleri:**

| Parametre | Açıklama |
|-----------|----------|
| `-sf` | source file (kaynak dosya) |
| `-p` | password (şifre) |

**Çıktı:**
```
wrote extracted data to "hidden_flag.zip"
```

✅ **Başarılı!** `hidden_flag.zip` dosyası oluşturuldu

---

### 📦 **6. ZIP Dosyasını Açma**

```bash
unzip hidden_flag.zip
```

**Çıkan Dosya:** `flag.txt`

---

### 📄 **7. Flag Dosyasını Okuma**

```bash
cat flag.txt
```

**Çıktı:**
```
eXp0Y3Rme1Axbmc6SzFrazBzX1Izc3AwbnMzOnIzYzMxdjNkX1N0NHR1czpHM2xkMW19
```

**Analiz:** Bu bir **Base64** encoded metin (genellikle sonda `=` olur ama bazen olmayabilir)

---

### 🔐 **8. Base64 Decode**

**Komut:**
```bash
echo "eXp0Y3Rme1Axbmc6SzFrazBzX1Izc3AwbnMzOnIzYzMxdjNkX1N0NHR1czpHM2xkMW19" | base64 --decode
```

**Alternatif Online:** https://www.base64decode.org/

**Decode Sonucu:**
```
yztctf{P1ng:K1kk0s_R3sp0ns3:r3c31v3d_St4tus:G3ld1m}
```

---

## 🚩 **FLAG**

```
yztctf{P1ng:K1kk0s_R3sp0ns3:r3c31v3d_St4tus:G3ld1m}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>Namechk</b><br><sub>Kullanıcı adı sorgulama</sub></td>
<td align="center">🐦<br><b>X (Twitter)</b><br><sub>Sosyal medya araştırma</sub></td>
<td align="center">📸<br><b>Instagram</b><br><sub>Parola bulma</sub></td>
<td align="center">🖼️<br><b>Steghide</b><br><sub>Steganografi çözme</sub></td>
<td align="center">🔐<br><b>Base64</b><br><sub>Decode işlemi</sub></td>
</tr>
</table>

**Kullanılan Siteler:**
- 🔍 **Kullanıcı Adı Sorgu:** https://namechk.com/
- 🐦 **X (Twitter) Hesap:** https://x.com/u.yaylacii
- 📸 **Instagram Hesap:** https://instagram.com/u.yaylaci
- 🔐 **Base64 Decode:** https://www.base64decode.org/

---

## 💻 **Kullanılan Komutlar**

```bash
# Steghide ile gizli veriyi çıkarma
steghide extract -sf kıbrıs_kikos.jpeg -p kıbrıs

# ZIP arşivini açma
unzip hidden_flag.zip

# Flag dosyasını okuma
cat flag.txt

# Base64 decode işlemi
echo "eXp0Y3Rme1Axbmc6SzFrazBzX1Izc3AwbnMzOnIzYzMxdjNkX1N0NHR1czpHM2xkMW19" | base64 --decode
```

---

## 💭 **Çözüm Akış Şeması**

```
📝 Soru: u.yaylacii kullanıcısını bul
                    ↓
        🔍 Namechk: Kullanıcı adı sorgusu
                    ↓
        🐦 X (Twitter): u.yaylacii hesabı ✅
                    ↓
        📅 28 Mayıs Tweet: "Bi şeyler var."
                    ↓
        📎 Dosya: kıbrıs_kikos.jpeg (9.9 MB)
                    ↓
        🖼️ Steghide Tespiti → ❌ Şifreli!
                    ↓
        💡 İpucu: "İnstadan bulabileceğini buldun"
                    ↓
        📸 Instagram: u.yaylaci (son i yok!)
                    ↓
        🗼 Galata Kulesi Fotoğrafı
                    ↓
        💬 Yorumlar: Password → Kıbrıs
                    ↓
        🔓 Steghide Extract: -p kıbrıs
                    ↓
        📦 Çıktı: hidden_flag.zip
                    ↓
        🗜️ Unzip: flag.txt
                    ↓
        📄 Base64 Encoded Text
                    ↓
        🔐 Base64 Decode
                    ↓
        🚩 FLAG: yztctf{P1ng:K1kk0s_R3sp0ns3:r3c31v3d_St4tus:G3ld1m}
```
