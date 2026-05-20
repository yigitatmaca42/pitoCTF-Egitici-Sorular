# 🚫 DontReverse - Reverse Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Reverse-Challenge-darkblue?style=for-the-badge" alt="Reverse">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Bonus-+1%20Point-success?style=for-the-badge&logo=star" alt="Bonus">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Reverse  
**Seviye:** Kolay  
**Puan:** 200  
**Soru No:** 69  
**Bonus:** +1 Puan (Flagi tek seferde doğru gönderenlere. İkinci denemede bonus yok!)  
**Açıklama:** Ekrana yazdığı görüntünün ekran resmini alıp google lenste aratsak? Acaba benzer soru varmıdır? Hazırı seviyorlardı.

**Challenge Dosyası:** [📥 Google Drive](https://drive.google.com/file/d/1nxkZo5h-gDNCMYpun3erNC_IxOB8W86c/view?usp=drivesdk)

---

## 🔍 Analiz

### İlk Deneme: Ghidra

İlk başta **Ghidra** ile bakıyoruz, belki içinde bir şifreleme veya gizli birşeyler saklanmıştır diye. Ancak çoğu şey okunmuyor, bu yüzden **IDA**'ya geçiyoruz.

---

## ✅ Çözüm Adımları

### 🛠️ **1. IDA'yı Açma ve Dosya Import**

**IDA'yı açıyoruz:**

1. **"do_not_reverse"** dosyasını IDA'ya atıyoruz
2. Hiçbirşey değiştirmeden **OK** diyoruz

---

### 📊 **2. Graph Overview Ayarları**

Dosyayı açtığımızda ilk yapacağımız şey **graph'lara** bakmak. Belki gizli bir metin bulabiliriz (ASCII art gibi ama farklı - burada dosyalar birbirine bağlı ve ortada bir yazı/resim/vb. şeyler çıkartıyorlar).

**Graph Ayarları:**

**Options → General → Graph → Max number of nodes → 1000**

> Not: Otomatik 1000 olur ve bu şimdilik yeterli.

---

### 🔍 **3. Graph Overview'da Flag Keşfi**

**Graph açma:**

1. Kodlar kısmında **rastgele bir satıra** tıklıyoruz
2. **Space** tuşuna basıyoruz
3. **"Graph overview"** kısmı açılıyor

**Flag bulma:**

Biraz **küçültünce** node'larda flagın gizlendiğini görüyoruz!

**Gizli Metin:**
```
BW{asm_asm_61}
```

---

## 🚩 **FLAG**

```
BW{asm_asm_61}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">👻<br><b>Ghidra</b><br><sub>İlk analiz (başarısız)</sub></td>
<td align="center">🔍<br><b>IDA Pro</b><br><sub>Graph analizi</sub></td>
<td align="center">📊<br><b>Graph Overview</b><br><sub>Görsel flag tespiti</sub></td>
</tr>
</table>

**IDA Komutları:**
- **Space:** Graph overview açma/kapama
- **Options → General → Graph:** Node sayısı ayarı
- **Mouse Wheel:** Zoom in/out

---

## 💭 **Çözüm Akış Şeması**

```
📥 Dosya İndirme: do_not_reverse
            ↓
🛠️ İlk Deneme: Ghidra → Okunmuyor ❌
            ↓
🔍 İkinci Deneme: IDA 
            ↓
📂 Dosya Import: Hiçbir ayar değiştirmeden OK
            ↓
⚙️ Graph Ayarları: Max nodes → 1000
            ↓
📊 Graph Overview Açma: Space tuşu
            ↓
🔎 Zoom Out: Node yapısını incele
            ↓
✅ Flag Keşfi: Node'larda gizli metin
            ↓
🚩 FLAG: BW{asm_asm_61}
```
