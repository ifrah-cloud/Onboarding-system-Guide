# Setup Guide — From Zero to Live

Follow these steps in order. Do not skip ahead — each step depends on the one before it.

---

## Prerequisites

Before starting you need:

- A GoHighLevel agency account with sub-account access
- A Supabase account (free tier works fine — supabase.com)
- A GHL AI Studio project with the codebase already loaded
- A custom domain pointed to your AI Studio project (e.g. `onboarding.youragency.com`)

---

## Step 1 — Create Your Supabase Project

1. Go to **supabase.com** and sign in
2. Click **New Project**
3. Name it something like `client-onboarding`
4. Set a strong database password and save it somewhere safe
5. Choose the region closest to your clients
6. Wait ~1 minute for the project to initialize

Once inside the project:

1. Click the **Settings icon** (bottom left sidebar)
2. Click **API**
3. Copy and save two values:
   - **Project URL** — looks like `https://abcdefghijk.supabase.co`
   - **anon public key** — long string starting with `eyJ...`

You will need both of these in Step 3.

---

## Step 2 — Run the Database Migrations

1. Inside your Supabase project, click **SQL Editor** in the left sidebar
2. Click **New query**
3. Paste and run each SQL block from [SUPABASE.md](./SUPABASE.md) in order
4. After each block you should see **"Success. No rows returned"**

There are 3 blocks to run:
- Create the `clients` table
- Create the `help_messages` table
- Create the `pipeline_requests` and `workflow_requests` tables

---

## Step 3 — Configure store.ts

Open `src/lib/store.ts` and update the values at the very top of the file:

```typescript
// Line 2 — replace with your Supabase project URL
const SUPA_URL = 'https://YOUR_PROJECT_ID.supabase.co';

// Line 3 — replace with your Supabase anon public key
const SUPA_KEY = 'eyJ...YOUR_ANON_KEY...';

// Line 8 — replace with your GHL completion webhook URL (set up in Step 5)
const GHL_WEBHOOK_URL = 'https://services.leadconnectorhq.com/hooks/YOUR_LOCATION_ID/webhook-trigger/YOUR_COMPLETION_TRIGGER_ID';

// Line 10 — replace with your GHL help/simulator webhook URL (set up in Step 5)
const HELP_MESSAGE_WEBHOOK_URL = 'https://services.leadconnectorhq.com/hooks/YOUR_LOCATION_ID/webhook-trigger/YOUR_HELP_TRIGGER_ID';
```

**That's the only change needed in store.ts for a new implementation.** Everything else is generic.

---

## Step 4 — Update the Notification Email Addresses

In `src/lib/store.ts`, find the `TEAM_EMAILS` section inside `sendCompletionEmail` and update with your team's addresses. By default these are the Enertia Flow team emails — replace them:

```typescript
// Around line 15 of store.ts
// Search for these email addresses and replace with yours:
'ifrah@growthguild.us'
```

Note: these emails are used for logging/reference only in the current implementation. Actual email delivery goes through GHL workflows (see Step 5 and [GHL.md](./GHL.md)).

---

## Step 5 — Set Up GHL Webhooks

You need two GHL workflows with inbound webhook triggers. Full instructions with email templates are in [GHL.md](./GHL.md). Quick summary:

**Workflow 1 — Onboarding Completion**
- Trigger: Inbound Webhook
- Action: Send email to internal team + send confirmation email to client
- Copy the webhook URL and paste it into `GHL_WEBHOOK_URL` in store.ts

**Workflow 2 — Help Messages + Simulator Requests**
- Trigger: Inbound Webhook
- Action: Send internal notification + conditional branch (pipeline vs workflow vs help message)
- Copy the webhook URL and paste it into `HELP_MESSAGE_WEBHOOK_URL` in store.ts

---

## Step 6 — Set Up the GHL Custom Menu Link

This is how clients access the portal directly from inside their GHL sub-account.

1. In GHL, go to **Agency Settings → Custom Menu Links** (or inside your snapshot settings)
2. Find the existing menu link pointing to the portal, or create a new one
3. Set the URL to:

```
https://onboarding.youragency.com/?location={{location.id}}&email={{user.email}}&name={{user.fullName}}
```

Replace `onboarding.youragency.com` with your actual portal domain.

The `{{location.id}}`, `{{user.email}}`, and `{{user.fullName}}` tokens are GHL dynamic values — GHL replaces them automatically when each client clicks the link.

4. Add this menu link to your **snapshot** so it appears in every new sub-account automatically

Full details in [GHL.md](./GHL.md).

---

## Step 7 — Customize the Checklist

Open `src/components/GamifiedChecklist.tsx` and find the `CHECKLIST_DATA` array near the top of the file. Replace the checklist items with your own steps.

Each item follows this structure:

```typescript
{ id: "unique_id", text: "What the client needs to do", points: 10 }
```

Rules:
- `id` must be unique across all items (use simple codes like `p1`, `ph2`, `c3`)
- `points` add up to your total XP — the gamification header shows this automatically
- Keep section `id` values consistent — they're used as keys in the database

See [CUSTOMIZATION.md](./CUSTOMIZATION.md) for the full list of things to change per business.

---

## Step 8 — Set the Dashboard Password

The team dashboard is protected by a simple hardcoded password. To change it:

Open `src/pages/DashboardLogin.tsx` and find:

```typescript
if (password === "enertia2024") {
```

Replace `"enertia2024"` with your own password. For production use, consider a more robust auth solution.

---

## Step 9 — Publish and Test

1. In AI Studio, click **Publish**
2. Wait for the deploy to complete
3. Follow the full verification checklist in [TESTING.md](./TESTING.md)

The most common failure points are:
- Missing `await` on `createUser` in ClientLogin.tsx (causes checklist to load before user row exists)
- Supabase RLS policies blocking writes (run the policy SQL from SUPABASE.md)
- Wrong webhook URLs in store.ts

---

## Step 10 — Add the Portal to Your Snapshot

Once everything is tested and working:

1. Go to your GHL agency snapshot
2. Ensure the custom menu link from Step 6 is included
3. When you create a new sub-account from this snapshot, the menu link will appear automatically for every new client

New clients click the menu link → land on the portal with their details pre-filled → complete the checklist → your team is notified. No manual setup needed per client.
