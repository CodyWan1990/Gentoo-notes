# Portage: Gentoo's Package Manager

Portage compiles packages from source, giving you full control over what features get built and how. This guide covers daily usage, package masking, USE flags, overlays, and license acceptance.

## Daily Commands

```bash
# Sync the Portage tree (fetch latest package metadata)
sudo emerge --sync

# Update all installed packages, check full dependency tree, apply new USE flags
sudo emerge -avuDN @world

# Remove packages that are no longer needed by anything
sudo emerge --depclean

# Check for broken library links after updates
sudo revdep-rebuild

# Search for a package
eix <name>             # fast (requires eix-update first)
emerge -s <name>       # built-in search
emerge -S <name>       # search descriptions too

# Show what would be installed without installing
emerge -pv <package>

# Install a package interactively
sudo emerge -av <package>

# Review post-install messages
elogv
```

## Package Masking Explained

Gentoo marks packages as `~amd64` (unstable/testing) when they haven't been through full stabilization. Most cutting-edge packages (Hyprland, Zed, etc.) are `~amd64` and must be explicitly unmasked. Some packages are also blocked by their license.

There are two reasons you can't install a package:
1. **Keyword mask** — the package is marked unstable (`~amd64`)
2. **License mask** — the package has a non-free license that must be explicitly accepted

## Unmasking Packages (~amd64)

Keyword files live in `/etc/portage/package.accept_keywords/` — this is a **directory**. One file per package (or grouped).

```bash
# Unmask a single package
echo "category/package-name ~amd64" | sudo tee /etc/portage/package.accept_keywords/package-name

# Unmask a live ebuild (9999 version — tracks git HEAD)
echo "category/package-name **" | sudo tee /etc/portage/package.accept_keywords/package-name
```

### Packages Unmasked on This System

| Package | Keyword | Notes |
|---|---|---|
| `gui-wm/hyprland` | `~amd64` | Wayland compositor |
| `gui-apps/hyprshot` | `~amd64` | Screenshot tool |
| `gui-apps/swaync` | `~amd64` | Notification daemon |
| `app-misc/nwg-look` | `~amd64` | GTK theme switcher GUI |
| `x11-themes/catppuccin-cursors` | `~amd64` | Cursor theme (from guru overlay) |
| `x11-themes/catppuccin-gtk` | `~amd64` | GTK theme (from guru overlay) |
| `app-editors/zed` | `~amd64` | Zed editor (binary is `zedit` on Gentoo) |
| `gui-libs/xdg-desktop-portal-hyprland` | `~amd64` | Hyprland XDG portal |
| `app-arch/unrar` | `~amd64` | rar extraction (also needs license, see below) |
| `app-arch/lha` | `~amd64` | lha/lzh archive support (also needs license) |

## Accepting Non-Free Licenses

`/etc/portage/package.license` is a **file** on this system (not a directory). Always append to it, never overwrite:

```bash
echo "category/package license-name" | sudo tee -a /etc/portage/package.license
```

### Licenses Accepted on This System

| Package | License |
|---|---|
| `sys-kernel/linux-firmware` | `@BINARY-REDISTRIBUTABLE` |
| `www-client/vivaldi` | `Vivaldi` |
| `games-util/steam-launcher` | `ValveSteamLicense` |
| `www-plugins/chrome-binary-plugins` | `google-chrome` |
| `app-arch/unrar` | `unRAR` |
| `app-arch/lha` | `lha` |

## USE Flags

USE flags control which optional features get compiled into a package. Set them in `/etc/portage/package.use/` — this is a **directory**. One file per package.

```bash
# Example: enable pulseaudio and sound-server for PipeWire
echo "media-video/pipewire pipewire-alsa pulseaudio sound-server" | sudo tee /etc/portage/package.use/pipewire
```

After changing USE flags, rebuild affected packages:
```bash
sudo emerge -avuDN @world    # picks up USE flag changes with -N (--newuse)
```

### Common USE Flags on This System

```
# /etc/portage/package.use/pipewire
media-video/pipewire pipewire-alsa pulseaudio sound-server

# /etc/portage/package.use/mesa
media-libs/mesa vulkan

# /etc/portage/package.use/waybar
gui-apps/waybar pulseaudio mpris tray

# /etc/portage/package.use/catppuccin-gtk
x11-themes/catppuccin-gtk mocha

# /etc/portage/package.use/swaybg
gui-apps/swaybg gdk-pixbuf
```

## Overlays

Overlays add packages not in the main Gentoo repository.

### guru (community overlay)

Maintained community overlay with many user-contributed packages (Catppuccin themes, etc.):
```bash
sudo eselect repository enable guru
sudo emaint sync -r guru
```

### steam-overlay

Required for native Steam installation. Use this URL — the old `steamcmd-overlay` is dead:
```bash
sudo eselect repository add steam-overlay git https://github.com/anyc/steam-overlay.git
sudo emaint sync -r steam-overlay
```

## Autounmask Workflow

When Portage refuses to install a package due to keywords or USE conflicts, use autounmask to let Portage calculate what needs to change:

```bash
sudo emerge -av --autounmask --autounmask-write <package>
sudo dispatch-conf   # review proposed changes; press 'u' to accept, 'z' to discard
sudo emerge -av <package>
```

## dispatch-conf

After any world update, Portage may have config file updates waiting. `dispatch-conf` lets you review them:

```bash
sudo dispatch-conf
```

Key bindings inside dispatch-conf:
- `u` — use new version (replace your file with the package's version)
- `z` — zap new (keep your current file, discard the update)
- `m` — merge interactively
- `q` — quit

Always review carefully — config updates can remove settings you've added (hostname entries, custom sudoers rules, etc.).

## Troubleshooting

**Invalid atom warning on every emerge:**
If you see `--- Invalid atom in /etc/portage/package.use/qt`, there is a malformed line in that file:
```bash
sudo nvim /etc/portage/package.use/qt
```

**Package masked by license:**
```bash
# Check what license it needs
emerge -pv <package> 2>&1 | grep -i license
# Then add the license acceptance
echo "category/package license-name" | sudo tee -a /etc/portage/package.license
```
