Review of automatically identified threats

**Project:** DefectDojo — Team 5  
**Assignment responsibility:** Examine each automatically identified threat, note which are relevant, and document reasonable mitigations  
**Source:** [DefectDojo Threat Modeling Tool report](https://github.com/creepysmilejpg/SWADjangoProject/blob/main/docs/DefectDojo_threat_report.html), and the accompanying Level 1 / Level 2 DFDs  
**Scope:** Issue #12 only. No new DFDs, HTML report exports, or OSS code testing.

## Summary

The TMT report lists **91 generated findings**, all originally marked **Not Started**: **19** for Level 1, **27** for Level 2 Import/Triage/Remediation and **45** for Level 2 Administration/Jira. Many are repeats of the same database threat pattern against different data flows. Each numbered finding appears in the review ledger below. I grouped repeated mitigation decisions to keep this review readable.

**Review decisions:** 81 relevant risk scenarios with proposed mitigations; 10 findings not applicable **as written** because they refer specifically to Azure subscription management not present in the documented Docker/PostgreSQL setup. “Relevant” means the risk deserves a control—not that DefectDojo currently lacks the control. **No mitigation has been verified as implemented**, and this review does not change the model’s saved TMT statuses.

## Decisions and proposed mitigations

| Code | TMT threat pattern / STRIDE | Decision and reason | Proposed mitigation for the modeled stack |
|---|---|---|---|
| **Q** | Valkey queue account / elevation of privilege | **Relevant.** The Docker task queue is an internal service, not an Azure subscription. A stolen or unauthenticated Valkey connection could let an attacker disrupt or influence background work. | Restrict Valkey to the private Docker network; use supported authentication/ACLs, protect service secrets, prevent published ports, and monitor access. Do **not** apply the report’s Azure storage RBAC instructions. |
| **I** | Tampering with database objects / tampering | **Relevant.** Unauthorized modification to findings, user/permission tables, or audit records could compromise integrity. Microsoft’s suggestion to sign SQL Server securables is not a direct fit for PostgreSQL. | Restrict database write privileges, constrain admin access, audit sensitive changes, protect deployment migrations, and keep tested backups. Verify that unauthorized writes are rejected. |
| **T** | Anomalous database activity / tampering | **Relevant.** Unexpected database operations or unusual access patterns may indicate a compromised container or account. Azure SQL threat detection is not part of the assumed deployment. | Collect PostgreSQL, container and application security logs; alert on suspicious access and high-volume or privileged changes; retain logs separately from the database where feasible. |
| **L** | Insufficient auditing / repudiation | **Relevant.** Without reliable attribution, changes to findings, credentials or access lists can be denied or go undetected. The source recommends SQL Server login audit settings, which are not applicable verbatim. | Log successful/failed authentication and sensitive mutations with actor, timestamp and affected object; protect log integrity and retention; review privileged changes. |
| **D** | Exposure of sensitive database records / information disclosure | **Relevant.** Findings, audit data, credentials and tokens may be sensitive. SQL Server TDE/Always Encrypted guidance is not an exact PostgreSQL control. | Restrict access to data and backups; encrypt backup/storage volumes as appropriate; minimize sensitive fields; protect Jira/API tokens using a secrets mechanism; rotate if exposed. |
| **S** | SQL injection into database operations / information disclosure | **Relevant.** Malformed input could expose data or modify records if raw SQL is assembled unsafely. Django ORM reduces this risk but does not rule out unsafe custom SQL. | Use Django ORM/parameterized SQL; avoid constructing queries from untrusted input; validate scan and webhook data; test any raw-SQL paths; keep DB permissions least privilege. |
| **N** | Unrestricted database network access / elevation of privilege | **Relevant.** Exposing PostgreSQL or other internal services beyond their Docker network enables unauthorized connection attempts. Windows Firewall advice does not apply directly. | Keep only nginx publicly reachable; do not publish PostgreSQL/Valkey ports; restrict service networks and inbound host firewall rules; verify from an external host. |
| **P** | Weak database credentials / elevation of privilege | **Relevant.** The report title mentions Azure SQL but its description explicitly references PostgreSQL brute-force attempts. The underlying weak-account risk applies; Azure AD authentication is not assumed. | Use unique strong secrets with limited exposure; rotate compromised credentials; restrict privileged database logins and network reachability; alert on failed login spikes. |
| **A** | Overbroad database authorization / elevation of privilege | **Relevant.** Over-privileged database roles can widen the effects of an app or worker compromise. SQL Server-specific stored-procedure advice and generic tenant RLS guidance may be unnecessary. | Grant PostgreSQL roles only required rights; segregate privileged maintenance identities; use DefectDojo object-level authorization for asset access; test unauthorized reads/writes. |
| **C** | Azure subscription database-account access / elevation of privilege | **Not applicable as written.** The named Azure subscription/storage-account management plane is outside the report’s assumed single-host Docker/PostgreSQL environment. This is a template mismatch, not evidence the database is secure. | Record N/A **only for the Azure subscription version** of this threat. The underlying account abuse is addressed under **P** and **A**; reassess if the team actually hosts its data in Azure. |

**Scope and trust-boundary observation:** The model includes external CI/CD-to-nginx/API interactions, users crossing the corporate-network boundary, and Jira webhooks entering DefectDojo. Those flows deserve attention, especially where tokens, scan data, authorization changes and webhook status changes enter a trusted application. However, the exported *automatic* findings are concentrated on data stores and the task queue. This review does not invent additional TMT-generated finding IDs for the external flows. The existing SR-1–SR-5 requirements address the broader risks.

## Every generated finding (traceability ledger)

Find each entry by its **report finding number**. The decision is specific to the threat as worded in TMT; the **code** points to the full reasoning and mitigation above. This preserves a review of every item without repeating the same recommendation nine or ten times.

| TMT # | Diagram | Data flow / interaction | Category | Decision | Mitigation |
|---:|---|---|---|---|---|
| 1 | L1 | 11. Async tasks | Elevation of privilege | Relevant — mitigation proposed | **Q** |
| 2 | L1 | 13. Task results | Tampering | Relevant — mitigation proposed | **I** |
| 3 | L1 | 13. Task results | Tampering | Relevant — mitigation proposed | **T** |
| 4 | L1 | 13. Task results | Repudiation | Relevant — mitigation proposed | **L** |
| 5 | L1 | 13. Task results | Information disclosure | Relevant — mitigation proposed | **D** |
| 6 | L1 | 13. Task results | Information disclosure | Relevant — mitigation proposed | **S** |
| 7 | L1 | 13. Task results | Elevation of privilege | Relevant — mitigation proposed | **N** |
| 8 | L1 | 13. Task results | Elevation of privilege | Relevant — mitigation proposed | **P** |
| 9 | L1 | 13. Task results | Elevation of privilege | Relevant — mitigation proposed | **A** |
| 10 | L1 | 13. Task results | Elevation of privilege | Not applicable — Azure only | **C** |
| 11 | L1 | 9. Queries, writes | Tampering | Relevant — mitigation proposed | **I** |
| 12 | L1 | 9. Queries, writes | Tampering | Relevant — mitigation proposed | **T** |
| 13 | L1 | 9. Queries, writes | Repudiation | Relevant — mitigation proposed | **L** |
| 14 | L1 | 9. Queries, writes | Information disclosure | Relevant — mitigation proposed | **D** |
| 15 | L1 | 9. Queries, writes | Information disclosure | Relevant — mitigation proposed | **S** |
| 16 | L1 | 9. Queries, writes | Elevation of privilege | Relevant — mitigation proposed | **N** |
| 17 | L1 | 9. Queries, writes | Elevation of privilege | Relevant — mitigation proposed | **P** |
| 18 | L1 | 9. Queries, writes | Elevation of privilege | Relevant — mitigation proposed | **A** |
| 19 | L1 | 9. Queries, writes | Elevation of privilege | Not applicable — Azure only | **C** |
| 20 | L2-Import | 11. Triage writes | Tampering | Relevant — mitigation proposed | **I** |
| 21 | L2-Import | 11. Triage writes | Tampering | Relevant — mitigation proposed | **T** |
| 22 | L2-Import | 11. Triage writes | Repudiation | Relevant — mitigation proposed | **L** |
| 23 | L2-Import | 11. Triage writes | Information disclosure | Relevant — mitigation proposed | **D** |
| 24 | L2-Import | 11. Triage writes | Information disclosure | Relevant — mitigation proposed | **S** |
| 25 | L2-Import | 11. Triage writes | Elevation of privilege | Relevant — mitigation proposed | **N** |
| 26 | L2-Import | 11. Triage writes | Elevation of privilege | Relevant — mitigation proposed | **P** |
| 27 | L2-Import | 11. Triage writes | Elevation of privilege | Relevant — mitigation proposed | **A** |
| 28 | L2-Import | 11. Triage writes | Elevation of privilege | Not applicable — Azure only | **C** |
| 29 | L2-Import | 13. Change history | Tampering | Relevant — mitigation proposed | **I** |
| 30 | L2-Import | 13. Change history | Tampering | Relevant — mitigation proposed | **T** |
| 31 | L2-Import | 13. Change history | Repudiation | Relevant — mitigation proposed | **L** |
| 32 | L2-Import | 13. Change history | Information disclosure | Relevant — mitigation proposed | **D** |
| 33 | L2-Import | 13. Change history | Information disclosure | Relevant — mitigation proposed | **S** |
| 34 | L2-Import | 13. Change history | Elevation of privilege | Relevant — mitigation proposed | **N** |
| 35 | L2-Import | 13. Change history | Elevation of privilege | Relevant — mitigation proposed | **P** |
| 36 | L2-Import | 13. Change history | Elevation of privilege | Relevant — mitigation proposed | **A** |
| 37 | L2-Import | 13. Change history | Elevation of privilege | Not applicable — Azure only | **C** |
| 38 | L2-Import | 14. New findings | Tampering | Relevant — mitigation proposed | **I** |
| 39 | L2-Import | 14. New findings | Tampering | Relevant — mitigation proposed | **T** |
| 40 | L2-Import | 14. New findings | Repudiation | Relevant — mitigation proposed | **L** |
| 41 | L2-Import | 14. New findings | Information disclosure | Relevant — mitigation proposed | **D** |
| 42 | L2-Import | 14. New findings | Information disclosure | Relevant — mitigation proposed | **S** |
| 43 | L2-Import | 14. New findings | Elevation of privilege | Relevant — mitigation proposed | **N** |
| 44 | L2-Import | 14. New findings | Elevation of privilege | Relevant — mitigation proposed | **P** |
| 45 | L2-Import | 14. New findings | Elevation of privilege | Relevant — mitigation proposed | **A** |
| 46 | L2-Import | 14. New findings | Elevation of privilege | Not applicable — Azure only | **C** |
| 47 | L2-Admin | 11. Jira creds, secret | Tampering | Relevant — mitigation proposed | **I** |
| 48 | L2-Admin | 11. Jira creds, secret | Tampering | Relevant — mitigation proposed | **T** |
| 49 | L2-Admin | 11. Jira creds, secret | Repudiation | Relevant — mitigation proposed | **L** |
| 50 | L2-Admin | 11. Jira creds, secret | Information disclosure | Relevant — mitigation proposed | **D** |
| 51 | L2-Admin | 11. Jira creds, secret | Information disclosure | Relevant — mitigation proposed | **S** |
| 52 | L2-Admin | 11. Jira creds, secret | Elevation of privilege | Relevant — mitigation proposed | **N** |
| 53 | L2-Admin | 11. Jira creds, secret | Elevation of privilege | Relevant — mitigation proposed | **P** |
| 54 | L2-Admin | 11. Jira creds, secret | Elevation of privilege | Relevant — mitigation proposed | **A** |
| 55 | L2-Admin | 11. Jira creds, secret | Elevation of privilege | Not applicable — Azure only | **C** |
| 56 | L2-Admin | 19. Status sync | Tampering | Relevant — mitigation proposed | **I** |
| 57 | L2-Admin | 19. Status sync | Tampering | Relevant — mitigation proposed | **T** |
| 58 | L2-Admin | 19. Status sync | Repudiation | Relevant — mitigation proposed | **L** |
| 59 | L2-Admin | 19. Status sync | Information disclosure | Relevant — mitigation proposed | **D** |
| 60 | L2-Admin | 19. Status sync | Information disclosure | Relevant — mitigation proposed | **S** |
| 61 | L2-Admin | 19. Status sync | Elevation of privilege | Relevant — mitigation proposed | **N** |
| 62 | L2-Admin | 19. Status sync | Elevation of privilege | Relevant — mitigation proposed | **P** |
| 63 | L2-Admin | 19. Status sync | Elevation of privilege | Relevant — mitigation proposed | **A** |
| 64 | L2-Admin | 19. Status sync | Elevation of privilege | Not applicable — Azure only | **C** |
| 65 | L2-Admin | 7. User changes | Tampering | Relevant — mitigation proposed | **I** |
| 66 | L2-Admin | 7. User changes | Tampering | Relevant — mitigation proposed | **T** |
| 67 | L2-Admin | 7. User changes | Repudiation | Relevant — mitigation proposed | **L** |
| 68 | L2-Admin | 7. User changes | Information disclosure | Relevant — mitigation proposed | **D** |
| 69 | L2-Admin | 7. User changes | Information disclosure | Relevant — mitigation proposed | **S** |
| 70 | L2-Admin | 7. User changes | Elevation of privilege | Relevant — mitigation proposed | **N** |
| 71 | L2-Admin | 7. User changes | Elevation of privilege | Relevant — mitigation proposed | **P** |
| 72 | L2-Admin | 7. User changes | Elevation of privilege | Relevant — mitigation proposed | **A** |
| 73 | L2-Admin | 7. User changes | Elevation of privilege | Not applicable — Azure only | **C** |
| 74 | L2-Admin | 8. Access changes | Tampering | Relevant — mitigation proposed | **I** |
| 75 | L2-Admin | 8. Access changes | Tampering | Relevant — mitigation proposed | **T** |
| 76 | L2-Admin | 8. Access changes | Repudiation | Relevant — mitigation proposed | **L** |
| 77 | L2-Admin | 8. Access changes | Information disclosure | Relevant — mitigation proposed | **D** |
| 78 | L2-Admin | 8. Access changes | Information disclosure | Relevant — mitigation proposed | **S** |
| 79 | L2-Admin | 8. Access changes | Elevation of privilege | Relevant — mitigation proposed | **N** |
| 80 | L2-Admin | 8. Access changes | Elevation of privilege | Relevant — mitigation proposed | **P** |
| 81 | L2-Admin | 8. Access changes | Elevation of privilege | Relevant — mitigation proposed | **A** |
| 82 | L2-Admin | 8. Access changes | Elevation of privilege | Not applicable — Azure only | **C** |
| 83 | L2-Admin | 9. Audit events | Tampering | Relevant — mitigation proposed | **I** |
| 84 | L2-Admin | 9. Audit events | Tampering | Relevant — mitigation proposed | **T** |
| 85 | L2-Admin | 9. Audit events | Repudiation | Relevant — mitigation proposed | **L** |
| 86 | L2-Admin | 9. Audit events | Information disclosure | Relevant — mitigation proposed | **D** |
| 87 | L2-Admin | 9. Audit events | Information disclosure | Relevant — mitigation proposed | **S** |
| 88 | L2-Admin | 9. Audit events | Elevation of privilege | Relevant — mitigation proposed | **N** |
| 89 | L2-Admin | 9. Audit events | Elevation of privilege | Relevant — mitigation proposed | **P** |
| 90 | L2-Admin | 9. Audit events | Elevation of privilege | Relevant — mitigation proposed | **A** |
| 91 | L2-Admin | 9. Audit events | Elevation of privilege | Not applicable — Azure only | **C** |

## Conclusions

1. **Highest-priority risks:** access to sensitive findings, API tokens, Jira integration secrets, unauthorized changes to findings/users, and database connectivity. Apply least privilege at the application, container/network and PostgreSQL layers.
2. **Logging and validation matter:** DefectDojo activity history, PostgreSQL logging, targeted authorization tests and backups are more appropriate here than the report’s generic SQL Server/Azure instructions.
3. **Template mismatches:** 10 Azure-subscription findings are N/A for the stated Docker deployment. Azure SQL threat detection, Windows Firewall, SQL Server TDE and other platform-specific suggestions should not be presented as implemented DefectDojo controls.
4. **No implementation claim:** This is a threat-review deliverable, not evidence that the mitigations already exist or work. The model owner may record the decisions in TMT if the team requires the tool states to be updated.

**Reference requirements:** [Team 5 Software Security Engineering Requirements](https://github.com/creepysmilejpg/SWADjangoProject/blob/main/docs/Software_Security_Engineering_Requirements.md) (SR-1 to SR-5).
