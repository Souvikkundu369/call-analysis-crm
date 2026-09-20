# AI Call Analysis CRM

> **Every sales call automatically transcribed, scored, and logged. ~10 salespeople. Zero manual steps.**

An AI-powered sales call analysis and CRM pipeline built on a zero-cost stack. Every call a salesperson makes is automatically captured, attached to the right person's record, scored by Gemini LLM on communication and pitch quality, and surfaced to management with coaching notes — all without the salesperson doing anything extra.

---

## The pipeline

```
Salesperson's phone (built-in call recording)
        ↓
FolderSync — instantly uploads each recording to Google Drive
(folder per salesperson: SalesCall_<Name>)
        ↓
Google Apps Script — detects new files, parses filename for
caller number + date + salesperson, appends to Call Logs sheet
        ↓
Gemini LLM — audio → transcription → scoring
(pitch quality, objection handling, close attempt, brand knowledge)
        ↓
AI Score (0–10) + Summary + Coaching note written back to sheet
        ↓
Manager dashboard — leaderboard, need-feedback flags, audio player
```

No per-seat SaaS cost. No app install required for salespeople. No manual tagging of recordings.

---

## AI scoring

Gemini receives the call audio directly (base64 encoded) and evaluates:

| Dimension | What it measures |
|---|---|
| Pitch quality | Did they explain the offer clearly? |
| Objection handling | Did they address concerns confidently? |
| Close attempt | Did they ask for the booking? |
| Brand knowledge | Did they represent all 3 brands correctly? |
| Call type | Birthday booking / Group / Corporate / Complaint / Franchise / Wrong number |

Output: `{ score: 0–10, summary: "...", coaching: "..." }` — structured JSON so it writes directly to the sheet without parsing.

Calls scored ≤ 5 are automatically flagged for manager feedback. Score and summary appear inline in the Call Logs tab.

---

## Smart filename parser

Salesperson phones use different recorder apps, producing inconsistent filenames. The parser handles all known formats:

- `+91XXXXXXXXXX 2026-07-15 14-30-00.m4a` — standard recorder
- `PHONE-YYMMDDHHMM.mp3` — knockout-style app
- `NAME_YYYYMMDDHHMMSS.aac` — saved-contact format

Falls back to `file.getLastUpdated()` timestamp if date can't be parsed from filename. Indian mobile heuristic: number starts 6–9, year starts with 2.

---

## Manager dashboard

The dashboard evolved from a read-only published-CSV view into a full Apps Script web app (`doGet`/`doPost` router serving both the salesperson tagging UI and the manager view from the same script):

- **Weekly leaderboard** — top performers by call quality score, resets Monday
- **Today's need-feedback calls** — one-click list of calls flagged for coaching
- **In-card audio player** — listen to any call without leaving the dashboard
- **AI score badge** — colour-coded 0–10 on every call card
- **Remark field** — managers add notes directly from the dashboard, written back to the sheet via the same webhook that tags calls
- **Salesperson self-tagging** — a lightweight phone web app (`doGet` page) lets each salesperson tag their own untagged calls by type, closing the loop on calls the automatic parser couldn't classify from the filename alone

The original separate **Bookings** and **Booking Dashboard** sheets were retired once the CRM layer's Leads tab (below) fully replaced what they were tracking — one less place for the same data to drift out of sync.

---

## CRM layer

Booking-type calls (Birthday / Group / Corporate) auto-populate a **Leads** tab:

- F1 / F2 / F3 follow-up dates calculated automatically
- Stage auto-advances on edit (In Progress → Booked → Won)
- **Today's Callbacks** live tab = filtered view of follow-ups due today
- No email or WhatsApp reminders — sheet-colour view only (by design)

---

## Scale

- ~10 salespeople across 3 brands (Jus Jumpin / Stoneberry Resort / Knockout Sports Bar)
- ~30,000 historical recordings backfilled
- `syncNewCalls` trigger runs every 1 minute — new calls appear within 60 seconds
- AI audit runs every 10 minutes on unscored calls from today

---

## Tech stack

`Google Apps Script` · `Google Drive API` · `Google Sheets` · `Gemini LLM` · `FolderSync` · `JavaScript` · `Netlify` · `HTML/CSS`
