# OpenRC: Init System & Service Management

OpenRC is Gentoo's default init system (as opposed to systemd). It's dependency-based, uses shell scripts for service definitions, and is significantly simpler than systemd. This covers runlevels, service management, user services, and common OpenRC-specific gotchas on a Wayland desktop.

## Runlevels

OpenRC uses runlevels to determine which services run at which stage of boot:

| Runlevel | Purpose | When it runs |
|---|---|---|
| `sysinit` | Very early kernel setup (udev, etc.) | First, before anything |
| `boot` | Core system services | After sysinit; serial (sequential) execution |
| `default` | Normal user-space services | After boot; parallel execution |
| `shutdown` | Cleanup on power-off | On shutdown |

### Boot vs Default Runlevel

The `default` runlevel starts services in **parallel** for speed. This can cause race conditions — a service may start before its dependency is ready, silently fail, and never log an error.

The `boot` runlevel is **serial** — each service waits for the previous one to finish. Use it for services that must reliably start every time (sshd, UFW firewall).

## Service Management Commands

```bash
# Add a service to a runlevel (persists across reboots)
sudo rc-update add <service> <runlevel>
sudo rc-update add sshd boot
sudo rc-update add NetworkManager default

# Remove a service from a runlevel
sudo rc-update del <service> <runlevel>

# Start/stop/restart a service now (does not affect runlevel)
sudo rc-service <service> start
sudo rc-service <service> stop
sudo rc-service <service> restart

# Show status of all services
sudo rc-status

# Show status of a specific service
sudo rc-service <service> status
```

## User Services (OpenRC 0.63+)

OpenRC 0.63+ supports user-level services that run as your user, without root. This is the correct way to run session-scoped daemons (PipeWire, Syncthing, etc.) on Gentoo instead of hacking them into a compositor's autostart.

```bash
# Add a user service to run at login
rc-update --user add <service> default

# Start a user service now
rc-service --user <service> start

# Check status of user services
rc-status --user
```

User service files live in `/usr/share/openrc/init.d/` or `~/.local/openrc/init.d/` (for custom user services).

### Services Configured as User Services on This System

```bash
rc-update --user add pipewire default
rc-update --user add wireplumber default
rc-update --user add pipewire-pulse default
rc-update --user add syncthing default
```

## D-Bus Session Bus on OpenRC

### The Problem

On systemd, a D-Bus **session** bus socket is automatically created at `/run/user/1000/bus` when you log in. On OpenRC this does NOT happen automatically.

Without a D-Bus session bus:
- Flatpak apps fail silently (zypak sandbox requires `DBUS_SESSION_BUS_ADDRESS`)
- xdg-desktop-portal fails to start
- Many Wayland apps misbehave
- Screen sharing, file picker portals don't work

### The Fix

The session bus must be started by your compositor or display manager session. For Hyprland, add to `autostart.conf`:

```ini
exec-once = dbus-daemon --session --address=unix:path=/run/user/1000/bus --fork --nopidfile --syslog-only 2>/dev/null
exec-once = dbus-update-activation-environment --all
```

And in `environment.conf`:
```ini
env = DBUS_SESSION_BUS_ADDRESS,unix:path=/run/user/1000/bus
```

> **Note:** `1000` is the UID of the first non-root user, which is standard on most systems. Verify yours with `id -u` — if it differs, replace `1000` in both paths above.

`dbus-update-activation-environment --all` propagates all current environment variables (including `WAYLAND_DISPLAY`, `XDG_RUNTIME_DIR`, etc.) into D-Bus, so apps launched via D-Bus activation get the correct environment.

### Verify It's Working

```bash
echo $DBUS_SESSION_BUS_ADDRESS
# Should output: unix:path=/run/user/1000/bus

ls /run/user/1000/bus
# File should exist
```

### CRITICAL WARNING

**Never modify `/usr/share/wayland-sessions/hyprland.desktop`** or create wrapper scripts around the Hyprland binary to inject environment variables. This has caused a broken SDDM login that required chroot recovery to fix. The `autostart.conf` + `environment.conf` approach is the correct solution.

## Common Gotchas

### sshd must be in boot, not default

The `default` runlevel's parallel startup silently skips sshd on some boots with no error or log entry. Always put it in `boot`:
```bash
sudo rc-update add sshd boot
```

### UFW firewall must be in boot, not default

UFW declares `before net` in its init script and must load before NetworkManager. The `default` runlevel's parallel startup doesn't guarantee ordering:
```bash
sudo rc-update add ufw boot
```

### Do not use netifrc alongside NetworkManager

OpenRC's built-in network interface management (`net.enp10s0` via netifrc) conflicts with NetworkManager. If present, remove it:
```bash
sudo rc-update del net.enp10s0 default
```

### PipeWire should use user services, not autostart

While you can launch PipeWire from a compositor's autostart, the correct OpenRC approach is user services — they handle automatic restarts on crash and proper dependency ordering. See [04-audio.md](04-audio.md).

### Power management uses loginctl, not systemctl

Since systemd is not present, power actions go through elogind:
```bash
loginctl poweroff
loginctl reboot
loginctl suspend
```
