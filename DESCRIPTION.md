# OpenLiteSpeed for Plesk

Manage OpenLiteSpeed directly from the Plesk Obsidian admin panel — the OpenLiteSpeed
counterpart of the LiteSpeed management experience you know from the official extension.

## Highlights

- "OpenLiteSpeed" entry in the admin menu (Server Management) and in Tools & Settings.
- Turkish and English interface: follows the panel language automatically, with a
  header language pill and an explicit per-module override.
- Service switch on the "Services Management" page, just like nginx and Apache:
  start/stop with one toggle.
- Dashboard with live status: running state, version, PID, worker processes,
  listener ports and the detected control method (lswsctrl or systemd unit).
- One-click Start, Stop, Restart, Graceful Reload and "Test Configuration".
- Web Admin Console access: shows the detected console listener (port, TLS, bind,
  reachability) and opens it in a new tab; console logins can be created from the
  Settings page using the same bcrypt hashing as `admpass.sh`. No Plesk password is
  involved.
- Built-in configuration editor for `httpd_config.conf` / `httpd_config.xml` and
  vhost configs with automatic backup, `lshttpd -t` validation and rollback on failure.
- Virtual host overview with document roots, TLS state and quick links to logs/config.
- Live log viewer for server, vhost and admin-console logs with selectable line count.
- Web front switching between nginx + Apache and OpenLiteSpeed, health-checked and
  automatically rolled back on failure. In the direct mode OpenLiteSpeed serves the
  domain docroots and runs PHP in the php-fpm pool Plesk already started for that
  domain, so Apache is stopped and disabled and a request makes one hop instead of
  three; Plesk stays authoritative for the PHP version. Switching back re-enables
  Apache, restores the original worker group and returns nginx to port 80/443.
  Uninstalling the extension reverts the web front to nginx + Apache so
  OpenLiteSpeed never keeps ports 80/443 unmanaged.
- First run opens a welcome page with a single button. It checks the server, installs
  OpenLiteSpeed with the distribution package manager when it is missing (only for the
  Linux families, releases and architectures LiteSpeed actually publishes), maps every
  Plesk domain, moves 80/443 to OpenLiteSpeed and starts the health-checked switch — all
  in a detached root worker, so a package install cannot time out inside the panel. The
  page hides itself once you have been through it, appears again after the extension is
  removed and reinstalled, and can be shown on purpose from the Settings page.
- Panel-style result toasts and a Back button into the Plesk admin pages.
- Works across the Linux distributions Plesk supports: Apache/nginx service names
  (`httpd` vs `apache2`) and the OS family are detected, not assumed.
- Hardened design: all privileged operations run through a fixed-command root helper
  invoked via `pm_ApiCli::callSbin()`; user input never reaches the shell and config
  paths are jailed to the OpenLiteSpeed configuration directory.

## Requirements

- Plesk Obsidian 17.8 or newer
- OpenLiteSpeed on the same server — the welcome page installs it with the distribution
  package manager when it is missing (default prefix `/usr/local/lsws`)

## Support

Author: **OmerAti - Ömer ATABER**.
