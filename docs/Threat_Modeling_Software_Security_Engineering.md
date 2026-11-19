# Threat Modeling for Software Security Engineering

**Team 5**

| Member | Role |
|---|---|
| Ray Donner | Contributor |
| Grant Nichter | Team Lead |
| Alex Bahwawsi | Contributor |
| Chandler Crist | Contributor |

**Team GitHub Repository:** [github.com/creepysmilejpg/SWADjangoProject](https://github.com/creepysmilejpg/SWADjangoProject)

Task assignments, discussion, and progress tracking for this project are managed through the repository's GitHub Project Board, where issues are linked to board cards and closed only after review/comment.

---

# Part 1
## 1. Data flow diagrams for critical data flows

The DFDs below support the five scenarios prioritized in the Security Requirements and Assurance Case assignments. They were drawn in the Microsoft Threat Modeling Tool (TMT) using the Azure Threat Model Template. One Level 1 diagram shows the DefectDojo container architecture across all five scenarios. Two Level 2 diagrams decompose the Django application into the components that enforce authentication, authorization, auditing, and the Jira integration.

| Scenario | Interaction (from Requirements assignment) | Assurance claim | Covered by |
|---|---|---|---|
| S1 | CI/CD pipeline imports scan results through the API | Claim 1 | Level 1, Level 2 (Import, Triage, Remediation) |
| S2 | Security analyst triages findings and records risk acceptance | Claim 2 | Level 1, Level 2 (Import, Triage, Remediation) |
| S3 | Developer views and remediates findings for their own asset | Claim 3 | Level 1, Level 2 (Import, Triage, Remediation) |
| S4 | Administrator manages users and access | Claim 4 | Level 1, Level 2 (Administration and Jira) |
| S5 | DefectDojo pushes findings to Jira | Claim 5 | Level 1, Level 2 (Administration and Jira) |

**Trust boundaries used in the DFDs**

| Trust boundary | What it separates |
|---|---|
| Corporate Network | Human users (analysts, developers, administrators) and their browsers |
| CI/CD Platform | The pipeline and security scanners that call the DefectDojo API |
| DefectDojo Docker Host | The containers of the default Docker Compose deployment; only nginx publishes ports |
| Django Application | The uWSGI process that performs authentication, authorization, and business logic (Level 2 only) |
| Jira (external) | The Atlassian Jira service that receives pushed findings and sends webhooks |

### Level 1: DefectDojo System

> **TODO:** Export the "Level 1 - DefectDojo System" tab from TMT (right-click the canvas, then **Export Image**, or take a screenshot) and save it as `docs/diagrams/DFD_Level1_System.png`.

![Level 1 DFD: DefectDojo system](./diagrams/DFD_Level1_System.png)

The Level 1 DFD shows the five external interactors reaching DefectDojo through nginx (TLS termination), the Django application reading and writing PostgreSQL, asynchronous work passing through the Valkey task queue to the Celery worker, and the Celery worker exchanging data with Jira. The Jira webhook is the only flow that enters DefectDojo from an external service without a user session.

> **TODO (optional):** Add two or three sentences on anything your team wants to highlight in this diagram.

### Level 2: Scan Import, Triage, and Remediation (S1-S3)

> **TODO:** Export the "Level 2 - Import, Triage, Remediation" tab and save it as `docs/diagrams/DFD_Level2_Import_Triage.png`.

![Level 2 DFD: import, triage, and remediation](./diagrams/DFD_Level2_Import_Triage.png)

Every request passes through Authentication (session cookie or API token, checked against Users & API Tokens) and then Authorization (checked against the Authorized Users lists) before reaching Findings Views or the Import Engine. Triage changes made in Findings Views are written to the Findings DB and recorded in the Audit Log.

### Level 2: Administration and Jira Integration (S4-S5)

> **TODO:** Export the "Level 2 - Administration and Jira" tab and save it as `docs/diagrams/DFD_Level2_Admin_Jira.png`.

![Level 2 DFD: administration and Jira integration](./diagrams/DFD_Level2_Admin_Jira.png)

Administrator requests pass through Authentication and a staff/superuser Authorization check before reaching User & Access Management or Jira Config. Two design points from earlier assignments are visible here. Access-list changes are written to Authorized Users without a corresponding audit event. The Jira webhook reaches the Webhook Handler without passing through Authentication, so it is protected only by the shared secret in the URL.

---

## 2. Threat analysis using the Microsoft Threat Modeling Tool

### Method

> **TODO:** Fill in the numbers after running **View → Analysis View** in TMT.

Threats were generated automatically by TMT's Analysis View for the Level 1 and Level 2 diagrams. Each threat was reviewed and assigned one of three states:

- **Mitigated:** DefectDojo or the deployment already addresses the threat; the justification names the control.
- **Needs Investigation:** the threat is relevant and not fully addressed; a mitigation strategy is documented.
- **Not Applicable:** the threat does not apply to this deployment; the justification explains why.

| Diagram | Threats generated | Mitigated | Needs Investigation | Not Applicable |
|---|---|---|---|---|
| Level 1 - DefectDojo System | TODO | TODO | TODO | TODO |
| Level 2 - Import, Triage, Remediation | TODO | TODO | TODO | TODO |
| Level 2 - Administration and Jira | TODO | TODO | TODO | TODO |
| **Total** | **TODO** | **TODO** | **TODO** | **TODO** |

| STRIDE category | Count |
|---|---|
| Spoofing | TODO |
| Tampering | TODO |
| Repudiation | TODO |
| Information Disclosure | TODO |
| Denial of Service | TODO |
| Elevation of Privilege | TODO |

### Threats on flows that cross trust boundaries

These flows cross a trust boundary in the DFDs, so their threats are reviewed first and in the most detail.

| Diagram | Flow | Boundary crossed | Why it matters |
|---|---|---|---|
| Level 1 | 1-2 Scan + API token / Import result | CI/CD Platform → Docker Host | A stolen pipeline token can import or alter findings (Claim 1) |
| Level 1 | 3-6 User and admin requests | Corporate Network → Docker Host | Session theft or credential guessing against local accounts; no MFA in OSS |
| Level 1 | 15-16 Create issue / Issue status | Docker Host ↔ Jira | Jira credentials leave DefectDojo; responses are trusted |
| Level 1 | 17 Webhook | Jira → Docker Host | Unauthenticated inbound call protected only by a URL secret |
| Level 2 (Import) | 5 Cookie / token | nginx → Django Application | Entry point for all authentication decisions |
| Level 2 (Import) | 11-14 Triage writes, Finding data, Change history, New findings | Django Application ↔ data stores | Integrity of findings and completeness of the audit trail (Claim 2) |
| Level 2 (Admin) | 7-9 User changes, Access changes, Audit events | Django Application → data stores | Access-list changes are not audited (Claim 4) |
| Level 2 (Admin) | 11-12 Jira creds, secret | Django Application → Jira Settings → Push Worker | Jira password stored unencrypted (Claim 5) |
| Level 2 (Admin) | 16-17 Webhook / Webhook (no login) | Jira → nginx → Webhook Handler | Bypasses Authentication entirely (Claim 5) |

> **TODO:** For each threat TMT generated on the flows above, add a row below. Use the threat ID and title exactly as shown in TMT's Threat List or the HTML report.

| TMT ID | Threat title | STRIDE | Flow / element | State | Mitigation strategy |
|---|---|---|---|---|---|
| TODO | TODO | TODO | TODO | TODO | TODO |
| TODO | TODO | TODO | TODO | TODO | TODO |

### Other relevant threats

> **TODO:** Add the remaining threats your team marked as Mitigated or Needs Investigation. These are threats on flows inside a single boundary. Group them by diagram.

| TMT ID | Threat title | STRIDE | Flow / element | State | Mitigation strategy |
|---|---|---|---|---|---|
| TODO | TODO | TODO | TODO | TODO | TODO |

### Threats marked Not Applicable

> **TODO:** List each Not Applicable threat with a one-line reason (for example, an Azure-specific threat that does not apply to a Docker Compose deployment).

| TMT ID | Threat title | Reason not applicable |
|---|---|---|
| TODO | TODO | TODO |

---

## 3. Mitigation strategies

> **TODO:** Summarize the mitigation strategies from Section 2 into a short list. Prefer controls that DefectDojo implements; note where a control is only available as configuration, in DefectDojo Pro, or in the hosting environment.

| # | Mitigation strategy | Threats addressed (TMT IDs) | Provided by | Related claim |
|---|---|---|---|---|
| M1 | TODO | TODO | DefectDojo / Configuration / Pro / Environment | TODO |
| M2 | TODO | TODO | TODO | TODO |
| M3 | TODO | TODO | TODO | TODO |

> **TODO:** Add a short paragraph comparing these mitigations with the gaps found in earlier assignments (no role separation, unaudited access-list changes, plaintext Jira password, no MFA in OSS).

---

## 4. TMT HTML report

> **TODO:**
> 1. In TMT, after all threats have a state and justification, run **Reports → Create Full Report** and save it as `DefectDojo_Threat_Model_Report.htm`.
> 2. Commit it to `docs/reports/DefectDojo_Threat_Model_Report.htm`, and commit the saved model as `docs/reports/DefectDojo_Threat_Model.tm7`.
> 3. The preview link below works once the file is pushed to the `main` branch.

**Full TMT report (rendered):** [DefectDojo Threat Model Report](https://htmlpreview.github.io/?https://github.com/creepysmilejpg/SWADjangoProject/blob/main/docs/reports/DefectDojo_Threat_Model_Report.htm)

**Source files:** [HTML report](./reports/DefectDojo_Threat_Model_Report.htm) | [TMT model (.tm7)](./reports/DefectDojo_Threat_Model.tm7)

---

## 5. Team GitHub repository

> **TODO:** Describe how this assignment was tracked on the Project Board (who owned which diagram or set of threats, and how cards were reviewed before being closed). Add the board link if the assignment asks for it.

---

## 6. Reflection

### Individual reflections

> **TODO:** Each member writes a short paragraph: what did you learn from this assignment, and what did you find most useful?

**Grant Nichter.** TODO

**Ray Donner.** TODO

**Alex Bahwawsi.** TODO

**Chandler Crist.** TODO

### Team reflection

> **TODO:** Combine the individual reflections into one paragraph for the team.
