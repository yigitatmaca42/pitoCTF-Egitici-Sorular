# 🎯 HileVarmi? - Reverse Challenge

<p align="center">
  <img src="https://img.shields.io/badge/Reverse-Challenge-darkblue?style=for-the-badge" alt="Reverse">
  <img src="https://img.shields.io/badge/Difficulty-Orta-darkorange?style=for-the-badge&logo=target" alt="Difficulty">
  <img src="https://img.shields.io/badge/Type-Unity%20Game%20RE-purple?style=for-the-badge" alt="Unity Game RE">
</p>

---

## 📋 Soru Bilgileri

**Kategori:** Reverse  
**Seviye:** Orta  
**Puan:** 500

**Challenge Dosyası:** [📥 Google Drive - BuradanKacisYok](https://drive.google.com/file/d/1QkxMJSjrld_-93aCdK_7mqmNkEjer-Ma/view?usp=drivesdk)

---

## 🔍 Analiz

**Hedef:** Unity oyununun içindeki flag'i bulmak

**Strateji:** ZIP aç → Oyun yapısını tanı → Assembly-CSharp.dll'i decompile et → Level asset'lerini tara → Flag'i çıkar

**İpuçları:**
- 🎯 Oyun bir Unity projesi, tüm oyun mantığı `Assembly-CSharp.dll` içinde
- 🔍 `BuildManager` sınıfında gizli bir tuş kombinasyonu var
- 💡 Level asset dosyaları binary olmalarına rağmen `strings` ile taranabilir
- 🔑 Flag doğrudan `level0` dosyasına gömülmüş

---

## ✅ Çözüm Adımları

### 🔎 **Adım 1: ZIP Dosyasını Aç ve İncele**

Drive linkinden `BuradanKacisYok.zip` indirildi ve çıkartıldı:

```bash
unzip BuradanKacisYok.zip -d ctf/
ls ctf/BuradanKacisYok/
```

**Çıktı:**
```
MonoBleedingEdge/
My project.exe
My project_Data/
UnityCrashHandler64.exe
UnityPlayer.dll
```

**Analiz:**
- ✅ Klasik bir Unity Windows build yapısı
- 💡 Tüm oyun mantığı `My project_Data/Managed/Assembly-CSharp.dll` içinde
- 🎯 Level verileri `My project_Data/level0` dosyasında

---

### 🧠 **Adım 2: Assembly-CSharp.dll'i Decompile Et**

`monodis` aracıyla DLL disassemble edildi:

```bash
monodis "My project_Data/Managed/Assembly-CSharp.dll" > disasm.txt
```

Sınıflar listelendi:

```
BuildManager
MyCharacterController
MyPlayer
ExampleCharacterController
PlanetManager
FrameratePanel
...
```

**Analiz:**
- ✅ Standart KinematicCharacterController örnek sınıfları mevcut
- 💡 `BuildManager` sınıfı dikkat çekici → adı CTF bağlamında şüpheli
- 🎯 `BuildManager::Update()` metoduna odaklanıldı

---

### 🔑 **Adım 3: BuildManager Sınıfını İncele**

`BuildManager::Update()` metodu incelendi:

```cil
.method private hidebysig
       instance default void Update () cil managed
{
    IL_0000:  ldc.i4 282
    IL_0005:  call bool class [UnityEngine.InputLegacyModule]UnityEngine.Input::GetKeyDown(...)
    IL_000a:  brfalse.s IL_0012

    IL_000c:  ldc.i4.0
    IL_000d:  call void class [UnityEngine.CoreModule]UnityEngine.SceneManagement.SceneManager::LoadScene(int32)
    IL_0012:  ret
}
```

**Kritik Bulgular:**
```
ldc.i4 282 → Unity KeyCode.F1 = 282
LoadScene(0) → Scene 0'ı yeniden yükle
```

**Analiz:**
- ✅ **Hile bulundu!** `F1` tuşuna basılınca oyun Scene 0'ı yeniden yüklüyor
- 💡 Challenge sorusu "HileVarmi?" → Cevap: EVET, F1 tuşu
- 🎯 Ama asıl flag henüz bulunamadı → asset dosyaları incelenmeli

---

### 🔢 **Adım 4: Level Asset Dosyalarını Tara**

`level0` binary dosyası `strings` komutuyla tarandı:

```bash
strings "My project_Data/level0"
```

**Çıktı (ilgili kısım):**
```
2022.3.10f1
globalgamemanagers.assets
sharedassets0.assets
level0.resS
Cube (22)
Directional light
Floor
Level
Cylinder
Capsule
Text (TMP)
...
INTIGRITI{Y0u_g0t_1t_g00d_J0b!}
```

**Analiz:**
- ✅ Flag doğrudan `level0` asset dosyasına gömülmüş
- 💡 Büyük ihtimalle oyun içindeki bir `Text (TMP)` objesinin içeriği
- 🎯 Flag elde edildi!

---

## 🚩 **FLAG**
```
INTIGRITI{Y0u_g0t_1t_g00d_J0b!}
```

---

## 🛠️ **Kullanılan Araçlar**

<table>
<tr>
<td align="center">🔧<br><b>monodis</b><br><sub>Assembly-CSharp.dll decompile</sub></td>
<td align="center">🔍<br><b>strings</b><br><sub>Binary asset tarama</sub></td>
<td align="center">📦<br><b>unzip</b><br><sub>ZIP çıkartma</sub></td>
<td align="center">💻<br><b>file / xxd</b><br><sub>Dosya analizi</sub></td>
</tr>
</table>

---

## 💭 **Çözüm Akış Şeması**

```
🎯 HileVarmi? Reverse Challenge
       ↓
📦 BuradanKacisYok.zip (Drive'dan indirildi)
   → unzip → Unity Windows build yapısı
   → My project_Data/Managed/Assembly-CSharp.dll tespit edildi
       ↓
🔧 monodis ile DLL decompile
   → Sınıflar listelendi
   → BuildManager::Update() şüpheli bulundu
       ↓
🧠 BuildManager analizi
   → ldc.i4 282 → Unity KeyCode.F1
   → LoadScene(0) → Scene yeniden yükleme
   → HİLE BULUNDU: F1 tuşu ✅
       ↓
🔍 Level asset taraması
   → strings "My project_Data/level0"
   → Binary içinde flag string'i tespit edildi ✅
       ↓
🚩 FLAG: INTIGRITI{Y0u_g0t_1t_g00d_J0b!}
```
