# Customer Project Kickoff Automation

A small n8n workflow that automates a recurring customer kickoff preparation step.

The workflow checks a project tracker, finds projects that are ready for kickoff, sends a prepared email to the customer contact, and updates the tracker so the same email is not sent twice.

---

## Why I built this

In customer onboarding and SaaS implementation, project starts often involve the same manual steps:

- checking whether a project is ready to start,
- sending preparation information to the customer,
- asking the customer to review the project plan,
- tracking whether the kickoff email was already sent.

This is simple work, but it is also easy to forget, repeat, or do inconsistently. I wanted to build a small automation that solves this specific business problem without turning it into a large system.

---

## What the workflow does

When a project has the status `Kickoff Ready` and the kickoff email has not been sent yet, the workflow:

1. reads the project tracker from Google Sheets,
2. filters only eligible projects,
3. prepares a personalized kickoff email,
4. sends the email through Gmail,
5. updates the tracker with the email status and timestamps.

After that, the project will be skipped in future runs because it is already marked as processed.

---

## Workflow overview

![n8n workflow overview](images/n8n-workflow.png)

| Step | Node | Purpose |
|---|---|---|
| 1 | Daily Project Check | Runs the workflow on a schedule |
| 2 | Load Project Tracker | Reads project data from Google Sheets |
| 3 | Filter Eligible Projects | Keeps only projects that should be processed |
| 4 | Prepare Kickoff Email | Builds the email subject and body |
| 5 | Send Kickoff Email | Sends the email via Gmail |
| 6 | Mark Project as Processed | Updates the tracker after the email is sent |

---

## Tools used

- n8n
- Google Sheets
- Gmail
- Google OAuth

---

## Project tracker fields

The workflow uses a Google Sheet as a simple project tracker.

Key fields:

| Field | Why it matters |
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

An anonymized sample file is included as:

```text
sample-project-tracker.csv
```

---

## Business logic

The workflow only continues when both conditions are true:

```text
Status contains "Kickoff Ready"
AND
Kickoff Email Sent is false
```

I used the project status as the main trigger condition because it is easy to understand, easy to test, and close to how many operational processes already work.

---

## Duplicate email prevention

This was one of the most important parts of the project.

After the email is sent, the workflow updates the project tracker:

```text
Kickoff Email Sent = TRUE
Kickoff Email Sent At = current timestamp
Last Automation Run = current timestamp
Automation Status = Success
```

The next time the workflow runs, that project does not pass the filter anymore. This prevents the same customer from receiving the same kickoff email multiple times.

---

## Example email

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

---

## What I learned

While building this workflow, I worked through a few realistic automation issues:

- Google Sheets values such as `TRUE` and `FALSE` need to be handled carefully in filters.
- A scheduled workflow needs a clear control flag so it does not process the same record again.
- Updating the tracker after the email step is important because the record should only be marked as processed after the customer communication was actually sent.
- Keeping the workflow small made it easier to test and explain.

---

## Possible improvements

I kept the first version intentionally small. Useful next steps could be:

- add a project plan link to the email,
- send an internal Slack notification after the customer email,
- add error handling for failed email sends or failed tracker updates,
- add an approval step before customer-facing emails are sent,
- create an audit log for each workflow run,
- support English and German email templates.

---

## Repository files

```text
workflow.json                  Sanitized n8n workflow export
sample-project-tracker.csv     Anonymized sample tracker data
images/n8n-workflow.png        Workflow screenshot
README.md                      Project documentation
```

---

## Privacy notes

The workflow and sample data are anonymized for portfolio use. The workflow export does not include real credentials. Anyone importing it into n8n will need to connect their own Google Sheets and Gmail credentials and replace the placeholder Google Sheet URL.

---

## Development note

I used AI assistance for brainstorming, wording, and documentation structure. The workflow was configured, tested, debugged, and refined manually in n8n.
