# Changelog

All notable changes to **kiro-wayland-dotfiles** are documented here.
Format: one dated entry per day (`YYYY.MM.DD`), newest first.

## 2026.10.08

### What Changed
- **Qt apps match the dark GTK look on the Wayland editions.** The skel Kvantum config came from kiro-kvantum and
  named ArcDark, a theme the Wayland ISOs don't install, so every Qt app (VLC, Kvantum Manager, …) fell back to
  Kvantum's built-in theme. New `kiro-kvantum-default`, run at session start, sets KvGnomeDark — shipped with the
  kvantum package and close to adw-gtk3-dark — whenever the config is missing, empty or names a theme that isn't
  installed. A theme the user picked is left alone.

### Technical Details
- Kvantum reads its theme only from `$XDG_CONFIG_HOME/Kvantum/kvantum.kvconfig` (no `/etc/xdg` lookup), so the
  default can't be a system file. Shipping it in `/etc/skel` would collide with kiro-kvantum and, with a
  `conflicts=`, would break ISO builds and the Wayland desktop installs ATT does on X11 Kiro (every Wayland edition
  depends on this package). A helper that only writes the user's own file has no package overlap.
- `Default`/`Kvantum` (the built-in theme) count as a deliberate choice. Tested on six cases (missing, empty,
  missing theme, installed theme, `[General]` without `theme=`, built-in) and on the hyprland-dms live ISO in
  QEMU: shipped ArcDark → KvGnomeDark; Kvantum Manager and VLC then match pavucontrol.
- The recipe installs the script explicitly (`usr/bin` files are listed one by one). The KIROTUX ISO lists drop
  kiro-kvantum. kiro-hyprland-dms calls the helper at start; other Wayland editions add the same one line.

### Files Modified
- usr/bin/kiro-kvantum-default (new)

## 2026.10.05

### What Changed
- Folders open in Thunar: tried here first as `/etc/xdg/hyprland-mimeapps.list` (Hyprland only), then moved the
  same day to **kiro-system-files** as `/usr/share/applications/mimeapps.list`, so every Kiro and KIROTUX desktop,
  X11 and Wayland, gets it. The file is removed from this package again; nothing else changes here.

### Files Modified
- `etc/xdg/hyprland-mimeapps.list` (added, then removed)

## 2026.09.05

### What Changed
- **New shared `/usr/bin/kiro-screenshot`.** Every non-niri Kiro Wayland edition bound Print to
  `grim -g "$(slurp)" - | wl-copy`. That worked, but wrote no file and raised no notification, so
  the image went to the clipboard and nowhere else — next to chadwm, where Print writes a PNG into
  `~/Pictures` via scrot, it read as a dead key. The same broken line had been copy-pasted into
  twelve editions, so the logic now lives here once, in the package they all already depend on.
- The helper takes `region` (default, drag a rectangle) or `screen`, saves a timestamped PNG in
  `~/Pictures/Screenshots`, still copies to the clipboard, and notifies with a thumbnail.

### Technical Details
- `usr/bin/kiro-screenshot` — installed `0755`. `depends` gains `grim slurp wl-clipboard libnotify`:
  the editions listed grim themselves already, but the package shipping the script should own what
  the script runs.
- The grab is wrapped in DankMaterialShell's `dms ipc call screenshot begin` / `end` handshake. That
  IPC pair is **not** a screenshot tool — its whole body flips `PopoutManager.screenshotActive`, which
  pulls DMS's popouts off screen so a half-faded panel doesn't land in the shot. It is guarded by
  `command -v dms`, so on the eleven editions that don't run DMS it is simply a no-op — one script
  body, no per-edition branching. Noctalia exposes no equivalent IPC, so it gets a plain grab.
- The handshake is closed **before** `notify-send`: while `screenshotActive` is set, the suppressed
  popout layer swallows the very toast the helper exists to show. A `trap ... EXIT` closes it too, so
  a cancelled selection can't leave DMS with its popouts stuck hidden.
- `slurp` exits non-zero when cancelled (Esc / right-click), so `geometry=$(slurp) || exit 0` treats
  cancel as success — under `set -euo pipefail` it would otherwise abort and leave a stray PNG.
  `notify-send` is `|| true` so a missing daemon can't fail a screenshot that already saved fine.
- Verified on picard against a live Hyprland session: both modes through the real keybind dispatcher,
  a stubbed geometry proving `grim -g` honours the region exactly (640x480 in, 640x480 out), clipboard
  `image/png`, the toast rendered, and DMS back to `SCREENSHOT_MODE_OFF`.

### Files Modified
- `usr/bin/kiro-screenshot` (new)
- `KIROTUX-PKG-BUILD/kiro-wayland-dotfiles/PKGBUILD`

## 2026.07.02

### What Changed
- **Now owns the shared `/etc/dconf` GTK appearance defaults** (`profile/user` +
  `db/local.d/00-kiro.conf`) — the same file-conflict pattern that created this package hit again:
  all 9 editions had independently shipped byte-identical (or near-identical) copies of these two
  files, which pacman refuses to install twice when two editions are co-installed on one machine.
  Moved here once; **all 9 editions, including `kiro-niri`**, now consume it for dconf (niri and
  ohmyniri were previously non-consumers/partial-consumers of the mako/hypr/waybar files — dconf
  makes both full dconf-consumers regardless).

### Technical Details
- Settings (`color-scheme`, `gtk-theme`, `icon-theme`, `cursor-theme`, `cursor-size`) were
  identical across all 9 source repos already — only the comment header differed per edition, so
  the merge is lossless.
- `kiro-niri` had no prior dependency on this package (noctalia owns everything else); it gains
  `kiro-wayland-dotfiles` in `depends=()` for dconf only.

### Files Modified
- `etc/dconf/profile/user`, `etc/dconf/db/local.d/00-kiro.conf` (new)
- `CLAUDE.md` (ownership list, architecture section, overview)

## 2026.07.01

### What Changed
- **`kiro-ohmyniri` is now a consumer** (mako + waybar colours/stylesheet only, not
  hyprlock/hypridle — that edition locks/idles with gtklock/swayidle instead). It had briefly
  shipped its own copies of `mako/config` + `waybar/colors.css`/`style.css` at the same absolute
  paths — a real file-ownership conflict against any other consumer of this package installed
  alongside — corrected same-day. niri's native `niri/workspaces` waybar module renders to
  `#workspaces`, so the existing sway/hyprland selector block in `style.css` already covers it;
  no dedicated niri block was needed.

### Files Modified
- `CLAUDE.md` (consumer list, niri caveat)

## 2026.06.30

### What Changed
- **Initial package** — the shared config base for the Kiro Wayland line, created to resolve the
  file-conflict between editions (e.g. kiro-hyprland + kiro-river both owned `~/.config/mako/config`,
  `~/.config/hypr/hyprlock.conf`, and the waybar files). Now owned once here; every wlroots edition
  depends on it, so the editions are **co-installable**.

### Technical Details
- Ships `mako/config`, `hypr/hyprlock.conf`, `hypr/hypridle.conf`, `waybar/colors.css`, and a
  **combined** `waybar/style.css` carrying every edition's workspace selector (`#workspaces` for
  sway/hyprland, `#tags` for river, `#taskbar` for wayfire/labwc — unused ones are harmless).
- `depends=(mako hyprlock hypridle)` — the binaries it configures. NOT `waybar` (so dwl, which uses
  dwlb, can depend on this for mako/hypr without pulling waybar).
- Each waybar edition now ships only its `waybar/config-<wm>.jsonc` (a unique path, no collision) and
  launches `waybar -c ~/.config/waybar/config-<wm>.jsonc`, picking up this shared style.css.

### Files
- `etc/skel/.config/{mako/config,hypr/{hyprlock,hypridle}.conf,waybar/{colors.css,style.css}}`
- `README.md`, `CLAUDE.md`, `up.sh`, `setup.sh`, `.gitignore`, `kiro.jpg`
