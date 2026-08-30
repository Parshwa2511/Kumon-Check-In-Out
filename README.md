# Kumon Desk

A lightweight check-in system for a Kumon center, built on Google Sheets — no database, no hosting bill. A phone/tablet-friendly check-in app for the front desk, an admin dashboard for staff, and a Google Apps Script backend that does all the bookkeeping in a spreadsheet you already own.

**Status: actively being built.** This started as a simple check-in/check-out tracker and keeps growing — see [What's Coming Next](#whats-coming-next) for what's planned.

---

## Overview

| Piece | File | What it's for |
|---|---|---|
| Check-In App | `kumon-desk.html` | The front-desk kiosk — students and staff search their name and tap to check in/out |
| Dashboard | `kumon-dashboard.html` | Live admin view — attendance table, weekly/monthly hours, editing, and an attendance leaderboard |
| Backend | `Code.gs` | Google Apps Script Web App — the only thing that talks to the spreadsheet |

The spreadsheet itself is the entire database: a **Students** tab (your enrollment roster), a **Staff** tab, and a **Log** tab (auto-created) that records every completed session for history and reporting.

---

## Features

### Check-In App
- Search-as-you-type for students or staff, matched by the start of first *or* last name (so "N" finds "Navya", not "Andrew")
- One-tap check-in / check-out with an instant confirmation screen — shows check-in time, and for students, **expected pickup time**
- **Expected pickup** is computed automatically as check-in time + that student's session length (adjustable per student — see below)
- Shows each student's parent name(s), pulled automatically from the roster's Mother/Father name fields
- Shows Math/Reading assignment pages for the day
- Center hours displayed on the home screen, per day of the week
- Feels instant — check-in/out applies to the screen immediately and confirms with the sheet in the background, rather than waiting on the round-trip

### Dashboard
- Live table of everyone on the roster — who's checked in, when, and their running weekly/monthly total
- Quick check-in/out button right from the table, with the same instant-apply behavior as the check-in app
- Search and a "checked in only" filter
- Click into anyone to see:
  - A day-by-day bar chart of hours for any given week, with exact numbers (not averages), and arrows to browse past weeks
  - An editable panel for **Math pages, Reading pages, session length, and a manual hours correction** — changes here show up identically on the check-in app and everywhere else in the dashboard, since everything reads from the same source
- **Attendance tab** — a leaderboard ranking everyone by hours this week, most to least (including people with zero hours, so it's easy to see who *hasn't* been in), plus "Most Present" / "Least Present" spotlight cards
- **Auto-checkout**: everyone still checked in gets checked out automatically at 8:00 PM daily. A hidden admin panel (tap the title 5× quickly) lets you toggle this on/off or trigger it manually for testing
- Efficient by design: only polls the tab you're actually looking at, skips re-rendering when nothing's changed, and pauses entirely when the browser tab isn't visible

### Backend / Data
- Auto-creates all the attendance columns it needs (`Checked In`, `Check In Time`, `Check Out Time`, `Time Elapsed`, etc.) — nothing to set up by hand
- `Time Elapsed` ticks live in the sheet itself via formula, no script polling required
- Rolling **Weekly Minutes** (students) and **Weekly + Monthly Minutes** (staff) totals, correctly reset on Sunday / the 1st of the month
- Every completed session is appended to a readable, filterable **Log** tab — the source of truth for history and the attendance leaderboard
- **Privacy-conscious**: the check-in app only ever receives a student's name, ID, and phone number(s) — none of the roster's other sensitive fields (address, DOB, parent email, etc.) ever leave the sheet
- Race-condition-safe column creation (locked, so simultaneous requests can't create duplicate columns)

---

## Setup

1. **Spreadsheet**: needs a `Students` tab (your roster) and a `Staff` tab (`Staff Name`, `ID`, `Phone` columns). Everything else — attendance columns, the `Log` tab — is created automatically the first time the script runs.
2. **Backend**: open the spreadsheet → Extensions → Apps Script → paste in `Code.gs` → Deploy as a Web App (Execute as: Me, Who has access: Anyone) → copy the `/exec` URL.
3. **Frontend**: paste that URL into the `WEBAPP_URL` constant near the top of both `kumon-desk.html` and `kumon-dashboard.html`.
4. **Auto-checkout** (optional): in the Apps Script editor, run `installAutoCheckoutTrigger` once from the function dropdown to turn on the daily 8 PM auto-checkout.

Redeploying the script after a code change gives you a new URL each time (unless you use "Manage deployments → Edit → New version" to keep the same one) — just update `WEBAPP_URL` in both HTML files whenever it changes.

---

## What's Coming Next

This is an evolving project — planned/under consideration:

- **SMS notifications** to parents when a student is ready for pickup, sent automatically at the expected pickup time
- Broader reporting (monthly rollups, exportable summaries)
- More admin controls alongside the auto-checkout panel

More to come.
