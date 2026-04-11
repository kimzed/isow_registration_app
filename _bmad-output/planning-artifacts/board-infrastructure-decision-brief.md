---
title: 'ISOW Registration App — Decisions for the Board'
status: 'draft'
created: '2026-04-11'
audience: 'ISOW Board (non-technical)'
---

# ISOW Registration App — Decisions for the Board

This document summarizes three decisions that need board input before the app can go live. The app itself is being built independently of these choices — they only affect where and how it runs in production.

---

## 1. Where does the app live? (Hosting)

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

## 2. Should the board use a shared password manager?

The app introduces several new accounts and credentials that the board needs to manage:

- The ISOW superadmin account (for managing the app)
- The hosting account (wherever the app runs)
- The Mollie account (payment processing)
- The Gmail app password (for sending automated emails)
- The Cloudflare account (website security)

These credentials need to survive board turnovers — every 6 months. If credentials are lost during a handover, the board loses control of the app, payments, or hosting. Recovering access can be difficult or impossible.

### The problem without a password manager

Today, ISOW likely passes credentials through WhatsApp messages, shared documents, or verbal handovers. This works until it doesn't — one missed message, one outgoing board member who forgets to share a password, and access is lost.

With the app adding 5+ new credentials on top of whatever ISOW already manages (website, email, social media, bank), the risk of losing something during handover grows.

### What a password manager does

A password manager is a secure app where all credentials are stored in one place, protected by a single master password. The board shares one vault — anyone with access can see all stored passwords. During handover, the outgoing board transfers the master password to the incoming board, and everything else comes with it automatically.

### The options

**Option A: Adopt a password manager (recommended)**

We recommend **Bitwarden**, which offers a free plan for small organizations:

- Free, no credit card required
- Works on all devices (phone, laptop, browser)
- End-to-end encrypted — only people with the master password can see the contents
- One master password to hand over during board transition — all other credentials come with it
- Can also be used for ISOW's existing accounts (website, social media, bank), not just the app

**Option B: Continue without one**

The board continues passing credentials however they currently do. This works, but the risk of lost access during handover increases with every new account the app introduces. If a credential is lost, recovering it may require contacting service providers, proving organizational identity, and waiting — potentially leaving the app non-functional during a critical period like orientation week.

### Board decision needed

Does the board want to adopt a shared password manager for ISOW credentials? This is not just about the app — it's about how ISOW manages all its digital accounts across board turnovers.

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

| Decision | Recommendation | Board action needed |
|---|---|---|
| **Hosting** | Ask WUR IT first. If not available, Hetzner Cloud (~EUR 42/year) | Someone contacts WUR IT to ask about hosting for student organizations |
| **Password manager** | Adopt Bitwarden free for all ISOW credentials | Board decides whether to adopt a shared password manager |
| **System steward** | Course Coordinator | Board confirms which position is responsible |

These decisions don't block development — the app is being built and tested locally. But they need to be made before the app can go live for members.

---

*This document was prepared as part of the ISOW Registration App architecture planning. For technical details, see the full Architecture Decision Document.*
