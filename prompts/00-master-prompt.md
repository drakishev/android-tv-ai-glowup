# Master prompt — full Android TV glow-up

Paste this into an AI agent that can run shell commands on a computer on the same network as the TV.
Edit the **My preferences** block first. For context on known pitfalls, also give the agent
[`../skills/android-tv-makeover/SKILL.md`](../skills/android-tv-makeover/SKILL.md).

---

```text
You are helping me speed up, de-ad and redesign my Android TV over adb. Work in phases.
Never skip an approval gate. Explain everything in plain language.

## My preferences (edit me)
- Apps I actually use: <e.g. YouTube, Netflix, my IPTV app>
- Keep: voice search <yes/no>, Chromecast <yes/no>, screensaver <yes/no>
- Home screen style: <minimal clock + apps | Google TV style with photo backgrounds>
- Ad blocking: <system-wide DNS + browser | browser only | none>
- Language for UI text: <English / ...>

## Ground rules
- Change nothing until I approve. Read-only first.
- Disable packages ONLY with `pm disable-user --user 0 <pkg>`. NEVER uninstall system apps.
- Log every change (timestamp, command, previous value) to a file on my computer and keep ONE
  command/script that undoes everything. Update it after every change.
- Ask me before: rebooting, downloading/installing anything, changing network/DNS settings,
  or pressing buttons on the TV on my behalf.
- If something fails twice, stop and tell me what you tried.
- Prefer tools already on my computer; install adb only from the official package manager
  (Homebrew / winget / apt).

## Phase 1 — Connect
1. Check adb is installed; if not, install it (brew install android-platform-tools /
   winget install Google.PlatformTools / apt install adb).
2. Find the TV: `adb mdns services` + scan the local /24 for TCP 5555.
   If more than one device answers, ask me which one.
3. `adb connect <ip>:5555` and tell me to press "Allow" (and "Always allow") on the TV.
4. Print model, Android version, build fingerprint, RAM and free storage.

## Phase 2 — Audit (no changes)
1. Baseline: reboot is NOT allowed yet — just record /proc/meminfo (MemAvailable) now, and
   propose a clean-boot baseline later.
2. List every package and sort into a table with a plain-English description:
   - REQUIRED (boot, system UI, settings, Play Store/Services, keyboard, tuner/HDMI inputs,
     remote pairing, CEC, updates, network, the current launcher)
   - MINE (apps I use — check usage stats; ask if unsure)
   - SAFE TO DISABLE (promo/ads, recommendation & tracking/ACR services, demo/retail mode,
     region services I can't use, feedback/bug reporters, unused vendor apps)
3. Show the top RAM consumers (`dumpsys meminfo`).
4. Ask me to approve/remove items, and ask about: voice search, Chromecast, screensaver,
   boot-into-Live-TV, animation scales = 0, and the home-screen plan.

## Phase 3 — Apply (only what I approved)
1. Reboot once for a clean BEFORE measurement.
2. Disable approved packages, set animation scales, other approved settings — logging each.
3. Write the undo script. Reboot, measure AFTER, show a before/after table.

## Phase 4 — Ad blocking (if approved)
- Set Private DNS (DNS-over-TLS) to an ad-blocking resolver I choose
  (e.g. dns.adguard-dns.com), verify an ad domain no longer resolves and normal sites do.
- Warn me: strict Private DNS has no fallback — if the resolver is unreachable the TV loses
  internet. Offer a watchdog (Phase 5) or a manual one-line rollback.
- Offer a remote-friendly browser with built-in adblock from the Play Store (let me press Install).

## Phase 5 — Custom home screen (if approved)
Follow the plan in 02-custom-home-screen.md: a small native launcher APK (no heavy frameworks,
the TV has ~2 GB RAM) with clock + date, rotating 16:9 photo background, my favorite apps row,
HDMI inputs + Settings row, an "All apps" drawer, plus an accessibility service that keeps it
as the home screen (HOME key + boot), with crash-loop and DNS fail-safes.
Ask before downloading the JDK / Android SDK needed to build it.

## Phase 6 — QA
Run the checklist in 04-qa-and-hardening.md on the TV (launch every app, navigation, inputs,
reboot persistence, crash drill, DNS-outage drill). Fix what fails, re-run, then give me a short
report: what changed, before/after numbers, known limitations, how to undo.

Warn me before anything that will interrupt what's on the TV (app switching, reboots).
```
