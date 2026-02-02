# 📱 Giriş Kapalı - Mobile Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Mobile-Challenge-darkblue?style=for-the-badge&logo=android" alt="Mobile">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-APK Reverse-purple?style=for-the-badge&logo=apk" alt="APK Reverse">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Mobile  
**Seviye:** Orta  
**Puan:** 300
  
**Açıklama:** Giriş yapabilirsen bir link bıraktım.

**Flag formatı:** Flag{link}

**Challenge Dosyası:** [📥 Google Drive - GirislerBurdan.apk](https://drive.google.com/file/d/1BEkFREX6K4LuPP5tr2tbLMfZQ-jI_qLV/view?usp=drivesdk)

**Firstblood:** +50 puan

---

## 🔍 Analiz

**Hedef:** APK'yı decompile edip giriş bilgilerini bulmak ve link'e ulaşmak

**Strateji:** APK decompile → Java kodu analizi → Login bypass → Link bulma

**İpuçları:**
- 🔐 Login ekranı var
- 📝 Giriş bilgileri hardcoded
- 🔗 Başarılı girişten sonra link gösterilecek
- 🎯 Flag formatı: `Flag{link}`

---

## ✅ Çözüm Adımları

### 📥 **1. APK İndirme ve İlk Analiz**

**APK dosyasını indirip tipini kontrol ediyoruz:**

```bash
file GirislerBurdan.apk
```

**Çıktı:**
```
GirislerBurdan.apk: Android package (APK), with gradle app-metadata.properties, 
with APK Signing Block
```

> ✅ **Android APK** dosyası doğrulandı!

---

### 📦 **2. APK'yı Unzip Etme**

**APK aslında bir ZIP arşividir:**

```bash
unzip GirislerBurdan.apk -d GirislerBurdan
cd GirislerBurdan
ls -la
```

**Çıktı:**
```
AndroidManifest.xml
classes.dex
META-INF/
res/
resources.arsc
```

**Önemli dosyalar:**
- 📄 **AndroidManifest.xml:** Uygulama yapılandırması
- 🔢 **classes.dex:** Derlenmiş Java/Kotlin kodu (burada login mantığı var!)
- 📂 **res/:** Resource dosyaları (layout, string, vb.)

---

### 🔨 **3. APK Decompile - JADX ile**

**JADX kullanarak APK'yı Java koduna çeviriyoruz:**

```bash
jadx GirislerBurdan.apk -d GirislerBurdan_decompiled
cd GirislerBurdan_decompiled/sources/com/example/myapplication/
ls -la
```

**Bulunan dosyalar:**
```
AccountPage.java
MainActivity.java
ProfilePage.java
R.java
UserPage.java
```

> 💡 MainActivity → Giriş ekranı burada olmalı!

---

### 🔍 **4. MainActivity Analizi**

**MainActivity.java dosyasını inceliyoruz:**

```bash
cat MainActivity.java
```

**Kod:**
```java
package com.example.myapplication;

import android.os.Bundle;
import android.widget.TextView;
import com.google.android.material.button.MaterialButton;
import d.l;
import w0.a;

public class MainActivity extends l {
    @Override
    public final void onCreate(Bundle bundle) {
        super.onCreate(bundle);
        setContentView(R.layout.activity_main);
        
        ((MaterialButton) findViewById(R.id.loginbtn))
            .setOnClickListener(new a(this, 
                (TextView) findViewById(R.id.username), 
                (TextView) findViewById(R.id.password)));
    }
}
```

**Analiz:**
- 📱 Login butonu var
- 👤 Username ve password field'ları var
- 🔍 Login kontrolü `w0.a` sınıfında yapılıyor

> 🎯 **Hedef:** `w0.a` sınıfını incelemek!

---

### 🔐 **5. Login Kontrolü - w0/a.java**

**w0/a.java dosyasını bulup inceliyoruz:**

```bash
cd ~/Desktop/GirislerBurdan_decompiled/sources/w0/
cat a.java
```

**Kod:**
```java
package w0;

import android.content.Intent;
import android.view.View;
import android.widget.TextView;
import android.widget.Toast;
import com.example.myapplication.MainActivity;
import com.example.myapplication.UserPage;

public final class a implements View.OnClickListener {
    public final TextView f3300a;  // username field
    public final TextView f3301b;  // password field
    public final MainActivity f3302c;

    public a(MainActivity mainActivity, TextView textView, TextView textView2) {
        this.f3302c = mainActivity;
        this.f3300a = textView;
        this.f3301b = textView2;
    }

    @Override
    public final void onClick(View view) {
        boolean zEquals = this.f3300a.getText().toString().equals("admin");
        MainActivity mainActivity = this.f3302c;
        
        if (!zEquals || !this.f3301b.getText().toString().equals("us0m14S3!!")) {
            Toast.makeText(mainActivity, "LOGIN FAILED", 0).show();
        } else {
            Toast.makeText(mainActivity, "LOGIN SUCCESSFUL", 0).show();
            mainActivity.startActivity(new Intent(mainActivity.getApplicationContext(), 
                (Class<?>) UserPage.class));
        }
    }
}
```

**🔑 GİRİŞ BİLGİLERİ BULUNDU!**

```
Username: admin
Password: us0m14S3!!
```

**Analiz:**
- ✅ Username kontrolü: `equals("admin")`
- ✅ Password kontrolü: `equals("us0m14S3!!")`
- ✅ Başarılı girişte `UserPage`'e yönlendiriliyor

---

### 🏠 **6. UserPage İnceleme**

**UserPage.java dosyasını açıyoruz:**

```bash
cd ~/Desktop/GirislerBurdan_decompiled/sources/com/example/myapplication/
cat UserPage.java
```

**Kod:**
```java
package com.example.myapplication;

import android.os.Bundle;
import com.google.android.material.button.MaterialButton;
import d.l;
import w0.b;

public class UserPage extends l {
    @Override
    public final void onCreate(Bundle bundle) {
        super.onCreate(bundle);
        setContentView(R.layout.activity_user_page);
        
        ((MaterialButton) findViewById(R.id.profile))
            .setOnClickListener(new b(this, 0));
        ((MaterialButton) findViewById(R.id.account))
            .setOnClickListener(new b(this, 1));
    }
}
```

**Analiz:**
- 🎯 İki buton var: **Profile** ve **Account**
- 🔍 `w0.b` sınıfı butonları handle ediyor
- 💡 Link bu sayfalardan birinde olabilir!

---

### 🔍 **7. Button Handler - w0/b.java**

**w0/b.java dosyasını inceliyoruz:**

```bash
cd ~/Desktop/GirislerBurdan_decompiled/sources/w0/
cat b.java
```

**Kod:**
```java
package w0;

import android.content.Intent;
import android.view.View;
import com.example.myapplication.AccountPage;
import com.example.myapplication.ProfilePage;
import com.example.myapplication.UserPage;

public final class b implements View.OnClickListener {
    public final int f3303a;
    public final UserPage f3304b;

    public b(UserPage userPage, int i2) {
        this.f3303a = i2;
        this.f3304b = userPage;
    }

    @Override
    public final void onClick(View view) {
        int i2 = this.f3303a;
        UserPage userPage = this.f3304b;
        
        switch (i2) {
            case 0:
                // Profile butonu
                userPage.startActivity(new Intent(userPage.getApplicationContext(), 
                    (Class<?>) ProfilePage.class));
                break;
            default:
                // Account butonu
                userPage.startActivity(new Intent(userPage.getApplicationContext(), 
                    (Class<?>) AccountPage.class));
                break;
        }
    }
}
```

**Analiz:**
- 📄 **Profile butonu** → ProfilePage
- 📄 **Account butonu** → AccountPage
- 🔍 Link bu sayfalarda olmalı!

---

### 📄 **8. AccountPage ve ProfilePage İnceleme**

**ProfilePage.java:**
```bash
cat ProfilePage.java
```

**Kod:**
```java
package com.example.myapplication;

import android.os.Bundle;
import d.l;

public class ProfilePage extends l {
    @Override
    public final void onCreate(Bundle bundle) {
        super.onCreate(bundle);
        setContentView(R.layout.activity_profile_page);
    }
}
```

**AccountPage.java:**
```bash
cat AccountPage.java
```

**Kod:**
```java
package com.example.myapplication;

import android.os.Bundle;
import d.l;

public class AccountPage extends l {
    @Override
    public final void onCreate(Bundle bundle) {
        super.onCreate(bundle);
        setContentView(R.layout.activity_account_page);
    }
}
```

**Analiz:**
- 📱 Her iki sayfa da sadece layout dosyasını gösteriyor
- 🔍 Link layout XML dosyalarında olmalı!

> 💡 **Hedef:** Layout XML dosyalarını incelemek!

---

### 📐 **9. Layout Dosyalarını İnceleme**

**Layout dizinine gidiyoruz:**

```bash
cd ~/Desktop/GirislerBurdan_decompiled/resources/res/layout/
ls -la | grep "activity"
```

**Bulunan layout dosyaları:**
```
activity_account_page.xml
activity_main.xml
activity_profile_page.xml
activity_user_page.xml
```

---

### 🔗 **10. AccountPage Layout İnceleme**

**activity_account_page.xml dosyasını açıyoruz:**

```bash
cat activity_account_page.xml
```

**Önemli kısım:**
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android" 
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:background="@drawable/android_background"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    
    <TextView
        android:textSize="35dp"
        android:textStyle="bold"
        android:textColor="@color/black"
        android:gravity="center"
        android:id="@+id/account"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="50dp"
        android:text="Account"/>
    
    <TextView
        android:textSize="32dp"
        android:textColor="@color/black"
        android:id="@+id/forgotpass"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="20dp"
        android:text="Your account is Admin. If you want to flag, you can go to this link: /bdc6a9d55a26ee383a9b5e7bf8e42c83.php"
        android:layout_below="@+id/account"
        android:layout_centerHorizontal="true"/>
</RelativeLayout>
```

**🚩 LİNK BULUNDU!**

```
/bdc6a9d55a26ee383a9b5e7bf8e42c83.php
```

**TextView içeriği:**
```
Your account is Admin. If you want to flag, you can go to this link: 
/bdc6a9d55a26ee383a9b5e7bf8e42c83.php
```

---

### 🎯 **11. Flag Oluşturma**

**Soru formatı:** `Flag{link}`

**Bulunan link:** `/bdc6a9d55a26ee383a9b5e7bf8e42c83.php`

**Flag:**
```
Flag{bdc6a9d55a26ee383a9b5e7bf8e42c83.php}
```

> ✅ **FLAG TAMAMLANDI!**

---

## 🚩 **FLAG**

```
Flag{bdc6a9d55a26ee383a9b5e7bf8e42c83.php}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔍<br><b>file</b><br><sub>Dosya tipi analizi</sub></td>
<td align="center">📦<br><b>unzip</b><br><sub>APK extraction</sub></td>
<td align="center">🔨<br><b>jadx</b><br><sub>APK decompiler</sub></td>
<td align="center">📝<br><b>cat</b><br><sub>Dosya okuma</sub></td>
</tr>
<tr>
<td align="center">🔤<br><b>grep</b><br><sub>Text arama</sub></td>
<td align="center">🗂️<br><b>find</b><br><sub>Dosya bulma</sub></td>
<td align="center">💻<br><b>Linux Terminal</b><br><sub>Komut çalıştırma</sub></td>
<td align="center">📱<br><b>Android</b><br><sub>APK platform</sub></td>
</tr>
</table>

**Kullanılan Komutlar:**
```bash
# APK analizi
file GirislerBurdan.apk
unzip GirislerBurdan.apk -d GirislerBurdan

# Decompile
jadx GirislerBurdan.apk -d GirislerBurdan_decompiled

# Kod analizi
cd GirislerBurdan_decompiled/sources/com/example/myapplication/
cat MainActivity.java
cd ../w0/
cat a.java
cat b.java

# Layout analizi
cd GirislerBurdan_decompiled/resources/res/layout/
cat activity_account_page.xml
```

---

## 💭 **Çözüm Akışı**

```
📱 "Giriş Kapalı" Mobile Challenge
            ↓
📥 GirislerBurdan.apk indirildi
            ↓
🔍 file → Android APK doğrulandı
            ↓
📦 unzip → classes.dex, res/ çıkarıldı
            ↓
🔨 jadx → Java koduna decompile
            ↓
📄 MainActivity.java
   → w0.a sınıfına yönlendirme
            ↓
🔐 w0/a.java analizi
   → Username: "admin"
   → Password: "us0m14S3!!"
            ↓
🏠 UserPage.java
   → Profile ve Account butonları
   → w0.b sınıfı handle ediyor
            ↓
🔍 w0/b.java
   → ProfilePage ve AccountPage
            ↓
📐 activity_account_page.xml
   → TextView içinde link!
            ↓
🔗 Link bulundu
   → /bdc6a9d55a26ee383a9b5e7bf8e42c83.php
            ↓
🚩 Flag{bdc6a9d55a26ee383a9b5e7bf8e42c83.php}
```
