# ŞİŞE VURMA

Sapanla renkli sıvılarla dolu **şeffaf cam şişeleri** vurduğun çizgi film tarzı bir mobil/masaüstü oyun.
Phaser 3 + düz JavaScript, derleme adımı yok, görsellerin ve seslerin tamamı kodla üretilir (harici dosya yok).

## 1. Yerel olarak çalıştırma

Gereken: [Node.js](https://nodejs.org) 18 veya üstü (yalnızca küçük bir yerel sunucu için).

```bash
cd sise-vurma
npm start                # http://localhost:8080
```

* **Bilgisayarda:** tarayıcıda `http://localhost:8080` adresini aç, fareyle oyna.
* **Telefonda:** telefon ve bilgisayar aynı Wi-Fi ağında olsun. `npm start` çıktısındaki
  `Telefonda : http://192.168.x.x:8080` adresini telefonun Chrome'unda aç ve telefonu **yatay** tut.
* Node yoksa: `python3 -m http.server 8080` de olur.
* `index.html` dosyasına çift tıklamak çoğu tarayıcıda çalışır, ama yazı tipi gibi yerel dosya kısıtları
  nedeniyle bir sunucu üzerinden açmak daha güvenlidir.

## 2. Nasıl oynanır

* Ekranın herhangi bir yerine bas, **geriye (sola/aşağı) çek**, bırak. Top fırlar. Fare ve dokunmatik aynı çalışır.
* Her bölümde **6 şişe** vardır, her şişenin sıvısı farklı renktir. Üstteki **HEDEF** bandı hangi renkli şişeyi
  vuracağını gösterir. 6 hedef **sırayla** vurulunca bölüm biter.
* Her bölüm **2 top hakkıyla** başlar (ekranda `2/2`).
  * **Doğru şişe:** puan kazanılır (100 + üst üste vuruşta seri bonusu), şişe patlar, top hakkı harcanmaz.
    Aynı atış kırılan şişeden geçip sıradaki hedefi de vurursa seri devam eder.
  * **Yanlış şişe ya da ıska:** 1 top hakkı gider.
  * **Top hakkı 0 olunca** "Topların bitti!" penceresi açılır:
    **REKLAM İZLE — 3 EKSTRA TOP KAZAN** ya da **BÖLÜMÜ YENİDEN BAŞLAT**.
* Bölüm bitince **BÖLÜM TAMAMLANDI!** ekranı: kazanılan puan, kalan top, toplam puan, yıldızlar
  (hatasız = 3 yıldız) ve **SONRAKİ BÖLÜM** düğmesi. Kalan her top 50 puan ekler.
* Alt köşede: ana menü, bölümü yeniden başlat, ses aç/kapat.
* **Bölümler:** 1. bölüm tek sıra, sabit platform (kolay). Sonrakilerde basamaklı platformlar, ayrı adalar,
  yana/yukarı-aşağı hareket eden platformlar, 6 şişe şekli ve 8 renkten 6'sı gelir; zorluk arttıkça yörünge
  noktaları azalır, hareketli platformlar hızlanır. Bölüm düzenleri `src/core/levels.js` içindeki şablonlardan üretilir.
* **Nişan yardımı:** bırakılan çekiş hedef şişeyi çok az ıskalıyorsa küçük bir "mıknatıs" etkisiyle
  düzeltilir (yalnızca hedef şişe için; ilerleyen bölümlerde zayıflar).

## 3. Ödüllü reklam sistemi

Reklam mantığı `src/core/ads.js` içindedir, oyun sahnesi yalnızca `AdManager` ile konuşur:

| Sağlayıcı | Ne zaman | Nasıl |
|---|---|---|
| `MockProvider` (sahte test reklamı) | Tarayıcıda / geliştirmede (varsayılan) | Ekranda "TEST REKLAMI" penceresi, 5 sn geri sayım |
| `AdMobProvider` | Capacitor ile paketlenmiş **Android uygulamasında** | `@capacitor-community/admob` eklentisi, AdMob Rewarded Ads |

Kurallar (kodda uygulanır ve testlerle doğrulanır):

* Reklam **yalnızca REKLAM İZLE düğmesine basılınca** gösterilir (arka planda yalnızca yüklenir).
* Reklam **sonuna kadar izlenirse** (ödül olayı gelirse) 3 top verilir; **erken kapatılırsa** top verilmez.
* Yüklenemezse/gösterilemezse hata mesajı ve **TEKRAR DENE** düğmesi çıkar.
* Aynı anda ikinci reklam isteği reddedilir (çift ödül yok).

### Geliştirmede test senaryoları

`config/ads.config.js` içinde `allowQueryOverrides: true` iken adrese parametre ekleyerek zorlayabilirsin:

| Adres | Davranış |
|---|---|
| `?ad=loadfail` | Reklam yüklenemez (hata + TEKRAR DENE) |
| `?ad=showfail` | Yükleme tamam, gösterme hata verir |
| `?ad=early` | Reklam erken kapatılmış gibi davranır (ödül yok) |
| `?adms=400` | Sahte reklam 400 ms sürer (hızlı test) |

### APK (Android uygulaması) oluşturma ve gerçek AdMob reklamı

**AdMob düz web sayfasında reklam gösteremez.** Gerçek reklamlı oyun için oyun, Capacitor ile bir Android
uygulamasına (APK) sarılır. Uygulama içinde `provider: 'auto'` AdMob'u kendiliğinden seçer. APK'yı üreten her şey hazırdır:

| Dosya | Ne yapar |
|---|---|
| `tools/build-apk.js` (`npm run apk`) | Tüm adımları sırayla çalıştırıp `dist/sise-vurma-debug.apk` üretir |
| `tools/android-patch.js` | AdMob **uygulama** kimliğini (`config/ads.config.js`'ten) ve yatay ekran kilidini Android projesine yazar |
| `.github/workflows/android-apk.yml` | Aynı işi GitHub'da, bilgisayarınıza hiçbir şey kurmadan yapar |

> **Dürüst not:** Bu araçlar yazıldığı ortamda (Android SDK, Gradle ve npm erişimi olmayan bir sandbox) **APK derlemesini
> yapamadı.** Android yamasının mantığı gerçek bir Capacitor 8.5.2 şablonuna karşı test edildi (`tests/android-patch.test.js`),
> ama Gradle derlemesi ve reklamın telefonda görünmesi ilk denemede sizin tarafınızda doğrulanmalıdır.

#### Yol A: Bilgisayara kurulum olmadan, bulutta (GitHub Actions, ücretsiz)

1. [github.com](https://github.com)'da ücretsiz hesapla yeni bir **boş** depo (repository) açın.
2. Zip'i açın; `sise-vurma` klasörünün **içindeki her şeyi** depoya yükleyin (`.github` klasörü dahil; bu klasör gizlidir:
   macOS Finder'da `Cmd+Shift+.` ile gösterilir). Web arayüzü `.github` klasörünü almazsa: depoda
   **Add file → Create new file** deyip adı `.github/workflows/android-apk.yml` yazın ve dosyanın içeriğini yapıştırın.
3. Depoda **Actions** sekmesi → **Android APK derle** → **Run workflow**. Yaklaşık 5-10 dakika sürer.
4. Bitince çalıştırmanın sayfasında **Artifacts** altından `sise-vurma-debug-apk` dosyasını indirin (zip içinden `.apk` çıkar).
5. APK'yı telefona aktarıp açın. Android, "bilinmeyen kaynaklardan yükleme" izni isteyecektir; izin verin.

#### Yol B: Kendi bilgisayarınızda

Gerekenler: **Node 22+**, **JDK 21**, **Android Studio** (Android SDK'yı kurar; `ANDROID_HOME` değişkeni SDK klasörünü göstermeli).

```bash
npm install
npm run apk        # sonuç: dist/sise-vurma-debug.apk
```

Telefonda USB hata ayıklama açıksa `adb install dist/sise-vurma-debug.apk` ile de yükleyebilirsiniz.
Elle yapmak isterseniz `npm run apk` içindeki adımlar: `npm run prepare:www` → `npx cap add android` →
`npm run android:patch` → `npx cap sync android` → `cd android && ./gradlew assembleDebug`.

#### Test reklamından gerçek reklama geçiş

Varsayılan ayar **test reklamıdır** (`useTestAds: true`; Google'ın herkese açık örnek kimlikleri, gelir getirmez, güvenle denenir).
Yayına çıkarmadan önce:

1. [AdMob](https://admob.google.com)'da uygulama ve **Ödüllü** reklam birimi oluşturun. İki farklı kimlik alırsınız:
   *uygulama kimliği* (`ca-app-pub-…~…`, **~** içerir) ve *reklam birimi kimliği* (`ca-app-pub-…/…`, **/** içerir).
2. İkisini de **yalnızca** `config/ads.config.js` içine yazın: `admob.appId.android` ve `admob.rewardedAdUnitId.android`.
   Kodun içinde hiçbir reklam kimliği yoktur. Yanlış alana yanlış kimlik yazılırsa `android-patch` açık bir hata verir.
3. Aynı dosyada `useTestAds: false` ve `allowQueryOverrides: false` yapın.
4. `capacitor.config.json` içindeki `appId` değerini kendi paket adınızla değiştirin (örn. `com.sirketim.sisevurma`).
5. Google Play'e yüklemek için imzalı **sürüm** paketi gerekir (Android Studio → Build → Generate Signed Bundle).
   `npm run apk` yalnızca kendi cihazınızda denemek için **debug** APK üretir.

## 4. Proje yapısı

```
index.html                 Giriş sayfası (yatay çevir uyarısı, sahte reklam penceresi)
config/ads.config.js       Reklam ayarları ve reklam birimi kimlikleri (tek yer)
lib/phaser.min.js          Phaser 3.90.0 (MIT)
assets/fonts/              Baloo 2 yazı tipi (OFL, Türkçe karakterli)
src/main.js                Ekran ölçüleri, Phaser kurulumu, yön değişimi
src/core/                  Saf mantık: state (puan/top/bölüm), levels, ads, audio, storage
src/game/                  physics.js (fizik + çözücü), aim.js (girdi), sling.js, fx.js (efektler)
src/gfx/                   Kodla çizilen dokular: cam şişe + sıvı, platform, arka plan, top
src/ui/                    HUD, düğmeler, pencereler, sahte reklam ekranı
src/scenes/                Boot, Menu, Game sahneleri
tests/                     core.test.js, android-patch.test.js, physics.test.js, e2e/run.js (tarayıcı testi)
tools/                     serve.js (yerel sunucu), prepare-www.js, android-patch.js, build-apk.js (APK)
.github/workflows/         android-apk.yml (APK'yı bulutta derler)
```

**Teknik notlar:** Mantıksal ekran yüksekliği 720, genişliği cihaz oranına göre 1152–1584 arasıdır
(Phaser `FIT`); yüksek çözünürlüklü ekranlarda tuval 2 kata kadar büyütülür. Fizik sabit 1/240 sn
adımla çalışır, bu yüzden hızlı top ince şişe boyunlarından geçmez ve sonuç ekran yenileme hızından
bağımsızdır. Şişeler statik gövdelerdir (devrilmez), çarpınca sallanır, sıvı yaylı biçimde çalkalanır.

## 5. Testler

```bash
npm run test:fast     # 32 mantık testi (puan, top hakları, bölümler, reklam akışı) + 9 Android yama testi ~1 sn
npm test              # + fizik testi: 12 bölümdeki 72 şişenin hepsi vurulabilir mi (1280 genişlik, ~1-2 dk)
npm run test:full     # aynısı 960/1280/1600 genişliklerde, 216 şişe (~4 dk)
npm i -D playwright && npx playwright install chromium
npm run test:e2e      # gerçek Chromium'da fare/dokunma ile 8 senaryo (~5 dk, yazılım GL yavaş)
```

`tests/e2e/run.js`, "son kontrol" listesindeki senaryoları oyunun gerçek arayüzünde çalıştırır
(K1 2 top · K2 yanlış atış · K3 "Topların bitti!" · K4 reklamla 3 top · K5 reklam tamamlanmazsa/yüklenmezse ·
K6 puan ve bölüm geçişi · K7 şeffaf cam + 6 sıvı rengi (piksel analizi) · K8 dokunmatik telefon).
Ekran görüntüleri `tests/e2e/screenshots/` klasörüne kaydedilir. Tek senaryo: `node tests/e2e/run.js --only=K7`.

## 6. Lisanslar

Phaser: MIT (`lib/PHASER-LICENSE.md`). Baloo 2 yazı tipi: SIL OFL 1.1 (`assets/fonts/OFL-Baloo2.txt`).
