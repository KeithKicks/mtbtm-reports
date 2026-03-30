# Changelog

All notable changes to the MTBTM Discovery Agent.

## [1.0.0] - 2026-03-30 — Premium Discovery Upgrade

### Added
- **7-phase discovery flow** (was 5): Idea Capture → Competitive Landscape → Feasibility → Configuration & Scope → Red Team Challenge → Blueprint → Complete
- **Competitive Landscape phase** — forces stakeholders to identify existing solutions, competitors, and articulate differentiation and "why now"
- **Red Team Challenge phase** — devil's advocate that presents the 3 biggest failure reasons and forces the stakeholder to defend or adapt
- **Scope negotiation** — agent actively pushes back when too many must-have features are listed, suggests cuts, uses "first value moment" test
- **Executive summary** — 2-4 sentence pitch in plain language added to PSD
- **User personas** — 1-3 structured persona cards (name, role, goals, pain points, tech comfort, context)
- **Competitive analysis** — competitors with strengths/weaknesses/differentiator, market gap, positioning statement
- **Viability score** — 4-dimension scoring (market size, differentiation, feasibility, time-to-value) with 1-10 scores, rationale, weighted overall, and go/no-go recommendation
- **Implementation roadmap** — phased build plan with features, dependencies, timeframes, milestones
- **Stack rationale** — explains why the recommended tech stack was chosen
- **User journey** — each feature maps to a step in the user's flow
- **Session persistence** — conversations saved to SQLite, survive server restarts and idle timeouts (7-day TTL)
- **Token-scoped session isolation** — `owner_id` on sessions, clients only see their own; `invite_tokens` table for access management
- **Tabbed PSD panel** — Overview, Validation, Features, Roadmap, Risks & NFRs, Tech
- **Replit/AI-ready export** — `.txt` prompt file optimized for pasting into Replit Agent, Cursor, or any AI coding tool
- **Multi-format export dropdown** — Markdown, JSON, and Replit/AI prompt from a single menu
- **Static survey form** at `/survey.html` — self-service 6-section questionnaire for tire-kicker intake, downloads as markdown
- **Phase progress bar** — 7-step visual indicator with completed/active/pending states and glow effect

### Changed
- PSD scoring expanded to **22 checks** across 7 categories (was 13 checks, 5 categories)
- PSD schema now includes `executiveSummary`, `targetUser.personas`, `competitiveAnalysis`, `viabilityScore`, `implementationRoadmap`, `technicalConstraints.stackRationale`, `mvpFeatures[].userJourney`
- Markdown export updated to include all new PSD sections
- UI redesigned with darker color scheme, better contrast, and premium visual hierarchy
- Session TTL increased from 2 hours to 7 days (DB-backed)

### Fixed
- **PSD generation timeout** — Anthropic SDK timeout increased from 120s to 300s; conversation history compressed before structured output call (37 messages → 1 summary); maxTokens reduced from 16384 to 8192
- Phase advance test updated for 7-phase flow

### Technical
- 35 tests passing (TypeScript + Vitest)
- Zero TypeScript errors
- End-to-end tested on Railway production: PSD generates in ~2.5 minutes, scores 100/100

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
