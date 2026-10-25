# Phase 2 — Your own Google-TV-style home screen (no root)

```text
Build me a custom home screen (launcher) for my Android TV and install it over adb.

Constraints
- Old hardware (~2 GB RAM, Android 9/10). Keep it native and tiny: plain Java/Kotlin + Android
  framework views, no Compose/Flutter/AndroidX unless you confirm it fits. Target the TV's API level.
- Build on my computer. Ask me before downloading a JDK and the Android command-line tools
  (platform + build-tools). A Gradle-free build is fine: aapt2 → javac → d8 → zipalign → apksigner.
- Sideload with `adb install -r`. Log everything and extend the undo script
  (uninstall the APK, restore any settings you change).

Design
- Big thin clock + localized date (top-left), soft gradient scrims for readability.
- Full-screen photo background that cross-fades every ~30 s with a slow Ken Burns drift
  (start the fade and the drift together — no "pause then zoom").
  Load photos from bundled assets AND a folder on the TV I can drop files into.
  Offer to generate 8–12 16:9 wallpapers with an image model if I want.
- Row 1: my favorite apps as 16:9 banner cards, focus zoom + white ring, name above the row.
  Long-press: move left/right or remove. Last card: "All apps".
- Row 2: HDMI/AV inputs (from TvInputManager, show connected state) + Settings.
- "All apps" drawer (MENU key also opens it): OK = open, long-press OK = add/remove favorite.
- Explicit D-pad links between rows so focus never gets stuck at row ends.
- Let me override ugly app banners with your own (only inside this launcher).

Making it stick as the home screen (no root)
- Many TVs give the stock launcher top priority and re-enable it on every boot, so
  "set as default" is ignored. Use an AccessibilityService that:
  1) filters key events and consumes HOME → opens our launcher (no stock flash);
  2) on TYPE_WINDOW_STATE_CHANGED from the stock launcher → opens ours;
  3) on its first connect after each boot (compare Settings.Global.BOOT_COUNT) → opens ours
     and shows a full-screen TYPE_ACCESSIBILITY_OVERLAY "cover" with a saved snapshot of our
     home screen until it has drawn (minimum ~7 s, hard timeout ~11 s).
  Enable it via `settings put secure enabled_accessibility_services ...` (append, don't overwrite)
  and record the previous value for undo.
- Fail-safe: record crashes; if the launcher crashed 3+ times in 3 minutes, stop consuming HOME
  so the stock launcher works.
- BOOT_COMPLETED receiver only as a backup, and only if the service hasn't handled this boot
  (it arrives ~40 s late and would yank me out of an app).

Verify on the TV with screenshots (`adb exec-out screencap -p`) and show me the result.
```
