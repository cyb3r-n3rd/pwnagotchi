# Pwnagotchi — build & setup guide

A **Pwnagotchi** is a Raspberry Pi Zero W that sits in your pocket and collects WPA/WPA2
handshake material from Wi-Fi networks **you own or are authorised to test**, so you can
study it later. It runs the open-source [Pwnagotchi](https://pwnagotchi.ai) firmware, shows
a little e-ink face, and uses a small reinforcement-learning loop to tune its own capture
settings over time.

This repo is my **build journal and setup guide** — the parts I used, how I wired and flashed
it, and the config I settled on. The core firmware is the upstream open-source project; links
are below.

> ⚠️ **Authorised use only.** Capturing Wi-Fi traffic from networks you do not own or have
> written permission to test is illegal in most countries. I use this on my own home lab AP
> and in lab environments during training. Use it the same way.

---

## Bill of materials

| Part | Why | Approx. price | Where |
| --- | --- | --- | --- |
| Raspberry Pi Zero **W** (or Zero 2 W) | the brains; needs the built-in Wi-Fi | ₹1,200–1,800 | [raspberrypi.com resellers](https://www.raspberrypi.com/products/raspberry-pi-zero-w/) |
| Waveshare 2.13" e-Paper HAT (V2/V3/V4) | the face / status display | ₹1,800–2,400 | [Waveshare](https://www.waveshare.com/2.13inch-e-paper-hat.htm) |
| microSD card, 16 GB+ (A1, good brand) | OS + captures | ₹300–600 | any electronics shop |
| PiSugar 2 / 3 battery module *(optional)* | makes it portable | ₹2,500–3,500 | [PiSugar](https://www.pisugar.com/) |
| USB-A to micro-USB cable | flashing & power | ₹100 | any shop |

A Pi Zero 2 W is worth the small extra cost — the stock Pwnagotchi image runs noticeably
smoother on it.

## Assembly

1. Solder the 40-pin header to the Pi Zero (or buy a pre-soldered "WH" model).
2. Seat the Waveshare e-Paper HAT directly on the GPIO header.
3. (Optional) Mount the PiSugar battery on the back per its guide.

![assembly placeholder](images/assembly.png)

## Flash the firmware

1. Download the latest image from the [Pwnagotchi releases](https://github.com/jayofelony/pwnagotchi/releases)
   (the `jayofelony` fork is the actively-maintained one in 2026).
2. Write it with [Raspberry Pi Imager](https://www.raspberrypi.com/software/) or
   [balenaEtcher](https://etcher.balena.io/):
   ```
   # or from the command line on Linux (double-check the device!):
   sudo dd if=pwnagotchi.img of=/dev/sdX bs=4M status=progress conv=fsync
   ```
3. Re-insert the card. Edit `/boot/config.toml` (see below) before first boot.

## Configure — `config.toml`

A minimal starting point (the whole point is you tune this yourself):

```toml
main.name = "cyb3r-gotchi"
main.lang = "en"
main.plugins.grid.enabled = true
main.plugins.grid.report = false   # do NOT upload anything to the public map
ui.display.enabled = true
ui.display.type = "waveshare_4"     # match YOUR HAT revision
personality.advertise = false        # stay quiet while learning
```

- First boot takes a few minutes while it resizes the filesystem.
- Connect over USB: the Pi appears as `10.0.0.2`, you are `10.0.0.1`.
  ```
  ssh pi@10.0.0.2      # default password: raspberry — change it immediately
  ```

## First run checklist

- [ ] Changed the default `pi` password
- [ ] `ui.display.type` matches your exact HAT revision (wrong value = blank screen)
- [ ] `grid.report = false` so nothing leaves the device
- [ ] Testing against **your own** access point

## How it works (short version)

Pwnagotchi puts the Wi-Fi interface in monitor mode via `bettercap`, watches for WPA
handshakes and PMKIDs, and saves them as `.pcap` files in `/root/handshakes/`. The RL agent
only tunes *when and how long* it listens — it does not attack anything on its own. Analysis
of the captured `.pcap` files is done later, offline, on hardware you control.

## Credits & upstream

- Firmware: [Pwnagotchi](https://github.com/jayofelony/pwnagotchi) (maintained fork) and the
  original [evilsocket/pwnagotchi](https://github.com/evilsocket/pwnagotchi)
- Packet engine: [bettercap](https://www.bettercap.org/)

Built and documented by **Shivam Verma (cyb3r_n3rd)** —
[portfolio](https://cyb3r-n3rd.github.io/portfolio/) ·
[LinkedIn](https://www.linkedin.com/in/shivamvermacyber)
