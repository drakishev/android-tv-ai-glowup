---
name: android-tv-makeover
description: Use when an AI agent is asked to speed up, debloat, ad-block, or redesign an Android TV / Google TV device over adb (no root). Holds safety rules, the phase plan, and non-obvious pitfalls.
---

# Android TV makeover — agent context

## Safety contract (always)
1. Read-only audit first; change nothing without the user's approval of a concrete list.
2. `pm disable-user --user 0` for system packages. Never `pm uninstall` a system app
   (exception: removing *Play Store updates* of a system app restores the factory version — ask first).
3. Log every change with the previous value; maintain ONE undo command/script; validate it parses.
4. Ask before reboots, downloads/installs, DNS/network changes, and before driving the TV UI.
5. Warn before interrupting the screen; if unexpected apps open during tests, a human is using the remote — stop.

## Connecting
- Discovery: `adb mdns services` + TCP 5555 scan of the local /24. Ask if several devices answer.
- First `adb connect` shows an RSA prompt on the TV — user must press Allow (tick "Always").
- Cloud/remote agents often can't reach the LAN: run adb on the user's computer (a script the user
  double-clicks, or a file-based command bridge the agent drops commands into).

## Measuring
- Compare `MemAvailable` between two **fresh boots** (reboot + ~90 s settle), never against a live snapshot.
- Boot timeline: `logcat -c` before reboot, then `logcat -d -b all -v time` and grep
  `START u0.*HOME`, `am_on_resume_called`, your service tag.

## Seeing and driving the TV
- Screenshot: `adb exec-out screencap -p > shot.png` (binary-safe redirect; on Windows use `cmd /c`).
  Secure/DRM surfaces (live tuner, some players) capture as empty files — that's expected.
- Keys: `input keyevent KEYCODE_DPAD_*|CENTER|BACK|HOME|MENU`.
- Long-press: `input keyevent --longpress` is often treated as a click on TV launchers. Real long press:
  `sendevent` on a virtual input device the shell can write (shell is in the `input` group), e.g. a
  "virtual-search" device: KEY down → sleep 1.3 → KEY up (find it with `getevent -lp`; KEY_SELECT = 353).
- Stock launcher channel rows: "Customize channels" at the bottom of the home screen. Some
  vendor/partner rows (e.g. a store's "recently added") can't be turned off there.

## Home-screen pitfalls (no root)
- The stock launcher may have higher intent-filter priority than any third-party HOME → "set default"
  (`cmd package set-home-activity`) is ignored. Disabling it may be **reverted at every boot** by firmware.
- Shell cannot disable another app's component (`Shell cannot change component state`).
- Working approach: AccessibilityService with `flagRequestFilterKeyEvents`:
  - consume KEYCODE_HOME and start your launcher (no flash, no 5 s app-switch delay);
  - redirect on TYPE_WINDOW_STATE_CHANGED from the stock launcher package;
  - on first connect per boot (Settings.Global.BOOT_COUNT) open home + TYPE_ACCESSIBILITY_OVERLAY cover
    (snapshot of your home) for ≥7 s, hard timeout ~11 s.
  - Shell-injected HOME bypasses the key filter → test with the physical remote.
  - Fail-safe: count crashes (UncaughtExceptionHandler → SharedPreferences.commit()); if ≥3 in 3 min,
    stop consuming HOME.
  - BOOT_COMPLETED arrives ~40 s after the UI; don't relaunch home from it if the service already did.
- Enable a11y via `settings put secure enabled_accessibility_services <existing>:<pkg>/<svc>` and
  `accessibility_enabled 1`; reinstalling the APK keeps it bound.

## Building an APK without Gradle (Windows/macOS/Linux)
- Tools: JDK 17 + Android cmdline-tools → `platforms;android-<api>`, `build-tools;34.0.0`.
- `aapt2 compile --dir res -o res.zip` → `aapt2 link -I android.jar --manifest ... -A assets --java gen`
  → `javac --release 8 -classpath android.jar` (NOT `-bootclasspath android.jar`: lambdas fail with
  "Unable to find method metafactory") → `d8 --min-api <api>` → add classes.dex to the APK → `zipalign -p 4` → `apksigner`.
- aapt2 on Windows stores asset subfolders with backslashes (`assets/bg\x.jpg`) → `AssetManager.list("bg")`
  is empty. Keep assets flat (`bg_01.jpg`) or build on Linux/macOS.
- Grant dev permissions after install: `pm grant <pkg> android.permission.WRITE_SECURE_SETTINGS`.
- Declare `ACCESS_NETWORK_STATE` if you touch ConnectivityManager (silent SecurityException otherwise).

## Ad blocking
- `private_dns_mode=hostname` + `private_dns_specifier=<DoT resolver>` (Android 9+). Strict mode has no
  fallback → add a watchdog or tell the user the one-line rollback.
- DNS probes must defeat caches: resolve random names on wildcard DNS (nip.io / sslip.io), off the main thread.
- DNS blocking can't remove ads served from the content's own domains.

## UI performance
- `settings put global window_animation_scale|transition_animation_scale|animator_duration_scale 0`.
- Decode backgrounds as RGB_565 at screen width; cross-fade + Ken Burns in one AnimatorSet starting together.
- Typical custom launcher footprint: ~35 MB PSS.
