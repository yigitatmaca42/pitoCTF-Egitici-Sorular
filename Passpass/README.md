# 🔓 Passpass - Reverse Engineering Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Reverse Engineering-Challenge-darkblue?style=for-the-badge&logo=hackaday" alt="Reverse">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-String%20Analysis-purple?style=for-the-badge&logo=key" alt="String Analysis">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Reverse Engineering  
**Seviye:** Kolay  
**Açıklama:** Şifre zor olsa ne yazar? Çözümü çok kolay.

**Challenge Dosyası:** [📥 GitHub - passpass](https://github.com/cihangungor/pitoctf/blob/main/passpass)

**Verilen:**
- passpass (Binary executable dosya)

---

## 🔍 Analiz

### Binary Dosya Analizi Nedir?

**Reverse Engineering**, bir programın nasıl çalıştığını anlamak ve içindeki gizli bilgileri (şifre, flag vb.) çıkarmak için yapılan analizdir.

| Özellik | Açıklama |
|---------|----------|
| 🔍 **Strings** | Programdaki okunabilir metinler |
| 🔐 **Hardcoded Password** | Kod içine gömülü şifreler |
| 🚩 **Flag** | Hedef bilgi |
| 💻 **Executable** | Çalıştırılabilir dosya |

**Örnek:** Binary içindeki strings analizi = gizli bilgileri açığa çıkarma

---

## ✅ Çözüm Adımları

### 📥 **1. Dosya İndirme ve Yetki Kontrolü**

İlk olarak dosyayı indirip yetkilerine bakıyoruz:

```bash
ls -l passpass
```

**Çıktı:**
```
-rwxrwxr-x passpass
```

> **Not:** Dosya zaten çalıştırma yetkisine sahip, ancak yoksa chmod ile vereceğiz.

---

### 🔓 **2. Çalıştırma Yetkisi Verme**

Dosyaya çalıştırma yetkisi veriyoruz (gerekirse):

```bash
chmod +x passpass
```

---

### 🎮 **3. Uygulamayı Çalıştırma**

Binary dosyayı çalıştırarak ne istediğini görüyoruz:

```bash
./passpass
```

**Çıktı:**
```
Welcome to the SPOOKIEST party of the year.
Before we let you in, you'll need to give us the password:
```

> **Not:** Program bizden bir şifre istiyor. Şimdi bu şifreyi bulmalıyız.

---

### 🔍 **4. Strings Analizi**

Binary dosyanın içindeki okunabilir stringlere bakıyoruz:

```bash
strings ./passpass
```

**Önemli Çıktılar:**
```
/lib64/ld-linux-x86-64.so.2
libc.so.6
puts
__cxa_finalize
strcmp
__libc_start_main
GLIBC_2.2.5
_ITM_deregisterTMCloneTable
__gmon_start__
_ITM_registerTMCloneTable
AWAVI
AUATL
[]A\A]A^A_
Welcome to the SPOOKIEST party of the year.
Before we let you in, you'll need to give us the password: 
s3cr3t_p455_f0r_gh05t5_4nd_gh0ul5
Welcome inside!
HTB{un0bfu5c4t3d_5tr1ng5}
Wrong password, try again!
;*3$"
```

> **Dikkat:** `s3cr3t_p455_f0r_gh05t5_4nd_gh0ul5` şifresi açıkça görünüyor!

---

### 🔐 **5. Şifreyi Deneme**

Bulduğumuz şifreyi programda deniyoruz:

```bash
./passpass
```

**Şifre girişi:**
```
s3cr3t_p455_f0r_gh05t5_4nd_gh0ul5
```

**Çıktı:**
```
Welcome inside!
HTB{un0bfu5c4t3d_5tr1ng5}
```

> **Başarılı!** Flag elde edildi.

---

## 🚩 **FLAG**

```
HTB{un0bfu5c4t3d_5tr1ng5}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>strings</b><br><sub>String çıkarma</sub></td>
<td align="center">💻<br><b>chmod</b><br><sub>Yetki düzenleme</sub></td>
<td align="center">📂<br><b>ls</b><br><sub>Dosya listeleme</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**
- 🔍 **strings** - Binary'den okunabilir metinleri çıkarma
- 💻 **chmod** - Dosya yetkilerini düzenleme
- 📂 **ls -l** - Dosya detaylarını görüntüleme

---

## 💻 **Kullanılan Komutlar**

```bash
# Dosya yetkilerini kontrol etme
ls -l passpass

# Çalıştırma yetkisi verme
chmod +x passpass

# Programı çalıştırma
./passpass

# Strings analizi
strings ./passpass

# Şifre ile giriş
./passpass
# (s3cr3t_p455_f0r_gh05t5_4nd_gh0ul5 giriyoruz)
```

---

## 💭 **Çözüm Akış Şeması**

```
🔓 Passpass Challenge: Binary dosya verildi
                    ↓
        📥 Dosyayı İndir ve Yetkileri Kontrol Et
                    ↓
        💻 chmod +x passpass (gerekirse)
                    ↓
        🎮 ./passpass → Şifre İsteniyor
                    ↓
        🔍 strings ./passpass → String Analizi
                    ↓
        📝 Şifre Bulundu: s3cr3t_p455_f0r_gh05t5_4nd_gh0ul5
                    ↓
        🔐 Şifreyi Programda Dene
                    ↓
        ✅ Welcome inside!
                    ↓
        🚩 FLAG: HTB{un0bfu5c4t3d_5tr1ng5}
```
