# 🎯 ezpz - Mobile Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Mobile-Challenge-darkblue?style=for-the-badge" alt="Mobile">
  <img src="https://img.shields.io/badge/Difficulty-Kolay-darkgreen?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-APK Reverse-purple?style=for-the-badge" alt="APK Reverse">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Mobile  
**Seviye:** Kolay  
**Puan:** 300

**Challenge Dosyası:** [📥 GitHub - ezpz.apk](https://github.com/cihangungor/pitoctf/blob/main/ezpz.apk)

---

## 🔍 Analiz

**Hedef:** Android APK dosyasını tersine mühendislik ile analiz ederek flag'i bulmak

**Strateji:** APK unzip → DEX decompile → Java kodu analizi → Firebase Firestore sorgusu

**İpuçları:**
- 🎯 APK = Android uygulaması → JADX ile Java koduna dönüştür
- 🔍 Firebase Firestore kullanımı → Cloud veritabanında flag olabilir
- 💡 REST API ile Firestore'a doğrudan erişim mümkün
- 🔑 `google-services.json` olmadan `resources.arsc` içinden project ID çekilebilir

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: Dosya Tipini Tespit Et**

```bash
file ezpz.apk
```

**Çıktı:**
```
ezpz.apk: Android package (APK), with zipflinger virtual entry, with APK Signing Block
```

---

### 📊 **Adım 2: String Analizi**

```bash
strings ezpz.apk | grep -i "flag\|FLAG\|PITO\|password\|{"
```

**Kritik Bulgular:**
```
Show password
flagINP
```

**Analiz:**
- ✅ `flagINP` → XML layout'ta bir input alanının ID'si
- 💡 Password input var → Flag girişi yapılıyor
- 🎯 DEX decompile gerekiyor

---

### 🔍 **Adım 3: APK'yı Unzip Et**

```bash
unzip -o ezpz.apk -d ezpz_extracted
ls ezpz_extracted/
```

**Çıktı:**
```
AndroidManifest.xml  classes.dex  classes2.dex  classes3.dex
firebase-firestore.properties  res/  resources.arsc  ...
```

**Analiz:**
- ✅ `classes.dex` → Ana uygulama kodu
- 💡 `firebase-firestore.properties` → Firebase Firestore kullanılıyor!
- 🎯 DEX → Java decompile gerekiyor

---

### 🧠 **Adım 4: JADX ile Java Koduna Dönüştür**

```bash
/usr/bin/jadx -d ezpz_decompiled ezpz.apk
find ezpz_decompiled/sources/com/application/ -name "*.java"
```

**Çıktı:**
```
MainActivity.java
uselessClass.java
R.java
whyAmIHere.java   ← ŞÜPHELİ!
```

---

### 💻 **Adım 5: MainActivity.java Analizi**

```java
// MainActivity.java - Kritik satırlar
final String[] YEET = new whyAmIHere().isThisWhatUWant();
...
if (YEET[0].equals(MainActivity.this.button.getText().toString())) {
    Toast.makeText(..., "Well thats the Correct Flag", 0).show();
}
```

**Analiz:**
- ✅ Flag `whyAmIHere.isThisWhatUWant()` fonksiyonundan geliyor
- 💡 Submit butonuna 500 kez tıklanması gerekiyor (anti-cheat)
- 🎯 `whyAmIHere.java` incelenmeli

---

### 🔥 **Adım 6: whyAmIHere.java Analizi**

```java
public String[] isThisWhatUWant() {
    FirebaseFirestore freeOnlineDatabaseYEEEET = FirebaseFirestore.getInstance();
    freeOnlineDatabaseYEEEET.collection("A_Collection_Is_A_Set_Of_Data")
        .get()
        .addOnSuccessListener(new OnSuccessListener<QuerySnapshot>() {
            public void onSuccess(QuerySnapshot queryDocumentSnapshots) {
                justAWaytoMakeAsynctoSync[0] = snapshot.getString("Points");
                Log.d("TypicalLogcat", justAWaytoMakeAsynctoSync[0]);
            }
        });
}
```

**Kritik Bulgular:**
- ✅ **Collection:** `A_Collection_Is_A_Set_Of_Data`
- ✅ **Field:** `Points`
- 💡 Firebase Firestore public REST API kullanılabilir!
- 🎯 Project ID gerekiyor

---

### 🔑 **Adım 7: Firebase Project ID Bul**

```bash
strings ezpz_extracted/resources.arsc | grep -i "firebase\|appspot\|project" | head -20
```

**Çıktı:**
```
ezpz-91e11
ezpz-91e11.appspot.com
309642636102
```

**Analiz:**
- ✅ **Project ID:** `ezpz-91e11`
- 💡 Firestore REST API endpoint hazır!

---

### 🚀 **Adım 8: Firestore REST API ile Flag Al**

```bash
curl -s "https://firestore.googleapis.com/v1/projects/ezpz-91e11/databases/(default)/documents/A_Collection_Is_A_Set_Of_Data"
```

**Çıktı:**
```json
{
  "documents": [
    {
      "name": "projects/ezpz-91e11/databases/(default)/documents/A_Collection_Is_A_Set_Of_Data/cfmYuJRypSBjiaOrvlcs",
      "fields": {
        "Points": {
          "stringValue": "darkCON{d3bug_m5g_1n_pr0duct10n_1s_b4d}"
        }
      }
    }
  ]
}
```

**KRİTİK BULGULAR:**
- ✅ **FLAG BULUNDU!** 🎉
- 💡 Firebase Firestore public erişime açık bırakılmış
- 🎯 `Points` field'ı flag'i içeriyor

---

## 🚩 **FLAG**
```
darkCON{d3bug_m5g_1n_pr0duct10n_1s_b4d}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">📦<br><b>unzip</b><br><sub>APK extraction</sub></td>
<td align="center">🔧<br><b>JADX</b><br><sub>DEX decompiler</sub></td>
<td align="center">🌐<br><b>curl</b><br><sub>Firestore REST API</sub></td>
<td align="center">🔍<br><b>strings</b><br><sub>String analizi</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 ezpz Mobile Challenge
       ↓
📂 APK analizi
   → Android package, APK Signing Block
   → firebase-firestore.properties var!
       ↓
🔧 JADX decompile
   → MainActivity.java → whyAmIHere.isThisWhatUWant()
   → whyAmIHere.java → Firebase Firestore sorgusu
   → Collection: "A_Collection_Is_A_Set_Of_Data"
   → Field: "Points"
       ↓
🔑 Project ID bul
   → resources.arsc → ezpz-91e11
       ↓
🌐 Firestore REST API
   → curl ile public erişim
   → Points field'ından flag al
       ↓
🚩 FLAG: darkCON{d3bug_m5g_1n_pr0duct10n_1s_b4d}
```
