# Customization Guide

This document lists every place you need to change something when implementing this system for a new business. Nothing else in the codebase needs to be touched for a standard implementation.

---

## File 1 — `src/lib/store.ts`

This is the only file with credentials and URLs. Change these values at the top:

| Line | Variable | What to put |
|---|---|---|
| ~2 | `SUPA_URL` | Your Supabase project URL |
| ~3 | `SUPA_KEY` | Your Supabase anon public key |
| ~8 | `GHL_WEBHOOK_URL` | Your completion workflow webhook URL from GHL |
| ~10 | `HELP_MESSAGE_WEBHOOK_URL` | Your help/simulator workflow webhook URL from GHL |

Also update the notification email addresses inside `sendCompletionEmail` if you're using them for logging purposes. Search for:

```
'ifrah@growthguild.us'
```

Replace with your team's email addresses. Note: actual email delivery is handled by GHL workflows, not this array directly.

**Everything else in store.ts is generic and should not need changes.**

---

## File 2 — `src/components/GamifiedChecklist.tsx`

### The checklist content

Find the `CHECKLIST_DATA` array and replace the items with your client's actual onboarding steps. Structure:

```typescript
export const CHECKLIST_DATA = [
  {
    id: "section_id",        // unique, no spaces, used as database key
    title: "Section Title",  // displayed as the section header
    items: [
      {
        id: "item_id",       // unique across ALL items, used as database key
        text: "What the client needs to do",
        points: 10,          // XP awarded when this item is checked
      },
      // more items...
    ],
  },
  // more sections...
];
```

**Important rules:**
- Never reuse an `id` — duplicates will cause checklist state bugs
- Keep item IDs short and consistent (e.g. `p1`, `p2`, `ph1`, `ph2`, `c1`)
- If you remove or rename items after clients have already started, their saved progress will reference the old IDs — existing clients won't be affected but their completed item list may contain IDs that no longer exist in the UI
- The pipeline section (`id: "pipeline"`) is where the Pipeline Simulator form renders — keep this section ID as `"pipeline"` or update the reference in the JSX (`isPipeline` check)
- The workflow simulator renders inside the section with `id: "contacts"` — change this if you want it elsewhere

### The encouragement messages

Find the `ENCOURAGEMENTS` array and replace with your own copy. There are 8 messages mapping to 8 progress milestones (0%, ~14%, ~28%, ~43%, ~57%, ~71%, ~86%, 100%).

### The completion confetti message

Find the section with `System Online` heading text inside the confetti modal and update with your branding:

```tsx
<h2 className="font-display text-3xl font-bold uppercase mb-2">System Online</h2>
<p className="text-muted-foreground mb-6">Your copy here...</p>
```

---

## File 3 — `src/pages/ClientLogin.tsx`

### Branding

The logo image URL:

```tsx
// Find this line and replace the src with your logo URL
<img src="https://your-logo-url.com/logo.png" alt="Your Agency" ... />
```

### Page copy

Find and update:
- `"Client Portal"` — the card title
- `"Enter your details to access your onboarding checklist"` — the subtitle
- `"Setting up your account…"` — the loading button text
- `"Start Onboarding"` — the submit button text

### Team access link

The "Team Access" link at the bottom points to `/dashboard-login`. Update the label if needed:

```tsx
<Link to="/dashboard-login" ...>
  <Shield className="w-3 h-3 mr-2" />
  Team Access  {/* ← change this label */}
</Link>
```

---

## File 4 — `src/pages/Dashboard.tsx`

### Logo

Find the `<img>` tag in the header and replace with your logo:

```tsx
<img src="https://your-logo-url.com/logo.png" alt="Your Agency" ... />
```

### Header copy

Find and update:
- `"Command Center"` — the header subtitle next to the logo
- `"Team Progress Overview"` — the main page heading
- `"Monitor engine initialization across all active personnel."` — the subheading

### Dashboard password

Open `src/pages/DashboardLogin.tsx` and find:

```typescript
if (password === "enertia2024") {
```

Replace `"enertia2024"` with a strong password of your choice.

---

## File 5 — `src/pages/DashboardLogin.tsx`

### Branding

Same logo swap as other pages. Also update:
- `"Command Center"` — the card title
- `"Restricted access. Internal personnel only."` — the subtitle
- `"Authorization Code"` — the input label
- `"Authenticate"` — the button text

---

## GHL Configuration

### Custom menu link URL

Replace `onboarding.enertiaflow.com` with your portal's domain:

```
https://onboarding.youragency.com/?location={{location.id}}&email={{user.email}}&name={{user.fullName}}
```

### Email templates

In both GHL workflows, update:
- The internal team email addresses (To field)
- `"The Enertia Flow Team"` sign-off → your agency name
- Any copy that mentions "Enertia Flow" by name

See [GHL.md](./GHL.md) for the full email templates.

---

## Optional Customizations

These are not required but worth knowing about:

### XP point values

Each checklist item has a `points` value. The total XP displayed in the header is calculated automatically from the sum of all items. Adjust point values to weight more important steps higher.

### Status labels

The status labels that appear in the dashboard ("Initializing", "Booting Up", "Calibrating", "Fully Operational") are set in `GamifiedChecklist.tsx` inside the save effect:

```typescript
const status = progressPercent === 100 ? "Fully Operational"
  : progressPercent > 50 ? "Calibrating"
  : progressPercent > 0 ? "Booting Up" : "Initializing";
```

Replace these with language that fits your brand.

### Pipeline simulator placement

The pipeline simulator form renders inside whatever checklist section has `id: "pipeline"`. If you want to rename that section or move the simulator, find this in `GamifiedChecklist.tsx`:

```tsx
{isPipeline && currentUserEmail && (
  <div className="px-3 pb-2">
    <PipelineSimulatorPanel ... />
  </div>
)}
```

### Workflow simulator placement

Similarly, the workflow simulator is placed inside the section with `id: "contacts"`. Find:

```tsx
{category.id === 'contacts' && currentUserEmail && (
```

Change `'contacts'` to whatever section ID makes sense for your checklist.

---

## What NOT to Change

These things are wired deeply into the data layer — changing them requires also updating the database schema or multiple files simultaneously:

- The Supabase table names (`clients`, `help_messages`, `pipeline_requests`, `workflow_requests`)
- The `item_id = 'general'` convention for inbox threads
- The column names in any table
- The `store_updated` custom event name (used to sync the dashboard in real time)
- The `currentUser` localStorage key (used by the checklist to identify the logged-in client)
