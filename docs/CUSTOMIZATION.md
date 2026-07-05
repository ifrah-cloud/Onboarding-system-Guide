# Testing Checklist

Before inviting real clients into your onboarding portal, complete the following validation steps using a test account.

> **Recommendation**
>
> Always test using a dedicated email address rather than a real client.

---

# Phase 1 — Database Verification

Confirm that your Supabase project has been configured correctly.

## Verify

- ✅ All required database tables exist.
- ✅ Row Level Security (RLS) policies have been configured.
- ✅ The application can read and write data successfully.

Expected tables:

- `clients`
- `help_messages`
- `pipeline_requests`
- `workflow_requests`

---

# Phase 2 — Client Login

Verify the onboarding portal can successfully create and recognize clients.

## Test

- Open the onboarding portal.
- Submit a new test client.
- Confirm the client record is created.
- Sign out and sign back in using the same email.

Expected results:

- Client account is created automatically.
- Returning users resume their existing onboarding progress.
- Progress is not reset between sessions.

---

## URL Parameter Test (GoHighLevel)

If you're using GoHighLevel, verify that the portal correctly receives client information from your custom menu link.

Expected behavior:

- Name is pre-filled.
- Email is pre-filled.
- GoHighLevel Location ID is stored.
- Client record is associated with the correct location.

---

# Phase 3 — Checklist

Verify the onboarding checklist functions correctly.

## Confirm

- Completing an item updates progress.
- XP increases correctly.
- Progress persists after signing out.
- Previously completed items remain checked.
- Progress bar updates correctly.
- Status labels change throughout onboarding.

---

# Phase 4 — Help Center

Verify client support messaging.

## Checklist

- Client can open a help thread.
- Client can send messages.
- Messages appear in the database.
- Messages trigger the configured webhook.
- General support inbox works correctly.
- Checklist-specific conversations remain separated.

---

# Phase 5 — Pipeline Simulator

Verify the Pipeline Simulator.

## Confirm

- Form opens correctly.
- Submission is accepted.
- Request is stored.
- Webhook is triggered.
- Success confirmation appears.

---

# Phase 6 — Workflow Simulator

Repeat the same validation for the Workflow Simulator.

Confirm:

- Request is stored.
- Webhook executes.
- Success confirmation appears.

---

# Phase 7 — Onboarding Completion

Complete the entire onboarding checklist.

Verify:

- Completion modal appears.
- Celebration animation displays.
- Completion webhook executes.
- Internal notification is sent.
- Client confirmation email is delivered.
- Completion is only processed once.

---

# Phase 8 — Dashboard

Verify the internal dashboard.

## Confirm

- Client appears in the dashboard.
- Progress updates correctly.
- Client details open successfully.
- Support conversations appear.
- Team replies sync correctly.
- Pipeline requests display correctly.
- Workflow requests display correctly.
- Status updates work.
- Reset Progress works.
- Delete Client works.

---

# Phase 9 — Cleanup

After testing:

- Remove all test records from Supabase.
- Remove test requests.
- Remove test conversations.
- Clear browser storage if required.

---

# Common Issues

| Problem | Possible Cause | Suggested Solution |
|----------|----------------|--------------------|
| Progress does not persist | Database connection or write failure | Verify Supabase credentials and database permissions. |
| Client cannot log in | Client record not created | Verify Supabase configuration and onboarding URL parameters. |
| Help messages do not appear | Webhook or database configuration | Verify webhook endpoint and database connectivity. |
| Pipeline or Workflow requests are missing | Simulator webhook not configured | Confirm webhook URL and workflow configuration. |
| Completion actions do not trigger | Completion webhook not configured | Verify webhook configuration and completion workflow. |
| Dashboard data is outdated | Browser cache or synchronization delay | Refresh the dashboard and verify database updates. |
| GoHighLevel information is missing | URL parameters not passed correctly | Verify the custom menu link includes the required merge fields. |

---

# Production Readiness Checklist

Before inviting your first client, confirm:

- Database configured
- Webhooks configured
- GoHighLevel integration tested
- Branding customized
-  Dashboard tested
- Help Center tested
- Pipeline Simulator tested
- Workflow Simulator tested
- Completion workflow tested
- Test client removed
