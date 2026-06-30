# Changelog

All notable changes to **kiro-wayland-dotfiles** are documented here.
Format: one dated entry per day (`YYYY.MM.DD`), newest first.

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
