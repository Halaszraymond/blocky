# Blocky
 
A small Windows desktop app that blocks distracting websites on a schedule I set — showing a shortlist of more productive things to do instead — with a manual override for when I genuinely need to get through. No editing config files by hand, no digging through the Windows hosts file.
 
## 1. The demo
 
I open the app. It shows my block list (`reddit.com`, `youtube.com`, `x.com`) and the current schedule — Study Mode, Mon–Fri 09:00–17:00 — with the status panel reading **"Blocking active — 3h 00m remaining"**, since it's 14:00 on a Tuesday. I try to open reddit.com anyway; instead of a broken connection, the browser shows a local page: "reddit.com is blocked until 17:00" with a shortlist underneath — *read tomorrow's lecture notes, 10-minute walk*. I click **Override** in the app, type a reason ("checking a work thread"), and reddit.com unblocks — the panel now reads **"reddit.com unblocked until 17:00"** with a countdown. Switching to the History tab, I see the override logged with the domain, timestamp, and my reason.
 
## 2. The shape
 
```
in           a block-list + weekly schedule + a shared shortlist of productive
             suggestions, entered through the app's UI
out          an updated Windows hosts file + a local page shown in the browser
             when a blocked site is requested + a live status display in the
             app + an override log
in between   a background checker compares the current time against each
             domain's schedule; it adds or removes hosts-file entries as
             windows open and close, points blocked domains at a small local
             web server that serves the block page with the suggestion
             shortlist, and temporarily exempts a domain when I override it,
             logging the reason
```
 
## 3. The size
 
**First useful version:**
- Add, edit, and remove blocked domains through a text input in the UI
- Define a weekly schedule (days + time range) that applies to the block list
- A background check (roughly once a minute) compares the clock against the schedule and writes/removes entries in the Windows hosts file accordingly
- A live status panel showing which domains are currently blocked and time remaining until the next change
- A manual override: pick a domain, type a required reason, and it's unblocked until the next scheduled window — logged with domain, timestamp, and reason
- A history tab listing past overrides
- A shared shortlist of productive suggestions, defined once in the app, shown on a local page in the browser whenever a blocked domain (over plain HTTP) is requested
**Not this term:**
- Blocking specific pages/paths rather than whole domains (would need a proxy or browser extension)
- Multiple named schedule profiles (e.g. "Study Mode" vs "Deep Work") — one active schedule only
- Per-site suggestion lists — one shared shortlist covers every block for now
- A working suggestion page for HTTPS sites — without a trusted certificate, HTTPS requests to a blocked domain will show the browser's own security warning instead of the custom page; see risks below
- Running as a persistent background service that survives a reboot without me relaunching it
- Any tamper-resistance — since I have admin rights on my own machine, I can always edit the hosts file back myself; this tool isn't meant to be uncircumventable
- macOS/Linux support
- Usage analytics beyond the raw override log (e.g. "you tried to visit reddit.com 4 times this week")
## 4. How we would know it works
 
- Given a domain on the block list and the current time inside its scheduled window, the Windows hosts file contains a `127.0.0.1` redirect entry for that domain.
- Given an override submitted with a reason for a currently-blocked domain, the hosts file entry for that domain is removed, and the history log gains an entry with the domain, timestamp, and reason.
- Given the current time outside the scheduled window for a domain, the hosts file contains no entry for that domain — including cleaning up any leftover entry from before the schedule changed.
- Given a blocked domain requested over plain HTTP while its block window is active, the browser displays the local block page with the shared suggestion shortlist, rather than a connection error.
- Given a malformed domain entered into the block list (e.g. missing a dot, containing spaces), the app rejects it with an error message and never writes it to the hosts file.
## 5. What could stop this
 
- **Admin privileges.** Editing `C:\Windows\System32\drivers\etc\hosts` requires elevated rights. The app needs to either run elevated from the start or trigger a UAC prompt — I haven't tested which is smoother in Tkinter/customtkinter yet.
- **Domain-level blunt blocking.** This blocks whole domains, not specific pages, and doesn't reliably handle sites served across many IPs/CDNs without extra care.
- **HTTPS.** Most sites redirect HTTP to HTTPS by default, and a local server can't present a valid certificate for someone else's domain. So for HTTPS sites, the browser will most likely show its own "connection not private" warning rather than my suggestion page — the friendly block page reliably works for plain-HTTP requests only. I'm treating this as an accepted limitation rather than solving it with self-signed certificates, which would add real scope.
- **Not tamper-proof by design.** Since I'm the same user with admin rights, I could edit the hosts file directly and bypass the tool entirely. That's acceptable here — the point is friction and logging, not enforcement — but worth stating plainly.
- **Data/privacy.** All data is local config (my own block list and schedule) with no personal or sensitive third-party data involved, so the full real setup can be shown in class.
## Tech
 
Python, with a `customtkinter` UI for a modern look with minimal setup overhead. Config (block list + schedule) is stored as YAML under the hood but never hand-edited — all changes go through the app.
