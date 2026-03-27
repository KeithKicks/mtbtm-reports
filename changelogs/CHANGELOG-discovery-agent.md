# Changelog

All notable changes to the MTBTM Discovery Agent.

## [0.7.0] - 2026-03-27

### Changed
- "Approve & Send" replaced with "Approve & Download" across entire UI
- Blueprint `.md` file auto-downloads immediately on approval
- Success screen simplified to download-focused flow (email removed)

## [0.6.0] - 2026-03-27

### Fixed
- **Hard stops at phase transitions** — textarea replaced with action buttons ("Continue" / "I want to make changes") so users can't free-type at the wrong time
- **PSD generation flow** — textarea hidden during blueprint phase; only "Generate Blueprint" button shown
- **Post-generation flow** — textarea hidden after PSD exists; only "Review Blueprint" and "Approve & Download" shown
- **Generate button appearing too early** — no longer shows during configuration phase
- **Random quick-reply buttons** — timeline/scale buttons no longer appear mid-conversation; now require actual question context
- **Raw JSON shown to users** — structured PSD data intercepted and replaced with friendly message pointing to Review Blueprint

## [0.5.0] - 2026-03-24

### Added
- **Role selection landing** — users choose: Business Owner, Building for a Client, Product Manager, or Exploring an Idea
- Modern dark UI with phase stepper, gradient accents, card-based layout
- Quick-reply suggestion chips for common answers
- File upload support (drag & drop, up to 5 files)
- PSD panel with formatted feature cards, risk table, config grid
- Download buttons for both `.md` and `.json` formats

### Fixed
- PSD generation reliability — retry button on failure
- Phase stepper now accurately reflects progress

## [0.4.0] - 2026-03-24

### Changed
- Simplified UI to chat + generate + approve flow
- Removed manual phase navigation bar (agent handles transitions automatically)

## [0.3.0] - 2026-03-24

### Changed
- Renamed "PSD" to "App Blueprint" throughout UI
- Added email-to-MTBTM flow on approval

### Fixed
- Approve action crash when PSD score below threshold

## [0.2.0] - 2026-03-24

### Fixed
- Phase transitions: auto-advance now works when agent signals `<<PHASE_COMPLETE>>`
- Bot re-engages properly when entering a new phase

## [0.1.0] - 2026-03-24

### Added
- Initial release: 5-phase guided discovery (Idea → Feasibility → Config → Blueprint → Complete)
- Claude-powered conversational agent with structured PSD output
- Phase readiness detection via keyword pattern matching
- PSD generation with Zod-validated structured output
- 13-point PSD scoring rubric (70/100 threshold)
- SQLite persistence for projects and artifacts
- Express API with session management (2hr TTL)
- React 18 frontend (inline Babel, no build step)
