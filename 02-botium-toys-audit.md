# Botium Toys — internal security audit

**Scope, risk rating and remediation plan for a simulated retail organisation**

| | |
|---|---|
| Type | Internal security audit simulation |
| Sector | Retail / e-commerce |
| Scope | Whole security program — assets, controls, processes, compliance |
| Overall risk score | **8 / 10 — High** |
| Status | Complete |

> The company is fictional: this audit works through the Botium Toys scenario from the Google
> Cybersecurity Professional Certificate, using its published asset inventory, control categories and
> risk assessment. The method is the audit; the organisation is a training case.

## Objective

Produce what an internal auditor produces: not a list of opinions, but a scored picture of where the
organisation stands, which gaps carry the most risk, which controls already work, and what to fix
first. The deliverable is a risk rating per finding and a prioritised plan a manager can act on.

## Method

1. Read the asset inventory and the control assessment to establish what exists and what is claimed.
2. Rate each control area for risk, using what is present versus what the control is supposed to
   achieve.
3. Check the findings against obligations that follow from the data being handled — payment card
   data and PII.
4. Order the recommendations by risk and by dependency, so the first one does not require the last.

## Findings

| Area | Risk | Current condition |
|---|---|---|
| Backup & recovery | **Critical** | Critical data is not backed up and there is no disaster recovery plan |
| Asset management | High | Assets are not fully identified, classified, or assessed for business impact |
| Access control | High | All employees can reach internally stored data; no least privilege, no separation of duties |
| Data encryption | High | Customer card data is not encrypted at rest or in transit |
| Password security | High | Requirements are weak and not centrally enforced |
| Intrusion detection | High | No IDS in place |
| Legacy systems | Medium | Monitored, but no regular maintenance schedule |
| Physical security | Low | Locks, CCTV, fire detection and prevention all present and working |
| Firewall | Low | Configured to block traffic by rules |
| Antivirus | Low | Installed and regularly monitored |

Backup and recovery is the only Critical, and it earns that rating for a reason that is easy to
underrate: encryption protects confidentiality, access control protects confidentiality, but neither
helps when the data stops existing. No backups plus no recovery plan means a ransomware event or a
dead disk is a business-ending event rather than an outage.

## Why the Highs are grouped together

Four of the five High findings are one story told from different angles. All employees can reach all
internal data; the card data in that store is unencrypted; the password policy that guards the door
is weak and unenforced; and nothing watches for unauthorised access once the door is open. Any single
one of those is a problem. Together they mean a single compromised account reaches live payment data,
and nobody finds out while it is happening.

## Compliance

The organisation handles payment card data and customer PII, which pulls in obligations rather than
choices:

- **PCI DSS** for cardholder data — the unencrypted card storage is the most direct gap.
- **GDPR** for applicable E.U. customers — breach notification within 72 hours and privacy procedures
  are already in place and should be maintained, not rebuilt.

A useful control that already exists: documented 72-hour breach notification for E.U. customers.
Auditing is as much about not breaking what works as it is about closing gaps.

## Priority recommendations

1. Implement least privilege and separation of duties.
2. Encrypt customer payment and sensitive personal data, at rest and in transit.
3. Strengthen authentication: central password policy, MFA, centralised password management.
4. Create regular, tested backups of critical data.
5. Develop and test a disaster recovery plan.
6. Deploy intrusion detection and security monitoring.
7. Build a complete asset inventory with classification.
8. Establish a maintenance process for legacy systems.
9. Continue reviewing PCI DSS and GDPR obligations.
10. Provide ongoing security awareness training.

The order matters. Encryption (2) without access control (1) still leaves every employee holding the
keys; backups (4) are worth little without a tested restore (5).

## Overall assessment

The organisation has real foundations — firewall, monitored antivirus, working physical controls,
data integrity controls and privacy procedures. The risk score of 8/10 is High because the
confidentiality and availability gaps are large and interlocking, not because nothing is in place.

## What this demonstrates

Turning an asset inventory and a control checklist into a scored risk register, writing findings that
name the business consequence rather than the missing product, and sequencing remediation so the plan
can actually be executed.

---

*Full audit document: [`Botium Toys — Internal Security Audit`](https://docs.google.com/document/d/1dzeY8OaihVyF70mCqvVJsee8t9fB-S9uZeVGAXUsKHg/edit)
in the Drive archive.*
