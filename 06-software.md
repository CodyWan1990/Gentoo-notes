# Software: Tools, Quirks & Recommendations

Gentoo-specific software notes: useful Portage tools, archive format support, Flatpak on OpenRC, and package quirks that are unique to Gentoo.

## Portage Utilities

These tools make working with Portage significantly easier and are worth installing on any Gentoo system.

| Package | Command | Purpose |
|---|---|---|
| `app-portage/eix` | `eix <pkg>` | Fast, colored package search; much better than `emerge -s`. Run `eix-update` after install |
| `app-portage/genlop` | `genlop -c` / `genlop -t <pkg>` | Shows ETA during active builds; compile time history from emerge logs |
| `app-portage/gentoolkit` | `equery f/d/u <pkg>` | List files owned by a package, reverse deps, USE flags |
| `app-portage/flaggie` | `flaggie <pkg> +<flag>` | Manage per-package USE flags from the CLI without editing files manually |
| `app-portage/elogv` | `elogv` | Browse saved post-install messages in a nice UI (requires elog settings in make.conf) |
| `app-portage/cpuid2cpuflags` | `cpuid2cpuflags` | Generate correct `CPU_FLAGS_X86` for your CPU for make.conf |

Install them all:
```bash
sudo emerge -av app-portage/eix app-portage/genlop app-portage/gentoolkit app-portage/flaggie app-portage/elogv app-portage/cpuid2cpuflags
eix-update
```

### equery examples

```bash
equery f <package>    # list all files installed by a package
equery d <package>    # what packages depend on this one
equery u <package>    # show available USE flags and their status
equery b <file>       # which package owns a given file
```

### genlop examples

```bash
genlop -c             # ETA for current active build
genlop -t <package>   # how long the last compile of this package took
genlop -l             # full emerge history log
```

## Archive Tools

For full archive format support (used by GUI archive managers like Xarchiver):

| Package | Format | Notes |
|---|---|---|
| `app-arch/xarchiver` | GUI frontend | Integrates with Thunar via thunar-archive-plugin |
| `app-arch/unar` | rar, zip, many others | Handles most formats including rar v3 |
| `app-arch/unrar` | rar (newer) | `~amd64` keyword + `unRAR` license required |
| `app-arch/lha` | lha/lzh | `~amd64` keyword + `lha` license required |
| `app-arch/7zip` | 7z, zip, tar, cab, arj, and more | Modern replacement for p7zip |
| `app-arch/zip` | zip (creation) | |
| `app-arch/lzop` | lzop | |
| `app-arch/arj` | arj | |

Unmask and accept licenses for the restricted ones:
```bash
echo "app-arch/unrar ~amd64" | sudo tee /etc/portage/package.accept_keywords/unrar
echo "app-arch/lha ~amd64" | sudo tee /etc/portage/package.accept_keywords/lha
echo "app-arch/unrar unRAR" | sudo tee -a /etc/portage/package.license
echo "app-arch/lha lha" | sudo tee -a /etc/portage/package.license
sudo emerge -av app-arch/unrar app-arch/lha app-arch/7zip app-arch/zip app-arch/lzop app-arch/arj
```

## Flatpak on OpenRC

Flatpak provides sandboxed apps for software not in Portage.

```bash
sudo emerge -av sys-apps/flatpak
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

**D-Bus requirement:** Flatpak apps silently fail to launch if the D-Bus session bus is not running. On OpenRC this does not start automatically — see [03-openrc.md](03-openrc.md) for the fix.

Useful commands:
```bash
flatpak list                           # list installed apps
flatpak search <name>                  # search Flathub
flatpak update                         # update all Flatpak apps
flatpak run <app.id>                   # launch an app
flatpak override --user <id> --show    # show overrides for an app
```

## Gentoo-Specific Package Quirks

These are behaviors that differ from other distros and will catch you off guard.

### Zed Editor — binary is `zedit`

The Zed editor package (`app-editors/zed`) installs its binary as `zedit`, not `zed`, because there is already a package named `zed` in Portage (a different, unrelated tool). All scripts and aliases should use `zedit`.

### Steam — requires a separate overlay

Steam is not in the main Gentoo repository. It requires the `steam-overlay`:

```bash
sudo eselect repository add steam-overlay git https://github.com/anyc/steam-overlay.git
sudo emaint sync -r steam-overlay
echo "games-util/steam-launcher ValveSteamLicense" | sudo tee -a /etc/portage/package.license
sudo emerge -av games-util/steam-launcher
```

Use `steam-overlay` — the old `steamcmd-overlay` URL no longer works.

### Vivaldi — keyring auto-unlock requires a flag

Vivaldi won't use gnome-keyring without the `--password-store=gnome-libsecret` flag. Without it, a keyring password prompt appears on every launch. Add it in two places:

1. **Compositor keybind config:**
   ```ini
   $browser = vivaldi --password-store=gnome-libsecret
   ```

2. **Local .desktop override** (so file managers and app launchers use it too):
   Copy to `~/.local/share/applications/vivaldi-stable.desktop` and add the flag to all three `Exec=` lines. The local `.desktop` takes priority over `/usr/share/applications/` and survives Vivaldi updates.

### Feishin Music Client — AppImage only

Feishin (a Jellyfin/Navidrome music client) is not in Portage or available as a Flatpak on Flathub. The only option is the AppImage from the project's GitHub releases page. Store it in `~/Applications/` and mark it executable.

### bat — binary is `bat` (not `batcat`)

On Debian/Ubuntu, the `bat` binary is called `batcat` to avoid a conflict. On Gentoo it's simply `bat`.
