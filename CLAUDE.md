# kiro-wayland-dotfiles — Claude project instructions

## Overview
The **shared config base** for the KIROTUX Wayland line. Owns the dotfiles common to the wlroots
editions (hyprland · wayfire · sway · river · labwc · dwl) so they don't each ship — and conflict on
— the same files. Public, shipped via `nemesis_repo`. This is the "shared Kiro Wayland shell" base
the per-WM studies kept flagging; the kiro-hyprland↔kiro-river install conflict was the forcing function.

## What it owns (and why)
- `~/.config/mako/config` — notifications (all editions except niri, which uses noctalia).
- `~/.config/hypr/hyprlock.conf` + `hypridle.conf` — the Wayland lock/idle pipeline.
- `~/.config/waybar/colors.css` — pywal-overwritten palette.
- `~/.config/waybar/style.css` — ONE combined stylesheet with every edition's workspace selector
  (`#workspaces`/`#tags`/`#taskbar`); unused selectors are harmless CSS.

## The architecture
- **`depends=(mako hyprlock hypridle)`** — the binaries it configures. **NOT `waybar`** so dwl (dwlb,
  no waybar) can depend on this for mako/hypr without pulling waybar; the waybar editions list
  `waybar` in their own depends.
- Each waybar edition: `depends=('kiro-wayland-dotfiles')`, drops the shared files, ships only its
  `waybar/config-<wm>.jsonc` (unique path → no collision), and launches
  `waybar -c ~/.config/waybar/config-<wm>.jsonc`.
- **niri is NOT a consumer** (noctalia shell — no waybar/mako/hypr).

## Gotchas
- The combined `style.css` must keep ALL editions' workspace selectors. When a new waybar edition is
  added, add its selector block here, not a private style.css.
- `mako/config` here is the canonical Tokyo Night; pywal editions overwrite the colour lines at login
  via their own `set-theme.sh` — that's the user's copy, not this read-only golden one.

## Build / delivery
- Public recipe `../KIROTUX-PKG-BUILD/kiro-wayland-dotfiles/build.sh` → `~/EDU/nemesis_repo/`.
  Rebuild this **and** every consuming edition when a shared file changes. See [../CLAUDE.md](../CLAUDE.md).
