# kiro-wayland-dotfiles

The **shared config base** for the Kiro Wayland line — the common dotfiles every wlroots edition
(hyprland · wayfire · sway · river · labwc · dwl) would otherwise ship its own copy of.

## What it is

The KIROTUX Wayland editions all use the same notification daemon (mako), the same Wayland lock
pipeline (hyprlock + hypridle), and — for the waybar editions — the same bar palette + stylesheet.
Shipping those in every edition meant six packages owning the same files (`~/.config/mako/config`,
`~/.config/hypr/hyprlock.conf`, …), so any two editions **conflicted at install time**. This package
owns those shared files once; each edition `depends` on it, so the editions are **co-installable**.

## What it ships

- `~/.config/mako/config` — notifications (Tokyo Night; pywal editions regenerate it at login).
- `~/.config/hypr/hyprlock.conf` + `hypridle.conf` — the Wayland lock/idle pipeline.
- `~/.config/waybar/colors.css` — the pywal-overwritten palette.
- `~/.config/waybar/style.css` — one combined stylesheet carrying every edition's workspace
  selector (`#workspaces` / `#tags` / `#taskbar`); each edition launches
  `waybar -c ~/.config/waybar/config-<wm>.jsonc`, which picks up this style.

## How to install

Pulled in automatically as a dependency of any `kiro-<wayland-edition>` package. It depends on
`mako`, `hyprlock`, `hypridle` (the binaries it configures).

```sh
sudo pacman -S kiro-wayland-dotfiles
```
