# Enertia Flow — Client Onboarding System

A fully custom client onboarding portal built on top of GoHighLevel (GHL), Supabase, and React. Clients complete a gamified checklist inside a branded portal, progress is tracked in real time, and your team is notified automatically via GHL workflows.

---

## What This System Does

When a new client is added to GHL, they click a menu link inside their sub-account that takes them to the onboarding portal. They fill in their details, work through a checklist of setup tasks, and can request help or submit custom pipeline/workflow builds directly from the portal. Your internal team sees all of this live in a dashboard and can reply to clients, manage requests, and track completion.

---

## Architecture

```
GHL Sub-Account (client clicks menu link)
        │
        │  URL params: ?location=xxx&email=xxx&name=xxx
        ▼
Onboarding Portal  (onboarding.enertiaflow.com)
  ├── ClientLogin.tsx       — captures client details, creates Supabase row
  ├── GamifiedChecklist.tsx — checklist, help panels, simulators, inbox
  └── store.ts              — all Supabase + webhook logic
        │
        │  REST API writes
        ▼
Supabase Database
  ├── clients               — one row per client, tracks progress + XP
  ├── help_messages         — all chat messages (per-step + general inbox)
  ├── pipeline_requests     — custom pipeline build requests
  └── workflow_requests     — custom workflow build requests
        │
        ├── read by Dashboard.tsx (team's live view, polls every 30s)
        │
        └── webhooks fire to GHL on:
              • Client sends a help message   → HELP webhook
              • Pipeline request submitted    → HELP webhook
              • Workflow request submitted    → HELP webhook
              • Checklist hits 100%           → COMPLETION webhook
                      │
                      ▼
              GHL Workflows
                ├── Send email to client (confirmation)
                └── Notify internal team (ifrah, ayesha, sean)
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend / Portal | React + TypeScript (GHL AI Studio / Lovable) |
| Database | Supabase (PostgreSQL) |
| CRM / Notifications | GoHighLevel (GHL) |
| Hosting | GHL AI Studio (custom domain: onboarding.enertiaflow.com) |
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

## Quick Numbers

- **4 Supabase tables** — clients, help_messages, pipeline_requests, workflow_requests
- **2 GHL webhooks** — completion (100%), help/simulator
- **5 checklist sections** — Profile, Phone, Contacts, Pipeline, (expandable)
- **3 notification recipients** — ifrah@growthguild.us, ayeshat@enertiaflow.com, sean@enertiaflow.com

---

## Original Implementation

Built by Ifrah (GrowthGuild) for Enertia Flow. For questions about this system contact ifrah@growthguild.us.
