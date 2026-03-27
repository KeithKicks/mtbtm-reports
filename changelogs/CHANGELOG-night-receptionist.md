# Changelog

All notable changes to The Night Receptionist will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [0.12.0] - 2026-03-27

### Added
- Multi-user team access per business with role-based permissions (owner, manager, staff)
- Team management API: list, invite by email, update role, remove member
- Team Settings UI tab with permission-aware controls
- Centralized access control utility (`verifyBusinessAccess` + `hasMinRole`) across all route files
- Unique constraint on `business_users` (businessId, userId) to prevent duplicate memberships
- Owner fallback in team list for legacy businesses without a `business_users` row
- `GET /api/businesses` now returns businesses where user is owner OR team member
- `POST /api/businesses` auto-creates a `business_users` entry with role "owner"

### Changed
- All 10 route files now use centralized RBAC instead of duplicated ownership checks
- RBAC matrix enforced: staff+ for reads (bookings, conversations), manager+ for operational reads/writes (services, analytics, config, calendar, team invite), owner-only for business mutations and team role/remove

### Security
- Prevents inviting the business owner as a non-owner team member
- Self-removal prevention on team endpoints
- Admin passthrough preserved for agency-level access

## [0.11.0] - 2026-03-26

### Added
- Contact directory page with lightweight CRM functionality
- Contact deduplication by phone/email across conversations and bookings
- Contact detail slide-over panel with full conversation and booking history
- Contact search and status filtering (lead/booked)
- Business analytics API with configurable date ranges (7d, 30d, 90d, all-time)
- Analytics dashboard with real data: daily activity charts, channel breakdown, top services, conversion rate
- Hourly distribution analysis for lead patterns
- `GET /api/businesses/:id/contacts` with search and status query params
- `GET /api/businesses/:id/contacts/:contactId` with linked conversations and bookings
- `GET /api/businesses/:id/analytics` with channel breakdown, daily activity, hourly distribution

## [0.10.0] - 2026-03-25

### Changed
- Improved login flow: proper redirects on protected routes instead of bare auth cards
- Added logout accessibility from onboarding screen
- Empty states now include helpful guidance across all pages
- Mobile responsiveness improvements across dashboard, bookings, conversations, and settings pages

### Fixed
- Login redirect behavior on protected routes
- Various mobile layout issues

## [0.9.0] - 2026-03-24

### Added
- Widget embed code section in Settings with copy-paste `<script>` tag
- Allowed origins configuration for widget CORS control
- Real-time notification system for new bookings and qualified leads
- SMS notifications via Twilio (configurable per business)
- Email notifications via SendGrid (configurable per business)
- Notification preferences in business settings

## [0.8.0] - 2026-03-23

### Added
- Full conversation viewer with complete message threads
- Qualification info sidebar on conversations (customer details, qualification data, timeline stats)
- Manual booking creation form with service picker, date/time scheduling, urgency, and notes
- Booking detail panel with click-to-view from table
- Search and filter controls on Bookings page

## [0.7.0] - 2026-03-22

### Added
- Public marketing landing page at `/` (no auth required)
- Hero section with headline, tagline, and call-to-action
- Features section, "How it works" walkthrough, trust section
- Auto-redirect for authenticated users (`/dashboard` for users, `/agency` for admins)
- Night/moon aesthetic branding consistent with product theme

## [0.6.0] - 2026-03-21

### Added
- Agency admin portal for managing all customer businesses
- `role` field on users table ("user" default, "admin" for agency admins)
- Admin bootstrapping via `ADMIN_USER_IDS` environment variable
- Admin-only API routes under `/api/admin/*` with `isAdmin` middleware
- Agency Dashboard (`/agency`) with business directory, aggregate stats, search/filter/sort
- Create business on behalf of customers via `/agency/create-business`
- Drill-into-business view (`/agency/business/:businessId`) with full dashboard scope
- Admin passthrough on all business routes for cross-tenant access

## [0.5.0] - 2026-03-20

### Added
- Embeddable chat widget (`/api/widget.js`) for business websites
- Real-time streaming AI responses via SSE (`/api/public/business/:slug/chat/stream`)
- Public booking funnel page (`/book/:slug`) for direct customer interaction without auth
- Widget configuration and customization options

## [0.4.0] - 2026-03-19

### Added
- Twilio Voice integration for inbound call handling by AI agent
- Twilio SMS integration for inbound text message handling
- TwiML voice response generation with natural conversation flow
- Webhook endpoints for Twilio call and SMS events
- Business owner notification system for new bookings

## [0.3.0] - 2026-03-18

### Added
- React + Vite dashboard frontend with Tailwind CSS and shadcn/ui
- Replit Auth (OpenID Connect with PKCE) authentication flow
- Onboarding wizard for new businesses (details, services, AI personality, notifications)
- Dashboard overview with stats cards and charts
- Settings pages for business configuration management
- Sidebar navigation (Dashboard, Bookings, Conversations, Contacts, Settings)
- Mobile-responsive layout

## [0.2.0] - 2026-03-17

### Added
- AI conversation engine with 5-state machine (greeting, service identification, detail collection, availability check, booking confirmation)
- OpenAI integration (gpt-5-mini) for natural language processing and lead qualification
- Qualification data stored as JSONB within conversation records
- Google Calendar OAuth integration for real-time availability checking
- Calendar-based booking event creation
- Graceful fallback to business hours when Google Calendar is not configured
- Calendar service abstraction layer

## [0.1.0] - 2026-03-16

### Added
- Multi-tenant PostgreSQL database schema with Drizzle ORM
- Tables: users, sessions, businesses, business_users, business_config, services, bookings, conversations, messages
- Express 5 REST API with `/api` prefix
- Zod validation integrated with Drizzle schemas
- OpenAPI specification with Orval codegen for API clients and Zod schemas
- CRUD endpoints for businesses, services, bookings, conversations
- Session-based authentication middleware
- pnpm monorepo structure with shared libraries (api-spec, api-client-react, api-zod, db)
- TypeScript 5.9 with composite project references
