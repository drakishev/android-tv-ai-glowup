# Phase 1 — Audit & safe debloat

```text
I want to speed up my Android TV and clean up its home screen. My computer and TV are on the
same network, and USB/network debugging is already on.

1. Setup: check if adb is installed. If not, install it (Homebrew on Mac, winget on Windows,
   apt on Linux).
2. Find my TV: use mDNS and a scan of my local network for port 5555. If you find more than one
   device, ask me which one. Run adb connect, and tell me to press "Allow" on the TV.
3. Audit (change nothing yet): note free memory (MemAvailable from /proc/meminfo) as a baseline.
   List every installed package and sort them into:
   - REQUIRED: system/launcher/Play Store/keyboard/anything the TV needs to boot or work
   - MINE: apps I actually use (check usage stats; ask me if unsure)
   - SAFE TO DISABLE: preinstalled bloatware, ad/recommendation/tracking services, unused apps
   Show this as a table with a plain-English description of each app.
4. Ask before changing: let me approve or remove items from the SAFE TO DISABLE list.
   Also ask if I want to:
   - turn off voice search, Chromecast, and the screensaver
   - stop the TV from booting into Live TV
   - set animation scales to 0 for snappier menus
5. Apply: disable apps only with `pm disable-user --user 0`. Never uninstall anything.
   Save every change to a log file on my computer, along with one command that undoes all of them.
6. Results: reboot once BEFORE changes for a fair baseline, apply, reboot again, check free memory
   and show me before vs after.
```

**Tips for the agent**
- Compare RAM only between two *fresh boots* — a running streaming app can skew a single snapshot by 150 MB+.
- Vendor "B2B / hotel / pro mode", tuner, HDMI-CEC, remote pairing, settings providers and the IME are **REQUIRED** — when unsure, keep it.
- Typical safe candidates: vendor promo/"select"/"discover" apps, ACR/viewing-tracking services (e.g. Samba TV), demo/retail mode, region-locked services, feedback & bug-report uploaders, unused Google apps (Play Games, Play Movies, Play Music), retired services.
