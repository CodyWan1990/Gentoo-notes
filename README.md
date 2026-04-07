# Gentoo Linux — Learning & Reference Guide

A Gentoo-specific knowledge base covering Portage, OpenRC, system configuration, and software quirks. This is a **Gentoo learning resource**, not a desktop config dump — compositor keybinds, theming, and app-specific config live in the dotfiles repo instead.

> **Note:** Some sections of this documentation — particularly [make.conf](02-make-conf.md) — reflect specific hardware (Ryzen 9 7900X, 64GB RAM, RDNA 3 GPU). Values like thread counts, RAM allocations, `CPU_FLAGS_X86`, and `VIDEO_CARDS` are not universal. Read carefully and adjust for your own system before applying anything.

## System

| Component | Details |
|---|---|
| CPU | AMD Ryzen 9 7900X (Zen4, 24 threads, AVX-512) |
| GPU | AMD Radeon RX 7800 XT (RDNA 3) |
| RAM | 64GB DDR5 |
| Storage | NVMe |
| Init | OpenRC (no systemd) |
| Distro | Gentoo Linux |

## What Makes Gentoo Different

- **Source-based** — every package is compiled from source on your machine; binary packages exist but aren't the default
- **USE flags** — per-package feature flags control what gets compiled in; must be set before installing
- **Portage** — the package manager; resolves dependencies, manages keywords, and handles config file updates
- **~amd64 masking** — most bleeding-edge packages are marked unstable and must be explicitly unmasked
- **OpenRC** — dependency-based init system; no systemd user services; session daemons need manual setup
- **D-Bus session bus** — does not start automatically on OpenRC; must be started by the compositor or session manager
- **elogind** — provides logind functionality (sessions, power management) without systemd

## Index

- [01 - Portage](01-portage.md) — daily commands, package masking, USE flags, overlays, license acceptance, dispatch-conf
- [02 - make.conf](02-make-conf.md) — compiler flags, parallelism, ccache, tmpfs builds, world rebuild
- [03 - OpenRC](03-openrc.md) — runlevels, service management, user services, D-Bus session bus gotcha
- [04 - Audio (PipeWire)](04-audio.md) — USE flags, OpenRC user services, wpctl commands, troubleshooting
- [05 - Networking](05-networking.md) — NetworkManager, UFW firewall, Syncthing, sshd, WireGuard
- [06 - Software](06-software.md) — Portage tools, archive format support, Flatpak on OpenRC, Gentoo package quirks
