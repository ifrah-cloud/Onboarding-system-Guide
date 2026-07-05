# Client Onboarding System
 
> *"It's not about solving bigger problems. It's about finding the loopholes and fixing the leaks — to respect time and add value."*
 
---
 
## The Problem This Solves
 
GHL onboarding is one of the most time-consuming parts of running an agency. Not because GHL is complicated — but because there's no structure to it. A new client gets access to a powerful sub-account, stares at a blank dashboard, and freezes. They don't know where to start. They don't know what they've already done. And without a system, neither does your team.
 
The result is hours of back-and-forth. Repeated check-in calls. Clients who bought a sub-account three weeks ago and still haven't connected their calendar. Team members manually tracking who's done what in a spreadsheet or, worse, in their heads.
 
This system fixes that — not by adding more complexity, but by removing the friction that was always there.
 
---
 
## What It Does
 
A branded, gamified onboarding portal that guides GHL clients through their setup from day one. Clients know exactly where they are, what's left to do, and who to ask when they're stuck. Your team gets a live dashboard showing every client's progress without having to ask.
 
**For the client:**
- Opens inside their GHL sub-account from a menu link — no separate login, no hunting for a URL
- A checklist tells them exactly what to do and in what order
- Each step has a help panel — they can ask a question right there, without leaving the page
- A general inbox lets them message the team about anything
- If they need a custom pipeline or workflow, they can request it with a form — no email, no guessing who to contact
**For your team:**
- A live dashboard shows every client, their progress percentage, and their status
- Help messages, pipeline requests, and workflow requests all surface in the dashboard automatically
- Replies go back to the client directly through the portal
- When a client hits 100%, GHL fires a notification to the whole team — automatically
---
 
## The Time Math
 
The 1.5 hours saved per client isn't an estimate pulled from thin air. Here's where it comes from:
 
| What used to happen | What happens now | Time saved |
|---|---|---|
| "Where do I start?" call after sub-account creation | Client opens the portal, checklist tells them exactly what to do | ~20 min |
| Manual progress check-in ("Have you set up your phone number yet?") | Dashboard shows real-time progress — no check-in needed | ~15 min |
| Back-and-forth email when client is stuck on a step | Help panel sends the message to the team, reply goes back inline | ~20 min |
| "Can you build me a pipeline for X?" email thread | Pipeline simulator form captures the full spec in one submission | ~15 min |
| Forgetting to follow up when a client finishes | GHL webhook fires automatically, team gets notified immediately | ~10 min |
| Client never finishes because they lose momentum | Gamification (XP, progress bar, section completion) keeps them moving | ~30 min |
 
**Total: ~1.5 hours per client, every time.**
 
At 10 new clients a month, that's 15 hours back. At 50, it's a part-time role eliminated.
---
How It Works
---

## How It Works
 
### Client journey
 
```
Client gets GHL sub-account
        ↓
Clicks "Client Onboarding Portal" in GHL sidebar
        ↓
Lands on portal — name and email pre-filled from GHL
        ↓
Completes checklist (Profile → Phone → Contacts → Pipeline)
        ↓
Asks help questions inline if stuck
        ↓
Requests custom pipeline or workflow via simulator forms
        ↓
Hits 100% → GHL notifies the whole team automatically
```
 
---

## Architecture

```
GHL Sub-Account (client clicks menu link)
        │
        │  URL params: ?location=xxx&email=xxx&name=xxx
        ▼
Onboarding Portal  
  ├──  captures client details, creates Supabase row
  ├──  checklist, help panels, simulators, inbox
  └──  all Supabase + webhook logic
        │
        │  REST API writes
        ▼
Supabase Database
  ├── clients               
  ├── help_messages         
  ├── pipeline_requests     
  └── workflow_requests     
        │
        ├── team's live view, polls every 30s
        │
        └── webhooks fire to GHL on:
                      │
                      ▼
              GHL Workflows
                ├── Send email to client (confirmation)
                └── Notify internal team

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend / Portal | React + TypeScript |
| Database | Supabase (PostgreSQL) |
| CRM / Notifications | GoHighLevel (GHL) |
| Hosting | custom domain |
| Animations | Framer Motion |
| UI Components | shadcn/ui + Tailwind CSS |

---

## Documentation Index

| File | What it covers |
|---|---|
| [SETUP.md](./SETUP.md) | Full step-by-step implementation guide from zero to live |
| [SUPABASE.md](./SUPABASE.md) | Database schema, SQL to run, table reference |
| [GHL.md](./GHL.md) | Webhook setup, custom menu link, email templates |
| [CUSTOMIZATION.md](./CUSTOMIZATION.md) | What to change per business when implementing |
| [TESTING.md](./TESTING.md) | End-to-end verification checklist before going live |

---

## Key Files in the Codebase

| File | Purpose |
|---|---|
| `src/lib/store.ts` | All Supabase queries and GHL webhook calls. **The single source of truth for data.** |
| `src/pages/ClientLogin.tsx` | Client-facing login page. Reads URL params from GHL menu link. |
| `src/components/GamifiedChecklist.tsx` | The full checklist UI — items, help panels, inbox, simulators. |
| `src/pages/Dashboard.tsx` | Internal team dashboard. Reads live from Supabase. |
| `src/pages/DashboardLogin.tsx` | Password-protected team login. |



---

## Original Implementation

Built by Ifrah (GrowthGuild) for Enertia Flow. For questions about this system contact ifrah@growthguild.us.
