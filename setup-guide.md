# 🎓 Study Abroad Automation System — Setup Guide

A full end-to-end automation system for study abroad agencies built on **n8n**, covering lead
capture, AI-powered qualification, counsellor assignment, application tracking, stage management,
and weekly reporting.

---

## 📋 Table of Contents

- [System Overview](#system-overview)
- [Workflows in This System](#workflows-in-this-system)
- [Architecture & Flow Diagram](#architecture--flow-diagram)
- [Prerequisites](#prerequisites)
- [External Services & Accounts Required](#external-services--accounts-required)
- [Google Sheets CRM Setup](#google-sheets-crm-setup)
- [Google Drive Setup](#google-drive-setup)
- [Workflow Setup Instructions](#workflow-setup-instructions)
  - [Workflow 1: Lead Intelligence & Auto-Assignment](#workflow-1-lead-intelligence--auto-assignment)
  - [Workflow 2: Create Student Record](#workflow-2-create-student-record)
  - [Workflow 3: Stage Update](#workflow-3-stage-update)
  - [Workflow 4: Weekly Pipeline Report](#workflow-4-weekly-pipeline-report)
- [Credential Configuration](#credential-configuration)
- [Placeholder Reference Table](#placeholder-reference-table)
- [Testing the System](#testing-the-system)
- [Troubleshooting](#troubleshooting)

---

## System Overview

This system automates the full lifecycle of a study abroad student from initial inquiry through
to enrollment. When a prospective student submits the eligibility form, the system:

1. Sends the student an instant acknowledgment email
2. Evaluates their profile using an AI model via the Groq API
3. Scores and qualifies the lead automatically
4. Assigns the next available counsellor using a round-robin rotation
5. Creates a dedicated Google Drive folder for the student
6. Logs the lead and all AI evaluation data into a Google Sheets CRM
7. Creates a student pipeline record with a unique Student ID
8. Notifies the assigned counsellor with the full evaluation report
9. Sends the student their counsellor assignment details
10. Allows counsellors to update student stages via a form
11. Sends stage-specific emails to students at every milestone
12. Delivers an automated weekly pipeline report every Monday at 8:00 AM WAT

---

## Workflows in This System

| # | File | Description |
|---|------|-------------|
| 1 | `lead-intelligence-auto-assignment.json` | Lead capture form, AI evaluation, counsellor assignment |
| 2 | `create-student-record.json` | Webhook receiver that creates the student pipeline record |
| 3 | `stage-update.json` | Form-based stage updater with student and team notifications |
| 4 | `weekly-pipeline-report.json` | Scheduled Monday report summarising the full pipeline |


[Eligibility Form]
│
▼
[Student Acknowledgment Email]
│
▼
[AI Evaluation — Groq API]
│
▼
[Parse & Score Lead]
│
▼
[Merge Form + AI Data]
│
├──▶ [Log Lead → Google Sheets: Leads_Master]
│ │
│ ▼
│ [Create Google Drive Folder]
│ │
│ ▼
│ [Get Counsellors from Sheet]
│ │
│ ▼
│ [Assign Next Counsellor (Round-Robin)]
│ │
│ ┌──────┴────────────────────────────┐
│ ▼ ▼
│ [Assignment Email → Student] [Evaluation Report → Counsellor]
│ │
│ ▼
│ [Update Last Assigned]
│ │
│ ▼
└──────────────────────────▶ [Webhook → Create Student Record]
│
▼
[Generate Student ID]
│
▼
[Append → Student Pipeline Sheet]
│
▼
[New Record Email → Counsellor]

[Stage Update Form]
│
├──▶ [Find Student Record in Sheet]
│
▼
[Merge Form + Sheet Data]
│
▼
[Validate Student Record]
│
▼
[Update Student Stage in Sheet]
│
▼
[Stage Email → Student]
│
▼
[Stage Update Notification → Team]

[Schedule: Every Monday 8AM WAT]
│
▼
[Read All Student Records]
│
▼
[Group Students by Stage]
│
▼
[Weekly Pipeline Report Email → Team]


---

## Prerequisites

Before setting up this system, ensure you have the following:

- An **n8n** instance (self-hosted or n8n Cloud)
  - Minimum recommended version: **1.40.0**
  - [n8n installation guide](https://docs.n8n.io/hosting/)
- A **Google account** with access to Gmail, Google Sheets, and Google Drive
- A **Groq API account** with an active API key
  - Sign up at [console.groq.com](https://console.groq.com)
- Basic familiarity with n8n workflow importing and credential setup

---

## External Services & Accounts Required

| Service | Purpose | Free Tier Available |
|---------|---------|-------------------|
| n8n | Workflow automation engine | ✅ (self-hosted) |
| Google Sheets | CRM — leads and student pipeline data | ✅ |
| Google Drive | Student document folder storage | ✅ |
| Gmail | All email notifications | ✅ |
| Groq API | AI lead evaluation and scoring | ✅ |

---

## Google Sheets CRM Setup

All workflows read from and write to a single Google Sheets workbook. Create a new Google
Sheets file and add the following sheet tabs with the exact column headers listed below.

---

### Sheet 1: `Leads_Master`

This sheet stores every lead submitted through the eligibility form along with the full
AI evaluation output.

| Column Header | Description |
|---------------|-------------|
| Submission Timestamp | Date and time of form submission |
| Full Name | Lead's full name |
| WhatsApp Number | Lead's WhatsApp contact |
| Email Address | Lead's email address |
| Preferred Destination | Target country for study |
| Has Passport | Passport status |
| Highest Qualification | Academic qualification level |
| Course of Study | Previous course studied |
| Grade Graduated With | Graduation grade |
| Intended Course Abroad | Course they want to study |
| Dependents Joining | Whether dependents will join |
| Intake | Target intake period |
| Budget (Naira) | Tuition and POF budget |
| Visa Refusal History | Previous visa refusal status |
| Status | AI qualification status |
| Readiness Score | AI readiness score out of 100 |
| Budget Assessment | AI budget analysis |
| Academic Assessment | AI academic analysis |
| Grade Assessment | AI grade analysis |
| Timeline Assessment | AI timeline analysis |
| Scholarship Flag | Whether student is a scholarship candidate |
| Risk Flags | Identified risk factors |
| Recommended Action | AI recommended next action |
| Reasoning | Full AI reasoning narrative |

---

### Sheet 2: `Student Pipeline`

This sheet tracks every active student through the application stages.

| Column Header | Description |
|---------------|-------------|
| Student_ID | Unique auto-generated student identifier |
| Full Name | Student's full name |
| WhatsApp Number | Student's WhatsApp contact |
| Email Address | Student's email address |
| Preferred Destination | Target country |
| Intended Course | Course to study abroad |
| Target Intake | Target intake period |
| Budget | Tuition and POF budget |
| AI Status | Qualification status from lead evaluation |
| Readiness Score | AI score from lead evaluation |
| Assigned Counsellor | Name of assigned counsellor |
| Counsellor Email | Email of assigned counsellor |
| Current Stage | Current pipeline stage |
| Stage Number | Numeric stage position |
| Stage Updated At | Timestamp of last stage update |
| Drive Folder Link | URL to student's Google Drive folder |
| Drive Folder ID | Google Drive folder ID |
| Date Created | Record creation date |
| Notes | Counsellor notes from stage updates |

---

### Sheet 3: `Counsellors`

This sheet manages the counsellor pool for the round-robin assignment system.

| Column Header | Description |
|---------------|-------------|
| Counsellor_ID | Unique counsellor identifier (e.g. C001) |
| Name | Counsellor's full name |
| Email | Counsellor's email address |
| Phone | Counsellor's WhatsApp/phone number |
| Active | Whether counsellor is active (`true` or `false`) |
| Last_Assigned | Timestamp of last lead assignment (leave blank initially) |

#### Example Counsellors Sheet Data

| Counsellor_ID | Name | Email | Phone | Active | Last_Assigned |
|---------------|------|-------|-------|--------|---------------|
| C001 | Counsellor A | counsellora@yourdomain.com | +234XXXXXXXXXX | true | |
| C002 | Counsellor B | counsellorb@yourdomain.com | +234XXXXXXXXXX | true | |
| C003 | Counsellor C | counsellorc@yourdomain.com | +234XXXXXXXXXX | true | |

> ⚠️ **Important:** The `Last_Assigned` column must exist but should be left **blank** for all
> counsellors when you first set up the sheet. The system uses blank values to identify
> counsellors who have never been assigned a lead, and prioritises them first in the
> round-robin rotation.

---

## Google Drive Setup

1. Open [Google Drive](https://drive.google.com)
2. Create a new folder and name it `Students_Intake_Folder`
3. Open the folder and copy its **Folder ID** from the URL: https://drive.google.com/drive/folders/YOUR_FOLDER_ID_IS_HERE
4. 4. Save this Folder ID — you will need it when configuring **Workflow 1**

Each time a new lead is submitted, the system automatically creates a subfolder inside
`Students_Intake_Folder` named after the student.

---

## Workflow Setup Instructions

### Workflow 1: Lead Intelligence & Auto-Assignment

**File:** `lead-intelligence-auto-assignment.json`

This is the primary entry point of the system. It handles the eligibility form, AI evaluation,
counsellor assignment, and triggers the student record creation webhook.

#### Step 1 — Import the Workflow

1. In your n8n instance, go to **Workflows → Add Workflow**
2. Click the **⋮ menu → Import from file**
3. Select `lead-intelligence-auto-assignment.json`
4. Click **Save**

#### Step 2 — Configure Credentials

Attach the following credentials to the nodes listed:

| Node | Credential Type | Credential to Attach |
|------|----------------|---------------------|
| AI Evaluation (Groq) | HTTP Header Auth | Your Groq API key (see [Credential Configuration](#credential-configuration)) |
| Student Acknowledgment | Gmail OAuth2 | Your Gmail account |
| Gmail: Notify | Gmail OAuth2 | Your Gmail account |
| Assignment Notification | Gmail OAuth2 | Your Gmail account |
| Drive: Create Folder | Google Drive OAuth2 | Your Google Drive account |
| Log Lead | Google Sheets OAuth2 | Your Google Sheets account |
| Get Counsellors | Google Sheets OAuth2 | Your Google Sheets account |
| Update Last Assigned | Google Sheets OAuth2 | Your Google Sheets account |

#### Step 3 — Update Placeholders

Open each node and replace the following placeholders with your actual values:

| Node | Field | Replace With |
|------|-------|-------------|
| AI Evaluation (Groq) | Authorization header value | `Bearer YOUR_GROQ_API_KEY` |
| Drive: Create Folder | Folder ID | Your `Students_Intake_Folder` Google Drive folder ID |
| Log Lead | Document ID | Your Google Sheets spreadsheet ID |
| Log Lead | Sheet Name / GID | GID of your `Leads_Master` sheet |
| Get Counsellors | Document ID | Your Google Sheets spreadsheet ID |
| Get Counsellors | Sheet Name / GID | GID of your `Counsellors` sheet |
| Update Last Assigned | Document ID | Your Google Sheets spreadsheet ID |
| Update Last Assigned | Sheet Name / GID | GID of your `Counsellors` sheet |
| Trigger: Create Student Record | URL | Your Workflow 2 webhook URL |

#### Step 4 — Get the Form URL

1. Open the **n8n Form Trigger** node
2. Copy the **Production URL** shown at the bottom of the node
3. Share this URL with prospective students as your eligibility form link

#### Step 5 — Activate the Workflow

Toggle the workflow to **Active** using the switch in the top-right corner.

---

### Workflow 2: Create Student Record

**File:** `create-student-record.json`

This workflow runs as a webhook receiver. It is triggered automatically by Workflow 1 and
creates the student's pipeline record in Google Sheets and notifies the counsellor.

#### Step 1 — Import the Workflow

1. In your n8n instance, go to **Workflows → Add Workflow**
2. Click the **⋮ menu → Import from file**
3. Select `create-student-record.json`
4. Click **Save**

#### Step 2 — Configure Credentials

| Node | Credential Type | Credential to Attach |
|------|----------------|---------------------|
| Append to Student Pipeline | Google Sheets OAuth2 | Your Google Sheets account |
| Gmail: New Student Record Notification | Gmail OAuth2 | Your Gmail account |

#### Step 3 — Update Placeholders

| Node | Field | Replace With |
|------|-------|-------------|
| Append to Student Pipeline | Document ID | Your Google Sheets spreadsheet ID |
| Append to Student Pipeline | Sheet Name / GID | GID of your `Student Pipeline` sheet |

#### Step 4 — Copy the Webhook URL

1. Open the **Webhook** trigger node
2. Copy the **Production URL**
3. Go back to **Workflow 1** and paste this URL into the **Trigger: Create Student Record**
HTTP Request node URL field

#### Step 5 — Activate the Workflow

Toggle the workflow to **Active**.

> ⚠️ **Important:** Workflow 2 **must be activated before Workflow 1**. If Workflow 2 is
> inactive when Workflow 1 runs, the webhook call will fail and the student pipeline record
> will not be created.

---

### Workflow 3: Stage Update

**File:** `stage-update.json`

This workflow provides counsellors with a form to update a student's current stage. It validates
the Student ID, updates the Google Sheets record, and sends tailored emails to both the student
and the internal team.

#### Step 1 — Import the Workflow

1. In your n8n instance, go to **Workflows → Add Workflow**
2. Click the **⋮ menu → Import from file**
3. Select `stage-update.json`
4. Click **Save**

#### Step 2 — Configure Credentials

| Node | Credential Type | Credential to Attach |
|------|----------------|---------------------|
| Find Student Record | Google Sheets OAuth2 | Your Google Sheets account |
| Update Student Stage | Google Sheets OAuth2 | Your Google Sheets account |
| Notify Student of Stage Update | Gmail OAuth2 | Your Gmail account |
| Notify Team of Stage Change | Gmail OAuth2 | Your Gmail account |

#### Step 3 — Update Placeholders

| Node | Field | Replace With |
|------|-------|-------------|
| Find Student Record | Document ID | Your Google Sheets spreadsheet ID |
| Find Student Record | Sheet Name / GID | GID of your `Student Pipeline` sheet |
| Update Student Stage | Document ID | Your Google Sheets spreadsheet ID |
| Update Student Stage | Sheet Name / GID | GID of your `Student Pipeline` sheet |
| Notify Team of Stage Change | Send To | Your internal team notification email address |

#### Step 4 — Update the Counsellor Dropdown

Open the **Stage Update Form** trigger node and update the **Counsellor Name** dropdown
options to match the actual names of your counsellors.

#### Step 5 — Get the Form URL

1. Open the **Stage Update Form** trigger node
2. Copy the **Production URL**
3. Share this URL with your counsellors so they can update student stages

#### Step 6 — Activate the Workflow

Toggle the workflow to **Active**.

---

### Workflow 4: Weekly Pipeline Report

**File:** `weekly-pipeline-report.json`

This workflow runs automatically every Monday at 8:00 AM West Africa Time (WAT). It reads all
student records, groups them by stage, and sends a formatted HTML pipeline report to the
specified team email address.

#### Step 1 — Import the Workflow

1. In your n8n instance, go to **Workflows → Add Workflow**
2. Click the **⋮ menu → Import from file**
3. Select `weekly-pipeline-report.json`
4. Click **Save**

#### Step 2 — Configure Credentials

| Node | Credential Type | Credential to Attach |
|------|----------------|---------------------|
| Read All Student Records | Google Sheets OAuth2 | Your Google Sheets account |
| Weekly Pipeline Report | Gmail OAuth2 | Your Gmail account |

#### Step 3 — Update Placeholders

| Node | Field | Replace With |
|------|-------|-------------|
| Read All Student Records | Document ID | Your Google Sheets spreadsheet ID |
| Read All Student Records | Sheet Name / GID | GID of your `Student Pipeline` sheet |
| Weekly Pipeline Report | Send To | Your report recipient email address |

#### Step 4 — Adjust the Schedule (Optional)

The default schedule is **every Monday at 8:00 AM**. To change this:

1. Open the **Schedule Trigger** node
2. Modify the `triggerAtDay` value (0 = Sunday, 1 = Monday ... 6 = Saturday)
3. Modify the `triggerAtHour` value (0–23 in 24-hour format)

#### Step 5 — Activate the Workflow

Toggle the workflow to **Active**.

---

## Credential Configuration

### Google Sheets & Google Drive OAuth2

1. In n8n, go to **Settings → Credentials → Add Credential**
2. Search for **Google Sheets OAuth2 API** and click it
3. Click **Connect my account** and follow the Google sign-in flow
4. Grant all requested permissions
5. Save the credential and note its name
6. Repeat the same process for **Google Drive OAuth2 API**
7. Attach both credentials to the relevant nodes in each workflow

> 💡 You can use a single Google OAuth2 credential for both Sheets and Drive if your n8n
> version supports shared credentials, or create separate ones for each service.

---

### Gmail OAuth2

1. In n8n, go to **Settings → Credentials → Add Credential**
2. Search for **Gmail OAuth2 API** and click it
3. Click **Connect my account** and sign in with the Gmail account
that will send all system emails
4. Grant the requested permissions
5. Save the credential
6. Attach this credential to every Gmail node across all four workflows

> ⚠️ All outgoing emails (student notifications, counsellor alerts, team reports) will be
> sent **from the Gmail account you connect here**. Ensure it is a professional account
> appropriate for client-facing communication.

---

### Groq API Key

1. Go to [console.groq.com](https://console.groq.com)
2. Sign in or create a free account
3. Navigate to **API Keys → Create API Key**
4. Copy the generated key
5. In n8n, open **Workflow 1 → AI Evaluation (Groq)** node
6. In the **Authorization** header value field, enter: Bearer YOUR_GROQ_API_KEY


> ⚠️ **Security Note:** Never commit your raw API key to a public GitHub repository.
> The workflow JSON files in this repository have been sanitised and use placeholder
> values. Always add your real credentials only inside your private n8n instance.

---

## Placeholder Reference Table

Use this table to track all values you need to collect before setup.

| Placeholder | Where to Find It | Workflows Affected |
|-------------|-----------------|-------------------|
| `YOUR_GROQ_API_KEY` | [console.groq.com](https://console.groq.com) → API Keys | 1 |
| `YOUR_GOOGLE_SHEETS_DOCUMENT_ID` | Google Sheets URL between `/d/` and `/edit` | 1, 2, 3, 4 |
| `YOUR_STUDENT_PIPELINE_SHEET_GID` | Google Sheets URL `#gid=XXXXXX` on the Student Pipeline tab | 2, 3, 4 |
| `YOUR_COUNSELLORS_SHEET_GID` | Google Sheets URL `#gid=XXXXXX` on the Counsellors tab | 1 |
| `YOUR_GOOGLE_DRIVE_FOLDER_ID` | Google Drive folder URL after `/folders/` | 1 |
| `YOUR_GOOGLE_SHEETS_CREDENTIAL_ID` | n8n → Settings → Credentials | 1, 2, 3, 4 |
| `YOUR_GOOGLE_DRIVE_CREDENTIAL_ID` | n8n → Settings → Credentials | 1 |
| `YOUR_GMAIL_CREDENTIAL_ID` | n8n → Settings → Credentials | 1, 2, 3, 4 |
| `YOUR_STUDENT_RECORD_WEBHOOK_URL` | Workflow 2 → Webhook node → Production URL | 1 |
| `YOUR_TEAM_NOTIFICATION_EMAIL` | Your internal team or manager email address | 3 |
| `YOUR_REPORT_RECIPIENT_EMAIL` | Your report recipient email address | 4 |

---

## Testing the System

Follow these steps in order to verify that the full system works end-to-end before going live.

### Step 1 — Verify Google Sheets Structure

- Open your Google Sheets CRM
- Confirm all three sheet tabs exist: `Leads_Master`, `Student Pipeline`, `Counsellors`
- Confirm all column headers are spelled exactly as listed in this guide
- Confirm at least one counsellor row exists in the `Counsellors` sheet with `Active = true`

### Step 2 — Test Workflow 2 First

- Open Workflow 2 in n8n
- Confirm it is **Active**
- Use a tool like [Postman](https://www.postman.com) or
[hoppscotch.io](https://hoppscotch.io) to send a test POST request to the webhook URL
with sample student data

### Step 3 — Submit a Test Lead

- Open the eligibility form URL from Workflow 1
- Fill in all fields with realistic test data
- Use a real email address you have access to so you can verify emails are received
- Submit the form

### Step 4 — Verify Each Step

After submitting the test form, check the following in order:

- [ ] Student receives acknowledgment email immediately
- [ ] Lead row appears in the `Leads_Master` sheet with AI evaluation data
- [ ] A new subfolder is created inside `Students_Intake_Folder` in Google Drive
- [ ] A row appears in the `Student Pipeline` sheet with a generated `STU-XXXXX` ID
- [ ] The assigned counsellor receives the evaluation report email
- [ ] The student receives the counsellor assignment email

### Step 5 — Test a Stage Update

- Open the Stage Update Form URL from Workflow 3
- Enter the Student ID generated in the previous step
- Select a new stage and submit the form
- Verify:
- [ ] The student's `Current Stage` is updated in the `Student Pipeline` sheet
- [ ] The student receives a stage-specific email
- [ ] The team receives a stage change notification email

### Step 6 — Test the Weekly Report (Manual Trigger)

- Open Workflow 4 in n8n
- Click **Execute Workflow** to run it manually
- Verify the pipeline report email is received at your report recipient address
- Confirm all students and stages appear correctly in the report

---

## Troubleshooting

### The form submits but no email is received

- Check that the Gmail credential is correctly attached to all Gmail nodes
- Verify the email address in the `Send To` field is correct
- Check your n8n execution logs for errors: **Executions → View All**
- Check the spam/junk folder of the recipient email

### The AI evaluation node fails

- Verify your Groq API key is valid and has not expired
- Confirm the `Authorization` header value is formatted as `Bearer YOUR_KEY` with a space
- Check your Groq account dashboard for rate limit or quota issues at
[console.groq.com](https://console.groq.com)

### No row is added to Google Sheets

- Confirm the Google Sheets credential has edit access to the spreadsheet
- Double-check the Document ID and Sheet GID values in each node
- Ensure the column headers in your sheet exactly match those listed in this guide
(headers are case-sensitive)

### The counsellor assignment always picks the same person

- Open the `Counsellors` sheet and check that more than one row has `Active = true`
- Verify the `Last_Assigned` column exists and is spelled exactly as `Last_Assigned`
- After the first assignment, check that the `Last_Assigned` timestamp was written
correctly to the sheet

### Workflow 2 webhook returns a 404 error

- Confirm Workflow 2 is toggled to **Active** — inactive workflows do not accept
webhook requests
- Verify the webhook URL in Workflow 1's HTTP Request node matches the Production URL
shown in Workflow 2's Webhook trigger node exactly
- Ensure you are using the **Production URL** and not the Test URL

### The Stage Update Form cannot find the student

- Confirm the Student ID is entered exactly as shown in the pipeline sheet,
including the `STU-` prefix
- Verify the `Student_ID` column exists in the `Student Pipeline` sheet
- Check that the Google Sheets credential on the `Find Student Record` node
has read access to the spreadsheet

### The weekly report runs but shows all zeros

- Confirm the `Student Pipeline` sheet has data rows beneath the header row
- Verify the `Current Stage` column values in the sheet exactly match the stage
names defined in the workflow (they are case-sensitive)
- Run the **Read All Student Records** node individually and inspect its output
to confirm data is being read correctly

---

## Security Recommendations

- Never commit real API keys, email addresses, spreadsheet IDs, or credential IDs
to a public repository
- Rotate your Groq API key immediately if it has ever been exposed publicly
- Use environment variables or n8n's built-in credential store for all sensitive values
- Restrict access to your n8n instance using authentication
- Limit Google OAuth2 scopes to only what is required by each workflow
- Share the Stage Update Form URL only with authorised counsellors

---

*Built with n8n · Google Workspace · Groq AI*
---

## Architecture & Flow Diagram
