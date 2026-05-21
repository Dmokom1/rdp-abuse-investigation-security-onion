# Build Notes
# Investigating RDP Abuse and Rogue Host Activity in Security Onion

This file provides supporting build context for the main README. It documents the investigation sequence, alert evidence, source-attribution notes, limitations, and screenshot mapping.

The README explains the full project story. These notes focus on how the investigation was reviewed and what the screenshots support.

---

## Purpose of This File

This project was built to practice defender-side investigation of RDP-related and scan-related activity in Security Onion.

These build notes focus on:

- RDP alert review
- Scan-related alert review
- Source and destination correlation
- Repeated alert-family analysis
- Virtual lab source-attribution challenges
- Evidence interpretation
- Screenshot-supported findings

---

## Lab Environment

| Component | Details |
|---|---|
| SIEM | Security Onion |
| Detection Engine | Suricata |
| Analysis Interface | Security Onion Alerts / Hunt |
| Domain Controller | `192.168.30.128` |
| Suspicious Source Observed | `192.168.30.1` |
| RDP Port | `3389` |
| Related Scan Port Observed | `5985` |

---

## Corrected Investigation Sequence

The final investigation workflow followed this order:

1. Reviewed RDP response alerts in Security Onion.
2. Identified repeated RDP traffic involving the domain controller.
3. Reviewed Nmap-related alert drilldown details.
4. Identified the suspicious source IP shown in alert telemetry.
5. Reviewed repeated Remote Desktop administrator login request alerts.
6. Compared RDP-related alerts with scan-related alerts.
7. Correlated repeated source and destination patterns.
8. Documented the source-attribution issue caused by virtual lab networking behavior.
9. Interpreted the activity from a defender’s point of view.

---

## Important Values and Alert Names

| Item | Value |
|---|---|
| Domain Controller IP | `192.168.30.128` |
| Suspicious Source IP Observed | `192.168.30.1` |
| RDP Port | `3389` |
| WinRM / HTTP Management Port Observed | `5985` |
| RDP Response Alert | `ET INFO RDP - Response To External Host` |
| Remote Desktop Login Alert | `ET REMOTE_ACCESS MS Remote Desktop Administrator Login Request` |
| Nmap Alert | `ET SCAN Nmap Scripting Engine User-Agent Detected` |
| Additional Scan Alert | `ET SCAN Possible Nmap User-Agent Observed` |
| Behavioral Scan Alert | `ET SCAN Behavioral Unusually fast Terminal Server Traffic Potential Scan or Infection` |
| OS Detection Alert | `ET SCAN NMAP OS Detection Probe` |

---

## Phase 1: RDP Response Activity Review

Security Onion showed repeated RDP response activity involving the domain controller.

### Screenshot Evidence

| Screenshot | What It Supports |
|---|---|
| `Screenshots/01-so-rdp-bruteforce-alerts.png` | RDP response activity involving the domain controller |

### Observation

The alert view showed repeated events named:

`ET INFO RDP - Response To External Host`

The visible traffic pattern showed:

- Source IP: `192.168.30.128`
- Source port: `3389`
- Destination IP: `192.168.30.1`

This supports the presence of RDP communication involving the domain controller.

This screenshot does not prove successful authentication or confirmed compromise.

---

## Phase 2: Nmap-Related Alert Drilldown

Security Onion also showed Nmap-related detections tied to the suspicious source.

### Screenshot Evidence

| Screenshot | What It Supports |
|---|---|
| `Screenshots/02-so-alert-drilldown-source-ip.png` | Nmap-related alert drilldown showing source and destination details |

### Observation

The alert drilldown showed:

- Source IP: `192.168.30.1`
- Destination IP: `192.168.30.128`
- Destination port: `5985`
- Severity: High

The visible rule name was:

`ET SCAN Nmap Scripting Engine User-Agent Detected (Nmap Scripting Engine)`

This supported the finding that the suspicious source was associated with scan-related traffic against the domain controller.

---

## Phase 3: Remote Desktop Login Request Pattern

Security Onion showed repeated Remote Desktop administrator login request alerts.

### Screenshot Evidence

| Screenshot | What It Supports |
|---|---|
| `Screenshots/03-rdp-alerts-same-source.png` | Repeated Remote Desktop login request activity |

### Observation

The alert view showed repeated events named:

`ET REMOTE_ACCESS MS Remote Desktop Administrator Login Request`

The visible traffic pattern showed:

- Source IP: `192.168.30.1`
- Destination IP: `192.168.30.128`
- Destination port: `3389`

This supports repeated RDP login request telemetry against the domain controller.

This screenshot does not prove that authentication succeeded.

---

## Phase 4: Correlation Across Alert Families

The grouped alert view showed several related RDP and scan alert families.

### Screenshot Evidence

| Screenshot | What It Supports |
|---|---|
| `Screenshots/04-correlated-scan-alerts-same-source.png` | Multiple alert families tied to related RDP and scan behavior |

### Observation

The grouped alert view showed several related alert families, including:

- `ET INFO RDP - Response To External Host`
- `ET REMOTE_ACCESS MS Remote Desktop Administrator Login Request`
- `ET SCAN Nmap Scripting Engine User-Agent Detected`
- `ET SCAN Possible Nmap User-Agent Observed`
- `ET SCAN Behavioral Unusually fast Terminal Server Traffic Potential Scan or Infection`
- `ET SCAN NMAP OS Detection Probe`

The strongest evidence came from the repeated relationship between the suspicious source pattern and the domain controller across multiple alert types.

---

## Source Attribution Notes

The suspicious source appeared as:

`192.168.30.1`

This did not perfectly match earlier expectations from other lab phases where Kali used different addressing.

This was treated as an attribution limitation, not ignored.

In a virtual lab, source representation can be affected by:

- NAT behavior
- Host-only adapter behavior
- Interface changes
- Routing changes
- Virtual network translation
- Earlier manual IP assignment changes

Because of that, the investigation relied on correlation instead of one expected attacker IP.

The strongest evidence path was:

1. Same suspicious source pattern.
2. Same domain controller destination.
3. Same RDP destination port.
4. Multiple related alert families.
5. Repeated event counts across the alert set.

---

## Evidence Interpretation Notes

These notes keep the project explanation accurate:

- The screenshots support RDP-related and scan-related alert telemetry.
- The screenshots do not prove successful login.
- The screenshots do not prove confirmed compromise.
- The source IP should be interpreted carefully because of virtual lab networking behavior.
- The strongest finding came from alert correlation, not host identity enrichment.
- More endpoint telemetry would be needed to confirm authentication success or post-login activity.
- This project should be described as a defender-side investigation, not an exploitation project.

---

## Key Lessons Learned

1. RDP activity can create repeated network alerts in Security Onion.
2. Multiple related alerts are stronger than one isolated alert.
3. Source and destination patterns are important investigation pivots.
4. Virtual lab networking can complicate source attribution.
5. Alert correlation can still provide useful evidence even when enrichment fields are limited.
6. RDP network telemetry does not automatically prove successful authentication.
7. A stronger investigation would combine network alerts with Windows authentication logs and endpoint telemetry.

---

## Improvements for a Future Version

Future improvements could include:

- Adding Windows Security event logs for RDP authentication review.
- Correlating Suricata alerts with Windows Event IDs.
- Capturing endpoint telemetry from the domain controller.
- Documenting the exact Kali IP and adapter state during the test.
- Reviewing whether `192.168.30.1` represented NAT, gateway, host adapter, or another virtual network artifact.
- Capturing packet-level evidence for selected RDP traffic.
- Building a detection rule that groups repeated RDP-related activity by source and destination.
- Creating a timeline that maps scan activity, RDP activity, and alert generation.

---

## Screenshot Map

| Screenshot | What It Supports |
|---|---|
| `Screenshots/01-so-rdp-bruteforce-alerts.png` | RDP response activity involving the domain controller |
| `Screenshots/02-so-alert-drilldown-source-ip.png` | Nmap-related alert drilldown showing source and destination details |
| `Screenshots/03-rdp-alerts-same-source.png` | Repeated Remote Desktop login request activity |
| `Screenshots/04-correlated-scan-alerts-same-source.png` | Multiple alert families tied to related RDP and scan behavior |