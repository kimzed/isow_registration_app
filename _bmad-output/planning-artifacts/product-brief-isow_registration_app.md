---
title: "Product Brief: ISOW Registration & Course App"
status: "complete"
created: "2026-04-06"
updated: "2026-04-06"
inputs:
  - "user brain dump and board meeting notes"
  - "ISOW prototype (/home/cedric/repos/isow_app_prototype)"
  - "tree_manager_app reference stack (/home/cedric/repos/tree_manager_app)"
  - "isow-wageningen.com website analysis"
  - "market and competitive research"
---

# Product Brief: ISOW Registration & Course App

## Executive Summary

ISOW (International Student Organisation Wageningen) serves international students at Wageningen University with free language and dance courses, cultural events, and a community hub. Today, becoming a member requires filling in a form, visiting the office, paying in person, and picking up a physical card. Course attendance is tracked manually with Google Docs and spreadsheets. This multi-step process loses prospective members to friction and creates significant admin overhead for a volunteer board that rotates every six months.

We propose a simple, mobile-first web app — no install required — that lets new members register, upload a photo, and pay online in minutes. Teachers open the app before class and see a clear roster with photos. The result: more completed registrations, proper course control, and dramatically less admin work for a board that already volunteers 15 hours a week.

More than a registration tool, this app is **institutional memory as software** — a system designed so that every new board inherits a working operation instead of a folder of spreadsheets and undocumented processes. The best feature of this app is zero-handover operations: the next board doesn't need to think about it.

This is a volunteer-built project. The strategy is to design carefully, build a proof of concept, and present it to the board before committing further. The app must be rock-solid and nearly maintenance-free — there may not be a developer available after initial deployment.

## The Problem

**Registration friction kills conversion.** A prospective member today must: (1) find and fill in a Squarespace or Google Form, (2) make a bank transfer to ISOW's IBAN, (3) visit the Global Lounge during limited office hours, (4) pick up a physical membership card. Each step is a chance to drop off. The website itself acknowledges this fragility — it includes a backup Google Form link because the primary one is unreliable. The highest-volume moment — orientation week, when hundreds of new international students arrive — is exactly when this friction costs the most.

**Course attendance is unverifiable.** Teachers have no reliable way to check who is a registered member. People attend courses without being members, which means lost membership revenue and creates ambiguity that teachers and the board must resolve manually. Registration for courses isn't even documented on the ISOW website — it happens informally via WhatsApp or in-person.

**Board turnover amplifies everything.** The ISOW board rotates every six months. Every new board inherits spreadsheets, Google Docs, and undocumented processes. Institutional knowledge walks out the door twice a year, and each new Course Coordinator and Treasurer must re-learn the admin workflow from scratch. No one at ISOW currently knows basic longitudinal metrics: how many members renewed, which courses grew or shrank, what the revenue trend looks like. That data simply doesn't exist in a usable form.

## The Solution

A single web app that works on any phone browser — no install needed. Built as a Progressive Web App (PWA), members can add it to their home screen for an app-like experience without any app store overhead.

**For new members:** Create an account, upload a photo, choose a 12-month (EUR 20) or 6-month (EUR 12.50) membership, and pay via iDEAL. Payment confirms instantly, and the member has a digital profile with their photo as ID. The entire process takes a few minutes from a phone.

**Cash and manual fallback:** For members who cannot use iDEAL (e.g., newly arrived students without a Dutch bank account), an admin creates the account on their behalf once cash or bank transfer payment has been received. The rule is strict: **a member only exists in the system if they have paid.** There are no pending or unpaid states — registration equals payment confirmed. This eliminates follow-up workflows, reconciliation complexity, and ambiguity about who is an active member.

**For teachers:** Open the app before class, see a roster of enrolled students with photos. If someone shows up who isn't on the list, they can register on their phone in about a minute. No more guesswork, no more paper lists.

**For the board:** A clean admin panel shows all members, payment status, and roles. The system handles what spreadsheets and Google Docs currently handle — but reliably, consistently, and without requiring a handover tutorial.

**Digital membership card:** A dedicated "Show Membership" screen optimized for showing at a counter — photo, name, validity dates. This is the feature members use beyond registration day, especially for partner discounts at local Wageningen businesses (5-30% off at shops like Mama Bowls, Fun Times, IceKootje, and others). PDF export available for offline use.

**What stays the same:** WhatsApp remains the communication channel. The existing static website stays as the public-facing presence, with a link to the registration/course app. Events continue to use Google Forms — they don't require membership and the flexibility works well.

## What Makes This Different

No existing product on the market combines iDEAL payment, digital photo membership ID, course enrollment, and teacher roster views in a lightweight, no-install web app. The closest alternatives are Dutch association management tools like Membro or Banster, but they lack course/attendance management. At ISOW's scale (~1000 members), Membro would cost an estimated EUR 500-1000+/year — and still wouldn't provide course rosters or digital photo IDs.

A custom-built app costs approximately EUR 40-60/year in hosting, with EUR 0.29 per iDEAL transaction handled by Mollie (~EUR 290/year for 1000 payments). Total operating cost: approximately EUR 330-350/year — a fraction of any SaaS alternative, delivering exactly the features ISOW needs.

The technical approach mirrors a proven stack: Django, Tailwind CSS with DaisyUI, PostgreSQL, and comprehensive automated testing — the same architecture running reliably in the developer's other projects.

Additionally, a self-hosted Django app on EU servers with explicit consent flows and no third-party analytics provides a stronger privacy posture than most SaaS tools that route data through US-based subprocessors.

## Who This Serves

**New members** — International students at Wageningen University who want to join ISOW. They expect a mobile-friendly, quick process. The current multi-step registration loses them; a one-session flow keeps them.

**Teachers** — Volunteer course instructors (language and dance) who need to know who is in their class. They need a glanceable roster with photos, not a spreadsheet to cross-reference. Teachers are ISOW's most committed weekly stakeholders — if they rely on the roster, they become the app's strongest advocates and the adoption flywheel that keeps the board using it across turnovers.

**Board administrators** — Volunteers (especially the Course Coordinator and Treasurer) who currently spend hours on manual member tracking, payment verification, and handover documentation. They need a system that simply works when the next board takes over.

## Success Criteria

- **Transition to app:** At least 70% of new registrations happen through the app within the first full semester of deployment.
- **Near-zero maintenance:** A stable version is reached before the end of the first semester of deployment, with close to zero bugs in production. The test suite catches regressions before they reach users.
- **Admin time reduction:** Board members report a significant and noticeable reduction in time spent on membership administration, payment tracking, and course roster management compared to the previous semester's spreadsheet-based process.
- **Board adoption:** The sitting board formally adopts the app as the primary registration method, and the old Google Form / Squarespace form is retired or demoted to a secondary fallback.

## Scope

### In for v1
- Member registration with photo upload
- Membership plans (12-month / 6-month) with iDEAL payment via Mollie
- Cash/manual registration fallback for admin (member created only after payment received)
- Digital membership profile with photo ID and "Show Membership" screen
- PDF export of membership card (for offline use and partner discounts)
- Course creation and management (quarterly schedule, sessions, capacity)
- Course enrollment and unenrollment
- Teacher roster view with photos
- Admin panel (member management, role assignment, payment status)
- Excel export/backup of member data
- Data migration pipeline from existing Google Sheet to PostgreSQL
- Comprehensive test suite with CI (GitHub Actions, protected main branch)
- Operations runbook: admin onboarding, cash registration flow, quarterly course setup, backup procedure, credential locations
- GDPR essentials: consent at registration, data deletion capability, EU-hosted data, privacy policy page
- Automated weekly data backup (Excel export emailed to association email)
- Basic uptime monitoring (e.g., UptimeRobot free tier, alerts to board email)

### Nice-to-have (post-v1, if feasible)
- Apple Wallet / Google Wallet membership card integration
- Course waitlists (with demand signal data for the board)
- Integration or merger with the static ISOW website

### Explicitly out of scope
- Event management or ticketing (Google Forms works, events don't require membership)
- WhatsApp replacement or in-app messaging
- Native mobile app
- Multi-language support (English only)
- Physical card printing workflow

## Continuity & Operations

This app will outlive its developer. Design decisions must prioritize survivability.

**Organizational ownership:** All paid services — hosting, domain, Mollie payment account — must be registered under ISOW organizational accounts using the association email, not personal accounts. Credentials are stored in a shared password manager that transfers with the board. The specifics of how ISOW manages its email accounts and organizational access need to be discussed with the board to align with their existing handover process.

**System steward:** One board role (recommended: Course Coordinator) is designated as the app's operational owner — responsible for verifying the app works at the start of each semester, performing board handover for admin access, and being the first contact for issues. This requires no technical skill, only ownership.

**Automated safety nets:** Weekly Excel backup emailed to the association address. Uptime monitoring that alerts the board if the app goes down. A visible "last backup" timestamp in the admin panel.

**Rollout strategy:** The app runs alongside existing processes for a full registration cycle before the old process is retired. Explicit go/no-go criteria and board sign-off before switching over. A rollback procedure is documented in case the app needs to be taken offline.

**Deployment timing:** The app should be deployed and tested well before orientation week — ideally mid-semester or during a break, giving time to iron out issues in lower-stakes conditions. Orientation week is the make-or-break stress test: if the app works flawlessly when hundreds of new students arrive, adoption is solved. It must be ready and proven before that moment, not launched during it.

## Vision

If the app proves itself, it becomes the single digital home for ISOW's operational needs — potentially absorbing the static website and serving as the first thing a new international student in Wageningen interacts with. The architecture is clean enough that other Wageningen student associations with similar needs could potentially adopt it.

But the near-term priority is clear: nail registration and course management, prove it's stable, and earn the board's trust. This app is institutional memory as software — every design decision is tested against one question: "Does this survive a complete board replacement?"
