# GoHighLevel Configuration

This document covers everything that needs to be set up inside GHL — the two webhook workflows, the custom menu link, and the email templates.

---

## Overview

The portal fires two webhooks to GHL:

| Webhook | When it fires | What GHL does with it |
|---|---|---|
| **Completion webhook** | Client hits 100% on checklist | Notifies internal team + sends client a congratulations email |
| **Help webhook** | Client sends a help message, submits a pipeline request, or submits a workflow request | Notifies internal team which specific thing happened |

---

## Workflow 1 — Onboarding Completion

### Create the workflow

1. Go to **Automation → Workflows → + Create Workflow**
2. Choose **Start from Scratch**
3. Name it: `Onboarding Completion Notification`

### Add the trigger

1. Click **Add Trigger**
2. Search for and select **Inbound Webhook**
3. Click **Save Trigger**
4. GHL will generate a webhook URL — copy it
5. Paste this URL into `store.ts` as the value of `GHL_WEBHOOK_URL`

### Send a test payload

To map the fields, GHL needs to receive a sample webhook. Use a tool like ReqBin (reqbin.com/curl) and run:

```
curl -X POST YOUR_WEBHOOK_URL_HERE \
-H "Content-Type: application/json" \
-d "{\"client_name\":\"Test Client\",\"client_email\":\"test@example.com\",\"status\":\"Onboarding Complete\",\"progress\":\"100\",\"xp\":\"185\",\"help_needed\":\"\",\"pipeline_requests\":\"\",\"workflow_requests\":\"\"}"
```

### Add workflow actions

**Action 1 — Internal team notification email**

- Add action: **Send Email**
- To: `ifrah@youragency.com, teammate2@youragency.com`
- Subject: `🎉 {{trigger.client_name}} completed onboarding!`
- Body:
```
Hi team,

{{trigger.client_name}} ({{trigger.client_email}}) has just completed their onboarding checklist!

Progress: {{trigger.progress}}%
XP Earned: {{trigger.xp}}
Help Requested: {{trigger.help_needed}}
Pipeline Requests: {{trigger.pipeline_requests}}
Workflow Requests: {{trigger.workflow_requests}}

Time to follow up and make sure they're fully set up.

— Onboarding System
```

**Action 2 — Client confirmation email**

- Add action: **Send Email**
- To: `{{trigger.client_email}}`
- Subject: `You're all set, {{trigger.client_name}}! 🎉`
- Body:
```
Hi {{trigger.client_name}},

Congratulations — you've completed your Enertia Flow onboarding!

Your account is now fully configured and your deal-flow engine is ready to go. A member of our team will be in touch shortly to walk you through your first steps.

If you have any questions in the meantime, just reply to this email.

Welcome aboard,
The Enertia Flow Team
```

5. Click **Publish** on the workflow

---

## Workflow 2 — Help Messages & Simulator Requests

### Create the workflow

1. Go to **Automation → Workflows → + Create Workflow**
2. Choose **Start from Scratch**
3. Name it: `Onboarding Help & Simulator Requests`

### Add the trigger

1. Click **Add Trigger**
2. Select **Inbound Webhook**
3. Save and copy the generated webhook URL
4. Paste it into `store.ts` as the value of `HELP_MESSAGE_WEBHOOK_URL`

### Send a test payload

```
curl -X POST YOUR_WEBHOOK_URL_HERE \
-H "Content-Type: application/json" \
-d "{\"type\":\"pipeline_request\",\"client_name\":\"Test Client\",\"client_email\":\"test@example.com\",\"help_needed\":\"Pipeline Simulator Request: Test Pipeline\",\"pipeline_name\":\"Test Pipeline\",\"stages\":\"New Lead, Contacted, Closed\",\"description\":\"Test description\"}"
```

### Add conditional branches

Add an **If/Else** condition after the trigger:

**Branch 1 — Pipeline Request**
- Condition: `trigger.type` equals `pipeline_request`
- Action: Send Email to internal team

Subject: `🔧 New Pipeline Request from {{trigger.client_name}}`

Body:
```
Hi team,

{{trigger.client_name}} ({{trigger.client_email}}) has submitted a pipeline build request.

Pipeline Name: {{trigger.pipeline_name}}
Stages: {{trigger.stages}}
Details: {{trigger.description}}

Log into the dashboard to review and update the status.

— Onboarding System
```

Also send a confirmation to the client:

Subject: `We received your pipeline request, {{trigger.client_name}}!`

Body:
```
Hi {{trigger.client_name}},

Great news — we've received your request to build a custom sales pipeline and our team is already on it.

Pipeline Name: {{trigger.pipeline_name}}
Stages: {{trigger.stages}}
Details: {{trigger.description}}

We'll have your pipeline built and ready inside your account within 1–2 business days. You'll hear from us as soon as it's live.

If you have any changes or additional context, just reply to this email or message us inside the onboarding portal.

Talk soon,
The Enertia Flow Team
```

**Branch 2 — Workflow Request**
- Condition: `trigger.type` equals `workflow_request`
- Action: Send Email to internal team

Subject: `⚡ New Workflow Request from {{trigger.client_name}}`

Body:
```
Hi team,

{{trigger.client_name}} ({{trigger.client_email}}) has submitted a workflow build request.

Workflow Name: {{trigger.workflow_name}}
Trigger: {{trigger.trigger_event}}
Actions: {{trigger.actions}}
Details: {{trigger.description}}

Log into the dashboard to review and update the status.

— Onboarding System
```

Also send a confirmation to the client:

Subject: `Your automation request is in the queue, {{trigger.client_name}}!`

Body:
```
Hi {{trigger.client_name}},

We've got your workflow request and we're on it.

Workflow Name: {{trigger.workflow_name}}
Trigger: {{trigger.trigger_event}}
Actions: {{trigger.actions}}
Additional Details: {{trigger.description}}

Our team will build this automation inside your account and test it before it goes live. Expect it within 1–2 business days.

If anything changes, just reply here or message us through the onboarding portal.

Talk soon,
The Enertia Flow Team
```

**Branch 3 — Help Message (default / else)**
- No condition needed — this catches everything that isn't a pipeline or workflow request
- Action: Send Email to internal team

Subject: `💬 {{trigger.client_name}} needs help`

Body:
```
Hi team,

{{trigger.client_name}} ({{trigger.client_email}}) has sent a help message through the onboarding portal.

Message: {{trigger.help_needed}}

Log into the dashboard to view the full thread and reply.

— Onboarding System
```

5. Click **Publish**

---

## Custom Menu Link

The custom menu link is what appears inside each GHL sub-account, giving clients one-click access to the portal with their details pre-filled.

### Setup

1. Go to **Agency Settings → Custom Menu Links**
   *(or navigate to this inside your snapshot settings)*
2. Click **+ Add Menu Link** or edit the existing one
3. Set the following:
   - **Name:** `Client Onboarding Portal` (or whatever you want it called in the sidebar)
   - **URL:**
   ```
   https://onboarding.youragency.com/?location={{location.id}}&email={{user.email}}&name={{user.fullName}}
   ```
   - **Icon:** choose anything relevant (a checkmark icon works well)
   - **Open in:** New tab

Replace `onboarding.youragency.com` with your actual portal domain.

### What the URL params do

| Param | GHL token | What the portal does with it |
|---|---|---|
| `location` | `{{location.id}}` | Saved to `clients.location_id` in Supabase. Used to build the direct pipeline deep link inside the checklist. |
| `email` | `{{user.email}}` | Pre-fills the email field on the login form |
| `name` | `{{user.fullName}}` | Pre-fills the name field on the login form |

### Including in your snapshot

Add this menu link to your GHL snapshot so it appears automatically in every new sub-account without manual setup per client.

---

## Webhook Payload Reference

### Completion webhook payload

```json
{
  "client_name": "Sara Ahmed",
  "client_email": "sara@example.com",
  "status": "Onboarding Complete",
  "progress": "100",
  "xp": 185,
  "help_needed": "p3, o2",
  "pipeline_requests": "Real Estate Leads (Stages: New Lead, Contacted, Closed)",
  "workflow_requests": "New Lead Follow-up (Trigger: New contact added, Actions: Send SMS)"
}
```

### Help webhook payload — help message

```json
{
  "type": "help_message",
  "client_name": "Sara Ahmed",
  "client_email": "sara@example.com",
  "status": "Booting Up",
  "progress": "45",
  "xp": 85,
  "help_needed": "p3: I can't find the calendar settings"
}
```

### Help webhook payload — pipeline request

```json
{
  "type": "pipeline_request",
  "client_name": "Sara Ahmed",
  "client_email": "sara@example.com",
  "help_needed": "Pipeline Simulator Request: \"Real Estate Leads\"",
  "pipeline_name": "Real Estate Leads",
  "stages": "New Lead, Contacted, Qualified, Offer Made, Closed",
  "description": "We need a separate pipeline for residential vs commercial"
}
```

### Help webhook payload — workflow request

```json
{
  "type": "workflow_request",
  "client_name": "Sara Ahmed",
  "client_email": "sara@example.com",
  "help_needed": "Workflow Simulator Request: \"New Lead Follow-up\"",
  "workflow_name": "New Lead Follow-up",
  "trigger_event": "When a new contact is added with tag 'lead'",
  "actions": "Send welcome SMS, wait 1 day, send follow-up email, assign to team member",
  "description": "The SMS should come from our main business number"
}
```
