---
stepsCompleted:
  - 'step-01-init'
  - 'step-02-discovery'
  - 'step-02b-vision'
  - 'step-02c-executive-summary'
  - 'step-03-success'
  - 'step-04-journeys'
  - 'step-05-domain'
  - 'step-06-innovation'
  - 'step-07-project-type'
  - 'step-08-scoping'
  - 'step-09-functional'
  - 'step-10-nonfunctional'
  - 'step-11-polish'
  - 'step-12-complete'
inputDocuments:
  - 'product-brief-isow_registration_app.md'
  - 'product-brief-isow_registration_app-distillate.md'
documentCounts:
  briefs: 2
  research: 0
  brainstorming: 0
  projectDocs: 0
workflowType: 'prd'
classification:
  projectType: web_app
  domain: edtech
  complexity: medium
  projectContext: greenfield
---

# Product Requirements Document - isow_registration_app

**Author:** Cedric
**Date:** 2026-04-06

## Executive Summary

ISOW (International Student Organisation Wageningen) serves ~1,000 international students annually with free language and dance courses, community events, and partner discounts across Wageningen. Today, two core workflows are broken:

**Registration is a manual burden.** Prospective members fill in a Google Form, make a bank transfer to an IBAN, visit the Global Lounge during limited office hours, and pick up a physical membership card. Each step loses members and creates admin work for a volunteer board that already commits 15 hours per week and rotates every six months. Member data ends up scattered across spreadsheets, Google Forms, and bank statements.

**Teacher verification is impossible.** Volunteer teachers have no reliable way to confirm whether someone in their class is a paid, registered member. There is no single source of truth — member data lives across disconnected tools with no unified view. The result: teachers either don't check (lost revenue, unregistered attendees) or attempt manual cross-referencing that wastes time and fails.

Both problems share a root cause: **there is no unified, reliable member database.** The app solves this by providing a single system where registration and payment happen in one flow, and teacher rosters draw from the same verified data.

The solution is a mobile-first Progressive Web App (no install required) built with Django, Tailwind CSS/DaisyUI, and PostgreSQL. New members register, upload a photo, and pay via iDEAL in minutes. Teachers open a roster with photos before class. The board gets a clean admin panel instead of spreadsheets. Cash registration remains available as a fallback — an admin creates the member only after payment is received. The strict rule: **a member exists in the system only if they have paid.**

## What Makes This Special

No existing product combines iDEAL payment, digital photo membership ID, course enrollment, and teacher roster views in a single lightweight web app. Dutch association tools (Membro, Banster) lack course management. International SaaS platforms lack iDEAL and cost 5-10x more. A custom build costs ~EUR 330-350/year in total operating costs (hosting + payment processing) and delivers exactly the features ISOW needs.

Beyond the feature gap, the app is designed to **outlive its developer.** Every decision is tested against one question: "Does this survive a complete board replacement?" Zero-handover operations, automated backups, and organizational account ownership ensure the next board inherits a working system, not a folder of undocumented processes.

## Project Classification

- **Project Type:** Web application (PWA, mobile-first, Django)
- **Domain:** Edtech — student organization membership and course management
- **Complexity:** Medium — GDPR compliance, payment processing (iDEAL/Mollie), photo data handling, security considerations for a volunteer-maintained system
- **Project Context:** Greenfield — new build informed by a working prototype

## Success Criteria

### User Success

- **New member:** Completes registration, payment, and photo upload from their phone in under 5 minutes — no office visit, no bank transfer follow-up, no waiting for a physical card
- **Teacher:** Opens roster before class, sees enrolled students with photos, and trusts the data without needing to cross-reference anything else. The roster is the single source of truth
- **Board admin:** No longer needs to be physically present at the Global Lounge for registrations. No manual payment reconciliation. No Google Form management. A new board member understands the admin panel within their first session — no training document required
- **Partner discount usage:** Member shows digital membership card at a local business counter in under 5 seconds — open app, show screen, done

### Business Success

- **Adoption:** At least 70% of new registrations happen through the app within the first full semester of deployment
- **Member conversion:** Measurable reduction in lost members due to registration friction — fewer people who start the process but never complete it
- **Board efficiency:** Board members report significantly less time spent on membership administration, payment tracking, and roster management compared to the previous semester
- **Board adoption:** Sitting board formally adopts the app as the primary registration method; Google Form process retired or demoted to emergency fallback
- **Handover quality:** New board takes over with minimal friction — the app is self-explanatory, no dedicated training or handover documentation for day-to-day operations

### Technical Success

- **Reliability:** App available with minimal downtime. Brief outages (a few hours) are acceptable but should be rare. Uptime monitoring alerts the board if the app goes down
- **Longevity without developer:** App runs without developer intervention for 1-2+ years. Pinned dependencies, automated backups, and documented operations make this realistic
- **Regression safety:** Comprehensive test suite (unit + integration + UI tests) with CI on GitHub Actions. Protected main branch. No code reaches production without passing tests — this is the primary defense against well-intentioned volunteers introducing regressions
- **Future maintainability:** Clean, well-documented codebase that WUR software engineering students or a future ISOW digital committee could contribute to. Contribution guardrails (CI, protected branches, test requirements) prevent regressions even with inexperienced contributors
- **Security:** Django security defaults enforced, HTTPS everywhere, rate limiting on auth endpoints, Cloudflare free tier. Annual security patch review acknowledged as a board responsibility (or delegated to a volunteer developer)

### Measurable Outcomes

| Metric | Target | Measured by |
|---|---|---|
| Registration completion rate | >80% of started registrations completed | App analytics (started vs. paid) |
| Registration time | <5 minutes from start to paid member | User testing |
| Board admin time on membership | >50% reduction vs. previous semester | Board self-report |
| Orientation week capacity | Handles peak registration (100+ in a week) without degradation | Monitoring during orientation |
| Board handover time | New board operational within first meeting | Board self-report |
| Unplanned downtime | <24 hours total per semester | Uptime monitoring |
| Test coverage | Critical paths (registration, payment, roster) at 100% | CI coverage reports |

## User Journeys

### Journey 1: New Member Registration (Primary — Success Path)

**Maria**, a 22-year-old Colombian exchange student, arrives in Wageningen for her MSc in Plant Sciences. During orientation week, she hears about ISOW's free salsa classes from a flyer at Forum. She scans the QR code on her phone.

The app loads instantly — no install needed. She taps "Register," enters her name, email, uploads a selfie, and picks a 12-month membership (EUR 20). She selects her bank from the iDEAL screen, confirms payment in her banking app, and is redirected back. Done. She's a member with a digital card showing her photo. Total time: about 3 minutes.

She browses courses, finds Salsa Beginners on Wednesdays, and enrolls. She gets a confirmation screen with the time, location (Global Lounge), and a link to the WhatsApp group. On Wednesday, she shows up, the teacher sees her photo on the roster. She's in.

**Reveals:** Registration flow, photo upload, iDEAL payment integration, course browsing, enrollment, digital membership card, WhatsApp link display.

### Journey 2: New Member — Manual/Cash Registration (Edge Case)

**Wei**, a Chinese PhD student, just arrived and doesn't have a Dutch bank account yet. He visits the Global Lounge during office hours. A board member takes his EUR 20 cash, then opens the admin panel and creates Wei's account — name, email, photo (taken on the spot with a phone). The system marks him as paid.

Wei receives an email with a link to set his password. He logs in, sees his membership card, and can browse and enroll in courses. From his perspective, the experience is the same as Maria's after registration.

**Reveals:** Admin manual member creation, cash payment recording, email invitation flow, admin panel member management.

### Journey 3: Teacher Roster Check (Secondary — Core Value)

**Priya**, a second-year MSc student, volunteers as a Japanese teacher. It's Tuesday at 17:55, five minutes before class at Forum. She opens the app on her phone and taps her course — Japanese Beginners, Tuesday Q2.

The roster shows 14 enrolled students with photos and names. She recognizes most faces. One student, **Tom**, has an orange "expired" badge next to his name — his 6-month membership ran out last week. Priya tells Tom before class: "Hey, your membership expired — you can renew on your phone right now." Tom pulls out his phone, taps renew, pays via iDEAL in 30 seconds. The badge disappears from Priya's roster.

A new face walks in — someone not on the roster. Priya doesn't challenge them — ISOW allows one free trial lesson for non-members. She makes a mental note. The following week, the same person shows up again. This time Priya says: "Hey, glad you're back! You'll need to register to keep coming — you can sign up on your phone right now." The student registers during the first few minutes and appears on the roster.

**Reveals:** Teacher roster view with photos, expired member flagging, real-time roster updates, free trial policy (first lesson free, no registration required), in-class registration flow, renewal flow.

### Journey 4: Board Admin — Day-to-Day Operations (Secondary)

**Sander**, the new Course Coordinator, just joined the board. The previous Course Coordinator showed him the app during handover — a 10-minute walkthrough. Sander logs in with his admin account.

It's the start of Q3. He needs to set up courses for the new quarter. He opens the admin panel, creates "Dutch Beginners — Q3" with the schedule (Mondays 18:00, Forum Room 123), assigns **Lisa** as teacher (she's already in the system as a teacher-role member), and sets max capacity to 20. He repeats for the other courses. Sessions are auto-generated based on the quarterly schedule.

A week later, a student emails asking to be removed from the system (GDPR request). Sander finds them in the member list, clicks delete, confirms. Gone.

Monthly, Sander glances at the admin panel — member count, recent registrations, last backup timestamp. Everything looks normal. He doesn't think about the app again until next quarter's course setup.

**Reveals:** Course creation with quarterly scheduling, teacher assignment, session auto-generation, capacity setting, GDPR deletion, admin dashboard overview, backup visibility.

### Journey 5: Board Handover (Edge Case — Critical for Continuity)

It's September. The outgoing board's term overlaps with the incoming board for a transition period. **Sander** (outgoing Course Coordinator) and **Amara** (incoming) sit down together.

Sander logs into the superadmin account (ISOW org credentials from the shared password manager) and creates an admin account for Amara. He walks her through the admin panel — member list, course setup, manual registration, export. It takes 15 minutes. He shows her where the operations runbook lives.

Amara logs in with her new admin account. She can see everything Sander sees except system settings. Over the next two weeks, she handles a few registrations and sets up one course with Sander available on WhatsApp if she has questions. After the overlap period, Sander's admin account is deactivated by the superadmin account.

The shared password manager credentials (including the ISOW org/superadmin account, Mollie, hosting) transfer with the board as part of the standard board handover process.

**Reveals:** Superadmin role (ISOW org account), admin account creation/deactivation, handover process, overlap period, shared credential management.

### Journey 6: Expired Member Renewal

**Lucas**, a Dutch student, bought a 6-month membership in September. In March, his membership expires. He gets an email notification that his membership has expired.

Next time he logs in, he sees a banner: "Your membership expired on March 15. Renew to keep attending courses." He can still see his profile and the course he was enrolled in, but the "Show Membership" screen clearly shows "Expired." If he shows up to class, the teacher sees him flagged.

He taps "Renew," picks 12-month this time, pays via iDEAL. His membership reactivates instantly — same account, same photo, same course enrollments. No re-registration needed.

Lucas can also delete his account entirely from his profile settings if he chooses — "Delete my account" with a confirmation step permanently removes his data, photo, and enrollment records (GDPR self-service).

**Reveals:** Expiry notification, expired state UX, renewal flow, account continuity, roster expired flagging, GDPR self-service account deletion.

### Journey 7: Teacher Onboarding

**Ahmed**, a PhD student fluent in Arabic, wants to volunteer as an Arabic teacher for ISOW. He contacts the board and meets them at the Global Lounge. They discuss the course schedule, expectations, and logistics.

After the meeting, a board admin opens the admin panel and creates Ahmed's account — name, email, photo. Since teachers volunteer for free, no payment is required. The admin marks the account as exempt from payment and promotes Ahmed to the teacher role.

Ahmed receives an email to set his password. He logs in and sees his profile with the teacher badge. He can now see the roster for his assigned course (Arabic Beginners, Thursdays Q2). Before his first class, he opens the roster — his enrolled students are listed with photos and names.

**Reveals:** Manual member creation without payment (teacher exemption), teacher role promotion, teacher-specific admin flow, face-to-face onboarding requirement.

### Journey Requirements Summary

| Journey | Key Capabilities Revealed |
|---|---|
| New member registration | Registration form, photo upload, iDEAL/Mollie payment, course browsing, enrollment, digital card |
| Manual/cash registration | Admin member creation, cash payment recording, email invitation |
| Teacher roster | Roster with photos, expired flagging, free trial policy, real-time updates |
| Board admin operations | Course CRUD, teacher assignment, session generation, GDPR deletion, admin dashboard |
| Board handover | Superadmin account (ISOW org), admin management, credential transfer |
| Expired member renewal | Expiry notifications, expired state restrictions, renewal payment, account reactivation, GDPR self-service deletion |
| Teacher onboarding | Admin member creation without payment (teacher exemption), role promotion, in-person meeting |

### Role Model

- **Member (student):** Register, pay, enroll in courses, view membership card, renew, delete own account (GDPR)
- **Teacher:** Everything a member can do + view roster for assigned courses. Exempt from payment — created manually by board
- **Admin (board):** Everything a teacher can do + create/remove members, manage courses, assign teacher roles, export data, view admin dashboard
- **Superadmin (ISOW org account):** Everything admin can do + manage admin accounts, configure system settings (pricing, Mollie config, backup settings)

## Domain-Specific Requirements

### Compliance & Regulatory (GDPR)

ISOW already stores member data without formal GDPR processes. The app improves the status quo by consolidating data, enabling self-service deletion, and hosting on EU servers. Approach: implement practical basics, don't overcomplicate.

- **Consent:** Checkbox at registration ("I agree to ISOW storing my name, email, and photo for membership purposes"). No separate cookie consent needed — no tracking cookies or analytics
- **Right to erasure:** Self-service account deletion in profile settings. Admin can also delete members
- **Right to access:** Members can view all their stored data in their profile. Excel export available to admins
- **Data hosting:** EU-based server (all hosting options under consideration are EU or can be configured as EU)
- **Data controller:** ISOW. No formal data processing agreement — developer hosts on behalf of ISOW using organizational accounts. If ISOW later wants to formalize this, the operations runbook documents the arrangement
- **Photo retention:** Photos stored until the member deletes their account. No automatic deletion on membership expiry. Operations runbook notes that an admin can periodically purge accounts inactive for 3+ years manually

### Security

- **Django built-in protections:** CSRF, XSS, SQL injection, clickjacking — all enabled by default, not to be disabled
- **HTTPS:** Enforced everywhere with TLS 1.2+. All hosting options provide free SSL/TLS certificates
- **Authentication hardening:** Rate limiting on login and registration endpoints (e.g., 5 attempts per minute per IP). Strong password requirements for admin/superadmin accounts. Session cookies marked HttpOnly, Secure, SameSite
- **Cloudflare free tier:** DNS proxy providing basic DDoS protection, CDN, SSL termination, and IP masking at zero cost
- **Admin protection:** Superadmin (ISOW org account) and admin accounts use strong passwords. 2FA as a nice-to-have for admin accounts. Admin and superadmin actions logged (member creation, deletion, role changes)
- **Data protection:** Passwords hashed using Django's default PBKDF2 (or Argon2). No sensitive data (payment details) stored in application database — Mollie handles all payment data. Database credentials and API keys stored as environment variables, never in code
- **Photo upload validation:** File type (JPEG, PNG only), file size limit (5MB), basic dimension checks. No server-side image processing beyond resizing/compression
- **Security maintenance trade-off:** Django publishes security releases. Pinned dependencies provide stability but become vulnerable over time. **Board decision required:** either budget for occasional developer time (a few hours per year) for security updates, or accept increasing risk. A future ISOW digital committee or WUR CS student volunteer could fill this role. Document the update procedure in the operations runbook

### Content Moderation

- **Photo policy:** Registration states "Upload a photo of your face — this is your membership ID." Trust-based, no automated review
- **Reporting flow:** If a teacher notices an inappropriate photo, they report to the board. Board admin deletes or contacts the member
- **No automated moderation:** Appropriate for ISOW's scale and community trust model

### Data Protection & Backup

- **Weekly Excel backup:** Automated export of member data (names, emails, membership status, enrollments) emailed to the ISOW association email. This is the disaster recovery baseline
- **Backup restore:** Admin can upload an Excel backup to restore member data in case of total data loss. Same standardized Excel format used for initial Google Sheet migration and ongoing backups
- **Photos are NOT backed up.** If the server is lost, members re-upload photos. This is an accepted trade-off — the cost and complexity of photo backup infrastructure contradicts the zero-maintenance goal
- **Hosting-level redundancy:** The chosen hosting provider's infrastructure (RAID storage, persistent volumes) provides implicit protection against hardware failure, but is not a substitute for off-server backups
- **Risk communicated to board:** In a total server loss scenario, member data is recoverable from the latest Excel backup (up to 1 week of data loss). Photos and enrollment history beyond what's in the Excel export would need to be recreated. Board must accept this risk profile

## Web Application Requirements

### Architecture

Server-rendered Multi-Page Application (MPA) using Django templates with HTMX for interactive partials where needed. No SPA framework — simpler to build, simpler to maintain, less JavaScript surface area. PWA capability for "Add to Home Screen" experience on mobile.

### Browser & Device Support

- **Target:** Modern evergreen browsers — last 2 versions of Chrome, Safari, Firefox, Edge
- **Primary device:** Mobile phones (iOS Safari, Android Chrome) — mobile-first design
- **Secondary device:** Desktop browsers for board admin operations
- **Not supported:** Legacy browsers (IE, Safari < 15, Chrome < 90). No polyfills or fallbacks for older devices
- **Rationale:** Target audience is university students in the Netherlands; 97%+ use modern phones and browsers

### Responsive Design

- **Mobile-first:** All core flows (registration, payment, show membership, course enrollment) designed for phone screens first
- **Tablet:** Should work but not specifically optimized
- **Desktop:** Admin panel and roster views should work well on larger screens. No desktop-specific features
- **Framework:** Tailwind CSS with DaisyUI components — responsive by default

### SEO Strategy

- **Not needed.** The app is accessed via direct link from the ISOW static website and QR codes on flyers/posters
- **robots.txt:** Disallow all. No sitemap needed
- **Public pages:** Registration/login page only. Everything else behind authentication

### Accessibility

- Standard web accessibility practices: semantic HTML, proper heading hierarchy, keyboard navigation, sufficient color contrast
- Form inputs with proper labels and error messages
- Photo alt text (member name)
- No WCAG audit required — follow sensible defaults from DaisyUI components

### Implementation Stack

- **HTMX** for partial page updates (roster refresh, enrollment actions) without full page reload
- **Django template inheritance** for consistent layout across pages
- **DaisyUI theme** for consistent component styling with minimal custom CSS
- **PWA manifest** for "Add to Home Screen" with app icon and splash screen
- **Service worker** — minimal, for PWA install prompt only. No offline caching (the app requires network for meaningful use)

## Project Scoping & Phased Development

### MVP Strategy

**Approach:** Problem-solving MVP — deliver the minimum that replaces ISOW's broken registration and roster workflows. Success = board adopts the app and retires the Google Form process.

**Resource:** Solo developer (Cedric), ML engineer background, with existing Django prototype. No ongoing development team after deployment.

**Rollout strategy:** App is the primary registration method from day one (after thorough testing). Google Form stays available as emergency fallback only — accessible but not promoted. If the app has critical issues during the first registration cycle, the board redirects to the Google Form while issues are resolved. Once the app proves stable through one full semester (including orientation week), the Google Form is retired entirely.

### MVP Feature Set (Phase 1)

Core registration and course management that solves both broken workflows:

- Member registration with photo upload (mobile-first)
- Membership plans (12-month / 6-month) with iDEAL payment via Mollie
- Cash/manual registration fallback (admin creates member only after payment received)
- Teacher account creation without payment (volunteer exemption)
- Digital membership profile with photo ID and "Show Membership" screen
- PDF export of membership card for offline use
- Membership renewal for expired members
- Course creation and management (quarterly schedule, sessions, capacity)
- Course enrollment/unenrollment
- Teacher roster view with photos and expired member flagging
- Admin panel (member management, role assignment, payment overview, dashboard)
- Superadmin system settings (admin account management, pricing, Mollie config)
- Excel export/backup of member data with automated weekly email
- Excel import for data migration (Google Sheets converted to Excel) and backup restore
- Comprehensive test suite with CI (GitHub Actions, protected main branch)
- GDPR essentials: consent at registration, self-service account deletion, privacy policy, EU-hosted data
- Basic uptime monitoring (UptimeRobot free tier, alerts to board email)
- Operations runbook: admin onboarding, cash registration, quarterly course setup, backup/restore procedure

**Core journeys supported:**
1. New member self-registration with iDEAL payment
2. Manual/cash member registration by admin
3. Teacher roster verification
4. Board admin day-to-day operations (course setup, member management)
5. Board handover
6. Member renewal
7. Teacher onboarding

### Growth Features (Phase 2 — Post-MVP)

- Apple Wallet / Google Wallet membership card integration (board interested — needs scoping to clarify effort, cost, and what it provides beyond the "Show Membership" screen)
- Course attendance tracking per session — teachers mark who showed up, board sees actual participation data (to be confirmed with board)
- Basic analytics dashboard: registration trends, course popularity, renewal rates (to be confirmed with board)

### Vision (Phase 3 — Future)

- Possible integration with the static ISOW website — only if stability is proven and risks are carefully weighed. Not a priority
- Primary long-term goal remains **stability and zero-maintenance operation** above all new features

### Board Decision Points

Decisions that require board input before or during development:

| Decision | Options | Impact | When needed |
|---|---|---|---|
| **Hosting provider** | Hetzner (~EUR 3.29/mo), Google Cloud free tier, Railway ($5/mo), WUR hosting (free?), PythonAnywhere (free), Oracle Cloud (free but risky), Raspberry Pi (~EUR 50 once) | Cost, reliability, maintenance burden | Before deployment |
| **Security maintenance** | Budget for occasional developer time vs. accept increasing risk vs. delegate to digital committee/CS student volunteer | Long-term app security | Before deployment |
| **Mollie account ownership** | Which board role signs up, which IBAN is linked, who holds API keys | Payment processing setup | Before development |
| **System steward role** | Which board position owns app operations (recommended: Course Coordinator) | Operational responsibility | Before deployment |
| **Shared password manager** | Adopt Bitwarden free org (or confirm existing solution) | Credential management across board turnovers | Before deployment |
| **Go-live confidence** | Define testing criteria that must pass before making the app the primary registration method | Determines when Google Form fallback can be retired | Before go-live |
| **Course attendance tracking** | Add post-MVP or not — teachers mark who showed up per session | Extra teacher workload vs. data value | Post-MVP |
| **Analytics dashboard** | Add post-MVP or not — registration trends, course popularity | Development effort vs. insight value | Post-MVP |

### Risk Mitigation Strategy

**Technical Risks:**
- **Payment integration complexity:** Mitigated by Mollie's well-documented Django SDK and existing prototype with mock payment flow. Mollie handles iDEAL-to-Wero transition transparently
- **Photo storage/performance:** Mitigated by client-side compression, file size limits, and reasonable resolution targets. Data volume is trivially small (~500MB/year)
- **Django security over time:** Mitigated by pinned dependencies for stability, documented update procedure in operations runbook, and CI guardrails for future contributors

**Operational Risks:**
- **Bus factor (single developer):** The #1 risk. Mitigated by comprehensive test suite, operations runbook, clean codebase, CI/CD guardrails, and designing for zero-handover. WUR CS students or ISOW digital committee as potential future maintainers
- **Board turnover breaks adoption:** Mitigated by superadmin on ISOW org account, overlap handover period, and self-explanatory admin panel. Teacher adoption flywheel: if teachers depend on the roster, they push the board to keep using it
- **Server failure/data loss:** Mitigated by weekly Excel backup with restore capability, hosting-level redundancy. Photos accepted as recoverable loss (members re-upload)

**Adoption Risks:**
- **Orientation week load:** The highest-stakes moment. Mitigated by deploying and testing well before orientation week, not during it
- **Members don't switch from Google Form:** Mitigated by app as primary method from day one after thorough testing, Google Form as emergency fallback only
- **Teachers don't use the roster:** Mitigated by making the roster experience genuinely delightful — fast, photo-based, mobile-first. If teachers love it, adoption is self-sustaining

## Functional Requirements

### Member Registration & Account Management

- **FR1:** Unregistered user can create an account with name, email, and photo
- **FR2:** Unregistered user can select a membership plan (12-month or 6-month) during registration
- **FR3:** Unregistered user can pay for membership via iDEAL during registration
- **FR4:** Member can log in and log out of their account
- **FR5:** Member can view and edit their profile information (name, email, photo)
- **FR6:** Member can reset their password
- **FR7:** Expired member can renew their membership by selecting a plan and paying via iDEAL
- **FR8:** Member can delete their own account and all associated data (GDPR self-service)
- **FR9:** Member receives an email notification 7 days before membership expiry and on the expiry date

### Digital Membership Card

- **FR10:** Active member can view a "Show Membership" screen displaying their photo, name, and membership validity dates
- **FR11:** Expired member sees an "Expired" status on their membership screen
- **FR12:** Member can export their membership card as a PDF for offline use

### Course Management

- **FR13:** Admin can create a course with name, schedule (day, time, quarter), location, maximum capacity, and WhatsApp group link
- **FR14:** Admin can assign a teacher-role member to a course
- **FR15:** Admin can edit and deactivate courses
- **FR16:** System auto-generates session dates based on the quarterly schedule when a course is created
- **FR17:** Member can browse available courses and view course details (schedule, location, capacity, teacher)
- **FR18:** Active member can enroll in a course (subject to capacity)
- **FR19:** Member can unenroll from a course
- **FR20:** Expired member cannot enroll in new courses

### Teacher Roster

- **FR21:** Teacher can view the roster of enrolled students for their assigned courses
- **FR22:** Roster displays each student's photo, name, and membership status
- **FR23:** Expired members appear on the roster with a visible "expired" flag
- **FR24:** Roster reflects enrollment changes without requiring the teacher to navigate away (page refresh)

### Admin Panel

- **FR25:** Admin can view a list of all members with search and filter capabilities
- **FR26:** Admin can manually create a member account (name, email, photo) and mark as paid — for cash/bank transfer registrations
- **FR27:** Admin can manually create a member account without payment — for teacher exemptions
- **FR28:** Admin can delete a member account and all associated data
- **FR29:** Admin can promote a member to teacher role and assign them to courses
- **FR30:** Admin can view an overview dashboard showing member count, recent registrations, and last backup timestamp
- **FR31:** Admin can export member data as an Excel file

### Superadmin

- **FR32:** Superadmin can promote/demote members to/from admin role
- **FR33:** Superadmin can create and deactivate admin accounts
- **FR34:** Superadmin can configure membership pricing (plan amounts)
- **FR35:** Superadmin can access system settings (Mollie configuration, backup settings)

### Payment Processing

- **FR36:** System integrates with Mollie for iDEAL payment processing
- **FR37:** System confirms payment via webhook and activates membership only upon confirmed payment
- **FR38:** System records payment details (amount, date, method) for each member
- **FR39:** Admin-created members (cash/bank transfer) are recorded with payment method and confirmation

### Data & Backup

- **FR40:** System automatically exports member data as Excel and emails it to the ISOW association email weekly
- **FR41:** Admin panel displays the timestamp of the last successful backup
- **FR42:** System supports importing member data from a standardized Excel format — used for both initial migration from Google Sheets (converted to Excel) and restoring from backup
- **FR43:** Admin can upload an Excel backup file to restore member data into the database

### Authentication & Authorization

- **FR44:** System enforces role-based access: member, teacher, admin, superadmin — each with increasing permissions
- **FR45:** System rate-limits login and registration attempts to 5 per minute per IP address
- **FR46:** New accounts created by admin receive an email invitation to set their password

### GDPR & Privacy

- **FR47:** Registration includes a consent checkbox for data storage
- **FR48:** A privacy policy page is accessible from registration and profile settings
- **FR49:** Member data deletion (self-service or admin-initiated) permanently removes all personal data, photo, and enrollment records

### Monitoring & Operations

- **FR50:** System is monitored for uptime with alerts sent to the board email when the app goes down
- **FR51:** System provides a health check endpoint for external monitoring services

## Non-Functional Requirements

### Performance

- Page load time under 2 seconds on 4G mobile for all core flows (registration, roster, membership card)
- Roster view with photos loads within 3 seconds for courses up to 30 students
- Photo upload processes (compress, store) within 5 seconds
- iDEAL payment redirect completes within 3 seconds (Mollie-dependent)
- "Show Membership" screen displays under 500ms (no network-dependent content beyond initial load)
- System handles 50 concurrent users without degradation — sufficient for orientation week peak
- Admin Excel export generates within 10 seconds for up to 5,000 member records
- No cold starts: production hosting must not have idle spindown

### Integration

- **Mollie:** iDEAL payment processing via Mollie REST API with webhook-based payment confirmation. Must handle: successful payment, failed/cancelled payment, and webhook retry scenarios
- **Email:** Transactional emails for account creation invitations, password reset, and membership expiry notifications. Via SMTP or email service provider (e.g., SendGrid free tier, or hosting provider's email)
- **UptimeRobot:** External HTTP health check polling. App exposes a lightweight health endpoint
- **Cloudflare:** DNS and proxy configuration. No application-level integration needed

### Reliability

- Target uptime: 99% (allows ~3.6 days downtime per year). Brief outages of a few hours are acceptable
- Automated weekly Excel backup with email delivery — single point of disaster recovery
- Application recoverable from backup Excel import in case of total data loss
- Database hosted on persistent storage with hosting provider's infrastructure redundancy
- Application logs errors to a persistent location for post-incident diagnosis
- Health check endpoint returns current application and database status

Note: Scalability NFRs are intentionally omitted — ISOW's user base (~1000 members/year) is well within the capacity of any single-server Django deployment. No horizontal scaling, caching layers, or CDN for dynamic content needed.
