# Runbook: Credential Stuffing / Account Takeover

**Runbook ID:** RB-IR-002  
**Incident Type:** Credential Stuffing Attack / Account Takeover  
**Severity:** P2 — High (escalates to P1 if admin accounts are compromised)  
**Source Risk:** R-002 (Risk Register)  
**Last Updated:** 2026-11-12  
**Owner:** Information Security Team / Identity & Access Team

---

## Overview

A credential stuffing attack uses large lists of stolen username/password pairs — obtained from breaches of other services — to attempt automated logins. Because many users reuse passwords, even a small percentage of attempts will succeed. For ShopCore, a successful attack results in customer account takeover: the attacker can view PII, place fraudulent orders, and harvest saved payment methods.

This runbook covers both the detection of an active attack and the response to confirmed account compromises.

---

## Indicators of Compromise (IoCs)

Activate this runbook if any of the following are observed:

- Spike in failed login attempts across many different accounts in a short time window
- Unusually high login success rate from a specific IP, IP range, or geographic region not typical for ShopCore customers
- Multiple accounts accessed from the same IP address or device fingerprint
- Login attempts using known breached credentials (detected by breach password list)
- Customer reports of unauthorized activity in their account
- Alert from rate-limiting or bot detection tooling on the `/login` endpoint
- Unusual account activity (password changes, address changes, large orders) across many accounts in a short period

---

## Phase 1 — Detection & Analysis

**Target time: 1 hour from alert to scope assessment**

- [ ] **Step 1.1** — Pull login attempt logs for the last 24 hours. Look for:
  - Total failed login attempts vs. baseline
  - Number of distinct accounts targeted
  - Source IPs, ASNs, and geographic distribution
  - User agents (scripted/automated tools often show identical or rotating user agents)
- [ ] **Step 1.2** — Determine the attack scale:
  - **< 1,000 attempts, no confirmed takeovers** → P3, monitor closely
  - **1,000–10,000 attempts or 1+ confirmed takeover** → P2, activate this runbook fully
  - **Admin account targeted or compromised** → Escalate immediately to P1
- [ ] **Step 1.3** — Identify confirmed compromised accounts: accounts that had a successful login from an anomalous IP or showed post-login activity inconsistent with normal behavior.
- [ ] **Step 1.4** — **[NOTIFY]** Alert Information Security Team. If any admin accounts are involved, notify CISO immediately.
- [ ] **Step 1.5** — Document: attack start time (estimated), number of accounts targeted, number confirmed compromised, source IPs identified.

---

## Phase 2 — Containment

**Target time: 8 hours from detection**

- [ ] **Step 2.1** — **Block attacking IPs** at the WAF or load balancer. If the attack is distributed across many IPs (botnet), move to rate-limiting and CAPTCHA challenges rather than IP blocking alone.
- [ ] **Step 2.2** — Implement or increase rate limiting on the `/login` endpoint: no more than 10 attempts per IP per minute; no more than 5 attempts per account per 10 minutes.
- [ ] **Step 2.3** — Enable CAPTCHA challenges for login attempts that exceed threshold, or for all logins if the attack is large-scale.
- [ ] **Step 2.4** — **Lock all confirmed compromised accounts** immediately. The account owner must verify identity before regaining access.
- [ ] **Step 2.5** — **[DECISION]** Is the attack ongoing (attempts still being logged at elevated rate)?
  - **Yes** → Keep WAF blocks and rate limits active. Consider temporarily requiring CAPTCHA for all logins site-wide.
  - **No** → Proceed with account remediation; lift site-wide restrictions after 24 hours of normal traffic.
- [ ] **Step 2.6** — If any admin accounts were targeted (even unsuccessfully), force immediate password resets and MFA re-enrollment for all admin accounts.

---

## Phase 3 — Eradication

- [ ] **Step 3.1** — For each confirmed compromised account:
  - Invalidate all active sessions
  - Force a password reset (require a password not in the known-breached list)
  - Require MFA enrollment if not already set up
- [ ] **Step 3.2** — Review post-compromise activity in each affected account:
  - Were orders placed? Cancel fraudulent orders; notify the fraud team.
  - Was the shipping address or email changed? Revert to the last known good state.
  - Was payment information accessed or used? Notify the customer and advise them to monitor their card statements.
- [ ] **Step 3.3** — Extract all attacker IPs, ASNs, and user agents. Add to permanent blocklist in WAF. Share IoCs with threat intelligence feeds if applicable.
- [ ] **Step 3.4** — Check whether the same credential pairs were attempted on admin accounts. If yes, treat as a separate P1 investigation.

---

## Phase 4 — Recovery

- [ ] **Step 4.1** — Notify all confirmed compromised customers individually. The notification must include:
  - What happened (in plain language — no jargon)
  - What ShopCore has done (account locked, session invalidated)
  - What the customer should do (reset password, check recent orders, monitor payment card)
  - A direct link to reset their password
- [ ] **Step 4.2** — Assess whether broader customer notification is warranted (e.g., if thousands of accounts were targeted, even if only some were compromised, customers deserve to know).
- [ ] **Step 4.3** — Confirm WAF rules, rate limiting, and CAPTCHA are functioning correctly and are set to remain active.
- [ ] **Step 4.4** — Verify that compromised accounts are fully restored to their pre-attack state or locked pending customer action.
- [ ] **Step 4.5** — **[NOTIFY]** Inform CISO that containment and customer notification are complete.

---

## Phase 5 — Post-Incident Activity

**Complete within 5 business days of incident closure.**

- [ ] **Step 5.1** — Conduct Post-Incident Review. Key questions:
  1. How long was the attack running before detection? What was the detection gap?
  2. How many accounts were targeted vs. compromised? What was the success rate?
  3. Did our rate limiting and CAPTCHA controls exist before this incident? If not, why not?
  4. How quickly were compromised accounts locked and customers notified?
  5. What would have reduced the blast radius?
- [ ] **Step 5.2** — Update Risk Register R-002 with revised scores if warranted.
- [ ] **Step 5.3** — Evaluate whether mandatory MFA enrollment should be extended to all customers (not just admin accounts).
- [ ] **Step 5.4** — File PIR and retain all logs and evidence for 12 months minimum.

---

## Notification Chain

| Recipient | Trigger | Deadline |
|---|---|---|
| Information Security Team | Detection | Immediate |
| Identity & Access Team | Detection | Immediate |
| CISO / Leadership | Admin account targeted OR 1,000+ attempts | Within 1 hour |
| Legal / Compliance | Customer PII or payment data accessed | Immediate |
| Affected customers | Confirmed account compromise | Within 72 hours |
| Fraud Team | Fraudulent orders detected | Immediate |

---

## References

- NIST SP 800-61r3: Incident Response Recommendations and Considerations for Cybersecurity Risk Management
- ShopCore Incident Response Policy (POL-IR-001)
- ShopCore Password & Authentication Policy (POL-IA-001)
- ShopCore Risk Register — R-002
