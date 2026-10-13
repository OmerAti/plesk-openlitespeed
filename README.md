# OpenLiteSpeed — Plesk Eklentisi

Plesk Obsidian içinden **OpenLiteSpeed** web sunucusunu yönetme eklentisi.
LiteSpeed'in resmi Plesk eklentisindeki yönetim deneyiminin OpenLiteSpeed karşılığı:
durum takibi, açma/kapama anahtarı, restart, kesintisiz (graceful) reload, yapılandırma
düzenleyici, sanal host listesi, canlı loglar ve web yönetim konsoluna erişim.
Arayüz **Türkçe ve İngilizce** çalışır.

**Yazar / Vendor:** OmerAti - Ömer ATABER

## Özellikler

- **Menüde "OpenLiteSpeed"** olarak görünür:
  - Admin paneli sol menüsü → *Server Management* bölümü (CustomButtons hook'u)
  - Tools & Settings → *Server Management* bölümü (ikinci erişim yolu)
- **Dil: Türkçe + İngilizce**: panel dilini otomatik izler; başlıktaki dil düğmesi veya
  Ayarlar → *Arayüz dili* alanı yalnızca bu eklenti için geçersiz kılar. Ekran metinleri
  ve root helper'ın hata mesajları (sonunda yol/PID/test çıktısı taşıyanlar dahil)
  çevrilidir; çeviri yoksa İngilizceye düşer.
- **Arayüz yenilemeleri (v1.3.0)**: işlemlerin sonucu Plesk tarzı **toast** bildirimleriyle
  gösterilir (otomatik kapanır, kapatılabilir), başlıkta panel'e dönen **Geri** düğmesi
  vardır, başlık altı "Plesk için OpenLiteSpeed" olarak sadeleşti, gereksiz BUILD/uptime
  metinleri ve tekrar eden ipuçları kaldırıldı, açılır kutuların içi artık sıkışmıyor.
- **İlk çalıştırma sihirbazı (v1.3.0)**: eklenti ilk açıldığında tek düğmeli bir
  "karşılama" sayfası gelir. Sayfa sunucuyu kontrol eder, OpenLiteSpeed kurulu değilse
  **dağıtım paket yöneticisiyle kurar** (yalnızca LiteSpeed'in gerçekten yayımladığı
  Linux aileleri, sürümler ve mimariler önerilir; desteklenmeyen bir dağıtımda düğme
  pasif kalır ve sayfa tek bir neden gösterir), sonra Plesk domain'lerini eşler,
  80/443'ü OpenLiteSpeed'e geçirir ve sağlık kontrollü geçişi başlatır. Kurulum + geçiş, panel API zaman aşımına
  takılmasın diye **kök yetkisinde, ayırık (detached) bir worker** içinde yürür; ilerleme
  sayfası kendini yeniler ve sayfa günlüğü (`setup.log`) ekranda görünür.
  - Sihirbazdan geçtiysen sayfa bir daha görünmez.
  - Eklenti **kurulduğunda veya yükseltildiğinde** (kaldır-yeniden kur ya da
    `plesk bin extension --install`) sayfa tekrar gelir; kurulum betiği "geçildi"
    kaydını siler. Ön yüz zaten OpenLiteSpeed ise sayfa tek bir "hazır" onayı
    gösterir ve kapısını kendisi kapatır, bir sonraki tıklama doğrudan panele gider.
  - Kaldırma sırasında da durum dosyaları silinir.
  - İsteyerek tekrar görmek istersen: Ayarlar → *İlk çalıştırma kurulumu* →
    "Kurulum sayfasını tekrar göster". Bu işlem sunucuda hiçbir şeye dokunmaz.
- **Web Yönetim Konsolu erişimi**: her sayfanın başlığında tek bir düğme; adres
  algılanan konsol dinleyicisinden (port + TLS + bağlama) üretilir ve yeni sekmede
  açılır. Port/bağlama/erişilebilirlik dökümü dashboard'da tekrar edilmez, detaylar
  Ayarlar sayfasında durur. Konsol kullanıcısı
  Ayarlar sayfasından oluşturulur; parola `admpass.sh` ile birebir aynı şekilde bcrypt
  hash'lenip yalnızca konsol kimlik dosyasına yazılır. İstersen konsol adresini elle
  girebilirsin (Load balancer / güvenlik duvarı arkası için).
- **Servis yönetimi anahtarı**: *Tools & Settings → Services Management* sayfasında
  nginx/Apache gibi **açık/kapalı switch** olarak listelenir (`pm_Hook_SystemServices`).
- **Dashboard**: çalışıyor/durdu rozeti, servis anahtarı, başlat-durdur-yeniden
  yükle/kesintisiz yenile/kanıtlama düğmeleri, dinleyici (listener) listesi ve
  **ikonlu hızlı bağlantılar**. Sürüm, PID, worker sayısı, kontrol yöntemi, kurulum
  dizini ve işletim sistemi/servis adı gibi teşhis bilgileri arayüzde tekrar
  edilmez; hepsi zaten ilgili sayfaların içinde (konsol detayları Ayarlar'da).
- **Start / Stop / Restart / Graceful Reload / Test Configuration** butonları.
- **Yapılandırma editörü**: `httpd_config.conf` (1.8+ düz metin) ve `httpd_config.xml`
  (eski XML) ile vhost konfları; kaydetmeden önce otomatik **yedek** alır ve
  `lshttpd -t` testiyle doğrular, test başarısız olursa **geri alır (rollback)**.
- **Web Engine sekmesi**: web trafiğini bitiren ön yüzü tek tıkla
  **nginx + Apache** ↔ **OpenLiteSpeed** arasında değiştirir (LiteSpeed eklentisindeki
  gibi düşünebilirsin; iki yön de geri alınabilir).
  - OpenLiteSpeed ön yüze geçince: Plesk'in nginx konfları okunur (domain'ler, SNI
    sertifikaları, public IP'ler, Apache backend portları), `httpd_config.conf` içine
    imarli bir blok olarak yazılır, 80/443 OpenLiteSpeed'e geçer, nginx durdurulur ve
    devre dışı bırakılır. Panel (8443) bundan etkilenmez.
  - **PHP çalıştırma seçimi** (Web Engine sayfası):
    - **Doğrudan OpenLiteSpeed (Apache durdurulur)** — varsayılan önerilen mod;
      arayüzde bu seçenek "Direct on OpenLiteSpeed / Doğrudan" olarak yazılır.
      OpenLiteSpeed docroot'u kendi servis eder, PHP'yi Plesk'in o domain için zaten
      başlattığı `php-fpm.sock` havuzuna (`type fcgi` extprocessor) bağlar; istek
      Apache'a hiç uğramaz. OpenLiteSpeed worker grubu Plesk web grubuna (`psaserv`)
      çevrilir ki docroot'ları ve socket'i okuyabilsin; eski değer kaydedilir ve geri
      alınır. PHP sürümü hâlâ Plesk'ten yönetilir, eklenti `lsphp` paketi kurmaz.
      Hiçbir domain Apache'a muhtaç değilse Apache **durdurulur ve devre dışı
      bırakılır**. HTTPS'e zorlayan domain'ler için 301 yönlendirmeli ayrı bir vhost
      üretilir (Apache olmadan da aynı davranış).
    - **Apache arka ucu üzerinden** — eski davranış: OpenLiteSpeed 80/443'ü tutar,
      PHP'yi Apache backend'ine (7080/7081) proxy'ler. php-fpm soketi ya da docroot
      algılanamayan domain'ler otomatik olarak bu modda kalır.
  - nginx + Apache'ye dönüş: üretilen blok ve üretilen vhost'lar silinir, 80/443
    bırakılır, Apache yeniden enable/start edilir, OpenLiteSpeed worker grubu eski
    değerine döner, nginx yeniden enable/start edilir ve OpenLiteSpeed kendi
    portlarında tekrar başlatılır.
  - Yakalanamayan/yeni eklenen domain'ler catch-all (`map *`) vhost sayesinde
    **senkronizasyon gerektirmeden** çalışmaya devam eder; mevcut domain'ler kendi
    sertifikalarıyla eşleşir. Apache arka ucu kullanılan modda her HTTPS domain'i
    kendi Apache TLS upstream'ini kullanır (nginx'in `proxy_ssl_name $host`
    davranışının karşılığı), böylece havuzda birleştirilen arka bağlantılar tek
    host'a ait olur ve mod_ssl **HTTP 421** vermez.
  - **Otomatik senkron**: ön yüze geçildiğinde `/etc/cron.d/plesk-openlitespeed-front`
    kurulur; Plesk domain/sertifika/backend bilgilerinin parmak izini 2 dakikada bir
    karşılaştırır, **değişiklik varsa** bloğu yeniden üretir. Yeni domain ekleme,
    silme veya sertifika yenileme panel'e dokunmadan kendiliğinden yansır; değişen
    bir şey yoksa OpenLiteSpeed hiç yeniden yüklenmez. Dinleyici portları değişmiyorsa
    graceful reload kullanılır (canlı bağlantılar kopmaz), yalnızca IP değişiminde
    tam durdur/başlat yapılır. nginx'e dönüşte ve eklenti kaldırılırken bu zamanlayıcı
    silinir; istersen dosyayı elle silerek otomatik senkronu kapatabilirsin.
  - Her geçişte: zaman damgalı yedek → `lshttpd -t` hatası olmaması → nginx durdurulur
    → OpenLiteSpeed başlatılır → **tüm domain'lerde HTTP sağlık kontrolü** → başarısızsa
    otomatik tam geri alma. Geçişten *önce* bozuk olan domain'ler ayrıma alınır; kötü
    bir site sağlıklı bir geçişi rollback'e zorlayamaz.
  - "Re-sync domains" butonu: Plesk'te domain/sertifika değiştiğinde ön yüz bloğunu
    yeniden üretir.
  - **Kaldırmada geri alma (v1.2.0)**: eklenti kaldırılırken aynı nginx + Apache dönüşü
    otomatik çalışır (cron zamanlayıcısı silinir, imli blok ve üretilen vhost'lar
    kaldırılır, Apache yeniden etkinleştirilip başlatılır, OpenLiteSpeed worker grubu
    eski değerine döner, nginx yeniden etkinleştirilip başlatılır, OpenLiteSpeed
    80/443'ü bırakır).
    Böylece OpenLiteSpeed sahipsiz port bırakamaz; yeniden kurmak istersen eklentiyi
    tekrar kurarsın.
- **Sanal hostlar**: OpenLiteSpeed vhost listesi, docroot, TLS durumu, hızlı log/konfig linkleri.
- **Loglar**: sunucu error/access, vhost logları ve admin console logları (satır sayısı seçilebilir).
  Vhost logları artık otomatik olarak gerçek bir sanal hosta çözülür; host yoksa hata
  yerine açıklayıcı mesaj gösterilir.
- **Güvenli mimari**: tüm root işlemleri `pm_ApiCli::callSbin()` ile sabit komutlu
  `sbin/openlitespeed` helper'ı üzerinden yürür; kullanıcı girdisi asla shell'e gitmez,
  dosya yolları `realpath` ile `conf/` dizinine hapsedilir.

## Gereksinimler

- Plesk Obsidian 17.8+ (meta.xml formatı destekleyen sürümler)
- Sunucuda OpenLiteSpeed — ols1clk ile, paket yöneticisiyle veya ilk çalıştırma
  sihirbazının tek düğmesiyle kurulabilir (varsayılan önek: `/usr/local/lsws`)
- Web Engine ön yüz geçişi için OpenLiteSpeed **1.8+** (düz metin `httpd_config.conf`)
- Linux (unix)

## Klasör Yapısı

```
openlitespeed/
├── meta.xml                     # Eklenti tanımı (id, sürüm, vendor: OmerAti - Ömer ATABER)
├── DESCRIPTION.md / CHANGES.md
├── htdocs/
│   ├── index.php                # Panel giriş noktası (pm_Context + pm_Application)
│   ├── assets/ols.css           # Arayüz (açık ve koyu panel temasında da okunur)
│   ├── assets/ols.js            # Onay diyalogları, otomatik gönderim, editör kilidi
│   └── images/icon.svg
├── plib/
│   ├── controllers/IndexController.php
│   ├── hooks/
│   │   ├── SystemServices.php   # pm_Hook_SystemServices  → Services Management switch
│   │   ├── Navigation.php       # pm_Hook_Navigation      → modül alt sayfaları
│   │   └── CustomButtons.php    # pm_Hook_CustomButtons   → menüde "OpenLiteSpeed"
│   ├── library/Ols/Manager.php  # callSbin sarmalayıcı (JSON protokol)
│   ├── library/Ols/Service.php  # pm_SystemService_Service (start/stop/restart/isRunning)
│   ├── library/Ols/L10n.php     # TR/EN çeviri + helper hata mesajı ön ek eşlemesi
│   ├── locales/tr.php           # İngilizce kaynak metin → Türkçe karşılıklar
│   ├── scripts/post-install.php # sbin helper'lara +x, var dizini
│   ├── scripts/pre-uninstall.php# kaldırmada nginx + Apache'ye otomatik dönüş +
│   │                            # sihirbaz durum dosyalarının silinmesi
│   └── views/scripts/index/*.phtml   # dashboard/webengine/vhosts/logs/editor/settings + wizard
└── sbin/
    ├── openlitespeed            # sh wrapper (php bulur)
    └── openlitespeed.php        # root helper: info/status/start/stop/restart/graceful/
                                 # test/vhosts/logs/config-list/config-get/config-save/
                                 # engine-status/engine-switch-ols/engine-switch-nginx/
                                 # engine-sync/engine-sync-check/engine-revert/
                                 # engine-cron-install/engine-cron-remove/
                                 # admin-console/admin-console-user/
                                 # setup-status/setup-run/setup-worker/setup-skip/setup-reset
```

## Paketleme (Windows)

Zip'in **kök dizininde** `meta.xml` bulunmalı. PowerShell:

```powershell
cd openlitespeed
Compress-Archive -Path * -DestinationPath ..\openlitespeed.zip
```

Git Bash / Linux:

```bash
cd openlitespeed && zip -r ../openlitespeed-1.3.8.zip .
```

## Kurulum

1. Plesk → **Extensions → Upload** → `openlitespeed.zip` (veya CLI):
   ```bash
   plesk bin extension --install /tmp/openlitespeed.zip
   ```
2. Kurulumdan sonra: Tools & Settings → **OpenLiteSpeed**.
3. OpenLiteSpeed standart olmayan bir dizindeyse eklentinin **Settings** sekmesinden
   kurulum dizinini girin.

## Notlar / Kısıtlar

- Eklenti yalnızca **admin** erişimlidir (`$_accessLevel = ['admin']`).
- Plesk'in desteklediği Linux dağıtımlarında çalışacak şekilde servis adları
  **algılanır**: Apache birimi RPM tabanlılarda (AlmaLinux/Rocky/CentOS/CloudLinux/
  Oracle/Fedora) `httpd`, Debian tabanlılarda (Debian/Ubuntu) `apache2`'dir; systemd
  yoksa init.d ve `lswsctrl` yollarına düşülür, `curl` yoksa HTTP kontrolü düz socket
  ile yapılır.
- Graceful reload `SIGUSR1` ile yapılır; bazı eski OLS derlemlerinde desteklenmeyebilir —
  başarısız olursa tam restart'a düşer.
- Config değişikliği ancak **restart** sonrası aktiftir; editör bunu hatırlatır.
- Web Engine geçişi **OpenLiteSpeed 1.8+** düz metin konfigürasyonu (`httpd_config.conf`)
  ve sunucuda tanımlı bir **nginx servisi** ister (geri dönüş yolu için). Eski XML
  konfigürasyonunda geçiş reddedilir, hiçbir şeye dokunulmaz.
- İlk çalıştırma sihirbazının **paket kurulumu** yalnızca LiteSpeed'in resmî deposunda
  gerçekten paketi olan kombinasyonlarda teklif edilir: RPM tabanlılarında CentOS/RHEL
  8, 9 ve 10 (x86_64 ve aarch64; `crb`/`powertools` kanalı gerekirse açılır) ve Debian
  tabanlılarında Ubuntu 18.04–24.04 ile Debian 11–13 (amd64/arm64). Amazon Linux, Fedora
  ve SUSE gibi diğer dağıtımlarda kurulum teklif edilmez, sayfa nedenini gösterir;
  el7 için yalnızca eski XML derlemesi bulunduğundan sihirbaz orada da kurulum yapmaz.
- Sunucuda LiteSpeed'in **ücretli (enterprise) `litespeed` paketi** kuruluysa sihirbaz
  kurulumu reddeder; iki derlemenin aynı portta savaşmasını önlemek için tek seçenek
  mevcut motoru kullanmaya devam etmektir.
- Eklenti Plesk'in kendi konflarını (`/etc/nginx/plesk.conf.d`, Apache conf'ları,
  subscription'lar) **değiştirmez**; yalnızca OpenLiteSpeed konfigine kendi imli bloğunu
  yazar ve geri alırken siler.
- **Doğrudan php-fpm modunda** OpenLiteSpeed'in docroot'ları okuyabilmesi için sunucu
  `group` değeri `psaserv` yapılır (eski değer durum dosyasına kaydedilir ve geri
  alınır). Apache istek yolundan çıktığı için **Plesk Web İstatistikleri** bu süre
  dolmaz; OpenLiteSpeed her vhost için kendi erişim günlüğünü yazar.
- Geçişte üretilen dosyalar: `/usr/local/lsws/conf/httpd_config.conf` içine işaretli blok,
  `conf/vhosts/pl-*` + `vhosts/pl-*` klasörleri, durum dosyası
  `/usr/local/psa/var/modules/openlitespeed/engine/state.json`, aynı yerdeki `backups/`
  ve otomatik senkron zamanlayıcısı `/etc/cron.d/plesk-openlitespeed-front`.
