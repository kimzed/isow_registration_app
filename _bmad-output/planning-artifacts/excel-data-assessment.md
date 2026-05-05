# ISOW Members.xlsx — Data Assessment for Migration

_Prepared for the ISOW board to verify what will be migrated into the new registration app._

## What's in the file

The file has 11 sheets but only 3 contain real data:

- **Members** — 2,010 rows, the full historical roster from 2019 to today.
- **Payments** — 656 rows, one row per payment received (date, amount, Cash/Bank, linked to a member).
- **Active Member Count** — appears to be an older auto-generated snapshot of Members. Not used for migration.

The other sheets (Registrations, FormResponses, TestingResponses, TestingMembers, TestingPayments, MailchimpUse, Map creation, Temp) are empty or contain test data and will be ignored.

## What we'll migrate

- **257 currently active members** — those whose membership expiration date is in the future as of migration day.
- The other ~1,750 expired or historical entries are **not** migrated. They stay in the Excel as a reference archive. Members whose memberships have already expired (even recently) will need to register again through the normal flow in the new app.
- **237 payment records** belonging to those 257 active members are imported alongside them, so each member keeps a record of their past payments. The other ~419 payment records belong to members we are not migrating and are left in the Excel archive.
- The 16 members whose Status field reads "close" (interpreted as "close to expiring") are part of the 257 active set and are migrated normally.
- Each migrated member receives an email invitation to set a password and upload a photo on first login.

## Data quality of the 257 active members

- **Email** is the only field the new system strictly needs. **256 of 257 have a valid email**; 1 row has no email and will need a manual decision before import (skip or hand-collect).
- All other fields (name, phone, nationality, occupation, notes) are optional. They are imported as-is from the Excel without normalization. Members can correct their own profile after their first login.
- **No member photos** exist in the file. All 257 members start photoless and upload a photo when they first log in. The teacher roster shows a placeholder (initials or silhouette) until a photo is uploaded.
- IDs in the active set are clean — all 257 have a unique numeric ID.

## Open question worth confirming

- 74 of the 257 active members have **no payment record** in the Payments sheet — possibly cash payments that were never logged. They are still migrated as active members; this is just flagged for awareness.
