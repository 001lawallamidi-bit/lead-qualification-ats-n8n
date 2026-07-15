# lead-qualification-ats-n8n
AI-powered lead intake, qualification, and student pipeline management system built with n8n

# Lead Qualification & ATS Pipeline System

An end-to-end, fully automated lead intake and student tracking system 
built with n8n for overseas education agencies in Nigeria. Four integrated workflows 
replace a manual lead management process with an intelligent, 
event-driven pipeline that qualifies leads using AI, assigns counsellors 
automatically, tracks students through every stage, and delivers weekly 
reports to the team.

---

## The Problem It Solves

Overseas education agencies receive leads from multiple sources 
simultaneously. Without automation, staff manually read each enquiry, 
decide if the lead is worth pursuing, figure out which counsellor to 
assign, send a notification, create a record somewhere, and repeat this 
for every single lead that comes in.

This process is slow, inconsistent, and entirely dependent on whoever 
happens to be available. Good leads get delayed. Records get created 
differently by different people. No one has a clear view of where every 
student currently stands in the pipeline.

This system eliminates all of that.

---

## How The System Works

### The Full Journey — From Lead to Weekly Report

New lead arrives via form or webhook
↓
Workflow 1 — AI evaluates and scores the lead
↓
Qualified lead is assigned to the right counsellor
↓
Team is notified instantly
↓
Workflow 2 — Unique Student ID generated
↓
Student record created in pipeline tracker
↓
Stage set to New Inquiry automatically
↓
Assigned counsellor notified
↓
Workflow 3 — Counsellor updates stage via form submission
↓
Student receives stage-specific email
↓
Team notified of stage change
↓
Workflow 4 — Every Monday 8AM Lagos time
↓
All records read, grouped by stage
↓
Full pipeline report sent to team

---

## The Four Workflows

### Workflow 1 — Lead Qualification & Assignment

**Trigger:** Webhook or form submission when a new lead arrives

**What it does:**
- Receives raw lead data including name, contact details, 
  country of interest, and programme type
- Passes the lead data through an AI node that evaluates 
  the quality of the lead based on defined criteria
- Scores and classifies the lead
- Identifies the appropriate counsellor based on 
  availability
- Assigns the lead to that counsellor
- Sends an immediate notification to the assigned counsellor 
  and the wider team
- Passes the qualified lead data downstream to Workflow 2

**Key nodes used:**
Webhook trigger → AI scoring node → conditional routing → 
email notification → workflow trigger

---

### Workflow 2 — Create Student Record

**Trigger:** Called by Workflow 1 after lead qualification

**What it does:**
- Receives the qualified lead data from Workflow 1
- Generates a unique Student ID in a defined format
- Creates a new row in the Student Pipeline Google Sheet 
  with all relevant student details
- Sets the initial pipeline stage to New Inquiry
- Sends a notification to the assigned counsellor confirming 
  the record has been created and is ready for action

**Key nodes used:**
Workflow trigger → ID generation → Google Sheets write → 
email notification

---

### Workflow 3 — ATS Stage Update

**Trigger:** Counsellor submits a web form

**What it does:**
- Counsellor fills in a simple form with the Student ID 
  and the new pipeline stage
- Workflow searches the Student Pipeline sheet for that 
  Student ID
- Updates the student's current stage in the sheet
- Sends the student a personalised email written specifically 
  for that stage of the journey
- Posts an internal notification to the team confirming 
  the stage change, who made it, and when

**Pipeline stages:**
New Inquiry → Document Collection → Application Submitted → 
Offer Received → Visa Applied → Visa Approved → Enrolled

**Key nodes used:**
Form trigger → Google Sheets lookup → Google Sheets update → 
conditional email selector → email send → team notification

---

### Workflow 4 — Weekly Pipeline Digest

**Trigger:** Cron schedule — every Monday at 08:00 WAT

**What it does:**
- Reads all student records from the Student Pipeline sheet
- Groups students by their current pipeline stage
- Counts the number of students in each stage
- Formats a clean, readable digest showing the full 
  pipeline overview
- Sends the report to all relevant team members so 
  Monday mornings start with a clear picture of 
  where every student stands

**Key nodes used:**
Cron trigger → Google Sheets read → data aggregation → 
report formatter → email send

---

## Tech Stack

| Tool | Purpose |
|---|---|
| n8n | Automation engine and workflow orchestration |
| AI Node | Lead scoring and qualification |
| Google Sheets | Student pipeline database |
| Webhook | Lead intake trigger |
| n8n Form | Counsellor stage update interface |
| Email (Gmail) | Student and team notifications |
| Cron | Weekly report scheduling |

---

## System Architecture

│ LEAD INTAKE │
│ Form / Webhook → Workflow 1 │
│ AI Scoring → Counsellor Assignment │

│
▼

│ RECORD CREATION │
│ Workflow 2 → Generate Student ID │
│ Create Row in Pipeline Sheet │
│ Notify Counsellor │

│
▼

│ PIPELINE MANAGEMENT │
│ Workflow 3 → Form Submission │
│ Update Stage → Email Student │
│ Notify Team │

│
▼

│ WEEKLY REPORTING │
│ Workflow 4 → Every Monday 08:00 WAT │
│ Read All Records → Group by Stage │
│ Send Digest to Team │

---

## Google Sheets Database Structure

### Student Pipeline Sheet

| Column | Description |
|---|---|
| Student ID | Unique identifier generated by Workflow 2 |
| Full Name | Student full name from lead form |
| Email | Student email address |
| Phone | Student phone number |
| Country of Interest | Destination country for study |
| Programme Type | Undergraduate, Postgraduate, etc |
| Lead Score | AI-generated quality score |
| Assigned Counsellor | Name of assigned counsellor |
| Current Stage | Active pipeline stage |
| Stage Updated | Timestamp of last stage change |
| Date Created | Record creation timestamp |

---

## Key Outcomes

- Lead response time reduced from hours to seconds
- Every lead scored and assigned consistently using the same criteria
- No leads fall through without a record being created
- Students receive timely, stage-appropriate communication 
  automatically
- Team has full pipeline visibility every Monday morning 
  without anyone compiling a manual report
- Counsellors updated instantly when they receive a new student


## Built By

Lawal Adeyemi Lamidi
muhlawwaladeyemi@gmail.com
github.com/001lawallamidi-bit
