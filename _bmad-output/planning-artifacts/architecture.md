---
stepsCompleted:
  - 'step-01-init'
  - 'step-02-context'
  - 'step-03-starter'
  - 'step-04-decisions'
  - 'step-05-patterns'
  - 'step-06-structure'
  - 'step-07-validation'
  - 'step-08-complete'
lastStep: 8
status: 'complete'
completedAt: '2026-04-11'
inputDocuments:
  - 'prd.md'
  - 'product-brief-isow_registration_app.md'
  - 'product-brief-isow_registration_app-distillate.md'
  - 'ux-design-specification.md'
  - 'excel-data-assessment.md'
workflowType: 'architecture'
project_name: 'isow_registration_app'
user_name: 'Cedric'
date: '2026-04-11'
revisions:
  - date: '2026-05-05'
    summary: 'Hosting decided (Hetzner Cloud CX23); migration scope narrowed to active-only members based on Excel data assessment; Member model extended with phone/nationality/occupation/notes; credential custody resolved as Gmail-anchored model (no shared password manager).'
---

# Architecture Decision Document

_This document builds collaboratively through step-by-step discovery. Sections are appended as we work through each architectural decision together._

## Project Context Analysis

### Requirements Overview

**Functional Requirements:**

51 functional requirements across 11 categories. The architecture must support two primary workflows that the PRD identifies as broken today:

1. **Registration workflow** (FR1-FR9, FR36-FR39, FR47): Self-service registration with photo upload and iDEAL payment in a single session, plus admin-created cash/bank transfer registrations. The strict invariant — a member exists only after payment is confirmed — eliminates pending states and simplifies the data model.

2. **Teacher roster verification** (FR21-FR24): Real-time photo roster with membership status. This is the adoption flywheel — if teachers depend on it weekly, the app survives board turnover.

Supporting capabilities: course management (FR13-FR20), admin panel (FR25-FR31), superadmin settings (FR32-FR35), digital membership card (FR10-FR12), data backup/restore (FR40-FR43), GDPR self-service (FR47-FR49), and monitoring (FR50-FR51).

**Non-Functional Requirements:**

- **Performance:** Page load <2s on 4G, roster with photos <3s, 50 concurrent users, no cold starts
- **Reliability:** 99% uptime (~3.6 days downtime/year acceptable), weekly backup, health check endpoint
- **Integration:** Mollie (payment), SMTP/email provider (transactional), UptimeRobot (monitoring), Cloudflare (DNS/proxy)
- **Security:** Django built-in protections enforced, HTTPS everywhere, rate limiting (5 req/min/IP on auth endpoints), Cloudflare free tier, admin action audit logging

**Scale & Complexity:**

- Primary domain: Full-stack web application (Django server-rendered MPA with PWA capability)
- Complexity level: Medium
- Estimated architectural components: ~8-10 (auth/accounts, membership/payment, courses/enrollment, roster, admin panel, backup/export, photo management, email, monitoring, PWA manifest/service worker)

### Technical Constraints & Dependencies

**Stack (decided in PRD/brief):**
- Django 5.x + Python 3.11+
- PostgreSQL
- Tailwind CSS + DaisyUI (themeable component library)
- HTMX for partial page updates
- Mollie Python SDK for iDEAL payment
- UV for package management
- pytest + pytest-django for testing
- Ruff + mypy (strict) for code quality
- GitHub Actions CI with protected main branch

**Operational constraints:**
- Solo developer (Cedric, ML engineer background) — no ongoing development team after deployment
- Must survive complete board replacement every 6 months with zero handover friction
- EU-based hosting required (GDPR)
- Budget: ~EUR 330-350/year total operating costs (hosting + payment processing)
- Pinned dependencies for stability (trade-off: security patches require manual updates)

**Existing codebase context:**
- Working prototype exists (`isow_app_prototype`) with Django 5.1, custom AbstractUser, role system, mock payment flow, and course management — 3 commits, SQLite, no tests
- Reference stack (`tree_manager_app`) provides proven patterns: split settings, services layer, strict mypy, pytest fixtures, DaisyUI theming — 45+ commits, actively maintained

### Cross-Cutting Concerns Identified

1. **Authentication & Authorization** — Custom AbstractUser with 4-tier role system (member/teacher/admin/superadmin). `@role_required` decorator pattern. Role-based view access, template rendering, and UI theming. Rate limiting on auth endpoints.

2. **Payment lifecycle** — Mollie iDEAL integration with webhook-based payment confirmation. Handles: successful payment (member activation), failed/cancelled payment (recovery flow), webhook retries, and admin-recorded cash payments. The strict "paid = exists" invariant simplifies the entire data model.

3. **Photo management** — Upload validation (JPEG/PNG, <5MB), client-side compression, server-side resize, persistent storage, serving in roster grids and membership cards. Photos NOT included in backup — accepted loss on disaster recovery.

4. **Transactional email** — Account invitation (admin-created members), password reset, membership expiry notifications (7 days before + on expiry date), weekly backup delivery. Via SMTP or email service provider.

5. **GDPR data lifecycle** — Consent capture at registration, self-service account deletion (cascading: profile + photo + enrollments), admin-initiated deletion, data export (Excel), EU hosting. No tracking cookies or analytics.

6. **Backup & data portability** — Automated weekly Excel export emailed to association email. Same Excel format used for Google Sheets migration, backup restore, and admin export. Import/restore capability in admin panel.

7. **Security hardening** — Django built-in protections (CSRF, XSS, SQL injection, clickjacking) never disabled. HTTPS enforced. Session cookies: HttpOnly, Secure, SameSite. Cloudflare free tier for DDoS/CDN. Admin action audit logging.

8. **Monitoring & health** — Lightweight health check endpoint for UptimeRobot polling. Last backup timestamp visible in admin dashboard. Error logging to persistent location.

## Starter Template Evaluation

### Primary Technology Domain

Full-stack Django web application (server-rendered MPA with PWA capability), based on project requirements analysis and existing developer expertise.

### Starter Options Considered

| Option | Evaluation | Verdict |
|---|---|---|
| **Cookiecutter Django** (2026.03.28) | Production-ready but heavyweight. Docker/Traefik, Celery, REST framework — features ISOW doesn't need. No HTMX/DaisyUI. Stripping > building. | Rejected — overkill |
| **Community HTMX+DaisyUI starters** | Low-activity GitHub repos. Solve initial setup but lack split settings, services layer, strict typing, production patterns. | Rejected — not production-ready |
| **`django-admin startproject` + tree_manager_app patterns** | Clean foundation with proven patterns already battle-tested in developer's own reference stack. Exact stack match. | **Selected** |

### Selected Starter: django-admin startproject + custom structure

**Rationale for Selection:**

The developer has 45+ commits of proven Django patterns in a reference stack (`tree_manager_app`) using the identical technology choices: Django + PostgreSQL + Tailwind/DaisyUI + HTMX + UV + pytest + mypy + Ruff. Starting from `django-admin startproject` and applying these established patterns provides a clean, fully-understood foundation without inherited complexity or unused features. For a solo-developer project that must survive without its creator, every line of code should be intentional and understood.

**Initialization Command:**

```bash
uv init isow_registration_app
cd isow_registration_app
uv add django
uv run django-admin startproject config .
```

**Architectural Decisions Provided by Starter:**

**Language & Runtime:**
- Python 3.11+ with Django 5.2 LTS (security support until April 2028)
- Type hints with `from __future__ import annotations`, strict mypy with `django-stubs`

**Styling Solution:**
- Tailwind CSS 4.2 via `django-tailwind-cli` 4.4 (no Node.js required — uses prebuilt CLI binary)
- DaisyUI 5.5 enabled via `TAILWIND_CLI_USE_DAISY_UI = True`
- Custom ISOW theme with role-based accent color variants

**Frontend Interactivity:**
- HTMX 2.0.8 for partial page updates (roster refresh, enrollment actions)
- No JavaScript framework — HTMX attributes on standard HTML elements

**Build Tooling:**
- UV for Python package management and virtual environment
- `django-tailwind-cli` handles Tailwind CSS compilation (dev watch mode + production build)
- No webpack, no Vite, no Node.js in the build chain

**Testing Framework:**
- pytest + pytest-django with `conftest.py` shared fixtures
- Test files in `apps/*/tests/test_*.py`
- GitHub Actions CI on every push to protected main branch

**Code Quality:**
- Ruff for linting (target py311)
- mypy strict mode with `django-stubs` for type checking
- Both enforced in CI

**Code Organization:**
- `config/` for project settings (split: `base.py`, `local.py`, `production.py`)
- `apps/` directory grouping Django apps
- Services layer (`apps/*/services/`) separating business logic from views
- Environment variables via `.env` + `python-dotenv`

**Development Experience:**
- `uv run manage.py runserver` + `uv run manage.py tailwind watch` for development
- Hot reload for both Django and Tailwind CSS changes
- Rich CLI error messages from django-tailwind-cli

**Note:** Project initialization using this approach should be the first implementation story.

## Core Architectural Decisions

### Decision Priority Analysis

**Critical Decisions (Block Implementation):**
- Data architecture (photo storage, Excel format, migration strategy)
- Authentication & security (rate limiting, password hashing, audit logging)
- Communication integrations (Gmail SMTP, Mollie webhooks, cron scheduling)
- Frontend architecture (WhiteNoise, PWA scope, photo compression)
- Deployment approach (Gunicorn, git-based deploy, .env config)

**Recommended Board Decisions (Do Not Block Deployment):**
- System steward role assignment — purely a designation; the app runs whether or not someone holds the title.

_Resolved 2026-05-05: hosting provider (Hetzner Cloud CX23); credential custody model (Gmail-anchored — no shared password manager needed)._

### Data Architecture

| Decision | Choice | Rationale |
|---|---|---|
| Photo storage | Local filesystem via Django `ImageField` | ~500MB/year at ISOW's scale. No object storage needed. File path stored in database, file on disk under `MEDIA_ROOT/photos/`. Deleted with member on GDPR removal. |
| Excel library | openpyxl | Standard Python `.xlsx` library. One format for backup export, admin export, Google Sheets migration import, and backup restore. |
| Caching | None | 50 concurrent users, ~1000 members. PostgreSQL handles all queries directly. No Redis, no Memcached. If a page is slow, optimize the query. |
| Data migration | Two-step: throwaway converter + standard import | Migration is split so `import_members` only ever consumes the standard backup `.xlsx` format. **Step 1:** export Google Sheets to `.xlsx`, then run `scripts/convert_legacy_xlsx.py legacy.xlsx → standard.xlsx` — a throwaway script that owns the messy concerns (active-only filtering, the 1 missing-email row, payment history filtering, column renames). The intermediate file is human-reviewable in Excel before commit. **Step 2:** `manage.py import_members standard.xlsx` — same code path used for DR restore. Migration exercises the restore path for real, and `import_members` stays single-purpose. See **Migration Scope** below for active-only cohort rules (enforced by the converter, not the importer). |
| Migration scope | Active members only | Of 2,010 historical rows in the Excel, only the ~257 with a future expiration date are imported. Recently-expired members re-register through the normal flow. The 16 rows with status `close` are part of the active cohort and are migrated. |
| Migration data rules | Email required, all other fields optional | The "paid = exists" invariant means the only field the system strictly needs is `email` (for login + invitation). `name`, `phone`, `nationality`, `occupation`, `notes` are imported as-is from Excel without normalization — members can correct their own profile after first login. The 1 active row missing an email requires manual decision before import. |
| Payment history migration | Filtered to active members | The Excel's 656 historical payment records are filtered to the ~237 belonging to the migrated active members and imported as `Payment` rows. Payments tied to non-migrated members are left in the Excel archive. |
| Photo migration | None — uploaded on first login | The Excel contains no member photos. All 257 migrated members start photoless; the teacher roster shows a placeholder (initials/silhouette) until a photo is uploaded. |

### Authentication & Security

| Decision | Choice | Rationale |
|---|---|---|
| Rate limiting | django-ratelimit | Decorator-based (`@ratelimit(key='ip', rate='5/m')`). No Redis dependency — uses Django's local memory cache. Single-server deployment makes this sufficient. |
| Password hashing | Argon2 (via argon2-cffi) | Django's recommended hasher, more resistant to GPU attacks than default PBKDF2. One-line settings change. |
| Audit logging | Custom `AuditLog` model | Lightweight model (actor, action, target, timestamp). Written by services layer on admin actions: member creation/deletion, role changes, payment recording. No third-party package — narrow, well-defined needs. |
| Session config | Django defaults + hardening | Database-backed sessions. Cookies: `HttpOnly=True`, `Secure=True`, `SameSite=Lax`. 2-week expiry for members. |

### Communication & Integration

| Decision | Choice | Rationale |
|---|---|---|
| Email provider | Gmail SMTP | ISOW organizational Gmail account with app password. `smtp.gmail.com:587/TLS`. Free. Daily send limit (500-2000/day) far exceeds ISOW's needs. Handles: account invitations, password reset, expiry notifications, weekly backup delivery. |
| Mollie webhooks | Idempotent endpoint + server-side verification | Single endpoint (`/payment/webhook/`). Always fetches payment status from Mollie API — never trusts webhook payload alone. Idempotent: already-activated member = no-op. `PaymentService` owns logic. |
| Scheduled tasks | System cron + management commands | Weekly: `manage.py export_backup`. Daily: `manage.py send_expiry_notifications`. No Celery. Two cron entries, documented in operations runbook. |
| Error handling | Typed service exceptions | Services raise `PaymentFailedError`, `MembershipExpiredError`, etc. Views catch and render user-facing messages. Unhandled exceptions log to file + generic 500 page. No Sentry — proportionate to scale. |

### Frontend Architecture

| Decision | Choice | Rationale |
|---|---|---|
| Static file serving | WhiteNoise | Serves static files from Django process. Gzip + cache headers automatic. No Nginx needed for static. One middleware line. |
| Media files (photos) | Django serves directly in production | At ISOW's scale (~500MB/year of photos), Django serving from `MEDIA_ROOT` is sufficient on the Hetzner CX23's 40GB SSD. No Nginx needed. If a future board decides to add Nginx as a reverse proxy, taking over `/media/` is a config change, not a code change. |
| PWA | Install prompt only, no caching | Minimal service worker for "Add to Home Screen" + manifest (icon, splash, theme). No offline cache — all actions require network. Avoids stale cache bugs. |
| Photo compression | Browser-native canvas API | ~30 lines vanilla JS in template partial. Resize to max 800px, export JPEG ~80% quality before upload. No npm package, no build step. Server-side validation (5MB, JPEG/PNG) as safety net. |

### Infrastructure & Deployment (Application-Level)

| Decision | Choice | Rationale |
|---|---|---|
| Hosting | **Hetzner Cloud CX23** (decided 2026-05-05) | 2 vCPU, 4GB RAM, 40GB NVMe SSD, 20TB traffic, EU data residency (Germany/Finland), ~€42/yr. Selected after evaluation of alternatives (see Board Decision Brief below). |
| Operating system | Ubuntu 24.04 LTS | Long-term support until 2029, standard target for Django deployments, well-documented. |
| Process management | systemd unit for Gunicorn | Auto-restart on failure, boot-time startup, logs to journald. Single `/etc/systemd/system/isow.service` file checked into the repo. |
| Firewall | ufw with allow 22/80/443 only | Default-deny inbound; SSH key-only login (no password auth). |
| OS security updates | unattended-upgrades | Automatic install of security patches. Pinned application dependencies are still upgraded manually. |
| WSGI server | Gunicorn | Standard Django production server. Workers = `2 * CPU + 1` (so 5 workers on CX23). |
| Deployment | Git-based with deploy script | CI runs on push (tests, lint, types). Production: `deploy.sh` runs git pull, uv sync, migrate, collectstatic, systemctl restart isow. No Docker. |
| Database | PostgreSQL on same server | Single-machine deployment. No managed database service at this scale. Belt-and-suspenders backup: weekly Excel export + `pg_dump` via cron, both retained on the server and emailed off-server. |
| SSL/TLS | Cloudflare edge + origin certificate | Cloudflare handles edge SSL (free). Origin encrypted via Cloudflare origin cert (free, 15-year validity). Zero certificate renewal headaches. |
| Environment config | `.env` + python-dotenv | All secrets in `.env` on server. Never in code, never in git. Same pattern as reference stack. |
| Container | None (no Docker) | Django runs directly on the server. Simpler to debug, maintain, and understand for future volunteers. If portability needed later, adding a Dockerfile is straightforward. |

### Board Decision Brief: Infrastructure

One decision remains open for the board (system steward role) and does not block deployment — it is purely a designation, recommended before the app is handed off to future boards. The other two decisions (hosting provider, credential custody model) are resolved.

#### Decision 1: Hosting Provider — RESOLVED (2026-05-05)

**Decision: Hetzner Cloud CX23.** The board approved the recommendation. The evaluation table below is retained for reference. Implementation now assumes a Hetzner CX23 host (Ubuntu 24.04 LTS) — see Infrastructure & Deployment section above for the resulting application-level configuration.

#### Decision 1 evaluation table (reference)

| | Hetzner Cloud CX23 | Railway Hobby | PythonAnywhere EU | WUR-Managed Server | Local Computer (Global Lounge) |
|---|---|---|---|---|---|
| **Monthly cost** | ~€3.49/mo (~€42/yr) | ~$5/mo (~€55/yr) | €10/mo (€120/yr) | Potentially free | ~€50-100 one-time |
| **Total annual cost** (hosting + Mollie) | ~€332/yr | ~€345/yr | ~€410/yr | ~€290/yr (Mollie only) | ~€290/yr (Mollie only) |
| **Specs** | 2 vCPU, 4GB RAM, 40GB SSD, 20TB traffic | Usage-based (~$5-8/mo for Django + PostgreSQL) | Managed Django environment | Unknown — depends on WUR offering | Depends on hardware (e.g., Raspberry Pi: 4-8GB RAM) |
| **EU data residency** | Yes — Germany/Finland | No — US by default, EU uncertain | Yes — eu.pythonanywhere.com | Yes — WUR infrastructure in NL | Yes — physically in Wageningen |
| **PostgreSQL** | Self-installed | One-click managed | Available on paid plans | Likely available | Self-installed |
| **Setup complexity** | Medium — one-time server config | Low — connect GitHub, deploy | Low — upload code, configure | Unknown — depends on WUR IT policies | Medium-High — hardware + network + maintenance |
| **Maintenance** | Self-managed (OS updates, security patches) | Platform-managed | Platform-managed | WUR IT-managed (potentially) | Self-managed + hardware reliability |
| **Cold starts** | None | None on Hobby plan | None on paid plans | None (presumably) | None |
| **Cron jobs** | Full system cron | Built-in cron | 1 daily task (Developer plan) | Likely full cron | Full system cron |
| **Reliability** | Professional data center, NVMe SSD, DDoS protection | Platform SLA | Platform SLA | University-grade infrastructure | Depends on office internet, power, hardware |
| **Key risk** | Self-managed maintenance | EU data residency unclear | Most expensive, least flexible | Availability unknown — must ask WUR IT | Single point of failure, no redundancy |

**Options evaluated and rejected:**
- **Google Cloud free tier:** e2-micro (1GB RAM) in US regions only — disqualified by EU residency requirement
- **Oracle Cloud Always Free:** Known for sudden account terminations — unacceptable reliability risk for production

**Recommendation:** Hetzner Cloud CX23 as the safe default — cheapest paid option, EU-native, generous specs. However, **WUR-managed server is the best option if available** — free, EU, professionally managed, university-grade reliability. ISOW should inquire with WUR IT before committing to a paid service. Local computer at Global Lounge is not recommended — office internet/power dependency and no redundancy make it unsuitable for a system that needs to survive board replacement.

#### Decision 2: Credential Custody Model — RESOLVED (2026-05-05)

**Decision: Gmail-anchored model. No shared password manager.**

Every third-party account that needs to outlive any single board (Hetzner, Cloudflare, Mollie, domain registrar, etc.) is **registered using the ISOW organizational Gmail address**, not a personal email. This makes the ISOW Gmail account the single root credential — whoever holds it can perform a password reset on every other service and take over operations.

**Operational rules:**

- All third-party accounts use `isow@wur.nl` (or the equivalent ISOW org Gmail) as the registered email.
- Two-factor authentication, where required, uses methods that don't tie to a personal device (Gmail-deliverable codes, recovery codes printed and stored with the board, or a TOTP secret saved alongside the credential).
- Runtime secrets (Django `SECRET_KEY`, PostgreSQL password, Mollie API key, Gmail SMTP app password) live only in `.env` on the server and in the current operator's personal vault as backup.
- The current operator stores their copies of credentials in whatever personal password manager they prefer (1Password, Bitwarden personal, KeePass, browser, etc.) — no organizational vault is mandated.
- Board handover is a single action: pass the ISOW Gmail password (in person, sealed envelope, or any informal method) to the incoming board. Everything else can be reset from there.

**Why no shared vault:** at ISOW's scale (one operator at a time, ~10 credentials, semi-annual handover), a shared vault adds onboarding friction without buying meaningful security over the Gmail-anchor model. A vault becomes worthwhile only if multiple maintainers operate the system simultaneously — at which point one can be added without rework.

**Implication for the operations runbook:** the runbook must list every account registered against ISOW Gmail and explicitly state the Gmail-anchor handover procedure.

#### Decision 3: System Steward Role

One board position designated as the app's operational owner. No technical skill required — just ownership.

**Responsibilities:** Verify app works at semester start. Handle admin account creation/deactivation during board handover. Be first contact for issues. Review backup timestamps monthly.

**Recommendation:** Course Coordinator — they interact with the app most (course setup, teacher assignment) and have the strongest incentive to keep it running.

## Implementation Patterns & Consistency Rules

### Pattern Categories Defined

**Critical Conflict Points Identified:** 6 areas where AI agents could make different choices — naming, structure, HTMX partials, template inheritance, error handling, and permission enforcement.

### Naming Patterns

**Model & Field Naming (Django/Python conventions — non-negotiable):**

| Element | Convention | Example |
|---|---|---|
| Model class | Singular PascalCase | `Member`, `Course`, `CourseSession`, `Enrollment` |
| Field names | snake_case | `start_date`, `is_paid`, `membership_plan` |
| Foreign keys | Named after related model (Django adds `_id`) | `teacher` (not `teacher_id`), `course`, `member` |
| Related names | Plural snake_case | `enrollments`, `taught_courses`, `payments` |
| Manager methods | Descriptive verbs | `Member.objects.active()`, `Course.objects.for_quarter(q)` |
| Service methods | Verb-first snake_case | `create_member()`, `process_payment()`, `handle_webhook()` |
| Constants | UPPER_SNAKE_CASE | `MEMBERSHIP_12_MONTHS`, `MAX_PHOTO_SIZE_MB` |

**URL Patterns:**

| Pattern | Convention | Example |
|---|---|---|
| List views | Plural noun | `/courses/`, `/members/` |
| Detail views | Plural + ID | `/courses/<int:pk>/` |
| Actions | Plural + ID + verb | `/courses/<int:pk>/enroll/`, `/courses/<int:pk>/unenroll/` |
| Admin namespace | `/admin/` prefix | `/admin/members/`, `/admin/courses/create/` |
| Webhooks | `/payment/webhook/` | Single Mollie webhook endpoint |
| Auth | `/accounts/` prefix | `/accounts/login/`, `/accounts/register/`, `/accounts/password-reset/` |
| URL names | `app:action` namespace pattern | `courses:list`, `courses:detail`, `courses:enroll`, `accounts:register` |

**Template Naming:**

| Pattern | Convention | Example |
|---|---|---|
| Page templates | `<app>/<model>_<action>.html` | `courses/course_list.html`, `accounts/registration.html` |
| Partials (HTMX) | `<app>/partials/<component>.html` | `courses/partials/course_card.html`, `roster/partials/student_grid.html` |
| Includes (shared UI) | `includes/<component>.html` | `includes/membership_card.html`, `includes/photo_upload.html` |
| Email templates | `emails/<action>.html` + `emails/<action>.txt` | `emails/account_invitation.html`, `emails/expiry_warning.txt` |

### Structure Patterns

**View Pattern: Function-Based Views (FBVs)**

All views are function-based. Class-based views only for Django's built-in auth views (LoginView, PasswordResetView). FBVs are more explicit, easier to read for future contributors, and consistent with the prototype.

```python
@login_required
@role_required('teacher', 'admin', 'superadmin')
def course_roster(request, pk):
    course = get_object_or_404(Course, pk=pk)
    students = enrollment_service.get_enrolled_students(course)
    template = 'roster/partials/student_grid.html' if request.headers.get('HX-Request') else 'roster/course_roster.html'
    return render(request, template, {'course': course, 'students': students})
```

**Business Logic: Services Layer**

| Layer | Responsibility | Example |
|---|---|---|
| Views | HTTP handling — request parsing, template selection, response | `course_roster(request, pk)` returns rendered HTML |
| Services | Business logic — validation, state changes, external calls | `PaymentService.process_payment()`, `MemberService.create_member()` |
| Models | Data — fields, relationships, simple computed properties | `Member.is_active`, `Membership.is_expired`, `Course.available_spots` |
| Forms | Input validation — cleaning, CSRF, field rules | `RegistrationForm`, `CourseCreateForm` |

**Rules:**
- Views never contain business logic beyond request/response handling
- Services never import `HttpRequest` or `HttpResponse`
- Models never call external services (Mollie, email)
- Forms are the only way to accept user input — never access `request.POST` directly

**Form Handling: Django Forms exclusively**

All user input goes through Django Forms or ModelForms. No raw `request.POST` access in views. Forms handle validation, cleaning, and CSRF. Services receive cleaned data from forms, never raw request data.

### HTMX Patterns

**Standard HTMX partial pattern for all interactive endpoints:**

- View checks `request.headers.get('HX-Request')` to detect HTMX calls
- If HTMX request: render partial template (`partials/*.html`)
- If regular request: render full page template (which includes the partial via `{% include %}`)
- Target convention: `hx-target="#<component-id>"`, `hx-swap="outerHTML"`
- All HTMX endpoints return HTML — no JSON responses, no special middleware
- HTMX attributes on server-rendered HTML elements — no client-side JavaScript component layer

**HTMX attribute conventions:**

```html
<button hx-post="{% url 'courses:enroll' course.pk %}"
        hx-target="#course-card-{{ course.pk }}"
        hx-swap="outerHTML"
        class="btn btn-primary">
    Enroll
</button>
```

### Template Patterns

**Three-level template inheritance:**

1. `base.html` — HTML skeleton, `<head>`, Tailwind/HTMX includes, messages block, content block
2. `base_member.html` / `base_teacher.html` / `base_admin.html` — role-specific DaisyUI theme class on `<html>` element (coral/teal/amber), role-appropriate bottom nav (mobile) and top nav (desktop)
3. Page templates extend the appropriate role base

**Role-based theming:**
- Role base templates set `data-theme="isow-member"` / `data-theme="isow-teacher"` / `data-theme="isow-admin"` on `<html>`
- DaisyUI theme config maps each theme to the correct accent color
- All components use DaisyUI semantic classes (`btn-primary`, `badge-error`) — accent color changes automatically with theme

### Error & Feedback Patterns

| Scenario | Pattern | Implementation |
|---|---|---|
| Form validation errors | Inline per field | Django form error rendering, red border + message below field |
| Action success (enrollment, member created) | Django messages framework | `messages.success(request, '...')` rendered as DaisyUI alert, auto-dismiss 5s |
| Action warning (course nearly full) | Django messages framework | `messages.warning(request, '...')` rendered as amber DaisyUI alert |
| Payment errors | Dedicated error template | `payments/payment_failed.html` with recovery actions ("Try Again", "Different Bank") |
| Payment cancelled | Dedicated cancel template | `payments/payment_cancelled.html` with retry action |
| Server errors (500) | Custom error template | `500.html` — "Something went wrong. Contact isow@wur.nl" |
| Not found (404) | Custom error template | `404.html` — "Page not found" with link to home |
| Permission denied (403) | Custom error template | `403.html` — "You don't have access to this page" |
| HTMX request errors | Response error handler | `hx-on::response-error` triggers toast notification |

**Service exceptions:**

```python
# apps/core/exceptions.py
class PaymentFailedError(Exception): ...
class MembershipExpiredError(Exception): ...
class EnrollmentFullError(Exception): ...
class InvalidPhotoError(Exception): ...
```

Views catch specific service exceptions and render appropriate responses. Unhandled exceptions log to file and return generic 500.

### Permission Patterns

**`@role_required()` decorator on views — single enforcement point:**

```python
@login_required
@role_required('admin', 'superadmin')
def admin_panel(request):
    ...
```

**Rules:**
- Views enforce access via decorators — this is the single source of truth for permissions
- Templates control visibility only (`{% if user.is_teacher %}`) — never enforce access
- Services trust that the calling view has already checked permissions
- Every view that isn't public must have `@login_required` at minimum
- Role hierarchy: superadmin > admin > teacher > member (higher roles inherit lower role access)

### Enforcement Guidelines

**All AI agents MUST:**

1. Follow Django/Python naming conventions exactly — PEP 8, Django model conventions, no deviations
2. Put business logic in services, HTTP handling in views, data in models — no exceptions
3. Use Django Forms for all user input — never access `request.POST` directly
4. Use the `@role_required()` decorator for permission enforcement — no inline role checks in views
5. Use HTMX partial pattern (check `HX-Request` header, return partial or full template) — no JSON API endpoints
6. Extend the correct role-based base template — never extend `base.html` directly from page templates
7. Use Django messages framework for action feedback — no custom notification systems
8. Raise typed service exceptions — no generic `Exception` or string error passing
9. Use `from __future__ import annotations` in every Python file — required for strict mypy
10. Write tests in `apps/<app>/tests/test_<module>.py` using pytest fixtures from `conftest.py`

## Project Structure & Boundaries

### Django App Boundaries

| Django App | Models | FR Categories | Notes |
|---|---|---|---|
| `accounts` | `Member` (custom AbstractUser) | FR1, FR4-FR8, FR44-FR46, FR47-FR49 | User model, registration, auth, profile, roles, GDPR. Fields: `name`, `email` (unique, required), `photo` (nullable — uploaded on first login for migrated members), `role`, `phone` (optional), `nationality` (optional, free-text), `occupation` (optional, free-text), `notes` (optional, admin-editable text). New Django primary keys are assigned on import; old Excel IDs are not preserved. |
| `memberships` | `Membership`, `Payment` | FR2-FR3, FR7, FR9-FR12, FR36-FR39 | Plans, Mollie/iDEAL, renewal, digital card, PDF export |
| `courses` | `Course`, `CourseSession`, `Enrollment` | FR13-FR20 | Course CRUD, sessions, enrollment |
| `roster` | _(no models)_ | FR21-FR24 | Teacher roster views — thin app, reads from courses/accounts |
| `admin_panel` | `AuditLog` | FR25-FR35, FR50-FR51 | Custom admin dashboard, member/course management, superadmin settings |
| `backup` | _(no models)_ | FR40-FR43 | Excel export/import, automated backup, migration command |
| `core` | _(no models)_ | Cross-cutting | Exceptions, decorators, context processors, middleware |

### Complete Project Directory Structure

```
isow_registration_app/
├── .github/
│   └── workflows/
│       └── ci.yml                          # GitHub Actions: pytest, ruff, mypy
├── .env.example                            # Template for environment variables
├── .gitignore
├── .python-version                         # Python version pin for UV
├── pyproject.toml                          # UV deps, ruff config, mypy config, pytest config
├── manage.py
├── deploy.sh                               # Production deploy script
├── conftest.py                             # Root pytest conftest with shared fixtures
│
├── config/
│   ├── __init__.py
│   ├── urls.py                             # Root URL configuration
│   ├── wsgi.py                             # Gunicorn entry point
│   └── settings/
│       ├── __init__.py
│       ├── base.py                         # Shared settings (installed apps, middleware, templates, etc.)
│       ├── local.py                        # Development: DEBUG=True, SQLite, console email backend
│       └── production.py                   # Production: PostgreSQL, Gmail SMTP, WhiteNoise, security hardening
│
├── apps/
│   ├── __init__.py
│   │
│   ├── accounts/                           # FR1, FR4-FR8, FR44-FR46, FR47-FR49
│   │   ├── __init__.py
│   │   ├── models.py                       # Member (custom AbstractUser: name, email, photo, role, phone, nationality, occupation, notes)
│   │   ├── forms.py                        # RegistrationForm, ProfileForm, LoginForm
│   │   ├── views.py                        # register, login, logout, profile, password_reset, delete_account
│   │   ├── urls.py                         # /accounts/ namespace
│   │   ├── decorators.py                   # @role_required()
│   │   ├── context_processors.py           # User role context for template theming
│   │   ├── services/
│   │   │   ├── __init__.py
│   │   │   └── member_service.py           # create_member(), delete_member(), update_profile()
│   │   ├── templates/
│   │   │   └── accounts/
│   │   │       ├── registration.html       # Self-service registration form
│   │   │       ├── login.html
│   │   │       ├── profile.html
│   │   │       ├── password_reset.html
│   │   │       ├── password_reset_confirm.html
│   │   │       └── delete_account_confirm.html
│   │   └── tests/
│   │       ├── __init__.py
│   │       ├── test_models.py
│   │       ├── test_views.py
│   │       ├── test_forms.py
│   │       └── test_services.py
│   │
│   ├── memberships/                        # FR2-FR3, FR7, FR9-FR12, FR36-FR39
│   │   ├── __init__.py
│   │   ├── models.py                       # Membership (plan, start/end, status), Payment (amount, method, date), PendingRegistration (temp pre-payment data)
│   │   ├── forms.py                        # PlanSelectionForm, RenewalForm
│   │   ├── views.py                        # select_plan, payment_redirect, payment_return, webhook, membership_card, renew
│   │   ├── urls.py                         # /memberships/ and /payment/ namespaces
│   │   ├── services/
│   │   │   ├── __init__.py
│   │   │   ├── payment_service.py          # create_payment(), handle_webhook(), verify_payment()
│   │   │   └── membership_service.py       # activate_membership(), renew_membership(), check_expiry()
│   │   ├── templates/
│   │   │   ├── memberships/
│   │   │   │   ├── plan_selection.html
│   │   │   │   ├── membership_card.html    # "Show Membership" screen
│   │   │   │   ├── membership_card_pdf.html # PDF export template
│   │   │   │   └── renewal.html
│   │   │   └── payments/
│   │   │       ├── payment_return.html     # Success/failure after Mollie redirect
│   │   │       ├── payment_failed.html
│   │   │       └── payment_cancelled.html
│   │   ├── management/
│   │   │   └── commands/
│   │   │       └── send_expiry_notifications.py  # Daily cron: email members with expiring memberships
│   │   └── tests/
│   │       ├── __init__.py
│   │       ├── test_models.py
│   │       ├── test_views.py
│   │       ├── test_payment_service.py
│   │       └── test_membership_service.py
│   │
│   ├── courses/                            # FR13-FR20
│   │   ├── __init__.py
│   │   ├── models.py                       # Course (name, schedule, location, capacity, teacher, whatsapp_link), CourseSession, Enrollment
│   │   ├── forms.py                        # CourseCreateForm, CourseEditForm
│   │   ├── views.py                        # course_list, course_detail, enroll, unenroll
│   │   ├── urls.py                         # /courses/ namespace
│   │   ├── services/
│   │   │   ├── __init__.py
│   │   │   ├── course_service.py           # create_course(), generate_sessions()
│   │   │   └── enrollment_service.py       # enroll_member(), unenroll_member(), get_enrolled_students()
│   │   ├── templates/
│   │   │   └── courses/
│   │   │       ├── course_list.html
│   │   │       ├── course_detail.html
│   │   │       └── partials/
│   │   │           └── course_card.html    # HTMX partial: course with enroll/unenroll action
│   │   └── tests/
│   │       ├── __init__.py
│   │       ├── test_models.py
│   │       ├── test_views.py
│   │       └── test_services.py
│   │
│   ├── roster/                             # FR21-FR24 (thin app — no models)
│   │   ├── __init__.py
│   │   ├── views.py                        # course_roster (photo grid with membership status)
│   │   ├── urls.py                         # /roster/ namespace
│   │   ├── templates/
│   │   │   └── roster/
│   │   │       ├── course_roster.html      # Full roster page
│   │   │       ├── my_courses.html         # Teacher's course list (if multiple assigned)
│   │   │       └── partials/
│   │   │           └── student_grid.html   # HTMX partial: photo grid of enrolled students
│   │   └── tests/
│   │       ├── __init__.py
│   │       └── test_views.py
│   │
│   ├── admin_panel/                        # FR25-FR35, FR50-FR51 (custom admin, not Django built-in)
│   │   ├── __init__.py
│   │   ├── models.py                       # AuditLog (actor, action, target, timestamp)
│   │   ├── forms.py                        # AdminMemberCreateForm, AdminCourseForm, AdminSettingsForm
│   │   ├── views.py                        # dashboard, member_list, member_create, member_delete, course_manage, settings, export
│   │   ├── urls.py                         # /admin/ namespace
│   │   ├── services/
│   │   │   ├── __init__.py
│   │   │   ├── admin_service.py            # admin_create_member(), promote_role(), demote_role()
│   │   │   └── audit_service.py            # log_action()
│   │   ├── templates/
│   │   │   └── admin_panel/
│   │   │       ├── dashboard.html          # Stats, last backup, quick actions
│   │   │       ├── member_list.html        # Search, filter, member table
│   │   │       ├── member_create.html      # Cash/manual registration form
│   │   │       ├── course_manage.html      # Course CRUD for admins
│   │   │       ├── settings.html           # Superadmin: pricing, Mollie config, admin accounts
│   │   │       └── partials/
│   │   │           └── member_row.html     # HTMX partial: member table row
│   │   └── tests/
│   │       ├── __init__.py
│   │       ├── test_views.py
│   │       └── test_services.py
│   │
│   ├── backup/                             # FR40-FR43 (no models)
│   │   ├── __init__.py
│   │   ├── services/
│   │   │   ├── __init__.py
│   │   │   ├── export_service.py           # export_members_excel(), email_backup()
│   │   │   └── import_service.py           # import_members_excel(), validate_excel()
│   │   ├── management/
│   │   │   └── commands/
│   │   │       ├── export_backup.py        # Weekly cron: Excel export + email
│   │   │       └── import_members.py       # Loads standard backup .xlsx — used for DR restore AND step 2 of legacy migration
│   │   └── tests/
│   │       ├── __init__.py
│   │       ├── test_export_service.py
│   │       └── test_import_service.py
│   │
│   └── core/                               # Cross-cutting (no models)
│       ├── __init__.py
│       ├── exceptions.py                   # PaymentFailedError, MembershipExpiredError, EnrollmentFullError, InvalidPhotoError
│       ├── middleware.py                    # Rate limiting middleware (if needed beyond decorator)
│       └── templatetags/
│           ├── __init__.py
│           └── isow_tags.py                # Custom template tags if needed
│
├── templates/                              # Project-level templates
│   ├── base.html                           # HTML skeleton, head, Tailwind/HTMX, messages block
│   ├── base_member.html                    # Coral theme, member bottom nav
│   ├── base_teacher.html                   # Teal theme, teacher bottom nav (Roster first)
│   ├── base_admin.html                     # Amber theme, admin bottom nav (Admin first)
│   ├── base_public.html                    # No nav, for registration/login pages
│   ├── includes/
│   │   ├── membership_card.html            # Reusable membership card component
│   │   ├── photo_upload.html               # Photo upload circle component
│   │   ├── plan_selection.html             # Plan selection cards component
│   │   ├── messages.html                   # DaisyUI alert rendering for Django messages
│   │   ├── bottom_nav.html                 # Mobile bottom navigation
│   │   └── top_nav.html                    # Desktop top navigation
│   ├── emails/
│   │   ├── account_invitation.html
│   │   ├── account_invitation.txt
│   │   ├── password_reset.html
│   │   ├── password_reset.txt
│   │   ├── expiry_warning.html
│   │   ├── expiry_warning.txt
│   │   ├── expiry_notice.html
│   │   └── expiry_notice.txt
│   ├── pages/
│   │   └── privacy_policy.html             # Static privacy policy page (FR48)
│   ├── 400.html
│   ├── 403.html
│   ├── 404.html
│   └── 500.html
│
├── static/                                 # Project-level static files
│   ├── css/
│   │   └── input.css                       # Tailwind input file (DaisyUI theme config, custom styles)
│   ├── js/
│   │   ├── htmx.min.js                     # HTMX 2.0.8 (vendored, not CDN)
│   │   └── photo_compress.js               # Client-side photo compression (~30 lines)
│   ├── images/
│   │   ├── isow_logo.svg                   # ISOW logo
│   │   └── icons/                          # PWA icons (various sizes)
│   ├── manifest.json                       # PWA manifest
│   └── sw.js                               # Minimal service worker (install prompt only)
│
├── media/                                  # User-uploaded files (gitignored)
│   └── photos/                             # Member photos (managed by Django ImageField)
│
└── scripts/                                # One-off scripts, not part of the running app
    └── convert_legacy_xlsx.py              # Throwaway: legacy Google Sheets .xlsx → standard backup .xlsx (active-only filter, payment history filter, column renames). Delete after migration go-live.
```

### Architectural Boundaries

**App Communication Rules:**

| From | Can Import From | Cannot Import From |
|---|---|---|
| `accounts` | `core` | memberships, courses, roster, admin_panel, backup |
| `memberships` | `core`, `accounts` (Member model) | courses, roster, admin_panel, backup |
| `courses` | `core`, `accounts` (Member model) | memberships, roster, admin_panel, backup |
| `roster` | `core`, `accounts`, `courses` | memberships, admin_panel, backup |
| `admin_panel` | `core`, `accounts`, `memberships`, `courses`, `backup` | roster |
| `backup` | `core`, `accounts`, `memberships`, `courses` | roster, admin_panel |
| `core` | _(nothing — no app imports)_ | all apps |

**Key principle:** Dependencies flow downward. `core` depends on nothing. `accounts` is foundational. Higher-level apps (`roster`, `admin_panel`) can import from lower-level apps but not vice versa.

### Requirements to Structure Mapping

| FR Category | Django App | Key Files |
|---|---|---|
| Member Registration (FR1-FR9) | `accounts` + `memberships` | `accounts/views.py:register`, `memberships/views.py:select_plan` |
| Digital Membership Card (FR10-FR12) | `memberships` | `memberships/views.py:membership_card`, `memberships/templates/memberships/membership_card.html` |
| Course Management (FR13-FR20) | `courses` | `courses/models.py`, `courses/views.py`, `courses/services/` |
| Teacher Roster (FR21-FR24) | `roster` | `roster/views.py:course_roster`, `roster/templates/roster/partials/student_grid.html` |
| Admin Panel (FR25-FR31) | `admin_panel` | `admin_panel/views.py`, `admin_panel/templates/admin_panel/` |
| Superadmin (FR32-FR35) | `admin_panel` | `admin_panel/views.py:settings`, `admin_panel/templates/admin_panel/settings.html` |
| Payment Processing (FR36-FR39) | `memberships` | `memberships/services/payment_service.py`, `memberships/views.py:webhook` |
| Data & Backup (FR40-FR43) | `backup` | `backup/services/`, `backup/management/commands/` |
| Auth & Authorization (FR44-FR46) | `accounts` | `accounts/decorators.py`, `accounts/models.py:Member` |
| GDPR & Privacy (FR47-FR49) | `accounts` | `accounts/views.py:delete_account`, `accounts/services/member_service.py:delete_member()` |
| Monitoring (FR50-FR51) | `config/urls.py` | Health check endpoint in root URL config (simple view, no app needed) |

### External Integration Points

| Integration | App | Entry Point | Direction |
|---|---|---|---|
| Mollie (iDEAL) | `memberships` | `payment_service.py` → Mollie API; `/payment/webhook/` ← Mollie POST | Bidirectional |
| Gmail SMTP | `config/settings/production.py` | Django email backend sends via Gmail | Outbound |
| Cloudflare | Infrastructure | DNS proxy, no app-level integration | External |
| UptimeRobot | `config/urls.py` | `/health/` endpoint polled by UptimeRobot | Inbound |

### Data Flow

**Registration flow:** `accounts/views.py:register` → `accounts/forms.py:RegistrationForm` → validated data stored as `PendingRegistration` in `memberships/models.py` → `memberships/views.py:select_plan` → `memberships/services/payment_service.py:create_payment()` (Mollie payment created with PendingRegistration ID) → Mollie redirect → Mollie webhook → `memberships/services/payment_service.py:handle_webhook()` → retrieves `PendingRegistration` → `accounts/services/member_service.py:create_member()` + `memberships/services/membership_service.py:activate_membership()` in one atomic transaction → deletes `PendingRegistration`. No `Member` record exists until payment is confirmed.

**Roster flow:** `roster/views.py:course_roster` → `courses/services/enrollment_service.py:get_enrolled_students()` → render `roster/partials/student_grid.html` with member photos and status

**Backup flow:** Cron → `backup/management/commands/export_backup.py` → `backup/services/export_service.py:export_members_excel()` → `backup/services/export_service.py:email_backup()` → Gmail SMTP

**Legacy migration flow (one-time):** Google Sheets export → `legacy.xlsx` → `scripts/convert_legacy_xlsx.py` (active-only filter, payment history filter, column renames) → `standard.xlsx` (human-reviewed in Excel) → `manage.py import_members standard.xlsx` → `backup/services/import_service.py:import_members_excel()` → DB

**Disaster recovery flow:** Latest weekly Gmail backup `.xlsx` → `manage.py import_members backup.xlsx` → `backup/services/import_service.py:import_members_excel()` → DB. Same code path as step 2 of legacy migration.

## Architecture Validation Results

### Coherence Validation

**Decision Compatibility:** All technology choices verified compatible. Django 5.2 LTS + Tailwind 4.2 + DaisyUI 5.5 + HTMX 2.0.8 + django-tailwind-cli 4.4 + PostgreSQL + Gunicorn + WhiteNoise + Mollie SDK + openpyxl + argon2-cffi + django-ratelimit + weasyprint. No version conflicts.

**Pattern Consistency:** All implementation patterns align with stack choices — FBVs with decorators, services layer, Django Forms, HTMX partials, three-level template inheritance, Django messages, typed exceptions. No contradictions.

**Structure Alignment:** Project structure supports all decisions. App boundaries respect dependency rules. Integration points properly placed. Cross-cutting concerns in `core`.

### Requirements Coverage

**Functional Requirements: 51/51 covered**

All FRs mapped to specific apps, files, and services. Three items clarified during validation:

1. **FR12 (PDF export):** weasyprint renders `membership_card_pdf.html` to PDF. Added to `memberships` app dependencies.
2. **FR48 (privacy policy page):** Added `templates/pages/privacy_policy.html` — static template linked from registration form and profile settings.
3. **Registration flow (FR1-FR3, FR36-FR37):** `PendingRegistration` model in `memberships` holds form data until Mollie webhook confirms payment, then `Member` is created in an atomic transaction. Preserves the "paid = exists" invariant. Abandoned registrations cleaned up by daily cron.

**Non-Functional Requirements: All covered**

| NFR | Architectural Support |
|---|---|
| Page load <2s | Server-rendered MPA, WhiteNoise static serving, no SPA overhead |
| Roster <3s with photos | HTMX partial, photo grid, client-side compressed images |
| 50 concurrent users | Gunicorn workers, PostgreSQL direct queries, no caching needed |
| No cold starts | Always-running server (not free-tier hosting) |
| 99% uptime | Hetzner Cloud CX23 (professional EU data center, NVMe SSD, DDoS protection) + UptimeRobot monitoring |
| Weekly backup | Cron + `export_backup` management command + Gmail SMTP |
| Health check | `/health/` endpoint in `config/urls.py` |
| GDPR | Consent at registration, cascading deletion, EU hosting, privacy policy |
| Security | Django defaults, Argon2, rate limiting, Cloudflare, HTTPS, audit logging |

### Gap Analysis

**Critical gaps:** None

**Resolved during validation:**

| Gap | Resolution |
|---|---|
| PDF library unspecified | weasyprint added to stack |
| Registration-to-payment handoff | `PendingRegistration` model in `memberships` — no Member until payment confirmed |
| Privacy policy page missing from structure | Added to `templates/pages/` |

**Additions to project structure:**

- `memberships/models.py` now includes: `Membership`, `Payment`, `PendingRegistration`
- `templates/pages/privacy_policy.html` added
- weasyprint added to project dependencies

**Deferred (not gaps — board-dependent):**

- System steward role assignment

**Resolved since initial drafting:**

- Hosting provider selection — Hetzner Cloud CX23 (decided 2026-05-05)
- Media file serving strategy — Django serves directly (Hetzner CX23 SSD has ample headroom)
- Migration scope and data rules — active members only (~257), email-required, photo-on-first-login, filtered payment history (see Data Architecture)
- Credential custody — Gmail-anchored model, no shared password manager (decided 2026-05-05); see Decision 2 in Board Decision Brief

### Architecture Completeness Checklist

**Requirements Analysis**
- [x] Project context thoroughly analyzed
- [x] Scale and complexity assessed (medium, ~1000 members/year)
- [x] Technical constraints identified (solo developer, zero-handover, EU hosting, budget)
- [x] Cross-cutting concerns mapped (8 concerns)

**Architectural Decisions**
- [x] Critical decisions documented with verified versions
- [x] Technology stack fully specified (Django 5.2 LTS, Tailwind 4.2, DaisyUI 5.5, HTMX 2.0.8)
- [x] Integration patterns defined (Mollie, Gmail SMTP, Cloudflare, UptimeRobot)
- [x] Performance considerations addressed (WhiteNoise, no caching, Gunicorn)
- [x] Security decisions documented (Argon2, rate limiting, audit logging, session hardening)

**Implementation Patterns**
- [x] Naming conventions established (models, fields, URLs, templates)
- [x] Structure patterns defined (FBVs, services layer, Django Forms)
- [x] HTMX patterns specified (partial detection, target conventions)
- [x] Template patterns documented (three-level inheritance, role theming)
- [x] Error handling patterns complete (service exceptions, messages, error templates)
- [x] Permission patterns defined (@role_required decorator)

**Project Structure**
- [x] Complete directory structure defined (7 Django apps, project-level templates, static, media)
- [x] Component boundaries established (app import rules)
- [x] Integration points mapped (Mollie, Gmail, Cloudflare, UptimeRobot)
- [x] Requirements to structure mapping complete (51 FRs → specific files)
- [x] Data flows documented (registration, roster, backup)

### Architecture Readiness Assessment

**Overall Status:** READY FOR IMPLEMENTATION

**Confidence Level:** High

**Key Strengths:**
- Clean "paid = exists" invariant simplifies the entire data model via `PendingRegistration`
- Services layer separates business logic from HTTP — testable, maintainable
- Proven patterns carried from reference stack (45+ commits of battle-tested code)
- App boundaries with clear dependency rules prevent circular imports
- Every FR traceable to a specific file and service method
- Role-based theming via DaisyUI keeps UI consistent without custom CSS per role

**Areas for Future Enhancement:**
- Operations runbook (documentation deliverable, not code architecture)
- Board-friendly infrastructure decision brief (separate document) — partially complete; system steward decision still open

### Implementation Handoff

**AI Agent Guidelines:**
- Follow all architectural decisions exactly as documented
- Use implementation patterns consistently across all components
- Respect app boundaries and import rules
- Use `PendingRegistration` for the registration-to-payment flow — never create a `Member` before payment confirmation
- Refer to this document for all architectural questions

**First Implementation Priority:**
```bash
uv init isow_registration_app
cd isow_registration_app
uv add django
uv run django-admin startproject config .
```
Then apply split settings, install apps, configure django-tailwind-cli with DaisyUI, and set up pytest/ruff/mypy in `pyproject.toml`.
