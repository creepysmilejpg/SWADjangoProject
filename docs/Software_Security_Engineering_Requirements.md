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

**1. CI/CD pipline imports scan results through the API**
![CI/CD pipline imports scan results through the API](./diagrams/CICD_usecase.png)

This is how nearly all data enters DefectDojo. Reports can be uploaded either directly through the UI or through an API endpoint that allows automated ingestion. API calls authenticate with a header containing the user's API key. In the open-source edition, auto-creating the Organization/Asset/Engagement/Test context during import is only available through the API; UI imports need the hierarchy to exist first. That means a pipeline token can create structure, not just findings, which is a useful point for the misuse cases.

---
**2. Security analyst triages findings and records risk acceptance**
![Security analyst triages findings and records risk acceptance](./diagrams/Analyst_Triage_usecase.png)

This is the core decision workflow, and it is exactly what your "integrity compromise" threat targets. Besides active states, a finding can be marked Duplicate, Mitigated, False Positive, Out Of Scope, Risk Accepted, or Under Defect Review. Risk Acceptances can carry uploaded files and notes as justification, and they have expiry dates that force re-evaluation. Each Asset can have its own SLA configuration, which sets the number of days allowed to remediate a finding.

---
**3. Developer or product owner views and remediates findings for their own asset**
![Developer or product owner views and remediates findings for their own asset](./diagrams/Developer_remediation_usecase.png)

This interaction carries your "cross-application disclosure" threat (App A's developer seeing App B), and that threat maps to DefectDojo's authorization boundary. An Asset's Authorized Users list grants access to that Asset and everything nested beneath it, while an Organization's list cascades to every Asset underneath. Open source now offers only local username/password login plus the password-reset flow.

---
**4. DefectDojo administrator manages users and access**
![DefectDojo administrator manages users and access](./diagrams/Admin_acess_usecase.png)

This maps to your "privilege escalation" threat. Superuser and staff accounts can see and act on every Asset and Organization regardless of the Authorized Users lists, and only those accounts get the controls to change the lists. The first account on a fresh install is automatically a superuser. Admins can also harden the API surface: API tokens can be turned off entirely with DD_API_TOKENS_ENABLED=False, or only the api-token-auth endpoint can be disabled with DD_API_TOKEN_AUTH_ENDPOINT_ENABLED=False.

---
**5. DefectDojo pushes findings to Jira**
![DefectDojo pushes findings to Jira](./diagrams/Jira_usecase.png)

This is the one interaction where DefectDojo holds another system's credentials and accepts inbound calls, so it carries your "credential theft" threat. The integration is gated by a System Settings toggle; while it is off, DefectDojo won't push findings (including push_to_jira API requests) and ignores incoming Jira webhooks. The troubleshooting page gives the open-source path as Configuration > System Settings > Enable JIRA integration, and notes that pushes run asynchronously. Those asynchronous pushes run on your Celery Workers box.

---

## 2. For each use case, derive security requirements using misuse case analysis.

**1. Injection of Malicious Information**

![DefectDojo attacker injects malicious info](./diagrams/Malicious_Injection.png)

A pipeline attacker could be someone who obtained a DefectDojo API key belonging to a CI/CD pipeline. The API key allows the pipeline to authenticate to DefectDojo. The attacker in questions can then make requests that appear legtimate. This could include modifying the information being imported, creating unauthorized application structures, or submitting fabricated information.

**Security requirement:** Validate/Authorize API Imports
DefectDojo should validate imported scan results and ensure that the API requests being made are authorized according to the permissions of the account that are associated with that API key. API activity should also be logged so administrators can identify which account made the request.

---

**2. Falsely Accepting Vulnerabilities**

![DefectDojo analyst misuses access](./diagrams/Falsely_Accept_Vulnerability.png)

A malicious security analyst would be someone who is a authenticated user and has legitimate access to vulnerability findings, but intentionally misues that access to mislead others on the current risks. This scenario works as an insider threat, where the analyst compromises the integrity of vulnerability information. They can prevent the proper remediation from taking place, leading to real security issues being left unchecked.

**Security requirement:** Audit Finding Changes
DefectDojo should restrict security sensitive finding changes to authorized users and maintain an audit of changes to finding status and risk acceptance. The system should record who made the change and when. Risk acceptances should retain their expiration information so that accepted vulnerabilities can be re-evaluated when the acceptance expires.

---
**3. Access Another Application's Findings**

![DefectDojo app dev misuses accesses](./diagrams/Asset_Authorization.png)

A malicious application developer is a legitimate developer who has access to one application in DefectDojo but attempts to access information belonging to another. If this is successful, this can expose vulnerability information belonging to another application and potentially allow them to modify or remediate findings they are not responsible for.

**Security requirement:** Asset Level Authorization
DefectDojo should verify a user is authorized to access an asset before providing access to its resources. These checks can apply to both the user interface and API requests so that a user can't bypass the application's authorization boundary.

---
**4. Grant Unauthorized Access**

![DefectDojo admin misuses authorization](./diagrams/Unauthorized_Access.png)

A compromised administrator is an attacker who has obtained control of a superuser account. These accounts have greater access than normal users, and can manage authorized users, access rights, etc. The attacker can add themselves or another account to an asset, create unauthorized privileged access, or even modify existing access controls to view application data.

**Security requirement:** Restrict and Audit Privilege Changes
DefectDojo should restrict user and access management functions to authorized administrators and maintain an audit trail of changes to user permissions and authorized user lists. Administrator privilege assignments should require appropriate authorization and they should be able to disable API token functionality when not required to reduce attack surface.

---
**5. Use Stolen Jira Credentials**

![DefectDojo credential thief](./diagrams/Credential_Theft.png)

A credential thief is an attacker who obtains credentials used by DefectDojo to communicate with Jira. DefectDojo stores these credentials and communicates with Jira asynchronously through the integration. The attacker can potentially use these credentials outside of DefectDojo to access or manipulate information in Jira.

**Security requirement:** Protect and Authenticate Integration Credentials
DefectDojo should protect Jira integration credentials from unauthorized access and restrict changes to the integration configuration to authorized administrators. Jira webhook requests should be authenticated before they are processed and integration activity should be logged so unauthorized requests or changes can be identified.

---

## 3. Iterate between use and misuse cases.

<img width="724" height="691" alt="image" src="https://github.com/user-attachments/assets/ab725d8c-8aa7-4503-a699-4673f13c0f9b" />

<img width="1375" height="417" alt="image" src="https://github.com/user-attachments/assets/3dc2b786-8ca7-4c94-b1e3-7278786609c6" />
<img width="1379" height="396" alt="image" src="https://github.com/user-attachments/assets/975bed62-003c-4281-ae2a-e6bae133eb1b" />
<img width="1378" height="187" alt="image" src="https://github.com/user-attachments/assets/5b40b52e-e486-462b-a63d-4cf7ed868357" />



The following is a prompt we utilized to both assist in both the creation and reviewal of our diagram: 
<img width="1347" height="116" alt="image" src="https://github.com/user-attachments/assets/416247b3-27b4-4360-a97e-efe348a78295" />



---

## 4. Build a list of security requirements derived from misuse case analysis.

---

## 5. Assess the alignment of security requirements
To assess alignment, we compared each requirement from Section 4 against what DefectDojo advertises in its README and documentation ([docs.defectdojo.com](https://docs.defectdojo.com/)) and then verified the behavior in the `master` branch of [django-DefectDojo](https://github.com/DefectDojo/django-DefectDojo) (release line 3.0.x, reviewed September 2026). The most important context for this assessment is that the open-source (OS) edition has recently shed several security features to the paid Pro edition. The docstring at the top of `dojo/authorization/authorization.py` states that the hierarchical RBAC role system has been moved out of OS into the `dojo-pro` plugin, and that OS deployments authorize an action only if the user is a superuser, is staff, or appears in the relevant `authorized_users` list somewhere up the Organization → Asset → Engagement → Test → Finding hierarchy. `dojo/authorization/roles_permissions.py` confirms that the `Roles` enum (Reader, Writer, Maintainer, Owner, API Importer) is preserved for backward compatibility only, and that the `Product_Member`, `Product_Type_Member`, and `Global_Role` tables are now "inert data tables" that nothing in OS reads. This single change affects three of our five requirements.

### Requirement-by-requirement alignment

| # | Security Requirement | Advertised Feature | Evidence in Code / Docs | Alignment |
|---|---|---|---|---|
| 1 | Validate and authorize API imports; log API activity by account | REST API v2 with token auth, 200+ parsers, `auto_create_context` | `dojo/authorization/api_permissions.py` calls `user_has_permission()` on the target object before import, including for objects created via `AutoCreateContextManager`. Parsers that read archives use `safe_read_all_zip()` (`dojo/tools/utils.py`), added after CVE-2026-3816. `DD_API_TOKENS_ENABLED` and `DD_API_TOKEN_AUTH_ENDPOINT_ENABLED` can disable tokens. `DD_RATE_LIMITER_RATE` throttles login. | **Partial.** Authorization and archive validation are present, but in OS a token holder in an Asset's authorized users list has *full* write on that Asset; there is no read-only or import-only token scope. API tokens do not expire (no token-expiry setting exists in `settings.dist.py`). Per-request API logging with the acting account is only what pghistory captures on object changes; there is no dedicated API access log. |
| 2 | Restrict finding status/risk-acceptance changes to authorized users; audit who changed what and when; retain risk-acceptance expiry | Audit history, Risk Acceptance with expiration, SLA tracking | `dojo/auditlog/services.py` registers `Finding`, `Risk_Acceptance`, `Test`, `Engagement`, `Product`, `Product_Type`, `Endpoint`, `Finding_Group` and others with django-pghistory (insert/update/delete events); `PgHistoryMiddleware` (`dojo/middleware.py`) records the acting user and client IP. `dojo/risk_acceptance/models.py` stores `expiration_date`, `expiration_date_warned`, and `expiration_date_handled`; `expiration_handler()` in `risk_acceptance/helper.py` reactivates findings via a daily job. | **Strong on auditing and expiry, weak on restriction.** The audit trail meets the requirement. However, because OS no longer distinguishes Reader from Writer, any authorized user on an Asset can mark findings False Positive, Mitigated, or Risk Accepted. The requirement's "restrict to authorized users" is satisfied only at the Asset boundary, not by role. Audit logging can also be switched off entirely with `DD_ENABLE_AUDITLOG=False`. |
| 3 | Asset-level authorization enforced in both UI and API | Authorization scoped by Asset and Organization | `authorization.py` climbs the object hierarchy to find an `authorized_users` membership; `api_permissions.py` and `authorization_decorators.py` apply the same function to REST and UI views; `query_filters.py` filters list endpoints. Historical advisories (GHSA-9jr7-2hgp-vhp8, GHSA-96vq-gqr9-vf2c, GHSA-jwr5-h452-j325) show this boundary has been broken repeatedly and repaired. | **Aligned, with a caveat.** The mechanism matches the requirement and is applied to both surfaces. The caveat is that "authorized" is now binary: a developer authorized on Application A can edit, delete, and import into A, not merely view it. The recurring IDOR history also argues for treating this boundary as a continuing verification target rather than a solved problem. |
| 4 | Restrict and audit privilege changes; allow disabling API tokens | Superuser/staff administration, `DD_API_TOKENS_ENABLED` | `user_has_configuration_permission()` reduces to `is_superuser`/`is_staff`. `Dojo_User` is pghistory-tracked (with `password` excluded). The Django admin can grant `auth.add_user` to non-staff users as a fallback. | **Partial.** The API-token kill switch exactly matches the requirement. The audit gap is significant: changes to an Asset's or Organization's `authorized_users` list are *not* tracked. `Product` and `Product_Type` are tracked models, but pghistory records column changes, and a ManyToMany membership change does not alter a column on the parent row, so adding an attacker to an Asset leaves no audit event. `Product_Member`/`Product_Type_Member`/`Global_Role` are likewise not registered. MFA is not present in OS (no `django_otp`/`two_factor` in `requirements.txt`), and SSO (SAML/OAuth2/LDAP) has moved to Pro, so a superuser is protected by password plus optional rate limiting only. |
| 5 | Protect Jira credentials at rest; restrict integration config to admins; authenticate inbound webhooks; log integration activity | "Credential encryption", Jira integration toggle, webhook secret | `Tool_Configuration` credentials **are** encrypted: `dojo/tool_config/ui/views.py` calls `dojo_crypto_encrypt()` (AES-256-GCM, migration `0272`). `Jira_Instance.password`, however, is a plain `CharField(max_length=2000)` in `dojo/jira/models.py` with no encryption call anywhere in `dojo/jira/`. `dojo/jira/views.py` rejects webhooks whose path secret does not match `system_settings.jira_webhook_secret`, unless `disable_jira_webhook_secret` is set. `Jira_Instance` is not pghistory-tracked. | **Misaligned on the core point.** The advertised "credential encryption" applies to scanner tool configurations but not to Jira, which is the exact integration our misuse case targets. The webhook secret is a shared secret in the URL path rather than an HMAC signature over the body, and it can be disabled with one checkbox. Configuration changes are restricted to superusers (aligned), but changes to the Jira instance itself are not audited (misaligned). |

### Summary of findings

Taken together, DefectDojo's OS edition provides a **reasonable floor but not the full set of controls** our misuse-case analysis expects.

**Where the software meets or exceeds expectations.** Auditing of findings and risk acceptances is more thorough than we anticipated: pghistory captures every insert, update, and delete on the core models with the acting user and source IP, and the risk-acceptance expiry workflow (warn, handle, reactivate) is exactly what Requirement 2 asks for. The Asset/Organization authorization boundary is implemented once and reused by both the UI and the REST API, and archive-handling hardening after CVE-2026-3816 shows the maintainers do close parser-level gaps. The API-token kill switches and login rate limiter are simple, effective attack-surface reducers that map directly to Requirements 1 and 4.

**Where the software falls short.** Three gaps stand out. First, the removal of role granularity from OS means "authorized" now equals "can do anything on this Asset." Our requirements assume a security analyst can accept risk while a developer can only view and remediate; that distinction no longer exists in the free edition, so Requirements 2 and 3 are satisfied only at a coarser level than the misuse cases demand. Second, the audit log does not cover the thing a compromised administrator would actually do: membership changes to `authorized_users` lists and edits to `Jira_Instance` produce no history events, which undercuts Requirement 4. Third, DefectDojo's marketing of "credential encryption" is accurate for tool configurations but does not extend to the Jira integration password, which sits in plaintext in the database, and the webhook is protected by a path-embedded shared secret that can be turned off. Requirement 5 is therefore the least well supported.

**Sufficiency judgment.** For the hypothetical 5,000-employee enterprise in our proposal, the OS edition alone is insufficient against the insider and compromised-administrator misuse cases without compensating controls: database-level encryption or a secrets manager for the Jira password, an external reverse proxy or identity provider for MFA and SSO, and either the Pro plugin or a customized fork to restore role separation and membership auditing. The distinction between the two editions is also poorly signaled in the documentation, which still describes Reader/Writer/Maintainer/Owner roles alongside `PRO__` prefixed pages; a team deploying from the docs could reasonably believe they have controls that the OS code no longer enforces. These gaps become the candidate areas for the assurance cases and code review in later milestones, with the `authorized_users` audit gap and Jira credential storage as the highest-value targets.

---
# Part 2
## 1. Review your OSS project documentation specifically for security-related configuration and installation issues. Summarize your observations about what could be improved or is missing.
