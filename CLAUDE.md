# Hallway Schedule — working notes for Claude

A school schedule display SaaS. Schools get a public hallway display at a clean URL (e.g. /ludus); admins manage schedules through a login-gated dashboard.

Live: https://hallway-schedule.vercel.app

## Stack

- Frontend: plain HTML/CSS/JS, no build step, no framework
- Hosting: Vercel (auto-deploys on push to main)
- Backend: Supabase (Postgres + Auth)
- Supabase project URL: https://zatmvuxtgunoruvvofcr.supabase.co

## Files

- `index.html` — public hallway display, loaded via /slug clean URLs (rewritten by vercel.json)
- `admin.html` — superadmin dashboard: manages schools, school admins, and per-school theming
- `school-admin.html` — per-school admin: schedule overrides and no-school dates only
- `login.html` — shared login, redirects by role (superadmin → admin.html, schooladmin → school-admin.html)
- `reset-password.html` — password reset flow via Supabase auth
- `vercel.json` — catch-all slug redirect to index.html?school=:slug

## Database

- `schools` — id, name, slug, theme (jsonb)
- `schedules` — school_id, data (jsonb) — full schedule blob: days, eods, dayNames, wads, overrides, noSchool, rotLen, rotMode, anchor
- `user_profiles` — id, role ('superadmin' | 'schooladmin'), school_id — RLS enabled

The superadmin uses the service role key (embedded in admin.html) to manage users via adminDb. The school admin and public display use the anon key only.

## Architecture notes worth remembering

**Theme system.** Each school has a `theme` jsonb with a `base` hex and 7 derived slots (bg, panel, cardBg, accentBg, accent, text, textMuted). The function `deriveSlots(hex)` exists in BOTH `index.html` and `admin.html` and they must stay in sync — if I change the derivation logic in one, change it in the other. Legacy themes (with just `accent`) are migrated on the fly in index.html's `applyTheme()`.

**Schedule resolution priority** (in `getActiveSchedule()` in index.html):
1. Date override for today → use it
2. Weekly Adjusted Day (WAD) for today's day-of-week → use it (with rotation-day variant if defined)
3. Otherwise: rotation day from calendar mode or sequential anchor

**Rotation modes.**
- `calendar` — Mon=Day1, Tue=Day2, etc., modulo rotLen
- `sequential` — counts school days forward/backward from an anchor date, skipping weekends and no-school dates

**No-school / weekend / before-school / after-school states** all render the `#noschool-screen` with different titles and subtitles. The display polls every 10s and re-fetches data every 60s.

## How I prefer to work

- Targeted find-and-replace edits, not full-file rewrites, unless I explicitly ask
- Walk me through VS Code/Cursor and Git operations step-by-step — I use the Source Control panel, not the git CLI
- Before any major change, confirm the approach with me first
- For CSS colors, prefer existing CSS variables over hardcoded hex
- Don't agree sycophantically. If I'm making a design or architectural mistake, push back. Say so plainly and explain why.

## Environment

- I edit in Cursor (VS Code fork), preview in Safari
- Vercel auto-redeploys on push to main
- I don't run a local dev server — changes are tested by pushing and reloading the live site, or by opening the HTML file directly