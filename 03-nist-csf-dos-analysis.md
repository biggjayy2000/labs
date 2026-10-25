# NIST CSF analysis of a DoS incident

**A two-hour ICMP flood, traced through Identify → Protect → Detect → Respond → Recover**

| | |
|---|---|
| Type | Incident analysis against the NIST Cybersecurity Framework |
| Incident | ICMP flood ("ping flood") through an unconfigured firewall |
| Business impact | Internal services unavailable for roughly two hours |
| Framework | NIST CSF 1.1 — five functions |
| Status | Complete |

> Scenario-based: this incident and its details come from the Google Cybersecurity Professional
> Certificate case study. The analysis below is my own.

## What happened

A malicious actor found a firewall that had not been configured to restrict or filter ICMP traffic
and used it to push a high volume of ICMP packets into the internal network. The flood consumed
network resources, and legitimate users could not reach internal services for about two hours.

The response team blocked inbound ICMP, temporarily disabled non-critical services, and restored
critical services to resume operations.

That is a reasonable emergency response — and it also describes a reactive posture, because the
attack was only noticed once services had already failed.

## The analysis, function by function

I went through all five functions rather than jumping to the fix, because the interesting output of a
framework is the shape of what was skipped.

### Identify

**Root cause:** an unconfigured firewall. Not a missing firewall — a firewall whose rules had never
been set to filter ICMP.

Recommendations: routine firewall configuration audits for overly permissive rules, a maintained
inventory of assets and exposed entry points, periodic privilege reviews to confirm least privilege,
and vulnerability scanning to surface misconfigurations before they are used.

The lesson worth carrying: a firewall is a control, not a guarantee. Its effectiveness is entirely a
function of its configuration, which makes configuration management an ongoing process rather than a
one-time setup step.

### Protect

Recommendations: rate-limit inbound ICMP so no single source can overwhelm the network while keeping
ICMP's legitimate diagnostic use, verify source IPs to catch the spoofing that flood attacks rely on,
establish firewall configuration standards and change management, and train staff to recognise and
escalate early indicators.

Rate limiting rather than an outright block is deliberate. It addresses the actual threat — volume —
without removing the protocol everyone uses to troubleshoot. Controls should be proportional to risk,
not maximally restrictive by default.

### Detect

Recommendations: monitor network traffic and establish a baseline so deviations are visible, deploy
IDS/IPS to filter ICMP with suspicious characteristics (abnormal volume, malformed packets,
single-source spikes), and alert in real time.

In the incident, detection happened when services failed — the users were the monitoring. A baseline
plus alerting shortens the gap between the first packet and the response, which is the entire
difference between an incident and an outage.

### Respond

The team's actions — blocking the traffic, shedding non-critical services, restoring critical ones
first — followed sound practice. What was missing was the paperwork that makes it repeatable: a
documented DoS response plan with escalation paths, communication protocols, and pre-assigned roles
for traffic mitigation, stakeholder communication and technical investigation.

Every minute spent deciding what to do during an active incident is a minute of extended business
impact.

### Recover

Recommendations: verify full restoration of critical services, confirm data integrity rather than
assuming it, update the firewall configuration that allowed the flood, communicate resolution to
stakeholders, and feed the lessons back into Identify.

Recovery is not finished when services are back online. It is finished when the root cause cannot
produce the same incident again — which closes the loop back to the start of the framework.

## What this demonstrates

Applying a framework as an analytical tool rather than a diagram: using the five functions to find the
gaps an incident exposes, writing recommendations that a team could implement, and reasoning about
trade-offs (rate limiting over blocking, proactive detection over outage-driven detection) instead of
reaching for the most restrictive control available.

---

*Full analysis document: [`NIST CSF Analysis of a DoS Attack`](https://docs.google.com/document/d/18aVa-0AJ-9QI2JFAupR3QR4i_NQruKuLrJ3wx0FQzmo/edit)
in the Drive archive.*
