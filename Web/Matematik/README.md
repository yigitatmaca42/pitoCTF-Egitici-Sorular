# 🎯 Matematik - Web Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Web-Challenge-darkblue?style=for-the-badge" alt="Web">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Automation-purple?style=for-the-badge" alt="Automation">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Web  
**Seviye:** Kolay  
**Puan:** 500

**Challenge URL:** `http://68.183.213.255:9014/challenge.php`

---

## 🔍 Analiz

**Hedef:** Web sitesindeki 10 matematik sorusunu 2 saniye içinde çözerek flag'i elde etmek

**Strateji:** Site keşfi → Protokol analizi → Form yapısı → Otomasyon → Flag

**İpuçları:**
- 🎯 2 saniye limiti → Manuel çözüm imkansız
- 🔍 Form alanları → `answer1..10` ve gizli `q1..q10`
- 💡 Otomasyon gerekli → Python requests ile çözülebilir
- 🔑 500 puan = Otomasyon becerisi gerektirir

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Siteyi Keşfet**

```bash
curl -s http://68.183.213.255:9014
```

**Çıktı:**
```
⚡ Math Speed Challenge
⏱️ Solve 10 questions in under 2 seconds!
⚠️ Warning: You have maximum 2 seconds to submit all answers!
💡 Tip: Manual solving is impossible. Automation required!
1. 71 - 72 = ?
2. 22 - 73 = ?
...
🚀 Submit Answers
```

**Analiz:**
- ✅ 10 matematik sorusu her seferinde rastgele değişiyor
- 💡 2 saniyelik süre limiti → Manuel çözüm kesinlikle imkansız
- 🎯 Site otomasyona yönlendiriyor

---

### 📊 **Adım 2: Form Yapısını Analiz Et**

```bash
python3 << 'EOF'
import requests
import re
from bs4 import BeautifulSoup

session = requests.Session()
url = "http://68.183.213.255:9014"

r = session.get(url)
soup = BeautifulSoup(r.text, 'html.parser')

questions = re.findall(r'\d+\.\s+(.+?)\s*=\s*\?', r.text)
answers = [str(eval(q)) for q in questions]
print("Cevaplar:", answers)

form = soup.find('form')
print("Form action:", form.get('action') if form else "Form bulunamadı")
print("Form fields:", [i.get('name') for i in form.find_all('input')] if form else "")
EOF
```

**Çıktı:**
```
Cevaplar: ['121', '-33', '805', '9', '1472', '5', '-30', '98', '1136', '-45']
Form action: None
Form fields: ['answer1', 'q1', 'answer2', 'q2', 'answer3', 'q3', 'answer4', 'q4',
              'answer5', 'q5', 'answer6', 'q6', 'answer7', 'q7', 'answer8', 'q8',
              'answer9', 'q9', 'answer10', 'q10']
```

**Analiz:**
- ✅ Form action yok → POST isteği aynı URL'ye yapılacak
- 💡 `answer1..answer10` → Kullanıcının gireceği cevaplar
- 🎯 `q1..q10` → Gizli input alanları, soru ifadelerini taşıyor
- 🔑 Gerçek endpoint: `/challenge.php`

---

### 🔍 **Adım 3: Tam Otomasyonla Flag'i Al**

```bash
python3 << 'EOF'
import requests
import re
from bs4 import BeautifulSoup

session = requests.Session()
url = "http://68.183.213.255:9014/challenge.php"

r = session.get(url)
soup = BeautifulSoup(r.text, 'html.parser')

questions = re.findall(r'\d+\.\s+(.+?)\s*=\s*\?', r.text)
answers = [str(eval(q)) for q in questions]

hidden = {i.get('name'): i.get('value', '') for i in soup.find_all('input')}
for i, ans in enumerate(answers, 1):
    hidden[f'answer{i}'] = ans

r2 = session.post(url, data=hidden)
soup2 = BeautifulSoup(r2.text, 'html.parser')
print(soup2.get_text())
EOF
```

**Çıktı:**
```
🎉 SUCCESS!
🚀 INCREDIBLE!
You solved all 10 questions in 0.053 seconds!
🏁 PITO{sp33d_m4th_b0t_pwn3d_5e69a85d}
Correct Answers: 10/10
```

**KRİTİK BULGULAR:**
- ✅ **FLAG BULUNDU!** 🎉
- 💡 Python `eval()` ile matematik ifadeleri anında hesaplandı
- 🎯 0.053 saniyede 10 sorunun tamamı doğru cevaplandı
- 🔑 Session cookie ile GET→POST akışı korundu

---

## 🚩 **FLAG**
```
PITO{sp33d_m4th_b0t_pwn3d_5e69a85d}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🐍<br><b>Python</b><br><sub>Otomasyon dili</sub></td>
<td align="center">🌐<br><b>requests</b><br><sub>HTTP istekleri</sub></td>
<td align="center">🍲<br><b>BeautifulSoup</b><br><sub>HTML parsing</sub></td>
<td align="center">🔧<br><b>eval()</b><br><sub>Matematiksel hesaplama</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 Math Speed Challenge
       ↓
📂 Siteyi keşfet
   → curl ile ilk bakış
   → 10 rastgele soru, 2 saniyelik limit
       ↓
📊 Form yapısını analiz et
   → BeautifulSoup ile HTML parse
   → answer1..10 + gizli q1..10 alanları
   → Gerçek endpoint: /challenge.php
       ↓
🤖 Tam otomasyon
   → Session aç → GET ile soruları çek
   → regex ile soruları parse et
   → eval() ile cevapları hesapla
   → POST ile gönder
       ↓
✅ FLAG: PITO{sp33d_m4th_b0t_pwn3d_5e69a85d}
```
