# Requirements for Software Security Engineering

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
## 1. The Five Essential Interactions of Defectdojo.

**CI/CD pipline imports scan results through the API**
![CI/CD pipline imports scan results through the API](./diagrams/CICD_usecase.png)
This is how nearly all data enters DefectDojo. Reports can be uploaded either directly through the UI or through an API endpoint that allows automated ingestion. API calls authenticate with a header containing the user's API key. In the open-source edition, auto-creating the Organization/Asset/Engagement/Test context during import is only available through the API; UI imports need the hierarchy to exist first. That means a pipeline token can create structure, not just findings, which is a useful point for the misuse cases.

**Security analyst triages findings and records risk acceptance**
![Security analyst triages findings and records risk acceptance](./diagrams/Analyst_Triage_usecase.png)
This is the core decision workflow, and it is exactly what your "integrity compromise" threat targets. Besides active states, a finding can be marked Duplicate, Mitigated, False Positive, Out Of Scope, Risk Accepted, or Under Defect Review. Risk Acceptances can carry uploaded files and notes as justification, and they have expiry dates that force re-evaluation. Each Asset can have its own SLA configuration, which sets the number of days allowed to remediate a finding.

**Developer or product owner views and remediates findings for their own asset**
![Developer or product owner views and remediates findings for their own asset](./diagrams/Developer_remediation_usecase.png)
This interaction carries your "cross-application disclosure" threat (App A's developer seeing App B), and that threat maps to DefectDojo's authorization boundary. An Asset's Authorized Users list grants access to that Asset and everything nested beneath it, while an Organization's list cascades to every Asset underneath. Open source now offers only local username/password login plus the password-reset flow.

**DefectDojo administrator manages users and access**
![DefectDojo administrator manages users and access](./diagrams/Admin_acess_usecase.png)
This maps to your "privilege escalation" threat. Superuser and staff accounts can see and act on every Asset and Organization regardless of the Authorized Users lists, and only those accounts get the controls to change the lists. The first account on a fresh install is automatically a superuser. Admins can also harden the API surface: API tokens can be turned off entirely with DD_API_TOKENS_ENABLED=False, or only the api-token-auth endpoint can be disabled with DD_API_TOKEN_AUTH_ENDPOINT_ENABLED=False.

**DefectDojo pushes findings to Jira**
![DefectDojo pushes findings to Jira](./diagrams/Jira_usecase.png)
This is the one interaction where DefectDojo holds another system's credentials and accepts inbound calls, so it carries your "credential theft" threat. The integration is gated by a System Settings toggle; while it is off, DefectDojo won't push findings (including push_to_jira API requests) and ignores incoming Jira webhooks. The troubleshooting page gives the open-source path as Configuration > System Settings > Enable JIRA integration, and notes that pushes run asynchronously. Those asynchronous pushes run on your Celery Workers box.

---

## 2. For each use case, derive security requirements using misuse case analysis.

---

## 3. Iterate between use and misuse cases.

---

## 4. Build a list of security requirements derived from misuse case analysis.

---

## 5. Assess the alignment of security requirements

---
# Part 2
## 1. Review your OSS project documentation specifically for security-related configuration and installation issues. Summarize your observations about what could be improved or is missing.
