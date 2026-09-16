# Vokalis

Sesli hatırlatıcı, kesintisiz fatura alarmı ve aylık arşiv uygulaması.
(E-posta otomasyonu kaldırıldı — kapsam dışı.)

## Bu klasörde ne var?

```
vokalis/
├── www/index.html                    → Uygulamanın tamamı (arayüz + mantık)
├── www/manifest.json                 → PWA ayarları
├── android-res/raw/alarm_sound.wav   → Gerçek alarm sesi (native tarafa otomatik kopyalanır)
├── capacitor.config.json             → Web'i Android kabuğuna saran yapılandırma
├── package.json                      → Gerekli paketler (local-notifications dahil)
└── .github/workflows/build-apk.yml   → GitHub'da otomatik APK üretimi
```

## Telefondan GitHub'a yükleme (bilgisayarsız, adım adım)

1. **github.com**'a telefon tarayıcından gir, sağ üstten **+ → New repository** ile `vokalis` adında yeni, boş bir repo oluştur (Public olabilir).
2. Repo açıldığında **"Add file" → "Upload files"** butonuna dokun.
3. Bilgisayardaysan bu zip'i olduğu gibi sürükleyip bırakabilirsin (klasör yapısı korunur). Sadece telefondaysan ve dosya seçici klasör yapısını korumuyorsa, bunun yerine her dosya için **"Add file" → "Create new file"** yolunu kullan: açılan isim kutusuna doğrudan `www/index.html` gibi tam yolu yaz (klasörü otomatik oluşturur), içeriğini yapıştır, **Commit**. Aynısını sırayla şu dosyalar için tekrarla:
   - `www/index.html`
   - `www/manifest.json`
   - `android-res/raw/alarm_sound.wav` *(bu ikili/ses dosyası olduğu için metin kutusuna yapıştırılamaz — bunun için mobil GitHub uygulamasından veya "Upload files" ekranından dosya seçiciyle doğrudan yüklemen gerekir)*
   - `capacitor.config.json`
   - `package.json`
   - `.github/workflows/build-apk.yml`
4. Ana dizine push/commit ettiğin an **Actions** sekmesinde "Vokalis APK Build" otomatik başlar (3-6 dakika sürer).
5. Bitince o çalışmanın sayfasında **Artifacts** kısmından `vokalis-debug-apk` dosyasını indir — içinde `app-debug.apk` var.
6. APK'yı arkadaşına WhatsApp/Drive ile direkt at. Telefonunda "bilinmeyen kaynaklardan yükleme" izni açması gerekecek (Play Store dışı her APK için geçerli).

> Bilgisayarın varsa çok daha kolay: zip'i aç, klasörü olduğu gibi `git init && git add . && git commit -m "ilk sürüm" && git push` ile kendi reponuza gönder. Actions aynı şekilde otomatik çalışır.

## Şimdi eklenen: gerçek native alarm (telefon kapalı/uygulama kapalıyken de çalışır)

`@capacitor/local-notifications` eklentisi eklendi. Bunun anlamı:

- Her fatura kaydettiğinde, o ayın belirlediğin gününde saat 09:00'da **işletim sistemi seviyesinde** bir alarm kuruluyor — uygulama tamamen kapalı olsa, telefon ekranı kilitli olsa bile Android bunu kendisi tetikliyor.
- Bildirim, projeye gömülü gerçek bir siren sesiyle (`alarm_sound.wav`) çalıyor, titreşimle geliyor ve "ongoing" (kapatılamaz/sabit) olarak ayarlandı.
- Bildirime dokunduğunda uygulama açılır ve **"siz kapatana kadar" çalan tam ekran alarm ekranı** devreye girer; orada "Kapat & Tamamlandı" veya "1 Saat Ertele" seçebilirsin.
- Fatura her ay aynı günde otomatik tekrar eder — tekrar kurmana gerek yok.
- Günlük görevler/randevular için de saatinde tek seferlik native bildirim kuruluyor.

### Dürüstlük payı — hâlâ %100 native olmayan tek nokta

Android'de bir bildirimin "siz elle kapatana kadar sonsuza dek, ekran kapalıyken bile yüksek sesle çalması" (gerçek çalar saat davranışı) için Android'in `AlarmClock` + tam ekran `Activity` + `foreground service` üçlüsünü **özel native kod (Kotlin)** ile yazmak gerekiyor — bu, bir eklenti ayarından fazlası, gerçek native geliştirme. Şu anki sürüm bunun **pratik olarak en yakınına** ulaşıyor: uygulama kapalıyken bile alarm zamanında sesli/titreşimli olarak tetikleniyor; uygulamayı açtığında (bildirime dokunarak) o an gerçekten "siz kapatana kadar" çalmaya devam ediyor. Tamamen ekran kapalıyken sonsuz döngü istiyorsan bir sonraki adımda seninle birlikte bu özel native Android servisini yazabiliriz — ama onu test edebilmem için gerçek bir Android cihazda/emülatörde deneme şart, burada köre kod yazıp "hatasız" diye sunmak istemem.

## Yol haritası

1. **v1.1 (şu an bu sürüm):** Gerçek native fatura/görev alarmı, izin yönetimi, arşiv, sesli komutla görev ekleme.
2. **v1.2:** Basit PIN kilidi (fatura verisi hassas olduğu için).
3. **v2 (istersen):** Tam ekran, ekran kapalıyken bile sonsuz çalan native Kotlin alarm servisi — gerçek cihazda test ederek adım adım ekleriz.
