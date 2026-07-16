# Changelog

All notable changes to **TCPV4MAC**. Format based on
[Keep a Changelog](https://keepachangelog.com/); this project follows
[Semantic Versioning](https://semver.org/).

## [Unreleased]

### Added — 2026-07-16 · External dashboard card
- Added an **External** card to the dashboard (last card, right beside
  **Loopback**), mirroring the IPv4/IPv6 and TCP/UDP two-way switch cards.

### Fixed — 2026-07-16 · README corrections vs actual code
- Removed "Combine" from Architecture (unused; the app uses the Observation
  framework, `@Observable`).
- Fixed highlight colors: "amber" not "yellow" (matches `RowColor.swift`), and
  documented the persistent unsigned/listening/loopback/root color cues.
- Fixed the export claim: no TXT export exists; JSON is clipboard-copy only,
  not a file export — only CSV is an actual file export.
- Fixed the roadmap: kill process is already shipped (Phase 1), not a
  Phase 4 item; only closing an individual connection remains a future item.

## [1.0.0] — 2026-07-02

First release. A complete, lightweight macOS TCP/UDP connection inspector
(Sysinternals-TCPView-inspired): live table, filters, search, inspector, color
rules, context menu, CSV/JSON copy & export, admin mode, settings, uninstaller.
