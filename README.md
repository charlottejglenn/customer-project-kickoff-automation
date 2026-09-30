# Customer Project Kickoff Automation

An n8n workflow that automates a recurring customer kickoff preparation step.

The workflow checks a project tracker, finds projects that are ready for kickoff, sends a prepared email to the customer contact, and updates the tracker so the same email is not sent twice.

**Stack:** n8n · Google Sheets · Gmail · Google OAuth

## The Problem

In customer onboarding and SaaS implementation, project starts often involve the same manual steps:

- checking whether a project is ready to start
- sending preparation information to the customer
- asking the customer to review the project plan
- tracking whether the kickoff email was already sent

This work is simple, but it can be forgotten, repeated, or handled inconsistently.

The workflow automates this specific preparation step without trying to cover the full customer onboarding process.

## The Workflow

When a project has the status `Kickoff Ready` and the kickoff email has not been sent yet, the workflow:

1. reads the project tracker from Google Sheets
2. filters for eligible projects
3. prepares a personalized kickoff email
4. sends the email through Gmail
5. updates the tracker with the email status and timestamps

After that, the project is skipped in future runs because it is already marked as processed.

![Customer Project Kickoff workflow in n8n](images/n8n-workflow.png)

### Workflow overview

| Step | Node | Purpose |
|---|---|---|
| 1 | Daily Project Check | Runs the workflow on a schedule |
| 2 | Load Project Tracker | Reads project data from Google Sheets |
| 3 | Filter Eligible Projects | Keeps only projects that should be processed |
| 4 | Prepare Kickoff Email | Builds the email subject and body |
| 5 | Send Kickoff Email | Sends the email via Gmail |
| 6 | Mark Project as Processed | Updates the tracker after the email is sent |

## Design Decisions

### Use project status to control eligibility

The workflow only continues when both conditions are true:

```text
Status contains "Kickoff Ready"
AND
Kickoff Email Sent is false
```

The project status acts as the readiness condition, while the sent flag keeps already processed projects out of later runs.

### Prevent duplicate emails

After the email is sent, the workflow updates the project tracker:

```text
Kickoff Email Sent = TRUE
Kickoff Email Sent At = current timestamp
Last Automation Run = current timestamp
Automation Status = Success
```

The next time the workflow runs, the project no longer passes the filter. This prevents the same customer from receiving the same kickoff email more than once.

### Update the tracker after the send

The project is marked as processed only after the customer email has been sent.

This keeps the tracker state tied to the actual communication step instead of marking a project as completed beforehand.

## Project Data

The workflow uses a Google Sheet as a simple project tracker.

Key fields:

| Field | Purpose |
|---|---|
| Project ID | Used to update the correct row |
| Project Name | Used in the email |
| Customer Contact Name | Used in the greeting |
| Customer Email | Email recipient |
| Start Date | Included in the email |
| Status | Controls whether the project is ready |
| Required Kickoff Preparation | Preparation steps for the customer |
| Kickoff Email Sent | Prevents duplicate emails |
| Kickoff Email Sent At | Stores when the email was marked as sent |
| Last Automation Run | Stores the last successful run timestamp |
| Automation Status | Shows whether the automation step completed |

An anonymized sample tracker is included as:

`sample-project-tracker.csv`

## Example Email

```text
Hallo Anna Müller,

ich freue mich auf den gemeinsamen Start von "Einführung der HR-Plattform" am 15.07.2026.

Wir haben den Projektplan im Projektmanagement-Tool für euch bereitgestellt. Bitte meldet euch vor dem Start einmal dort an und schaut euch den Plan kurz an.

Im Projektplan findet ihr die nächsten Schritte, erste To-dos und wichtige Infos zum Ablauf. Fragen oder Ergänzungen könnt ihr gerne direkt im Tool hinterlassen, damit wir sie vor dem Kickoff berücksichtigen können.

Bitte erledigt vor dem Start außerdem diese Schritte:

- Im Projektmanagement-Tool anmelden
- Projektplan prüfen
- offene Fragen kommentieren

Vielen Dank und viele Grüße
Avery Morgan
```

## Setup

1. Import `workflow.json` into n8n.
2. Connect your own Google Sheets and Gmail credentials.
3. Use the included `sample-project-tracker.csv` or connect your own project tracker.
4. Replace the placeholder Google Sheet reference in the workflow.
5. Run the workflow with a project that meets the kickoff conditions.

The workflow and sample data are anonymized for portfolio use. The workflow export does not include real credentials.

## Adapting It

The workflow can be extended with additional steps around the existing kickoff process.

Possible additions include:

- adding a project plan link to the email
- sending an internal Slack notification after the customer email
- adding error handling for failed email sends or tracker updates
- adding an approval step before customer-facing emails are sent
- creating an audit log for each workflow run
- supporting English and German email templates

## Scope

This version focuses on the recurring kickoff preparation step: checking whether a project is ready, sending the preparation email, and recording the result in the project tracker.

It does not cover the wider customer onboarding process.
