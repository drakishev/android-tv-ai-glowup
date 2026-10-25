# Phase 4 — QA, fault tolerance, reboot checks

```text
Check the whole TV setup for errors, bugs and fault tolerance. Warn me before anything that
interrupts what's on screen (app switching, reboots), and pause if someone is using the TV.

Diagnostics (read-only)
- Crash buffer (`logcat -b crash`), ANRs, our tags in logcat.
- RAM (MemAvailable), our launcher's PSS, storage, WebView version.
- Every package I disabled is still disabled; settings (animations, DNS, accessibility) persist.
- The undo script parses (PowerShell: Parser::ParseFile; bash: `bash -n`).

On-device functional tests (automate with screencap + input keyevent, report PASS/FAIL)
- Each favorite card opens the right app; "All apps" opens and Back closes it.
- D-pad: from the LAST card of row 1, Down lands in row 2; Up returns. No dead ends.
- An HDMI input opens; Settings opens.
- Stock launcher appears → our home comes back.
- Real remote HOME (ask me to press it) → our home, instantly.
- Reboot: capture screenshots every ~1–2 s from the moment adb is up; the stock launcher must
  not be visible; open an app right after boot and confirm nothing yanks it away after 60 s.

Fault drills
- Crash the launcher 3× (`am crash <pkg>`): HOME must fall back to stock; after the window
  expires, our home works again.
- Break DNS (bogus Private DNS hostname): the watchdog must restore internet automatically;
  then restore the real resolver.

Code review
- Missing permissions (check logcat for SecurityException), main-thread network calls in the
  accessibility service, leaks on destroy, races after async decode, unbounded overlays.

Fix what fails, rebuild, re-run only the failed tests, then give me a short report.
```

**Gotchas seen in practice**
- In PowerShell `R` is an alias for `Invoke-History` — don't name a helper function `R`.
- Shell-injected keys (`input keyevent HOME`) bypass accessibility key filters; test HOME with the real remote.
- After a HOME press Android delays non-system activity starts ~5 s ("app switch" protection); key-filter interception avoids that.
- If tests suddenly open an app nobody asked for — someone may be using the remote. Stop and ask.
