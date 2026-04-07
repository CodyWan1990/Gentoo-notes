# PipeWire Audio on OpenRC

PipeWire is the modern Linux audio server, replacing both PulseAudio and JACK. On systemd it runs as a user service automatically. On OpenRC it must be set up manually.

## Step 1: Set USE Flags

Before installing, set the required USE flags so PipeWire is built with ALSA and PulseAudio compatibility layers. Without these, apps that use PulseAudio or ALSA APIs won't be able to play audio through PipeWire.

```
# /etc/portage/package.use/pipewire
media-video/pipewire pipewire-alsa pulseaudio sound-server
```

- `pipewire-alsa` — ALSA compatibility layer (allows ALSA-only apps to use PipeWire)
- `pulseaudio` — PulseAudio compatibility (required for most desktop apps)
- `sound-server` — enables PipeWire to act as the primary sound server

Then install:
```bash
sudo emerge -av media-video/pipewire media-video/wireplumber
```

## Step 2: Enable as OpenRC User Services

The recommended approach on OpenRC is user services — they auto-restart on crash and have proper dependency ordering. Order matters: PipeWire must start before WirePlumber and pipewire-pulse.

```bash
rc-update --user add pipewire default
rc-update --user add wireplumber default
rc-update --user add pipewire-pulse default
```

Start them for the current session:
```bash
rc-service --user pipewire start
rc-service --user wireplumber start
rc-service --user pipewire-pulse start
```

See [03-openrc.md](03-openrc.md) for more on OpenRC user services.

## Verifying Audio Works

```bash
# Show all audio devices and their status
wpctl status

# Set default output to 50% volume
wpctl set-volume @DEFAULT_AUDIO_SINK@ 0.5

# Raise/lower volume by 5%
wpctl set-volume -l 1 @DEFAULT_AUDIO_SINK@ 5%+
wpctl set-volume @DEFAULT_AUDIO_SINK@ 5%-

# Toggle mute on default output
wpctl set-mute @DEFAULT_AUDIO_SINK@ toggle

# Toggle mute on default microphone
wpctl set-mute @DEFAULT_SOURCE@ toggle
```

## Troubleshooting

**`wpctl status` shows no devices:**
1. Check USE flags are set: `emerge -pv media-video/pipewire` — `pulseaudio` and `sound-server` should appear in USE
2. Rebuild with the flags if missing: `sudo emerge -av media-video/pipewire`
3. Confirm all three user services are running: `rc-status --user`

**Apps can't find audio:**
- Most apps use PulseAudio — make sure `pipewire-pulse` is running, not just `pipewire`
- ALSA-only apps need the `pipewire-alsa` USE flag set at compile time
