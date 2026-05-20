# 🎯 Orda Görüşelim - Forensics Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Forensics-Challenge-darkblue?style=for-the-badge" alt="Forensics">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-PCAP-purple?style=for-the-badge" alt="PCAP">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Forensics  
**Seviye:** Kolay  
**Puan:** 300  

**Challenge Dosyası:** [📥 Google Drive - hedefimiz.pcap](https://drive.google.com/file/d/1XeA75QlFlX5k8JTuhKslHtmmQwB6ZN3d/view?usp=drivesdk)

---

## 🔍 Analiz

**Hedef:** PCAP içerisinden URL ve login bilgilerini bulmak

**Strateji:** HTTP trafik analizi → Login isteğini bul → Şifreyi çıkar → Siteye gir → Flag

**İpuçları:**
- 🎯 HTTP paketleri kritik
- 🔍 POST istekleri incelenmeli
- 💡 Login endpoint önemli
- 🔑 Credentials flag’e götürür

---

## ✅ Çözüm Adımları

### 🔎 Adım 1: HTTP Trafiğini İnceleme

```bash
tshark -r hedefimiz.pcap -Y "http.request" -T fields -e http.request.method -e http.host -e http.request.uri
```

**Çıktı:**
```
GET     172.16.0.1      /api/data
GET     10.0.0.1        /index.html
POST    192.168.1.200   /admin/login
POST    46.101.237.185:2001     /login
GET     172.16.0.254    /dashboard
GET     10.10.10.1      /api/v1/users
```

**Analiz:**
- ✅ Birden fazla endpoint bulundu
- 💡 En kritik endpoint: /login
- 🎯 POST isteğine odaklanılmalı

---

### 🔬 Adım 2: Login İsteğini İnceleme

```bash
tshark -r hedefimiz.pcap -Y 'ip.dst == 46.101.237.185 && http.request.method == "POST"' -T fields -e http.file_data
```

**Çıktı:**
```
username=mebrobotctf&password=hedefbirincilik
```

**Analiz:**
- ✅ Username bulundu: mebrobotctf
- 💡 Password bulundu: hedefbirincilik
- 🎯 Giriş yapılabilir

---

### 🧠 Adım 3: Siteye Giriş

**URL:**
```
http://46.101.237.185:2001/login
```

**Credentials:**
```
mebrobotctf : hedefbirincilik
```

**Analiz:**
- ✅ Başarılı login
- 💡 Dashboard erişimi sağlandı
- 🎯 Flag görüntülendi

---

### 🚩 Adım 4: Flag'i Elde Et

**Çıktı:**
```
Flag{teknoparkbeklebizigeliyoruz}
```

---

## 🚩 FLAG
```
Flag{teknoparkbeklebizigeliyoruz}
```

---

## 🛠️ Kullanılan Araçlar

| Araç | Açıklama |
|------|---------|
| tshark | PCAP analizi |
| Wireshark | Trafik inceleme |
| Browser | Siteye giriş |

---

## 💭 Çözüm Akış Şeması

```
🎯 Orda Görüşelim
       ↓
🔎 HTTP analiz
   → endpointler bulundu
   → login endpoint tespit edildi
       ↓
🔬 POST analiz
   → username bulundu
   → password bulundu
       ↓
🧠 Siteye giriş
   → login başarılı
       ↓
🚩 FLAG: Flag{teknoparkbeklebizigeliyoruz}
```
