# Testing & Verification Checklist

Run through every item below before going live with real clients. Use a test email address (not a real client's) for all of this.

---

## Prerequisites

- System is fully set up per [SETUP.md](./SETUP.md)
- All four Supabase tables exist (verify with the SQL in [SUPABASE.md](./SUPABASE.md))
- Both GHL workflows are published
- Portal is deployed and accessible at your domain

---

## Phase 1 — Database Connectivity

Open your browser's developer tools (F12 → Console tab) before starting. Errors will appear there.

**1.1 — Tables exist**
Run this in Supabase SQL Editor:
```sql
select table_name from information_schema.tables
where table_schema = 'public'
  and table_name in ('clients','help_messages','pipeline_requests','workflow_requests');
```
✅ Expected: 4 rows returned

**1.2 — RLS policies allow writes**
Run this in Supabase SQL Editor:
```sql
select tablename, policyname from pg_policies where schemaname = 'public';
```
✅ Expected: at least one policy per table

---

## Phase 2 — Client Login Flow

**2.1 — Portal loads**
Navigate to your portal URL.
✅ Expected: Login form appears with your logo and branding

**2.2 — Login creates a Supabase row**
1. Fill in a test name, email, and phone
2. Click Start Onboarding
3. Go to Supabase → Table Editor → clients
✅ Expected: A new row exists with your test email, progress = 0, completed_items = []

**2.3 — Returning client loads saved progress**
1. Log out (if there's a logout button) or clear localStorage in dev tools
2. Log back in with the same test email
✅ Expected: Same name/email recognized, checklist loads with previous state

**2.4 — GHL URL params pre-fill the form**
Open this URL in your browser (replace values):
```
https://your-portal-domain.com/?location=testloc123&email=test@example.com&name=Test+Client
```
✅ Expected: Name and email fields are pre-filled. Location ID stored in Supabase after login (check `location_id` column).

---

## Phase 3 — Checklist Progress Tracking

**3.1 — Checking an item saves to Supabase**
1. Log in with your test account
2. Check one item
3. Go to Supabase → clients → your test row
✅ Expected: `completed_items` array contains the item ID, `progress` > 0, `xp` > 0

**3.2 — Progress persists across sessions**
1. Check 3–4 items
2. Clear localStorage (dev tools → Application → Local Storage → clear)
3. Log back in
✅ Expected: The same items are still checked — progress loaded from Supabase, not browser storage

**3.3 — Progress bar updates correctly**
1. Check items one at a time
✅ Expected: Progress bar and percentage update after each item. Status changes from "Initializing" → "Booting Up" → "Calibrating" as you progress.

**3.4 — XP accumulates correctly**
✅ Expected: XP score increments by the correct point value shown on each item badge

---

## Phase 4 — Help Messaging

**4.1 — Per-step help panel opens**
1. Click "Need help with this step?" under any checklist item
✅ Expected: Accordion expands showing a message thread (empty) and an input box

**4.2 — Client can send a help message**
1. Type a message in the input and press Enter or click Send
✅ Expected: Message appears in the thread with "You" label

**4.3 — Message saved to Supabase**
Go to Supabase → Table Editor → help_messages
✅ Expected: A row exists with your client_email, the item's ID, sender = 'client'

**4.4 — Help webhook fires to GHL**
Go to your GHL help workflow → Execution History
✅ Expected: A new execution appears with your test client's details

**4.5 — General inbox works**
1. Open the Team Inbox card (top of checklist)
2. Send a test message
✅ Expected: Message saved to help_messages with item_id = 'general'

---

## Phase 5 — Pipeline Simulator

**5.1 — Form opens inside pipeline section**
1. Find the "Review Your Sales Pipeline" section
2. Click "Launch Pipeline Simulator"
✅ Expected: Form expands with Pipeline Name, Stages, and Description fields

**5.2 — Submission saves to Supabase**
1. Fill in the form and submit
✅ Expected: Row appears in Supabase → pipeline_requests table with status = 'pending'

**5.3 — Webhook fires to GHL**
Go to your GHL help workflow → Execution History
✅ Expected: Execution with type = 'pipeline_request' and your submitted pipeline details

**5.4 — Success state shows**
✅ Expected: After submitting, form shows a green checkmark and "Request Submitted" message

---

## Phase 6 — Workflow Simulator

**6.1 — Form opens inside contacts section**
Find "Launch Workflow Simulator" inside the Import Contacts section
✅ Expected: Form expands with Workflow Name, Trigger, Actions, Description fields

**6.2 — Submission saves to Supabase**
✅ Expected: Row in workflow_requests with status = 'pending'

**6.3 — Webhook fires to GHL**
✅ Expected: Execution with type = 'workflow_request' in GHL workflow history

---

## Phase 7 — Completion Flow

**7.1 — Completion fires when checklist hits 100%**
1. Check all items (or manually update Supabase to set completed_items to all IDs and progress to 100)
✅ Expected: Confetti modal appears

**7.2 — Completion webhook fires to GHL**
Go to your GHL completion workflow → Execution History
✅ Expected: New execution with status = 'Onboarding Complete', progress = '100'

**7.3 — Internal team notification email received**
✅ Expected: Email arrives at all three team addresses within a few minutes

**7.4 — Client confirmation email received**
✅ Expected: Email arrives at the test client's email address

**7.5 — Completion only fires once**
1. Uncheck an item then re-check it to hit 100% again
✅ Expected: No second webhook fires (controlled by `hasSentCompletionEmail` ref)

---

## Phase 8 — Team Dashboard

**8.1 — Dashboard loads and shows the test client**
1. Navigate to `/dashboard-login`
2. Enter the team password
✅ Expected: Dashboard loads showing the test client with correct progress %

**8.2 — Progress updates are reflected in real time**
1. Open the portal in one tab, the dashboard in another
2. Check some items in the portal
3. Click Refresh in the dashboard (or wait 30 seconds for auto-refresh)
✅ Expected: Progress bar updates to match

**8.3 — Client detail modal works**
Click a client row in the dashboard
✅ Expected: Modal opens with three tabs — Progress, Help Messages, Inbox

**8.4 — Help Messages tab shows messages**
✅ Expected: Any messages sent in Phase 4 appear here with correct sender labels

**8.5 — Team can reply to help messages**
1. Type a reply in the dashboard message thread
2. Click Reply
✅ Expected: Reply appears in the thread. Check that it also appears in the checklist's help panel when viewed as the client.

**8.6 — Inbox tab shows general messages**
✅ Expected: Messages from Phase 4 step 4.5 appear here

**8.7 — Pipeline Requests tab shows submissions**
✅ Expected: Your test pipeline request appears with Pending status

**8.8 — Status update buttons work**
Click "Start" then "Mark Done" on a pipeline or workflow request
✅ Expected: Status badge updates immediately

**8.9 — Reset button works**
Click the reset icon on a client row → confirm
✅ Expected: Client's progress drops to 0% in the dashboard and in Supabase

**8.10 — Delete button works**
Click the trash icon on a client row → confirm
✅ Expected: Client row disappears from the dashboard and is removed from Supabase

---

## Phase 9 — Clean Up After Testing

1. Delete all test client rows using the dashboard delete button or this SQL:
```sql
delete from clients where email = 'your-test-email@example.com';
delete from help_messages where client_email = 'your-test-email@example.com';
delete from pipeline_requests where client_email = 'your-test-email@example.com';
delete from workflow_requests where client_email = 'your-test-email@example.com';
```

2. Clear browser localStorage on any devices used for testing

---

## Common Failure Points

| Symptom | Likely cause | Fix |
|---|---|---|
| Progress resets to 0% every time page loads | `createUser` called without `await` in ClientLogin | Add `await` before `createUser()` |
| Checkboxes don't save | `isLoaded` flag not set before save effect runs | Check that `setIsLoaded(true)` is called after `fetchUser` completes |
| Supabase returns 406 error | Missing `Content-Type` header | Verify HEADERS object in store.ts includes all three headers |
| Supabase returns 403/401 | RLS policy missing or anon key wrong | Re-run the policy SQL from SUPABASE.md, check the key in store.ts |
| Webhook fires but GHL doesn't receive it | Wrong webhook URL in store.ts | Copy URL fresh from GHL workflow trigger step |
| GHL field mapping shows blank | Test payload never sent to GHL | Send the curl test payload from GHL.md before building the workflow |
| Location ID not saving | `location_id` column missing from clients table | Run: `alter table clients add column if not exists location_id text;` |
| Pipeline link doesn't appear | location_id is null for this client | Client logged in before GHL params were added to menu link URL |
