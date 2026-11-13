# Incident Response Runbook — ShopCore E-Commerce Application

**Author:** Danielle Law  
**Framework:** NIST SP 800-61r3 — Incident Response Recommendations and Considerations for Cybersecurity Risk Management  
**System:** ShopCore (fictional e-commerce web application)  
**Status:** Complete

---

## About This Project

This project contains step-by-step incident response runbooks for ShopCore. A runbook is the operational document responders follow during an active incident — it translates the high-level Incident Response Policy into concrete, numbered actions with decision points, timelines, and notification chains.

These runbooks are built on risks identified in the companion [Risk Register](https://github.com/danielletjlaw/risk-register-lab) and enforce the procedures defined in the [Incident Response Policy](https://github.com/danielletjlaw/security-policies-lab/blob/main/incident-response-policy.md).

---

## Runbooks in This Repo

| File | Incident Type | Source Risk |
|---|---|---|
| `runbook-ransomware.md` | Ransomware / Destructive Malware | R-004 |
| `runbook-credential-stuffing.md` | Credential Stuffing / Account Takeover | R-002 |
| `runbook-data-breach.md` | Confirmed Data Breach (PII / Payment Data) | R-001, R-003 |

---

## How to Use These Runbooks

Each runbook follows the NIST SP 800-61r3 incident response lifecycle:

```
Detection & Analysis → Containment → Eradication → Recovery → Post-Incident Activity
```

Each phase has numbered steps. Steps marked **[DECISION]** require a judgment call — the responder must assess conditions before proceeding. Steps marked **[NOTIFY]** require an escalation or external communication action.

---

## Incident Severity Levels (from IR Policy)

| Level | Description |
|---|---|
| P1 — Critical | Active compromise with ongoing data loss or system unavailability |
| P2 — High | Confirmed breach or significant compromise, contained |
| P3 — Medium | Suspected compromise under investigation |
| P4 — Low | Minor anomaly with no confirmed impact |

---

## Notification Chain

| Recipient | When |
|---|---|
| Information Security Team | Immediately on any suspected incident |
| IT Operations | Immediately on P1 or P2 |
| CISO / Leadership | Within 1 hour of P1 or P2 confirmation |
| Legal / Compliance | Immediately if customer PII or payment data may be involved |
| Affected Customers | Within 72 hours of confirmed breach (or per regulatory requirement) |

---

## Relationship to Other Portfolio Projects

| Project | Relationship |
|---|---|
| [Risk Register](https://github.com/danielletjlaw/risk-register-lab) | Identifies the threats these runbooks respond to |
| [Security Policies](https://github.com/danielletjlaw/security-policies-lab) | The IR Policy sets the rules; this runbook operationalizes them |

---

*This is a fictional scenario created for portfolio purposes. No real customer data or systems are involved.*
