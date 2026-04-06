---
title: "Product Brief Distillate: isow_registration_app"
type: llm-distillate
source: "product-brief-isow_registration_app.md"
created: "2026-04-06"
purpose: "Token-efficient context for downstream PRD creation"
---

# Product Brief Distillate: ISOW Registration & Course App

## ISOW Organization Context

- Full name: International Student Organisation Wageningen, recognized by WUR since 1996
- ~1000 members/year, courses in hundreds, small volunteer-run association
- Located at Global Lounge, Plantage 2, 6708 WJ Wageningen; email: isow@wur.nl
- Board of 7 positions (President, Secretary/VP, Treasurer, Marketing, Activity, Course, Excursion Coordinator), rotates every 6 months, up to 15hrs/week commitment
- Office hours: Tue/Wed/Thu lunch + Tue/Thu evenings — this is when in-person registration and card pickup happen
- Current website: Squarespace (template: blackbird-bassoon), static, no dynamic features
- Communication: WhatsApp groups per course + main community, Instagram @isow_wur as primary updates channel
- Partner discounts at local businesses (Mama Bowls 5%, IceKootje 10%, Games&Geekery 10%, Copypoint 10%, Fun Times 30%, Jazz In Wageningen EUR 2) — require physical membership card
- Collaborates with ~12 other student organizations (ISA, PPI, UCA, CASSW, MEXA, IAAS, Thymos, etc.)

## Current Courses (from website, as of April 2026)

- 9 language courses: Dutch (3 levels), Japanese, Spanish, Persian, Chinese, Arabic, Calligraphy
- 7 dance courses: Bachata (2 levels), Kizomba, Salsa (2 levels), Twerkout, Belly Dance
- Several marked "currently unavailable": French, German, Portuguese, Dance HIIT, Bodyweight Fitness, Bollywood Dance, Yoga
- All courses free for members; non-members get one free trial lesson
- All teachers are volunteers
- Held at Forum building (languages) and Global Lounge (dance)
- Each course has its own WhatsApp group for reminders
- No mandatory attendance but registration is required
- Students can join mid-semester

## Current Registration Pain Points (observed from website)

- Membership form is Squarespace-embedded with a backup Google Form link (explicitly because "the google form isn't working")
- Google Form backup URL: `https://docs.google.com/forms/d/e/1FAIpQLSfcKjg4jtAeoM1BvVfvPE9jn2jOttE2Tq9lD986grhZkdpJ9A/viewform`
- Payment: bank transfer to IBAN NL48 INGB 0006 9015 79 or in-person cash/card at Global Lounge
- Physical card pickup required at Global Lounge during office hours
- Course registration process is not documented anywhere on the website — happens informally
- Event registration uses ad-hoc Google Forms per event, each with unique link
- Calendar page contains no calendar, just redirects to Instagram/WhatsApp
- Committee member page 404s (broken/unpublished)

## Competitive Landscape (detailed)

- **Membro** (membro.nl): EUR 180/yr for <=100 members, scales steeply. No course management, no digital photo ID. Named "Best Platform" by BUILD Magazine 2025
- **Banster** (banster.nl): Modular pricing, not transparent. No course enrollment/attendance. General-purpose association tool
- **Procurios** (procurios.com): Enterprise CRM, 25 years, Amsterdam-hosted. Vastly overkill and expensive for a student association
- **AllUnited** (allunited.nl): Sports club focused, 2500+ associations. No course enrollment, app-based, likely expensive
- **Mijn Club App**: Native app required (iOS/Android), fitness/gym oriented. No photo ID, no teacher roster, no web-only option
- **WildApricot**: USD 100-150/mo at 1000 contacts. No iDEAL, no course management, no digital card, US-centric
- **Join It**: USD 29-199/mo + service fees. Has Apple/Google Wallet cards (interesting precedent). No course management, Stripe-only
- **ClubExpress**: USD 0.30-0.42/member/mo. UI widely criticized. No iDEAL, no course management, US-focused
- **CiviCRM**: Free/open-source but requires WordPress/Drupal, steep learning curve, no course/attendance, no iDEAL OOTB
- **CourseStorm, Enrollsy, Regpack**: Course enrollment only, no membership management, no iDEAL, US-focused
- **Key gap**: No single product combines iDEAL + photo ID + course enrollment + teacher roster + no-install web app

## Payment Integration Analysis

- **Mollie (recommended)**: EUR 0.29-0.32/transaction flat, no monthly/setup fees, no lock-in. Official `mollie-api-python` SDK (PyPI, Python >= 3.8). Django packages: `django-mollie-ideal`, `django-payments-mollie`. Dutch company, native iDEAL, handles iDEAL-to-Wero transition transparently. Estimated cost for ISOW: ~EUR 290/yr for 1000 transactions
- **Stripe (alternative)**: EUR 0.29/transaction for iDEAL, 1.5% + EUR 0.25 for cards. Better ecosystem, supports Apple/Google Pay alongside iDEAL. Merchants need to update for Wero by end Q2 2026. More complex for iDEAL-only use case
- **iDEAL-to-Wero transition**: Starting March 31, 2026, full transition by end 2027. Payment providers (Mollie/Stripe) handle it transparently for developers — existing integrations continue to work
- **Design decision**: Registration = paid. No pending/unpaid states. Member exists in system only after payment confirmed (iDEAL webhook or admin recording cash receipt)

## Hosting Options Analysis

- **Render**: Free tier 512MB RAM but cold starts after 15min inactivity (30-60s wait, bad UX). Starter $7/mo. Free Postgres 30 days then $6/mo
- **Railway**: $5 one-time trial credit, then Hobby $5/mo with included usage. One-click Postgres. Fastest Django deploy
- **Fly.io**: No free tier (removed 2024). ~$1.94/mo VM + $2/mo IPv4 + $38/mo managed Postgres minimum. Not recommended for budget
- **Hetzner Cloud**: EUR 20 initial credits, CX11 ~EUR 3.29/mo (1 vCPU, 2GB RAM, 20GB SSD). Self-managed Postgres. European data residency (GDPR). Best value for production
- **Oracle Cloud**: Always-free ARM VPS (24GB RAM) but known for sudden account terminations. DIY setup
- **Recommendation**: Railway for dev/MVP, Hetzner for production. Avoid free tiers for production — cold starts kill the teacher-opening-roster-before-class experience

## Wallet Integration Feasibility

- **Apple Wallet**: HIGH feasibility. Libraries: `django-walletpass` (Django-native, push notifications), `wallet-py3k` (used by pretix), `applepassgenerator`. Requires Apple Developer Account ($99/yr) and Pass Type ID certificate (renew every 398 days). Generic pass layout supports photo, name, member number, QR/barcode
- **Google Wallet**: HIGH feasibility. Official REST API with Python samples. "Generic Pass" type. Requires Google Cloud project + Wallet API Issuer account (free). No certificate renewal needed
- **Combined**: `django-wallets` package claims both platforms. Membership card can include photo, name, QR code, expiry, branding
- **Cost**: Apple $99/yr + Google free = ~EUR 90/yr total
- **Alternative**: Simple mobile-web membership card page with QR code — zero cost, works on all phones. Wallet passes are a nice-to-have upgrade
- **Status**: Post-v1 nice-to-have

## Existing Prototype Architecture (from /home/cedric/repos/isow_app_prototype)

- Django 5.1 + Tailwind CSS 4 (via `django-tailwind-cli`) + SQLite + Pillow
- **Models**: `Member` (custom AbstractUser: role student/teacher/admin, photo, is_paid, role_color properties), `Membership` (OneToOne: plan 12m/6m, start/end dates, paid boolean), `Course` (ForeignKey to teacher, quarterly Q1-Q4, day/time/location/max_students), `Enrollment` (unique together constraint), `CourseSession` (auto-generated dates per quarter)
- **Auth**: `@role_required(*roles)` decorator, context processor for role-colored navbar (emerald=student, blue=teacher, amber=admin)
- **Payment flow**: Mock iDEAL with Dutch bank selection UI (ING, Rabobank, ABN AMRO). `/payment/` shows price, `/payment/confirm/` sets paid flags
- **Views**: register, profile, payment, payment_confirm, admin_panel, toggle_role, create_course, course_list, course_detail, enroll, unenroll, course_roster
- **Tests**: Empty placeholder files (accounts/tests.py, courses/tests.py)
- **Dependencies**: django>=5.1, django-tailwind-cli>=2.16, Pillow>=10
- 3 commits total

## Reference Stack Patterns (from /home/cedric/repos/tree_manager_app)

- Django 5.0+ / Python 3.11+ / PostgreSQL / Tailwind 3.4 + DaisyUI / HTMX 2.0 / UV package management
- **Settings split**: `config/settings/base.py` + `local.py` (dev overrides). Environment variables via `.env` + `python-dotenv`
- **Services layer**: Separate business logic from views (`apps/*/services/`), custom exception classes per service
- **Type hints**: `from __future__ import annotations`, strict mypy with `django-stubs`, `cast()` for type narrowing
- **Testing**: pytest + pytest-django, conftest.py with shared fixtures (`user_data`, `user`), test files in `apps/*/tests/test_*.py`
- **Code quality**: Ruff (linter, target py311) + mypy (strict) configured in pyproject.toml
- **Frontend**: DaisyUI theme customization in tailwind.config.js, HTMX for server-rendered partials (hx-get/hx-post/hx-target)
- **ETL pipeline**: `scripts/etl/` with download_sources.py, build_tree_database.py, load_to_django.py — pattern to replicate for Google Sheet migration
- **Auth**: Custom AbstractUser, password validators enabled
- 45+ commits, actively maintained

## Patterns to Carry Over to Production App

- Split settings (base/local/production)
- Services layer for business logic (especially payment processing)
- Strict mypy + Ruff in CI
- pytest + fixtures in conftest.py
- UV for package management
- DaisyUI theme customization
- Custom AbstractUser with role system
- `@role_required` decorator pattern
- Context processor for role-based UI theming

## Board Feedback (from meeting after prototype demo)

- Cash and admin manual registration must remain possible for people who can't use iDEAL
- Google Wallet / Apple Wallet integration would be nice if realistic
- PDF of the card for offline cases would be nice
- Sander provided verification steps for the prototype (course date checkboxes, session dates display, enroll/unenroll flow, nav label "Members" vs "Admin")

## Requirements Hints (captured during discovery)

- Data engineering pipeline needed: migrate existing Google Sheet member database to PostgreSQL
- Test suite must be very strong — no engineer available to maintain after deployment
- Minimal CI: GitHub Actions running tests, main branch protected
- Integration/UI tests (Selenium or similar) to verify website + UI is working
- Backup solution: member data exported as Excel sent to board member email
- Image data: accept risk of loss on migration/disaster recovery, may ask members to re-upload photos
- Membership pricing must be configurable (currently EUR 20/12mo and EUR 12.50/6mo but could change)
- Registration = paid, strict rule. No pending/unpaid member states
- Admin creates manual members only after receiving cash/bank transfer payment
- PWA capability for "Add to Home Screen" experience
- "Show Membership" screen optimized for counter display at partner businesses

## Rejected Ideas and Out-of-Scope Decisions

- **Event management/ticketing**: OUT — events don't require ISOW membership, Google Forms is flexible and maintainable for ad-hoc events
- **WhatsApp replacement/in-app messaging**: OUT — WhatsApp works well for communication, no reason to change it
- **Native mobile app**: OUT — adds app store maintenance burden, PWA achieves the same goal for free
- **Multi-language support**: OUT — English only, ISOW is an international English-speaking community
- **Physical card printing workflow**: OUT — physical cards may exist during transition but the app doesn't manage printing
- **v1/v1.1 split for course management**: REJECTED — course management stays in v1 because (a) prototype already has it built, (b) infrastructure work is the same either way, (c) teacher roster is half the value proposition, (d) splitting adds coordination overhead
- **Replacing the static website in v1**: OUT — both websites coexist with a link between them. Merging is a future consideration only

## Open Questions

- Organizational account structure: How does ISOW manage its email accounts and digital service access? Needs board discussion to align hosting/Mollie/domain accounts with their handover process
- GDPR specifics: Data controller is ISOW, but who is data processor if Cedric hosts? Formal data processing agreement may be needed. Retention periods for member data and photos undefined
- Which board role becomes the "System Steward"? Recommended Course Coordinator but needs board agreement
- Shared password manager: Does ISOW already use one? If not, recommend Bitwarden free org
- Photo moderation: What if members upload inappropriate images? Simple validation (file type, size) or manual review?
- Membership expiry mid-course: If a 6-month membership expires while enrolled in a course, what happens? Auto-remove from roster? Grace period?
- Data migration validation: How to confirm ~1000 members migrated correctly from Google Sheet? Duplicate handling? Expired membership treatment?
- Orientation week timing: When is the next one? Deployment should be tested and stable well before it
- Mollie account: Who signs up? Which IBAN is linked? Who has API key access?

## Reviewer Insights (from skeptic, opportunity, and adoption reviews)

- **Bus factor**: Single developer risk is the #1 continuity threat. Operations runbook and organizational account ownership are the mitigations
- **Teacher adoption flywheel**: If teachers rely on the roster weekly, they pressure the board to maintain the app across turnovers. Teacher experience should be delightful, not just functional
- **Orientation week**: Single highest-volume registration period. App must be proven before this moment. Design capacity and error handling for this peak
- **Longitudinal data**: After 2-3 semesters the app passively accumulates data ISOW has never had (renewal rates, course growth, revenue trends). Useful for university funding requests
- **Zero-handover operations**: Strongest positioning angle. Commercial SaaS tools require config knowledge; this app should require none
- **Replicability**: Other Wageningen student associations have similar problems. Clean codebase could be offered to them eventually — drives volunteer developer interest
- **Automated health monitoring**: Weekly Excel backup email + UptimeRobot free tier + "last backup" timestamp in admin panel — costs nothing, prevents silent failures
- **Parallel rollout**: Run alongside existing process for a full cycle. Define go/no-go criteria. Get explicit board sign-off before retiring old process. Document rollback procedure
