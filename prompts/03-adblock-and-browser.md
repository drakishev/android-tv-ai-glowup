# Phase 3 — Ad blocking + an ad-free browser

```text
Add ad blocking to my Android TV.

1. System-wide: set Private DNS (DNS-over-TLS) via adb:
     settings put global private_dns_specifier <resolver hostname, e.g. dns.adguard-dns.com>
     settings put global private_dns_mode hostname
   Record the previous values for undo. Verify: a normal site resolves, a known ad domain
   (e.g. doubleclick.net) does not, and the network is still "validated".
   Tell me honestly what this can't block (ads served from the same domain as the content,
   e.g. in-app video ads on some platforms).
2. Resilience: strict Private DNS has NO fallback. If the resolver is unreachable the whole TV
   loses internet. If I have a custom launcher with an always-on service, add a watchdog:
   every 60 s resolve a random, never-cached name (e.g. r<random>.10.0.0.1.nip.io / sslip.io);
   after 3 failures in a row switch private_dns_mode to "opportunistic", retry the resolver
   hourly. It needs WRITE_SECURE_SETTINGS (grant with `pm grant`) and ACCESS_NETWORK_STATE.
   Test it by temporarily pointing the specifier at a bogus hostname.
3. Browser: open the Play Store page of a lightweight, remote-friendly, open-source TV browser
   with built-in adblock and let me (or you, if I say so) press Install. Confirm adblock is on.
   Add it to my home screen.
4. Log everything and extend the undo script.
```
