# Networking on OpenRC

## NetworkManager

NetworkManager handles Wi-Fi and wired connections. On OpenRC it runs as a system service.

### Enable as OpenRC service

```bash
sudo rc-service NetworkManager start
sudo rc-update add NetworkManager default
```

### Important: Do not use netifrc alongside NetworkManager

OpenRC's built-in network interface service (netifrc) conflicts with NetworkManager — they both try to manage network interfaces. If netifrc is present, remove it:

```bash
# Replace enp10s0 with your actual interface name (check with: ip link show)
sudo rc-update del net.enp10s0 default
```

Do NOT add `rc_provide="net"` to `/etc/conf.d/NetworkManager` — it is not needed and causes problems.

## Fix: IPv6 Overriding Homelab DNS

When using a homelab with custom DNS, the router's IPv6 auto-DNS assignment can override your manual DNS settings. To disable auto-DNS for IPv6 on a specific connection:

```bash
# Replace "Wired connection 1" with your actual connection name (check with: nmcli connection show)
nmcli connection modify "Wired connection 1" ipv6.ignore-auto-dns yes
nmcli connection up "Wired connection 1"
```

Verify the fix:
```bash
nmcli connection show "Wired connection 1" | grep dns
cat /etc/resolv.conf
```

## UFW Firewall

UFW (Uncomplicated Firewall) provides a simple frontend for iptables/nftables.

```bash
sudo emerge -av net-firewall/ufw
sudo ufw enable
sudo rc-update add ufw boot   # boot runlevel — UFW must load before NetworkManager
```

> UFW must be in the **boot** runlevel because it declares `before net` in its init script. The `default` runlevel's parallel startup doesn't guarantee ordering, so UFW may start after the network is already up.

Common rules:
```bash
sudo ufw allow ssh             # port 22
sudo ufw allow 22000/tcp       # Syncthing data sync
sudo ufw allow 21027/udp       # Syncthing local discovery
```

Check status:
```bash
sudo ufw status verbose
```

## Syncthing

Syncthing syncs files between devices. Run it as a **user-level OpenRC service**, not the system service. The system service runs as a `syncthing` system user with config in `/var/lib/syncthing/` — not what you want for a personal setup.

```bash
rc-update --user add syncthing default
rc-service --user syncthing start
```

Config lives at `~/.local/state/syncthing/`.
Web UI: `http://localhost:8384`

## SSH Server (sshd)

```bash
sudo rc-update add sshd boot   # boot runlevel, not default
```

> **Important:** sshd must be in the **boot** runlevel, not `default`. The default runlevel uses parallel startup which can silently skip sshd on some boots — no error, no log entry. The boot runlevel is serial and always runs sshd reliably.

## WireGuard

WireGuard can be managed by NetworkManager or via an OpenRC init script. Using the init script approach (requires `/etc/wireguard/wg0.conf` to exist first):

```bash
sudo rc-update add wg-quick.wg0 default
```
