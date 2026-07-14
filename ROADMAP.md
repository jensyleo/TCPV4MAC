# TCPV4MAC — TODO and roadmap

> Working document. Status as of **2026-07-02**. Updated on every change
> (see the documentation rule in the CHANGELOG).
>
> 🎉 **v1.0 COMPLETE** (2026-07-02) — the utility is finished. The only real
> pending item is **publishing on GitHub** (3 user data points missing, see
> section C).

## ✅ Done

- **`TCPV4MACCore`** (Swift package, no UI, Swift 6 / macOS 14+):
  - Models `Connection` / `ProtocolType` / `TCPState` (with stable identity).
  - `ConnectionProvider` (protocol) + `LsofProvider` (parses `lsof -F`).
  - `LsofParser` (IPv4, IPv6 `[..]:port`, zones, `*:*`, `local->remote`).
  - `ConnectionDiffEngine` (added / removed / modified).
  - `tcpv4mac-cli` (headless smoke tool).
  - **29 XCTest tests** green; pipeline validated against real `lsof`.
- **Docs and publishing**: README/CHANGELOG (English), `LICENSE` (GNU GPL v3.0,
  Jensy Leonardo Martínez Cruz), GPLv3 headers in every Swift source,
  `.gitignore`, `Publicar GitHub/` folder. Repo **ready to publish** under GPL.
  App name decided: **TCPV4MAC**.

---

## 🚧 Phase 1 — MVP

### A. Core (`TCPV4MACCore`) — ✅ COMPLETE (2026-07-02)
- [x] **`ConnectionRepository`** (Repository Pattern): composes provider + diff
      engine + enrichment; `refresh()` = fetch → enrich → diff.
- [x] **Refresh engine**: async timer-based refresh (500 ms / 1 / 2 / 5 / 10 s),
      with pause/resume/change-interval, published via `AsyncStream`.
- [x] **Process enrichment (Foundation part)**: executable path and PPID via
      `libproc` (`LibprocMetadataProvider`).
- [x] Repository/refresh tests with mock providers → 29 tests green.
- [ ] *(Deferred)* `bytesSent`/`bytesReceived`: `lsof` does not provide them →
      requires a `nettop` backend (Phase 4 / bandwidth).

### B. SwiftUI app (`TCPV4MAC.app`)
- [x] **B1 — Scaffold + live table** (2026-07-02): project via XcodeGen
      (`project.yml`), no App Sandbox, its own bundle id (`com.jensyleo.tcpv4mac`),
      `ConnectionsViewModel` (MVVM) + `ContentView` auto-refreshing from the
      `RefreshEngine`. Status bar with counters.
- [x] **B2 — Table** (2026-07-02): sortable, resizable, show/hide + reorder
      columns. Later migrated to `NSTableView` (unlimited columns).
- [x] **B3 — Toolbar + Dashboard** (2026-07-02): search (`.searchable`),
      pause/resume, manual refresh, interval selector; live search
      (`Connection.matches`); summary cards; status bar with "Showing X of Y".
- [x] **B4 — Color rules** (2026-07-02): added=green, modified=amber,
      listening=blue, loopback=purple, root=gray (row text tint) + legend
      popover. Removed=red and Unsigned=orange in B6.
- [x] **B5 — Search + Filters** (2026-07-02): search (B3) + filter menu
      (protocol TCP/UDP, IP IPv4/IPv6, state Listening/Established/Other, include
      loopback) with active indicator and Reset. `ConnectionFilter.swift`.
- [x] **B6 — Inspector + AppKit enrichment** (2026-07-02): native inspector
      (`.inspector`) with icon/bundle/PID/PPID/user/architecture/signature/path/
      CPU/memory/threads/connection count; `ProcessEnricher` (NSWorkspace/
      NSRunningApp/SecStaticCode/proc_pidinfo, cached); Icon/Bundle ID/Duration
      columns; unsigned (orange) and removed (red ghost rows ~1.5s) colors.
- [x] **Main window**: Toolbar · Dashboard · Table · Inspector · Status bar.
- [x] **B7 — Context menu + CSV export** (2026-07-02): right-click → copy
      local/remote/PID/bundle/path, Reveal in Finder, Open Terminal, Kill
      process (confirmation, multi-select); CSV/JSON copy & export (NSSavePanel).
- [x] **B8 — Settings + icon** (2026-07-02): Settings (⌘,: appearance, persisted
      interval, auto-refresh, show icons) + full persistence + **app icon** (set
      from a 1024 PNG via `Scripts/make-appicon.sh`).

> 🎉 **MVP (Phase 1) COMPLETE** — B1–B8 done.

### Tests done
- [x] **Orange "unsigned" color** (2026-07-02): resolved and verified. On Apple
      Silicon a fully unsigned binary cannot run, so it was redefined: orange =
      no recognized identity (unsigned **or ad-hoc**). Verified with a throwaway
      ad-hoc listener (Signature "Ad-hoc (unsigned)" shown in orange), since
      removed — no trace remains in the repo.

### Known limitations / notes
- [ ] **Current-user connections only**: without root, `lsof` only sees the
      current user's processes → root/other-user connections (and the gray color)
      do not appear. To see the whole system: a privileged helper (`SMAppService`)
      or launching elevated.
  - **In-app option** (2026-07-02): ⋯ menu → **"Run as Administrator…"** relaunches
    the app as root. It shows the whole system (root rows in gray).
    Manual: `sudo /Applications/TCPV4MAC.app/Contents/MacOS/TCPV4MAC`.
  - [x] **Graphical sudo prompt (DONE 2026-07-02)** → **in-app password sheet**:
    validates with `sudo -S -k` (password via stdin) and relaunches the app as
    root (continuous, no re-prompt, no Terminal, no signing).
    `SudoRelaunch.swift` + `AdminPasswordSheet.swift`.
    Note: the app **is not signed**; a privileged `SMAppService` helper (which
    would require signing) is therefore not used.
- [x] **Double-click column-divider autofit** (Excel-style): RESOLVED with the
      migration to `NSTableView` (2026-07-02).
- [x] **Column limit / desync on reorder**: RESOLVED with `NSTableView` (unlimited
      columns, stable reorder/hide-show). Added CPU/Memory/Threads/Arch/Signature
      columns.

### C. Publishing — ✅ DONE (2026-07-13)
- [x] GPLv3 license + per-file headers + README notice → repo ready to publish.
- [x] `git init` + first commit (author: Jensy Leonardo Martínez Cruz
      <jensyleo@live.com>).
- [x] **Published on GitHub**: https://github.com/jensyleo/TCPV4MAC (public,
      pushed with `gh`).
- [x] **GitHub Release `v1.0.0`** with the compiled `.app` (zipped) attached,
      install instructions, and a Gatekeeper notice (app ships unsigned).
- [x] README screenshots (overview + inspector), sensitive data blurred.
- [ ] **Developer ID** signing + **notarization** (Gatekeeper) — optional; the
      app currently ships unsigned by design.

---

## 💡 Ideas / future improvements
- **Revisit the "External" toggle in the filter menu's Scope section** (2026-07-06):
  kept for now for symmetry with IPv4/IPv6 and to avoid an empty-list dead-end.
  Pending decision: whether to drop it and rely on a single "Loopback" toggle
  plus a guard that reverts to "show all" if both scope buckets end up off.
- **Event log (added/modified/removed)** — with fast refresh rates the
  green/amber/red flashes are visible only briefly. Add a way to **review them
  later**: a timestamped event panel/log (which connection appeared/changed/
  closed and when), and/or "freeze" or pin recently highlighted connections until
  dismissed. Fits naturally with the **Phase 2 history** (SQLite + timeline).

## 🔭 Future phases (summary)

> **SCOPE DECISION (2026-07-02):** TCPV4MAC stays a **lightweight utility**.
> Phase 2 onward (SQLite history, statistics, charts, etc.) will NOT be done here
> — it will be a **sister app in a new folder**, documented as a "fuller
> version". Here the app is only polished and, at most, could gain **lightweight,
> database-free** extras (reverse DNS, whois, in-memory event log) if decided.

- **Phase 2 → SEPARATE APP**: history (SQLite), statistics, charts, alerts, DNS, whois.
- **Phase 3**: GeoIP, ASN, threat intel (VirusTotal / AbuseIPDB / Shodan) — separate app.
- **Phase 4**: Endpoint Security, bandwidth, kill process, close connection — to evaluate.

---

## ➡️ Next step

Phase 1 (A + B1–B8) ✅ complete and shipped as **v1.0.0**. The only remaining
step is **publishing on GitHub** — waiting on the user data listed in section C.
