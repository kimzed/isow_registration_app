---
title: 'ISOW Registration App — Decisions for the Board'
status: 'partially resolved'
created: '2026-04-11'
updated: '2026-05-05'
audience: 'ISOW Board (non-technical)'
---

# ISOW Registration App — Decisions for the Board

This document originally listed three decisions for the board. **Two are now resolved** (hosting and credential custody — see notes below). **One remains open**: who on the board is the app's "system steward."

None of the remaining decision blocks development or even deployment — they are recommendations for how ISOW operates the app over time.

---

## 1. Where does the app live? (Hosting) — RESOLVED

**Decision: Hetzner Cloud (~EUR 42/year, data stored in Germany or Finland).**

The board approved Hetzner Cloud on 2026-05-05. The full evaluation of the alternatives (PythonAnywhere, Railway, WUR-managed server, local computer) is retained below for reference and historical context.

---

### Original evaluation (kept for reference)

The app needs a computer that's always on and connected to the internet, so members can register and teachers can check rosters at any time. This is called "hosting." There are several options, each with different trade-offs.

### What matters for ISOW

- **Cost** — ISOW runs on membership fees. Lower is better.
- **Reliability** — The app must work during orientation week and before every class. Downtime means lost registrations.
- **EU data storage** — Member data (names, emails, photos) should be stored in Europe. This is good practice under European privacy rules (GDPR).
- **Maintenance** — Someone needs to keep the server updated. Less maintenance = better for a volunteer board.
- **No "cold starts"** — When a teacher opens the roster 5 minutes before class, the app must respond immediately, not take 30-60 seconds to wake up.
- **Automated tasks** — The app needs to run scheduled tasks automatically: sending weekly data backups and daily membership expiry reminders. The hosting must support this without limitations.

### The options

#### Option A: Hetzner Cloud (Recommended)

A European cloud provider with data centers in Germany and Finland.

| | |
|---|---|
| **Cost** | ~EUR 3.50 per month (~EUR 42 per year) |
| **Total annual cost** (hosting + payment fees) | ~EUR 332 |
| **Data stored in** | Germany or Finland (EU) |
| **Reliability** | Professional data center, always on |
| **Maintenance** | The developer sets it up once. After that, occasional security updates are needed (a few hours per year by a technically skilled volunteer) |
| **Best for** | Lowest cost with full reliability |

#### Option B: PythonAnywhere EU

A managed hosting service specifically designed for Python/Django apps, with an EU option.

| | |
|---|---|
| **Cost** | EUR 10 per month (EUR 120 per year) |
| **Total annual cost** (hosting + payment fees) | ~EUR 410 |
| **Data stored in** | EU |
| **Reliability** | Managed platform, always on |
| **Maintenance** | Minimal — the platform handles most server maintenance |
| **Limitation** | Only allows 1 scheduled task per day on the Developer plan. The app needs at least 2 (daily expiry reminders + weekly backup). This would require workarounds or a more expensive plan. |
| **Best for** | If the board wants the least technical maintenance, is willing to pay more, and the task limitation can be resolved |

#### Option C: Railway

An easy-to-use cloud platform popular with developers.

| | |
|---|---|
| **Cost** | ~EUR 5 per month (~EUR 55 per year) |
| **Total annual cost** (hosting + payment fees) | ~EUR 345 |
| **Data stored in** | United States by default — EU option is uncertain |
| **Reliability** | Good, always on |
| **Maintenance** | Low — code is deployed automatically |
| **Best for** | If EU data storage is not a priority |

#### Option D: WUR-Managed Server

Wageningen University may be able to provide server space for ISOW as a recognized student organization.

| | |
|---|---|
| **Cost** | Potentially free |
| **Total annual cost** (hosting + payment fees) | ~EUR 290 (only payment processing fees) |
| **Data stored in** | Netherlands (EU) |
| **Reliability** | University-grade infrastructure |
| **Maintenance** | Managed by WUR IT (potentially) |
| **Best for** | Best overall option *if WUR offers this* |
| **Action needed** | Someone from the board needs to ask WUR IT if this is possible |

#### Option E: Computer at the Global Lounge

A small computer (like a Raspberry Pi) running at the ISOW office.

| | |
|---|---|
| **Cost** | ~EUR 50-100 one-time purchase |
| **Total annual cost** (hosting + payment fees) | ~EUR 290 (only payment processing fees) |
| **Data stored in** | Physically at the Global Lounge (EU) |
| **Reliability** | Depends on office internet and power. If the internet goes down or there's a power cut, the app goes down too. No backup if the device fails. |
| **Maintenance** | Board is responsible for the hardware |
| **Not recommended** | The app needs to be available 24/7, including outside office hours and during holidays. A single device with no backup is too fragile for a system the board depends on. |

### Our recommendation

**First choice: Ask WUR IT** whether they can host the app for ISOW. If yes, this is the best option — free, reliable, EU-based, professionally managed. Someone from the board should reach out to WUR IT to explore this.

**If WUR hosting is not available: Hetzner Cloud.** It's the cheapest reliable option at ~EUR 42/year, with data stored in Germany/Finland. The developer will set everything up. After that, the only ongoing need is occasional security updates — a technically inclined volunteer (CS student, digital committee member, or the developer) could handle this in a few hours per year.

---

## 2. How are passwords handed over between boards? — RESOLVED

**Decision: Gmail-anchored model. No shared password manager needed.**

The app introduces several new accounts and credentials (hosting, Mollie payment processing, Cloudflare, the Gmail app password, the ISOW superadmin account). These need to survive every board turnover.

Rather than introducing a shared password manager (which would add ongoing onboarding friction for every incoming board member), the app uses a simpler model that takes advantage of something ISOW already has: **the organizational Gmail account.**

### How it works

- **Every third-party account is registered using the ISOW Gmail address** (not a personal email). This applies to the hosting account, Mollie, Cloudflare, the domain registrar, and any future service.
- The ISOW Gmail account becomes the "master key" — whoever holds it can perform a password reset on every other service and immediately take over.
- During board handover, the only credential that needs to be passed is the **ISOW Gmail password**. Everything else can be reset from there.
- The current operator (the developer or whoever runs the app day-to-day) keeps a copy of the active credentials in whatever personal password manager they prefer. This is a personal choice, not an organizational requirement.

### Why this works for ISOW

- **No new tool to learn or maintain.** The board already uses Gmail.
- **One thing to hand over instead of many.** Pass the Gmail password — everything else follows.
- **No vendor lock-in.** If a future board wants to adopt a shared password manager later, they can do so at any time without rework.
- **Realistic for ISOW's scale.** A shared vault makes sense when multiple people operate the system at the same time. ISOW's app has one operator at a time.

### What the board needs to do

- Make sure ISOW Gmail uses a strong password and that 2-factor authentication recovery codes are stored in a known place (printed copy held by the chair, for example).
- During handover, pass the Gmail password to the incoming board. That's it.
- The operations runbook (a separate document the developer will produce) lists every account that's registered against ISOW Gmail, so the new operator knows what they have access to.

### Open follow-up (not blocking)

If ISOW already uses a shared vault for other things (website login, social media, bank), the app credentials can be added there too — but this is optional and doesn't change the deployment plan.

---

## 3. Who is responsible for the app? (System Steward)

The app is designed to run without developer intervention. But someone on the board should be the "owner" — not to fix technical problems, but to make sure the app is working and to handle the administrative side.

### What this person does

- **At the start of each semester:** Open the app, make sure it works, check that the backup is running (there's a "last backup" timestamp on the admin dashboard)
- **During board handover:** Create admin accounts for new board members, deactivate old ones (15-minute task)
- **If something goes wrong:** Be the first person to notice and coordinate a response (contact the developer or a tech volunteer)
- **Monthly:** Glance at the admin dashboard — member count, recent registrations, last backup. This takes 2 minutes.

### No technical skill required

This is an organizational role, not a technical one. The app is designed to be self-explanatory. The admin panel requires no training beyond a short walkthrough from the outgoing board member.

### Our recommendation

**The Course Coordinator.** This board position already interacts with the app the most — they set up courses each quarter, assign teachers, and handle course-related questions. It's a natural fit.

---

## Summary

| Decision | Status | Outcome / Action |
|---|---|---|
| **Hosting** | RESOLVED (2026-05-05) | Hetzner Cloud (~EUR 42/year, EU data centers) |
| **Password handover** | RESOLVED (2026-05-05) | Gmail-anchored model — no shared password manager. Register every account against ISOW Gmail; hand over only the Gmail password between boards. |
| **System steward** | OPEN | Recommended: Course Coordinator. Board confirms which position is responsible before the app is handed off to a new board. |

The remaining open decision does not block development or even deployment — it is a recommendation for how the board operates the app over time.

---

*This document was prepared as part of the ISOW Registration App architecture planning. For technical details, see the full Architecture Decision Document.*
