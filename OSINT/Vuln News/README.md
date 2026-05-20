# 🎯 VulnNews - OSINT Challenge

<p align="center">
  <img src="https://img.shields.io/badge/OSINT-Challenge-darkblue?style=for-the-badge" alt="OSINT">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Cyber Intelligence-purple?style=for-the-badge" alt="Intelligence">
</p>

---

## 📋 Soru Bilgileri
  
**Kategori:** OSINT  
**Seviye:** Orta  
**Puan:** 300

**Açıklama:** Bu hafta bizim okulda da bazen kullandığımız bir programı hacklemişler. Programın adını bulup silmemiz gerek. Update yaparken malware indiriyormuş.

**Flag Formatı:** `Flag{programadı}`

**Deneme Hakkı:** Brute atmayın. Flag denemek için 5 hakkınız var :)

**Tarih:** 7 Şubat 2026 (Bu hafta)

---

## 🔍 Analiz

**Hedef:** Şubat 2026 başında hacklenen ve okullarda kullanılan bir yazılımı tespit etmek

**Strateji:** İpuçları analizi → OSINT araştırması → Supply chain attack haberleri → Okul bağlamı → Program tespiti → Flag

**İpuçları:**
- 🎯 "Bu hafta" → 7 Şubat 2026 civarı
- 🏫 "Okulda bazen kullandığımız" → Eğitim kurumlarında yaygın
- 🔄 "Update yaparken malware" → Supply chain attack
- 💻 "Program" → Yazılım/uygulama
- 🎓 "Meslek lisesi" → IT/Programlama dersleri var

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: İpuçlarını Analiz Etme**

**Soruda verilen bilgiler:**

```
✅ Zaman: "Bu hafta" (7 Şubat 2026)
✅ Yer: Okul (Meslek lisesi)
✅ Kullanım sıklığı: "Bazen" (Her gün değil)
✅ Saldırı türü: Supply chain attack
✅ Yöntem: Update mekanizması
✅ Sonuç: Malware dağıtımı
```

**Analiz:**
- ✅ Şubat 2026'nın ilk haftası (1-7 Şubat arası)
- ✅ Okullarda kullanılan bir yazılım
- ✅ Güncelleme sistemi hacklendi
- ✅ Supply chain attack gerçekleşti
- 💡 Meslek lisesi → IT/programlama dersleri → Kod editörü olabilir

---

### 🌐 **Adım 2: OSINT Araştırması Başlatma**

**Arama terimleri belirleme:**

```bash
# Ana arama terimleri
"supply chain attack February 2026"
"malware supply chain 2026"
"software compromised update February"
"supply chain attack Şubat 2026"
```

**OSINT kaynakları:**
- 🌐 Siber güvenlik haber siteleri
- 🐦 Twitter/X (#supplychain #malware)
- 💬 Reddit (r/cybersecurity, r/netsec)
- 📰 Google News
- 🔍 CVE veritabanları

**Analiz:**
- ✅ Supply chain attack = Güncelleme zinciri saldırısı
- ✅ February 2026 = Şubat 2026
- 💡 Okullarda kullanılan yazılım olmalı

---

### 🔍 **Adım 3: Web Araştırması**

**Google araması yapıyoruz:**

```bash
# Arama sorgusu
"malware supply chain"
```

**Sonuçlar:**
- 🎯 **Notepad++** ile ilgili haberler çıkıyor
- 📅 Tarih: Şubat 2026 başı
- 🔴 Supply chain attack
- ⚠️ Update mekanizması hacklendi

**Detaylar:**
```
- Duyuru tarihi: 2 Şubat 2026
- Saldırı dönemi: Haziran-Aralık 2025
- Yöntem: Hosting provider güvenlik açığı
- Hedef: Güncelleme altyapısı
- Sonuç: Seçili kullanıcılara malware dağıtımı
```

**Analiz:**
- ✅ Tarih uyuyor (2 Şubat 2026 = "Bu hafta")
- ✅ Supply chain attack uyuyor
- ✅ Update mekanizması uyuyor
- 💡 Notepad++ okullarda kullanılıyor mu?

---

### 🏫 **Adım 4: Okul Bağlamı Doğrulaması**

**Notepad++ nedir?**
- Ücretsiz kaynak kod ve metin editörü
- Windows için geliştirilmiş
- Programcılar tarafından yaygın kullanılır
- Syntax highlighting desteği
- Eklenti sistemi

**Meslek liselerinde kullanımı:**

✅ **Programlama Dersleri:**
- HTML/CSS öğretimi
- JavaScript kodlama
- PHP/Python geliştirme
- Web tasarım dersleri

✅ **Bilgisayar Bilimleri:**
- Algoritma yazımı
- Kod düzenleme
- Syntax öğrenimi

✅ **"Bazen kullanılan" ifadesi:**
- Her ders değil
- Sadece IT/programlama derslerinde
- Opsiyonel editör (IDE'lerin yanında)

**Pendik İTO Ticaret Mesleki ve Teknik Anadolu Lisesi:**
- ✅ Meslek lisesi → Programlama dersleri var
- ✅ Bilgisayar bölümleri mevcut
- ✅ Web tasarım/kodlama eğitimi
- 💡 Notepad++ yaygın kullanılır

**Analiz:**
- ✅ Tüm ipuçları uyuyor
- ✅ Okul bağlamı mantıklı
- ✅ Supply chain attack doğrulandı
- 🎯 Notepad++ kesinlikle doğru cevap

---

### 🎯 **Adım 5: Flag Formatını Belirleme**

**Flag formatı:** `Flag{programadı}`

**Olası varyasyonlar:**
```
Flag{Notepad++}         # Orijinal yazım
Flag{notepad++}         # Küçük harf
Flag{NotepadPlusPlus}   # Camel case
Flag{notepadplusplus}   # Tek kelime
Flag{Notepad}           # Sadece isim
```

**CTF flag kuralları:**
- Özel karakterler korunabilir
- Tam program adı kullanılır

**En olası format:**
```
Flag{Notepad++}
```

**Analiz:**
- ✅ `++` işaretleri program adının parçası
- 💡 5 deneme hakkımız var, dikkatli olmalıyız

---

### 🏆 **Adım 6: Flag Denemesi**

**İlk deneme:**

```
Flag{Notepad++}
```

**✅ BAŞARILI!**

```
🎉 Flag doğru! 500 puan kazandınız!
```

---

## 🚩 **FLAG**

```
Flag{Notepad++}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🌐<br><b>Google Search</b><br><sub>Web araştırması</sub></td>
<td align="center">🔍<br><b>OSINT Teknikleri</b><br><sub>Açık kaynak istihbaratı</sub></td>
<td align="center">📰<br><b>Haber Kaynakları</b><br><sub>Siber güvenlik haberleri</sub></td>
<td align="center">🧠<br><b>Mantıksal Analiz</b><br><sub>İpucu çıkarımı</sub></td>
</tr>
</table>

**Kullanılan Araştırma Sorguları:**
```bash
# OSINT araştırması
"malware supply chain"
"supply chain attack February 2026"
"notepad++ supply chain"
"notepad++ malware update"

# Doğrulama
"notepad++ schools education"
"notepad++ türkiye okul"
```

**Siber Güvenlik Haber Kaynakları:**
- The Hacker News
- BleepingComputer
- SecurityWeek
- Krebs on Security
- CVE Database

---

## 💭 **Çözüm Akış Şeması**

```
🎯 VulnNews Challenge
            ↓
📋 İpuçlarını analiz et
   → "Bu hafta" (7 Şubat 2026)
   → "Okulda bazen kullanılan"
   → "Update yaparken malware"
   → Supply chain attack
            ↓
🔍 OSINT araştırması başlat
   → "malware supply chain" ara
   → Google, haber siteleri
            ↓
📰 Haberler bulundu
   → Notepad++ supply chain attack
   → 2 Şubat 2026 duyuruldu
   → Update mekanizması hacklendi
            ↓
🏫 Okul bağlamı kontrol
   → Meslek lisesi
   → Programlama dersleri
   → Notepad++ kod editörü
   → ✅ "Bazen kullanılan" uyuyor
            ↓
🎯 Flag formatı belirle
   → Flag{Notepad++}
   → Büyük harf tercih et
   → ++ işaretlerini koru
            ↓
✅ Flag denemesi
   → Flag{Notepad++}
   → ✅ DOĞRU!
            ↓
🚩 Flag{Notepad++}
   → 500 puan! 🎉
```
