# Runbook: Confirmed Data Breach (PII / Payment Card Data)

**Runbook ID:** RB-IR-003  
**Incident Type:** Confirmed Data Breach — Customer PII or Payment Card Data  
**Severity:** P1 — Critical  
**Source Risks:** R-001 (SQL Injection), R-003 (Third-Party Processor Breach)  
**Last Updated:** 2026-11-12  
**Owner:** Information Security Team / Legal & Compliance

---

## Overview

A data breach occurs when an unauthorized party accesses or exfiltrates protected data. For ShopCore, the highest-consequence breach scenarios involve customer PII (names, addresses, emails) and payment card data. This runbook applies when a breach is **confirmed** — meaning there is evidence that data was actually accessed or taken, not merely that an attack was attempted.

This runbook is often activated *after* another runbook (e.g., RB-IR-001 Ransomware or RB-IR-002 Credential Stuffing) reveals that customer data was exposed during that incident. It may also be activated directly when a third-party vendor notifies ShopCore of a breach on their side.

**Legal and Compliance must be involved from Step 1.4 onward. Do not send external notifications without Legal approval.**

---

## Indicators That a Breach Is Confirmed

A breach is confirmed when at least one of the following is true:

- Database query logs show bulk data extraction (e.g., `SELECT *` across customer or payment tables by an unexpected user or process)
- Exfiltration detected by DLP or network monitoring (large outbound data transfers to unknown IPs)
- Attacker has posted or offered ShopCore customer data for sale online
- A third-party vendor has notified ShopCore that their systems — which held ShopCore customer data — were compromised
- A law enforcement agency has notified ShopCore that customer data was found in their investigation
- Internal investigation of another incident (ransomware, SQL injection, insider threat) has confirmed data was accessed

---

## Phase 1 — Detection & Analysis

**Target time: 1 hour from confirmed breach signal to initial scope assessment**

- [ ] **Step 1.1** — Confirm the breach signal is real (not a false positive). Review the evidence directly: query logs, network logs, or the third-party notification letter.
- [ ] **Step 1.2** — Determine the data types involved:
  - Customer PII only (names, addresses, emails)?
  - Payment card data (card numbers, CVVs, expiration dates)?
  - Both?
  - Authentication credentials (password hashes)?
- [ ] **Step 1.3** — Estimate the scope: how many records? Which customers? What date range?
- [ ] **Step 1.4** — **[NOTIFY]** Immediately notify: Information Security Team, CISO, and Legal / Compliance. This notification must happen before any external communication of any kind.
- [ ] **Step 1.5** — **[DECISION]** Is payment card data confirmed in scope?
  - **Yes** → Notify the payment card brands and acquiring bank per PCI-DSS breach notification requirements. Legal will manage this communication.
  - **No** → Proceed; re-evaluate if new information emerges.
- [ ] **Step 1.6** — Open a formal incident case. Assign a lead investigator. All findings, actions, and communications must be logged in the case record from this point forward.
- [ ] **Step 1.7** — Place a legal hold on all relevant logs, database backups, and system images. Nothing is deleted, rotated, or overwritten during the investigation.

---

## Phase 2 — Containment

**Target time: 2 hours from breach confirmation**

- [ ] **Step 2.1** — Identify and close the breach vector immediately:
  - SQL injection: deploy a WAF rule to block the specific pattern; patch the vulnerable query.
  - Compromised admin credential: lock the account, invalidate all sessions, force password reset.
  - Third-party vendor: revoke the vendor's API access and data-sharing integrations until the scope is understood.
  - Insider: disable the employee's access immediately; coordinate with HR and Legal.
- [ ] **Step 2.2** — Verify the breach vector is closed: confirm that the same method cannot be used again right now.
- [ ] **Step 2.3** — Isolate any systems that were directly involved in the breach if they may still be under attacker control.
- [ ] **Step 2.4** — Rotate all credentials and API keys that may have been exposed or that had access to the breached data.
- [ ] **Step 2.5** — Confirm that backup copies of the breached data have not also been compromised.
- [ ] **Step 2.6** — **[NOTIFY]** Update CISO and Legal with containment status and revised scope estimate.

---

## Phase 3 — Eradication

- [ ] **Step 3.1** — Conduct a full forensic review of affected systems to identify all attacker activity: what was accessed, what was taken, what was modified, and what backdoors may have been left.
- [ ] **Step 3.2** — Remove all attacker tools, scripts, or backdoors identified during forensic review.
- [ ] **Step 3.3** — Patch or remediate the root cause vulnerability (e.g., parameterize SQL queries, enforce input validation, revoke over-privileged vendor access).
- [ ] **Step 3.4** — Confirm remediation is complete with a targeted test (e.g., attempt the same SQL injection pattern against the patched endpoint and confirm it is blocked).

---

## Phase 4 — Recovery

- [ ] **Step 4.1** — Restore any data integrity issues caused by the breach (e.g., if attacker modified records).
- [ ] **Step 4.2** — Return affected systems to production only after Legal and the Information Security Team have approved.
- [ ] **Step 4.3** — **[NOTIFY — EXTERNAL]** Legal leads all external notifications. The following notifications may be required — Legal will confirm which apply:

| Recipient | Regulatory Basis | Deadline |
|---|---|---|
| Affected customers | State breach notification laws (varies by state) | Typically 30–72 hours after discovery |
| State attorneys general | State breach notification laws | Varies by state |
| HHS (if health data involved) | HIPAA Breach Notification Rule | 60 days |
| PCI card brands / acquiring bank | PCI-DSS | Per card brand requirements |
| Federal Trade Commission | FTC Act (if applicable) | As required |
| Law enforcement | If criminal activity is involved | At Legal's discretion |

- [ ] **Step 4.4** — Customer notification must include (in plain language):
  - What happened
  - What data was involved
  - What ShopCore has done to stop it
  - What the customer should do (monitor accounts, place a fraud alert, etc.)
  - A direct point of contact for questions
  - Whether credit monitoring is being offered (Legal will advise)

---

## Phase 5 — Post-Incident Activity

**Complete within 5 business days of incident closure.**

- [ ] **Step 5.1** — Conduct Post-Incident Review. Key questions:
  1. What was the root cause, and how long had the vulnerability existed?
  2. How was the breach discovered? Was it internal detection or external notification?
  3. How many records were actually exfiltrated (confirmed vs. estimated)?
  4. What was the detection-to-containment time? What delayed us?
  5. What controls, if in place, would have prevented this?
- [ ] **Step 5.2** — Update the Risk Register (R-001 or R-003) with revised scores.
- [ ] **Step 5.3** — Assign corrective actions with owners and deadlines. Track to completion.
- [ ] **Step 5.4** — File the full incident case record — all logs, forensic images, notifications sent, and PIR — and retain for a minimum of 3 years (or longer per applicable regulation).
- [ ] **Step 5.5** — Brief the board or executive leadership if the breach was P1 or involved a large volume of customer records.

---

## Notification Chain

| Recipient | Trigger | Who Sends | Deadline |
|---|---|---|---|
| Information Security Team | Breach signal detected | Discoverer | Immediate |
| CISO | Breach confirmed | InfoSec Lead | Immediate |
| Legal / Compliance | Breach confirmed | InfoSec Lead | Immediate |
| Payment card brands / bank | Payment data in scope | Legal | Per PCI-DSS |
| Affected customers | Breach confirmed | Legal drafts, InfoSec approves | Within 72 hours or per law |
| Regulators | Per applicable regulation | Legal | Varies |

---

## References

- NIST SP 800-61r3: Incident Response Recommendations and Considerations for Cybersecurity Risk Management
- PCI-DSS v4.0: Requirement 12.10 (Incident Response Plan)
- ShopCore Incident Response Policy (POL-IR-001)
- ShopCore Data Classification Policy (POL-DC-001)
- ShopCore Risk Register — R-001, R-003
