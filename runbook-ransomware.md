# Runbook: Ransomware / Destructive Malware

**Runbook ID:** RB-IR-001  
**Incident Type:** Ransomware or Destructive Malware  
**Severity:** P1 — Critical  
**Source Risk:** R-004 (Risk Register)  
**Last Updated:** 2026-11-12  
**Owner:** Information Security Team

---

## Overview

Ransomware is malware that encrypts files or systems and demands payment for decryption. For ShopCore, the highest-risk vector is a phishing email delivered to an admin employee whose workstation has direct or indirect access to production systems. This runbook covers the full response from initial detection through recovery and post-incident review.

**Do not pay the ransom without Legal and leadership approval.** Payment does not guarantee decryption, may violate sanctions laws, and signals willingness to pay again.

---

## Indicators of Compromise (IoCs)

Activate this runbook if any of the following are observed:

- Files on a workstation or server have been renamed with an unknown extension (e.g., `.locked`, `.enc`, `.crypt`)
- A ransom note file appears on the filesystem (`README.txt`, `HOW_TO_DECRYPT.html`, etc.)
- Files are inaccessible or corrupted with no known cause
- Unusual disk I/O activity on a server with no scheduled job explanation
- Endpoint Detection & Response (EDR) alert for ransomware behavior (mass file encryption, shadow copy deletion)
- CloudTrail or system logs show commands like `vssadmin delete shadows` or `bcdedit /set recoveryenabled no`

---

## Phase 1 — Detection & Analysis

**Target time: 15 minutes from alert to confirmed scope**

- [ ] **Step 1.1** — Confirm the alert. Check the source: EDR alert, user report, monitoring system, or IT ticket.
- [ ] **Step 1.2** — Identify the affected system(s). Note hostname, IP address, owner, and function (workstation, server, database server).
- [ ] **Step 1.3** — Determine if the affected system has access to network shares, databases, or cloud storage. Document all connected resources.
- [ ] **Step 1.4** — Check for lateral movement: review recent login events on the affected system and any systems it connects to.
- [ ] **Step 1.5** — **[NOTIFY]** Alert the Information Security Team and IT Operations immediately. Do not wait for full scope confirmation.
- [ ] **Step 1.6** — **[DECISION]** Is the production database or cloud environment reachable from the infected system?
  - **Yes** → Proceed immediately to Phase 2 (Containment). Speed is critical.
  - **No** → Continue analysis but still move to containment within 30 minutes.
- [ ] **Step 1.7** — Document initial findings: time of detection, affected systems, IoCs observed, and who was notified and when.

---

## Phase 2 — Containment

**Target time: 2 hours from detection to containment**

- [ ] **Step 2.1** — **Isolate the affected system immediately.** Disconnect it from the network (pull the network cable or disable the NIC). Do NOT power off the machine — memory forensics may be needed.
- [ ] **Step 2.2** — If the affected system is a VM (cloud instance), isolate it at the network security group / firewall level. Do not terminate the instance.
- [ ] **Step 2.3** — Revoke all active sessions and credentials for any accounts that were logged into the affected system at the time of infection. Force password resets.
- [ ] **Step 2.4** — **[DECISION]** Are any other systems showing encryption activity or IoCs?
  - **Yes** → Isolate each additional system using Steps 2.1–2.2. Treat as a network-wide incident.
  - **No** → Continue with single-system containment.
- [ ] **Step 2.5** — Check cloud storage (S3 buckets) and database backups for signs of encryption or deletion. If backups are intact, do NOT mount them yet — confirm the backup environment is clean first.
- [ ] **Step 2.6** — Preserve evidence: take a forensic memory image of the affected system if EDR tooling supports it. Document chain of custody.
- [ ] **Step 2.7** — **[NOTIFY]** Escalate to CISO and Legal. Provide: affected systems, timeline, whether customer data or payment data was accessible from the infected system.
- [ ] **Step 2.8** — **[DECISION]** Was customer PII or payment card data accessible from the infected system?
  - **Yes** → Notify Legal immediately. A breach notification assessment is required.
  - **No** → Continue containment; re-evaluate if new information emerges.

---

## Phase 3 — Eradication

**Do not begin eradication until containment is confirmed and evidence is preserved.**

- [ ] **Step 3.1** — Identify the infection vector. Review email logs for phishing emails received in the 48 hours before the incident. Look for:
  - Emails with attachments (.docm, .xlsm, .js, .vbs, .zip containing executables)
  - Emails with links to external sites that were clicked
  - Emails impersonating known vendors or internal staff
- [ ] **Step 3.2** — If a phishing email is identified, preserve it and extract indicators (sender domain, link URLs, attachment hashes). Block the sender domain and URLs at the email gateway and firewall.
- [ ] **Step 3.3** — Run a full EDR scan across all systems in the same network segment as the infected system.
- [ ] **Step 3.4** — Confirm no persistence mechanisms were installed (scheduled tasks, registry run keys, new admin accounts, modified startup scripts).
- [ ] **Step 3.5** — Remove all malware artifacts identified by the EDR tool. Document every artifact removed.
- [ ] **Step 3.6** — **[DECISION]** Is the affected system a workstation or a server?
  - **Workstation** → Reimage the machine from a known-good image. Do not attempt to clean and reuse.
  - **Server** → Reimage from a known-good snapshot taken before the infection date, confirmed clean.

---

## Phase 4 — Recovery

- [ ] **Step 4.1** — Restore data from the most recent clean backup. Confirm backup integrity before restoring. Do not restore from a backup taken after the first possible infection date.
- [ ] **Step 4.2** — Bring restored systems online in an isolated environment first. Verify they are functioning correctly and show no signs of re-infection before returning to production.
- [ ] **Step 4.3** — Reset all credentials that were active on the infected system, even if no compromise of those credentials is confirmed.
- [ ] **Step 4.4** — Re-enable network connectivity to restored systems in stages. Monitor closely for 24–48 hours post-recovery.
- [ ] **Step 4.5** — Verify that all production services are functioning normally. Confirm with the business that operations have been restored.
- [ ] **Step 4.6** — **[NOTIFY]** Inform CISO and relevant business owners that systems have been restored. Provide estimated data loss window (time between last clean backup and incident).

---

## Phase 5 — Post-Incident Activity

**Complete within 5 business days of incident closure.**

- [ ] **Step 5.1** — Conduct a Post-Incident Review (PIR) with the full response team.
- [ ] **Step 5.2** — Document the full incident timeline: first possible infection → detection → containment → eradication → recovery → closure.
- [ ] **Step 5.3** — Answer the five PIR questions:
  1. What happened, and when?
  2. What was the root cause?
  3. What did we do well?
  4. What did we do poorly or too slowly?
  5. What specific actions will we take to prevent recurrence?
- [ ] **Step 5.4** — Update the Risk Register (R-004) with revised likelihood or impact scores if the incident revealed new information.
- [ ] **Step 5.5** — File the PIR report and all evidence in the incident case record. Retain for a minimum of 12 months.
- [ ] **Step 5.6** — Assign corrective action owners and due dates. Track to completion.

---

## Notification Chain

| Recipient | Trigger | Deadline |
|---|---|---|
| Information Security Team | Detection | Immediate |
| IT Operations | Detection | Immediate |
| CISO / Leadership | P1 confirmed | Within 1 hour |
| Legal / Compliance | Customer data may be affected | Immediate |
| Affected customers | Breach confirmed | Within 72 hours |
| Cyber insurance carrier | Per policy terms | Per policy terms |

---

## References

- NIST SP 800-61r3: Incident Response Recommendations and Considerations for Cybersecurity Risk Management
- ShopCore Incident Response Policy (POL-IR-001)
- ShopCore Risk Register — R-004
