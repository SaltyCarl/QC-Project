# Weekly QC Audit: Case Closures & Escalations

## Why We Do This

When analysts resolve security incidents, they sometimes close them as "benign" even when the threat was real and required action. This weekly audit catches those misclassifications so our reporting stays accurate and clients get the full picture of what we handled on their behalf.

**Frequency:** Weekly
**Time Required:** ~30 minutes per client
**Output:** QC Report via MSSP QC Report Generator

---

## What You'll Need

- Access to Security Operations Mailbox
- Access to [CLIENT_NAME] Sentinel workspace
- Access to Google SecOps
- MSSP QC Report Generator open and ready

---

## Step 1: Check the Mailbox

Start by seeing what actually got escalated to the client this week.

1. Open the **Security Operations Mailbox**
2. Filter by **[CLIENT_NAME]** and **past 7 days**
3. Review each escalation email and jot down:
   - Incident ID
   - What happened (malware, phishing, compromised account, etc.)
   - How it was resolved

**What you're looking for:** A count of how many incidents were genuinely malicious and required client notification or action.

Keep this list handy—you'll compare it against Sentinel next.

---

## Step 2: Audit Sentinel Classifications

Now let's see if Sentinel's closure data matches reality.

### Run the Query

In the [CLIENT_NAME] Sentinel workspace, run:

```kql
SecurityIncident
| where TimeGenerated > ago(7d)
| summarize arg_max(TimeGenerated, *) by IncidentNumber
| where Status == "Closed"
| summarize Count = count() by Classification
```

### Compare the Numbers

The **TruePositive** count from Sentinel should match the number of malicious escalations you found in the mailbox.

**If they match:** Export the query results to CSV. You'll import this into the report generator.

**If they don't match:** You've found misclassified incidents. Here's how to fix them:

1. Run this query to see the details:

```kql
SecurityIncident
| where TimeGenerated > ago(7d)
| summarize arg_max(TimeGenerated, *) by IncidentNumber
| where Status == "Closed"
| project IncidentNumber, Title, Severity, Classification, ClassificationReason, ClosedTime
| order by ClosedTime desc
```

2. Cross-reference with your escalation list
3. Open any misclassified incidents in Sentinel and update:
   - Classification (e.g., BenignPositive → TruePositive)
   - ClassificationReason
   - Add a note explaining the correction

Once everything lines up, export the corrected results to CSV.

---

## Step 3: Verify SecOps Cases

Last step—make sure the case documentation in SecOps tells the same story.

1. Open **Google SecOps**
2. Filter cases by **[CLIENT_NAME]** and **last 7 days**
3. For each escalated incident, check:

| Check | What to Look For |
|-------|------------------|
| **Case notes** | Do they reflect what actually happened? |
| **Closure status** | Does it match the email thread outcome? |
| **Malicious tag** | Applied to all confirmed true positives? |

If anything's off, update the case notes and apply the malicious tag where needed.

---

## Step 4: Generate the Report

With your audit complete, pull everything together in the **MSSP QC Report Generator**.

1. Select the **reporting period** (Last 7 Days)
2. **Import the CSV** from your Sentinel query—this auto-populates the classification counts
3. Enter the **Total Escalated** count from your mailbox review
4. Add each escalation with:
   - Service Ticket / Incident Number
   - Incident Title
   - Classification (should all be TruePositive for escalations)
   - Threat type, severity, affected assets
   - Actions taken / closure notes
5. Add any **analyst notes** about corrections made or patterns observed
6. **Generate** and export to PDF

---

## Quick Reference: Classifications

| Classification | What It Means | Use When |
|----------------|---------------|----------|
| **TruePositive** | Real threat, we took action | Malware removed, account secured, threat contained |
| **BenignPositive** | Alert was valid but activity was authorized | Known admin activity, approved security testing |
| **FalsePositive** | Alert shouldn't have fired | Bad detection logic, needs tuning |
| **Undetermined** | Can't say for sure | Not enough data to make a call |

---

## Common Mistakes to Watch For

These are the misclassifications you'll see most often:

| What Happened | Often Closed As | Should Be |
|---------------|-----------------|-----------|
| Malware found and cleaned | BenignPositive | TruePositive |
| Phishing reported and blocked | FalsePositive | TruePositive |
| Compromised credentials reset | BenignPositive | TruePositive |
| Alert from authorized pentest | TruePositive | BenignPositive |

The pattern: analysts close resolved threats as "benign" because the situation is handled. But "handled" doesn't mean "wasn't malicious"—it means we did our job.

---

## Notes

_Use this space for anything worth flagging—recurring issues, analyst coaching opportunities, process improvements, etc._

---

**Document Version:** 1.0
**Last Updated:** [DATE]
**Owner:** Security Operations
