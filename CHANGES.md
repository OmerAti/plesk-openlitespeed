# Changelog

## 1.3.9

- The Web engine page no longer carries a domain table. The numbers that matter
  (mapped domains, PHP execution mode, automatic sync, last sync) stay in the
  status card; the per-domain certificate and health details are what the
  Virtual hosts page and the logs already show.
- The update answer is one compact line inside the service status card instead of
  a separate card below it, and the quick links sit side by side instead of
  stacking.
- The first-run page finishes with a single action - "Open the dashboard". The
  "Web engine" button next to it was redundant.
- Fixed: the PHP pool this extension starts for Plesk's own service hostnames
  (webmail) never came up. Plesk's php-fpm rejects a `syslog` entry, so the pool
  died during initialization, and its socket sat in a directory the OpenLiteSpeed
  worker (`nobody:psaserv`) is not allowed to enter. The pool files now live in
  their own directory created 0710 root:psaserv, exactly like Plesk's
  `/var/www/vhosts/system/<domain>`, so Apache can leave the request path.
- Removing an extension copy of stale pool files left over from the old location
  is done by the install script.

## 1.3.8

- Simplified the Dashboard: it no longer repeats the OpenLiteSpeed version,
  process id, worker count, control method, installation directory,
  configuration format or virtual host count, no longer spells out the web admin
  console's port, TLS, binding, reachability and user list (the console button
  in the page header already opens it), and no longer shows the operating system
  or the nginx/Apache service names. Quick links now carry icons.
- Simplified the Web Engine page: the public listener table and the summary line
  under it (PHP backend, public IP addresses, worker group) are gone, the front
  status line no longer prints a PID, and the domain table is headed simply
  "Domains".
- The interface no longer names the PHP backend: the execution mode reads
  "Direct on OpenLiteSpeed" or "Apache backend", and the per-domain badge reads
  "Direct".
- Fixed: reinstalling the extension never brought the first-run page back. Both
  the stored "already been through it" record and the mere presence of the front
  block silenced it permanently. Installing now clears that record, and on a
  server that already runs on OpenLiteSpeed the page shows the finished state
  once and then closes its own gate, so the next click lands on the panel
  instead of looping.
- Fixed: the configuration editor's save confirmation stayed in English in the
  Turkish panel - its catalog key was written in single quotes, so the `\n`
  escapes in the source never became the newlines the catalog stores.

## 1.3.7

- Fixed: with "PHP execution: direct" a PHP site returned **500 — Failed opening
  required 'wp-config.php': Permission denied**. The request never reached
  Plesk's php-fpm pool, for two separate reasons:
  - The generated `type fcgi` extprocessor let OpenLiteSpeed take ownership of
    the pool. It then tried to lock a pid file beside Plesk's socket
    (`php-fpm.sock.pid: Permission denied` as `nobody`) and dropped the handler.
    The extprocessor now declares `autoStart 0` and no `path`/`instances`, which
    is how OpenLiteSpeed expresses "this backend is started by somebody else,
    only connect to it". (`path` is only required when the server starts the
    app, so `lshttpd -t` no longer reports `missing <path>`.)
  - The `.php` association was written into the `virtualHost` registration
    inside `httpd_config.conf`, where the parser accepts it but never uses it.
    It belongs in the virtual host's own `vhconf.conf`, as
    `scripthandler { add fcgi:<app> php }`, and is now generated there - which is
    also the only place it can override the stock `lsphp` handler.
    The plain-text form is the one the server writes itself; the `addsuffix`
    node the web admin console shows has no plain-text syntax and is rejected
    (`Not support [addsuffix php]`).
  PHP is now served by the pool Plesk runs, so the answer comes back as
  `Server: LiteSpeed` with the PHP version selected for the subscription, and
  Apache stays stopped.
- Fixed: "Graceful Reload" failed with `Usage: lswsctrl {start|stop|restart|…}`.
  lswsctrl has no `graceful` verb; it is `reload`. The same mapping is applied to
  `systemctl` (whose `lsws.service` defines `ExecReload`).
- Fixed: the reported PID, uptime and worker count could name a process that was
  not OpenLiteSpeed. LiteSpeed rewrites its process titles
  (`openlitespeed (lshttpd - main)`) and ships no pid file at the usual paths, so
  the search by binary path found nothing and the fallback matched unrelated
  processes; the master is now found by its own process title.
- Verified on a live Plesk box: single click into direct mode, the reverse switch
  back to nginx + Apache and the single click again all completed with no
  downtime, no generated leftovers and no `lshttpd -t` errors; HTTP 301 to HTTPS,
  WordPress front page and `wp-login.php` answered 200, a missing page answered
  404, and a graceful restart kept every site serving.

## 1.3.1

- Fixed: "PHP execution: direct" left every domain on the Apache backend
  (`PHP direct domains: 0`), so Apache stayed running and the request still made
  the hop the mode was meant to remove. A php-fpm pool socket is a unix socket,
  not a regular file, and `is_file()` returns false for it, so
  `plesk_php_socket()` never found Plesk's `/var/www/vhosts/system/<domain>/php-fpm.sock`
  and no domain ever qualified as direct-capable. Pool sockets are now detected
  with `file_exists()`, which sees sockets too. After this fix a domain with a
  live Plesk pool and a document root goes direct, Apache is stopped and
  disabled when nothing needs it, and PHP is answered with
  `Server: LiteSpeed` and no Apache hop.
- Re-syncing domains, or running the welcome page single click again, is enough
  to pick the pools up; no configuration from earlier versions has to be undone
  by hand.

## 1.3.0

- First run now opens a welcome page with a single button. It checks the server,
  installs OpenLiteSpeed with the distribution package manager when it is
  missing (LiteSpeed's official repository for RHEL 8/9/10 and Debian/Ubuntu
  releases that publish a 1.8+ build), maps every Plesk domain, moves 80/443 to
  OpenLiteSpeed and health-checks the result - all in a detached root worker so
  a package install cannot time out in the panel. The page hides itself once the
  administrator has been through it, appears again after the extension is
  removed and reinstalled, and can be shown on purpose from the Settings page.
- OpenLiteSpeed can now be the whole web front: a new "PHP execution" choice on
  the Web Engine page runs PHP directly in the php-fpm pool Plesk already starts
  for each domain (`type fcgi` extprocessor on the domain's
  `/var/www/vhosts/system/<domain>/php-fpm.sock`) and serves the domain's
  docroot itself. The Apache hop is gone, which is the point of moving to
  OpenLiteSpeed. The Plesk PHP version stays authoritative - no `lsphp` packages
  are installed and no Plesk configuration is edited.
- Apache is stopped and disabled whenever no domain needs it any more, and the
  OpenLiteSpeed worker group is switched to the Plesk web group (`psaserv`) so
  it can read the docroots and connect to the sockets. The original group is
  recorded and restored, and Apache is re-enabled and started again on
  "Switch to Plesk", on a re-sync that needs it, and on uninstall.
- Domains that Plesk forces to HTTPS keep that behaviour without Apache: the
  front generates a small redirect virtual host with a `RewriteRule` 301 to the
  https listener.
- OpenLiteSpeed rewrites `httpd_config.conf` in its own block order when the
  server restarts, which can orphan the extension's begin marker. The generated
  stanzas are therefore recognised and cleaned by their own names
  (`PleskHttpBackend`, `PleskTls*`, `pl-*` vhosts, `PleskPlain-*`/`PleskSecure-*`
  listeners), and switching back to Plesk now also removes the generated `pl-*`
  vhost directories from disk instead of leaving them behind.
- Per-domain PHP mode, the Apache verdict and the worker group are all shown on
  the Web Engine page (new PHP column in the domain table, PHP execution row in
  the status grid, listener footer).
- Results are presented as panel-style toasts instead of inline message blocks,
  and a Back button returns to the Plesk admin pages, so the module no longer
  feels like a dead end.
- Interface cleanup: the verbose `BUILD built: ...` text is stripped from the
  reported version, the Uptime badge is gone, the subtitle reads "OpenLiteSpeed
  for Plesk", the redundant firewall/service-name hints are removed, select
  controls get proper padding instead of clipped text and the meta grids are
  restyled.
- Fixed: "Yapılandırmayı eşitle" returned
  `Zend_Controller_Action_Exception — Action "enginesync" does not exist`. Zend
  lowercases the action token, so the controller actions and the view links now
  all use the lowercase names.

## 1.2.0

- Turkish and English interface. The module follows the panel language
  automatically; a header language pill and an "Interface language" setting
  override it for this module only. All screen text and all root-helper error
  messages are translated, including messages that carry a dynamic tail
  (paths, PIDs, `lshttpd -t` output).
- Web Admin Console access: the dashboard and the header show the detected
  console listener (port, TLS, bind, reachability, existing usernames only) and
  open it in a new tab. Console credentials can be created from the Settings
  page - bcrypt hashed exactly like `admpass.sh`, written only to the console
  credentials file, never to the Plesk database. A manual console URL override
  is available for load-balanced or firewalled setups.
- OpenLiteSpeed logo and a modernised interface: redrawn layout with status
  badges, icon action buttons, a service toggle, card-based dashboard, and the
  author/version footer.
- Fixed: `<select>` controls were unreadable when the panel ran in dark mode.
  Form controls now declare their own colours and `color-scheme`.
- Fixed: "A valid --vhost name is required for vhost logs." The vhost log views
  now default to a real virtual host, resolve log paths across the layouts
  OpenLiteSpeed uses, and the UI shows an explanatory message instead of an
  error when the installation genuinely has no vhost.
- Uninstall now reverts: removing the extension runs the same switch back to
  nginx + Apache (cron timer removed, front block stripped, generated vhosts
  deleted, nginx re-enabled and started, OpenLiteSpeed released from 80/443)
  with a timestamped backup and a health check. Reinstalling afterwards stays a
  deliberate choice.
- Cross-distribution support: Apache and nginx unit names are detected instead
  of assumed (`httpd` on RPM families, `apache2` on Debian families), the
  operating system family is reported on the dashboard, service control falls
  back where systemd is absent, and HTTP probing falls back from `curl` to a
  plain socket request.
- System service registration moved to `hooks/SystemServices.php`; Plesk loads
  hooks by file name, so the previous `hooks/Services.php` never registered the
  switch.
- Added a second entry point under Tools & Settings -> Server Management, with
  an explicit menu order for the admin navigation entry.

## 1.1.0

- New "Web Engine" tab: one-click switch of the public web front between
  nginx + Apache (Plesk stock) and OpenLiteSpeed, in both directions.
- OpenLiteSpeed front generation reads Plesk's own nginx configuration:
  per-domain SNI certificates, all public IPs, port 80/443 listeners and an
  Apache backend proxy, so existing and newly added domains keep working.
- Every switch is protected by `lshttpd -t` validation, a timestamped backup,
  a live HTTP health check of all domains and automatic rollback on failure.
- Domains that were already unreachable before the switch are detected first
  and excluded from the rollback decision.
- Each HTTPS domain gets its own Apache TLS upstream, so pooled backend
  connections stay single-host and mod_ssl never answers HTTP 421
  (the equivalent of nginx `proxy_ssl_name $host`).
- Automatic front sync: a `/etc/cron.d` timer compares a fingerprint of Plesk's
  domain/certificate/backend facts every 2 minutes and rebuilds the front only
  when something actually changed. New, renamed or removed domains and renewed
  certificates are picked up without touching the panel; an idle server is never
  reloaded. The timer is installed on the switch to OpenLiteSpeed and removed on
  the way back and on uninstall.
- Re-sync button rebuilds the front mapping on demand after Plesk domain changes;
  applying a rebuild that changes no listener port uses a graceful reload, so
  live connections survive. Only an IP change forces a full stop/start.
- Generated vhost directories left behind by removed domains are pruned on every
  rebuild; nginx is stopped + disabled while OpenLiteSpeed fronts traffic and is
  re-enabled and started again on the way back.
- Configuration editor now supports the OpenLiteSpeed 1.8+ plain-text format
  (httpd_config.conf / vhconf.conf) as well as the legacy XML format, and
  validates with error-line detection instead of exit code.
- OLS admin listener is moved off port 7080 (Plesk Apache backend) when the
  front is installed.

## 1.0.0

- Initial release by OmerAti - Omer ATABER.
- Dashboard: OpenLiteSpeed status, version, uptime, PID, workers, listeners.
- Start / Stop / Restart / Graceful Reload / Test Configuration actions.
- Service switch registered in Plesk "Services Management" (pm_Hook_SystemServices).
- Menu entry "OpenLiteSpeed" in the admin Server Management section.
- Configuration editor for httpd_config.xml and vhost configs with backup,
  lshttpd -t validation and automatic rollback.
- Virtual host list with docroot/TLS info and per-vhost log shortcuts.
- Log viewer for server/vhost/admin error and access logs.
- Settings: custom OpenLiteSpeed installation prefix override.
