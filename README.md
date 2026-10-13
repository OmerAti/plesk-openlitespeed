# OpenLiteSpeed — Plesk Eklentisi

Plesk Obsidian içinden **OpenLiteSpeed** web sunucusunu yönetme eklentisi.
LiteSpeed'in resmi Plesk eklentisindeki yönetim deneyiminin OpenLiteSpeed karşılığı:
durum takibi, açma/kapama anahtarı, restart, kesintisiz (graceful) reload, yapılandırma
düzenleyici, domainler, canlı loglar ve web yönetim konsoluna erişim.
Arayüz **Türkçe ve İngilizce** çalışır.

## Gereksinimler

- Plesk Obsidian 17.8+ (meta.xml formatı destekleyen sürümler)
- Sunucuda OpenLiteSpeed — ols1clk ile, paket yöneticisiyle veya ilk çalıştırma
  sihirbazının tek düğmesiyle kurulabilir (varsayılan önek: `/usr/local/lsws`)
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

## Kurulum

1. Plesk → **Extensions → Upload** → `openlitespeed.zip`:

   (veya CLI)
   
   ```bash
   plesk bin extension --install /tmp/openlitespeed.zip
   ```
**Yazar / Vendor:** OmerAti - Ömer ATABER
