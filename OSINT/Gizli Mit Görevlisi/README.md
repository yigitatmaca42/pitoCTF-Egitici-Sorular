# 🕵️ GizliMitGörevlisi - OSINT Challenge

<p align="center">
  <img src="https://img.shields.io/badge/OSINT-Challenge-darkblue?style=for-the-badge&logo=searchengineland" alt="OSINT">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Multi Step Research-purple?style=for-the-badge&logo=magnifying-glass" alt="Multi-Step">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** OSINT  
**Seviye:** Orta  
**Puan:** 300  
**Açıklama:** Gizli mit görevlisine bir zarf geldi. İçinde bir kitap, kitabın içinde de gizli bir mesaj var. Gizli mesajın olduğu sayfayı bulduk. Bize zarfın kime geldiğini bulur musun? Zarfa soyadını yanlış yazmışlar. Bu yüzden kızdığını da duyduk.

**Flag Formatı:** `Flag{GelenZarftaYazanAdSoyad}`

**Challenge Dosyası:** [📥 Google Drive - GizliMit.jpg](https://drive.google.com/file/d/1Oum8S_GViLCg9cy1F-AVVs4uNdiUS3mR/view?usp=drivesdk)

---

## 🔍 Analiz

### Fotoğraf İçeriği

Soruyu açtığımızda **"GizliMit.jpg"** adında bir fotoğraf geliyor. Açtığımızda içinde şunlar yazıyor:

```
Beşinci Cild
Yazan: Naima Mustafa efendi
Çeviren: Zuhiri Danışman
Kitap No: (Random Sayılar)
Cilt No: (Random Sayılar)
Sıra No: (Random Sayılar)
Konu No: (Random Sayılar)
```

**İpuçları:**
- 📚 Kitap: **5. Cilt**
- ✍️ Yazar: **Naima Mustafa Efendi**
- 🔢 Nokia kodlaması ipucu (sayılar)

---

## ✅ Çözüm Adımları

### 📚 **1. Kitap Araştırması**

Kitabı bulmak için tarayıcımıza **"Naima Mustafa efendi 5. Cild"** yazıyoruz.

**Bulunan Kitap:** **NAİMA TARİHİ**

---

### 🔍 **2. Google Lens ile Görsel Arama**

**Google Lens** kısmından:
1. **GizliMit.jpg**'yi yüklüyoruz
2. Yanına **"naima tarihi"** yazıyoruz

**Sonuç:**
Karşımıza **1. sırada** şu link çıkıyor:

🔗 **X (Twitter) Paylaşımı:** https://x.com/mycapella01/status/1710673623623277613068?s=46

Bu paylaşımda **GizliMit.jpg ile aynı olan** bir fotoğraf var!

---

### 📹 **3. Video İncelemesi (X/Twitter)**

Linke gittiğimizde videoyu izliyoruz:

**Video İçeriği:**
- **Aslan Akbey** bu numaraları **Nokia kodlaması** sayesinde çözüyor
- **51. saniye**'de şu metin çıkıyor:

```
Operasyon hedef kurtlar vadisi
```

Bu yazıyı oradan alıyoruz.

---

### 🎬 **4. YouTube Araştırması**

YouTube arama kısmına **"Operasyon hedef kurtlar vadisi"** yazıyoruz.

**Sonuçlar:**
- Birkaç video çıkıyor
- Kimisi çok uzun
- Kimisi not kısmını almıyor

Biraz aşağı iniyoruz ve **"Edit Vadisi"** adlı kanalın videosuna denk geliyoruz:

🔗 **YouTube Video:** https://www.youtube.com/watch?v=pDnAR9Ym6_A

---

### 📮 **5. Zarf Keşfi**

Videoyu açtığımız zaman:
- **Aslan Akbey**'e bir zarf geliyor
- Zarfın üzerinde **"Aslan Akbay"** yazıyor

**Soruda belirtilen ipucu:**
> "Zarfa soyadını yanlış yazmışlar. Bu yüzden kızdığını da duyduk"

Bu kısım ile uyuşuyor! Doğru yazılış: **Aslan Akbay** ama zarfta **"Aslan Akbey"** yazıyor (soyadı yanlış).

---

## 🚩 **FLAG**

```
flag{AslanAkbay}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>Google Search</b><br><sub>Kitap araştırması</sub></td>
<td align="center">📸<br><b>Google Lens</b><br><sub>Görsel arama</sub></td>
<td align="center">🐦<br><b>X (Twitter)</b><br><sub>Video kaynağı</sub></td>
<td align="center">🎬<br><b>YouTube</b><br><sub>Video analizi</sub></td>
</tr>
</table>

**Kullanılan Linkler:**
- 🔗 **X (Twitter) Post:** https://x.com/mycapella01/status/1710673623277613068?s=46
- 🔗 **YouTube Video (Edit Vadisi):** https://www.youtube.com/watch?v=pDnAR9Ym6_A
- 🔗 **Google Drive Fotoğraf:** https://drive.google.com/file/d/1Oum8S_GViLCg9cy1F-AVVs4uNdiUS3mR/view?usp=drivesdk

---

## 💭 **Çözüm Akış Şeması**

```
📥 GizliMit.jpg İndirme
            ↓
📖 Fotoğraf Analizi:
   - Beşinci Cild
   - Naima Mustafa Efendi
   - Random sayılar (Nokia kodlaması)
            ↓
🔍 Google Search: "Naima Mustafa efendi 5. Cild"
            ↓
📚 Kitap Bulma: NAİMA TARİHİ
            ↓
📸 Google Lens: GizliMit.jpg + "naima tarihi"
            ↓
🐦 X (Twitter) Bulma: Aynı fotoğraf paylaşımı
            ↓
📹 Video İzleme: Aslan Akbey + Nokia kodlaması
            ↓
📝 51. Saniye: "Operasyon hedef kurtlar vadisi"
            ↓
🎬 YouTube Arama: "Operasyon hedef kurtlar vadisi"
            ↓
🎥 Edit Vadisi Kanalı: Video bulma
            ↓
📮 Zarf Keşfi: Üzerinde "Aslan Akbay" yazıyor
            ↓
✅ Soyadı Hatası: Akbey → Akbay
            ↓
🚩 FLAG: flag{AslanAkbay}
```
