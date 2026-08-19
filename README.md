# gnome-desktop

GNOME desktop recipes for Zainium OS. Same `ZEXBUILD` format as
[`syshub-recipes`](https://github.com/Zainium-Dynamics/syshub-recipes)
and [`userland-recipes`](https://github.com/Zainium-Dynamics/userland-recipes).
`ci/build.sh` is a direct copy of theirs, unmodified.

## Status

Work in progress. Recipes here are staged locally before publishing
into the real `syshub-recipes` tree. Not CI-runnable yet:

- `quantra-logind`: source is the `quantra-system` monorepo, not a
  fetched tarball. Rust recipe plumbing isn't wired into `ci/build.sh`
  yet (see `userland-recipes/README.md`, "Rust packages - target spec").
- `librsvg`: same Rust CI plumbing gap as `quantra-logind` (needs
  cargo + the `x86_64-zainium-linux-musl.json` target spec wired into
  `ci/build.sh`, plus a `cargo-c` recipe that doesn't exist yet).
  Written against the layout, not CI-runnable today. `gnome-commander`
  and `gnome-user-share` sidestep the same Rust gap by pinning to
  their last pre-Rust upstream release instead.
- Every other recipe here: each has at least one unpublished dependency
  somewhere in its chain (glib, wayland, alsa-lib, cups,
  NetworkManager, and others are still missing, among many more).
  Listed in each `manifest.toml`'s `depends`, none built yet. gtk4,
  mutter, polkit, and gjs (the hard blockers for `gnome-shell`) now
  have recipes, as do gdm, gnome-control-center, gnome-keyring, dconf,
  dconf-editor, gnome-flashback, gnome-panel, gvfs, nautilus, and
  evolution-data-server (the rest of a minimal working session), a
  first batch of `gnome-base`/`gnome-extra` bucket A stragglers
  (gnome-menus, libgtop, upower, geoclue, libgweather, gucharmap,
  gnome-online-accounts, polkit-gnome), and a second, larger batch
  covering the rest of `gnome-extra` plus the remaining GTK2-era
  `gnome-base` libraries (gnome-applets, librsvg, libglade,
  libgnomecanvas, libgnomekbd, eiciel, gnome-browser-connector,
  gnome-calculator, gnome-calendar, gnome-characters, gnome-clocks,
  gnome-color-manager, gnome-commander, gnome-contacts,
  gnome-directory-thumbnailer, gnome-firmware,
  gnome-integration-spotify, gnome-network-displays,
  gnome-power-manager, gnome-shell-frippery, gnome-system-monitor,
  gnome-tweaks, gnome-user-share, gnome-weather, krb5-auth-dialog,
  libgda, libgsf, mousetweaks, nautilus-dropbox, nautilus-sendto,
  nm-applet, pch-session, sushi, tecla, yelp, yelp-xsl, zenity, plus
  the two shell extensions gnome-shell-extension-dash-to-panel and
  gnome-shell-extension-pop-shell). `gnome-software` is deliberately
  excluded from this repo - it needs a git-clone-plus-custom-patches
  recipe, handled separately.

## systemd-free approach

Every recipe here targets `libelogind` (pkg-config name), not
`libsystemd`. That's satisfied by `quantra-logind`'s `libqlogind.so` +
a `libelogind.pc` alias (see `quantra-logind/`), not the real elogind
project. Most GNOME packages already have a working elogind fallback
upstream (`dependency('libelogind', required: false)`), so most
recipes need zero source patches, just the right meson flag.

Two exceptions needed a patch, both already resolved (patches carried
here, originally authored by Swagtoy for Gentoo's `gnome-session`/
`gnome-shell` ebuilds):

- `gnome-session/0001-make-systemd-optional.patch`
- `gnome-shell/0001-shell-elogind-support-to-avoid-stubbed-sd_notify.patch`

`gnome-logs` and `office-runner` are out of scope: they read the
systemd binary journal format directly, not just logind, so an
elogind-style shim doesn't help.

## Packages

| Package | Patches | Blocking deps |
|---|---|---|
| `quantra-logind` | none | Rust CI plumbing not wired |
| `gnome-session` | systemd-optional | glib, gnome-desktop, gsettings-desktop-schemas, gnome-settings-daemon |
| `gnome-shell` | elogind sd_notify | mutter, gjs, gtk4, gnome-desktop, gnome-autoar, json-glib |
| `gnome-settings-daemon` | none | geocode-glib, libgweather, libcanberra, geoclue, libpulse, polkit, upower, libgudev, wayland, alsa-lib, libnotify |
| `gnome-desktop` | none | gtk4, gdk-pixbuf, xkeyboard-config, libxkbcommon, iso-codes, cairo |
| `gsettings-desktop-schemas` | none | glib |
| `gtk4` | none | glib, cairo, pango, fribidi, harfbuzz, gdk-pixbuf, librsvg, libepoxy, graphene, wayland, mesa, libX11 family |
| `mutter` | none (`-Dlogind=true` only) | glib, gtk4, gnome-settings-daemon, libxkbcommon, at-spi2-core, colord, harfbuzz, libei, glycin, libcanberra, wayland, gnome-desktop |
| `polkit` | none (`-Dsession_tracking=elogind` only) | glib, expat, duktape |
| `gjs` | none | glib, libffi, gobject-introspection, spidermonkey, cairo |
| `gdm` | none (`-Dlogind-provider=elogind` only) | glib, libgudev, dconf, gnome-settings-daemon, gsettings-desktop-schemas, accountsservice, pam |
| `gnome-control-center` | none | gtk4, libadwaita, accountsservice, colord, upower, gcr, polkit, networkmanager, gnome-bluetooth, mit-krb5, cups, samba |
| `gnome-keyring` | none | glib, gcr, libgcrypt, p11-kit |
| `dconf` | none | glib, dbus |
| `dconf-editor` | none | dconf, glib, gtk3, libhandy |
| `gnome-flashback` | none (pkg-config CFLAGS/LIBS override) | gtk3, gnome-desktop, gnome-panel, libcanberra, polkit, ibus, upower, gnome-bluetooth, libpulse, alsa-lib, pam, gdm |
| `gnome-panel` | none (pkg-config CFLAGS/LIBS override) | gnome-desktop, gtk3, libwnck, gnome-menus, libgweather, dconf, gdm, polkit |
| `gvfs` | none (`-Dlogind=true` only) | glib, gsettings-desktop-schemas, dbus |
| `nautilus` | none | gtk4, gnome-desktop, gnome-autoar, libadwaita, libportal, icu, tinysparql, localsearch, gvfs |
| `evolution-data-server` | none | libsecret, sqlite, glib, libical, libxml2, nspr, nss, libsoup3, json-glib, icu |
| `gnome-menus` | none | glib |
| `libgtop` | none | glib |
| `upower` | none (`-Dsystemdsystemunitdir` dummy path only) | glib, dbus, libgudev, polkit |
| `geoclue` | none | glib, json-glib, libsoup3, libnotify |
| `libgweather` | none | glib, libsoup3, geocode-glib, libxml2, json-glib |
| `gucharmap` | none | freetype, glib, gtk3, libpcre2, unicode-data, at-spi2-core |
| `gnome-online-accounts` | none | glib, libadwaita, gtk4, json-glib, libsecret, libsoup3, rest |
| `polkit-gnome` | none | glib, polkit, gtk3 |
| `gnome-shell-extension-dash-to-panel` | none | glib, gnome-shell |
| `gnome-shell-extension-pop-shell` | none | glib, gnome-shell, fd |
| `gnome-applets` | none | gtk3, glib, gnome-panel, libgtop, libwnck, libnotify, upower, adwaita-icon-theme, libxml2, libgweather, gucharmap, polkit, libx11, pango |
| `librsvg` | none (Rust/cargo, same CI plumbing gap as quantra-logind) | cairo, freetype, gdk-pixbuf, glib, harfbuzz, libxml2, pango, cargo-c |
| `libglade` | none | glib, gtk2, atk, libxml2 |
| `libgnomecanvas` | none | glib, gtk2, libart_lgpl, pango |
| `libgnomekbd` | none | glib, gtk3, libx11, libxklavier |
| `eiciel` | none | acl, gtkmm, glibmm |
| `gnome-browser-connector` | python-path (meson relative-script fix) | python3, pygobject, gnome-shell |
| `gnome-calculator` | mpc-1.4, strict-aliasing | glib, libxml2, libsoup3, libgee, mpc, mpfr, gtk4, libadwaita, gtksourceview5 |
| `gnome-calendar` | none | libical, gsettings-desktop-schemas, evolution-data-server, libsoup3, libadwaita, glib, gtk4, libgweather, geoclue |
| `gnome-characters` | none | gjs, glib, gtk4, libadwaita, gdk-pixbuf, pango, gnome-desktop |
| `gnome-clocks` | none | glib, gtk4, libgweather, gnome-desktop, geocode-glib, geoclue, libadwaita |
| `gnome-color-manager` | none | glib, gtk3, colord, lcms2 |
| `gnome-commander` | none (pinned to pre-Rust 1.18.6) | glib, gtk3, gdk-pixbuf |
| `gnome-contacts` | eds-3.60 vCard API rename | folks, libgee, glib, gtk4, libadwaita, evolution-data-server, libportal, gstreamer, qrencode |
| `gnome-directory-thumbnailer` | gnome-desktop-43 API change | glib, gdk-pixbuf, gnome-desktop, gtk3 |
| `gnome-firmware` | none (`-Delogind=true` only) | gtk4, glib, fwupd, libxmlb, libadwaita |
| `gnome-integration-spotify` | command-line-parsing, correct-interface, use-glib | python3, pygobject, dbus-python, imagemagick, wmctrl, xautomation, xdotool, xwininfo |
| `gnome-network-displays` | none | glib, json-glib, libportal, gtk4, libadwaita, gst-rtsp-server, libpulse, avahi, libsoup3, networkmanager, mutter, gnome-desktop, xdg-desktop-portal |
| `gnome-power-manager` | none | glib, gtk3, cairo, upower, adwaita-icon-theme |
| `gnome-shell-frippery` | none | glib, gnome-menus, gnome-shell |
| `gnome-system-monitor` | none (`-Dsystemd=false` only) | glibmm, glib, gtk4, gtkmm4, libgtop, libadwaita, librsvg, polkit |
| `gnome-tweaks` | none | python3, pygobject, gsettings-desktop-schemas, gnome-settings-daemon, glib, gobject-introspection, gtk4, libadwaita, libgudev, gnome-desktop, libnotify, pango, mutter, gnome-shell, sound-theme-freedesktop |
| `gnome-user-share` | none (pinned to pre-Rust 47.2, `-Dsystemduserunitdir` dummy path only) | glib, mod_dnssd, apache2 |
| `gnome-weather` | typescript6 | glib, gobject-introspection, gtk4, gjs, geoclue, libadwaita, libgweather, gsettings-desktop-schemas |
| `krb5-auth-dialog` | remove-postinstall-script | gcr, glib, pam, gtk3, mit-krb5 |
| `libgda` | fix-gcc14, c23 | glib, libxml2, libxslt, readline, ncurses, sqlite, iso-codes |
| `libgsf` | none | glib, libxml2, zlib |
| `mousetweaks` | none | glib, gtk3, gsettings-desktop-schemas, libx11, libxtst, libxfixes, libxcursor |
| `nautilus-dropbox` | none (inline dropbox-path sed, no patch file) | glib, gtk4, pygobject, python3, nautilus |
| `nautilus-sendto` | meson-0.61 | glib, nautilus |
| `nm-applet` | none | glib, libsecret, libnma, gtk3, networkmanager, freedesktop-icon-theme |
| `pch-session` | none | gnome-shell, gnome-shell-extensions and friends, gnome-tweaks, gnome-clocks |
| `sushi` | none | libepoxy, evince, freetype, gdk-pixbuf, glib, gstreamer, gst-plugins-base, gtk3, gtksourceview4, harfbuzz, gobject-introspection, gjs, nautilus |
| `tecla` | none | gtk4, libadwaita, libxkbcommon |
| `yelp` | none | glib, gtk4, libadwaita, libxml2, libxslt, sqlite, webkitgtk6, yelp-xsl, xz-utils, bzip2 |
| `yelp-xsl` | none | (host tools only: libxslt, itstool) |
| `zenity` | none | libadwaita, gdk-pixbuf, pango |
