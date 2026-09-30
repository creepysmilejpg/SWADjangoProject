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

The misuse case analysis in Sections 2 and 3 produced five top-level security requirements, one for each essential interaction. Below, each one is broken into specific, testable "shall" statements that describe functions the software itself must provide. The numbering (SR-1 to SR-5) matches Requirements 1–5 assessed in Section 5.

### Traceability summary

| ID | Security Requirement | Derived From (Misuse Case) | Misuser | Threat Addressed (Proposal) | Use Case Protected |
|---|---|---|---|---|---|
| SR-1 | Validate and Authorize API Imports | Injection of Malicious Information | Pipeline Attacker | Integrity compromise, Denial of service | CI/CD pipeline imports scan results through the API |
| SR-2 | Audit Finding Changes | Falsely Accepting Vulnerabilities | Malicious Security Analyst | Integrity compromise | Security analyst triages findings and records risk acceptance |
| SR-3 | Asset-Level Authorization | Access Another Application's Findings | Malicious Application Developer | Unauthorized access / cross-application disclosure | Developer views and remediates findings for their own asset |
| SR-4 | Restrict and Audit Privilege Changes | Grant Unauthorized Access | Compromised Administrator | Privilege escalation | Administrator manages users and access |
| SR-5 | Protect and Authenticate Integration Credentials | Use Stolen Jira Credentials | Credential Thief | Credential theft | DefectDojo pushes findings to Jira |

### Detailed requirements

**SR-1: Validate and Authorize API Imports**
*Misuse case:* A Pipeline Attacker uses a stolen CI/CD API key to import fabricated or modified scan data, or to create unauthorized application structures.

| ID | Requirement |
|---|---|
| SR-1.1 | The system shall authenticate every API request using a token bound to a single user account. |
| SR-1.2 | The system shall authorize each import against that account's permissions on the target Engagement or Asset before accepting data. |
| SR-1.3 | The system shall apply the same authorization check to any Organization, Asset, Engagement, or Test that an import request would create automatically (`auto_create_context`). |
| SR-1.4 | The system shall validate uploaded scan files before parsing them, rejecting files that exceed size limits, are malformed, or are compressed archives that exceed safe decompression limits. |
| SR-1.5 | The system shall log API import activity with the acting account, so that administrators can identify which account submitted each import. |

**SR-2: Audit Finding Changes**
*Misuse case:* A Malicious Security Analyst uses legitimate access to falsely mark findings as accepted, false positive, or mitigated, hiding real vulnerabilities from remediation.

| ID | Requirement |
|---|---|
| SR-2.1 | The system shall restrict changes to finding status, severity, and risk acceptance to users authorized for the affected Asset. |
| SR-2.2 | The system shall record an audit entry for every change to a finding's status or severity and to every risk acceptance, including the acting user, the time of the change, and the old and new values. |
| SR-2.3 | The system shall require every risk acceptance to have an expiration date. |
| SR-2.4 | The system shall re-evaluate accepted findings when their risk acceptance expires, returning them to an active state so they are reviewed again. |

**SR-3: Asset-Level Authorization**
*Misuse case:* A Malicious Application Developer who is authorized for one application tries to view or modify findings belonging to another application.

| ID | Requirement |
|---|---|
| SR-3.1 | The system shall verify that a user is authorized for an Asset before displaying or modifying any of its engagements, tests, or findings. |
| SR-3.2 | The system shall enforce the same authorization checks in both the user interface and the REST API, so the boundary cannot be bypassed through either one. |
| SR-3.3 | The system shall filter list, search, report, and metrics results so they include only Assets the user is authorized to access. |
| SR-3.4 | The system shall deny access by default when no authorization for the requested object can be established. |

**SR-4: Restrict and Audit Privilege Changes**
*Misuse case:* A Compromised Administrator uses a stolen superuser account to add themselves or others to Assets, create privileged accounts, or change existing access controls.

| ID | Requirement |
|---|---|
| SR-4.1 | The system shall restrict user creation, privilege assignment (staff/superuser), and changes to Asset and Organization authorized-user lists to authorized administrators. |
| SR-4.2 | The system shall record an audit entry for every change to user privileges and every addition to or removal from an authorized-user list, including the acting administrator, the affected user, and the time. |
| SR-4.3 | The system shall allow administrators to disable API token authentication, or only the token-issuing endpoint, when it is not required. |

**SR-5: Protect and Authenticate Integration Credentials**
*Misuse case:* A Credential Thief obtains the credentials DefectDojo uses to connect to Jira and uses them outside DefectDojo to access or manipulate Jira data.

| ID | Requirement |
|---|---|
| SR-5.1 | The system shall never return stored Jira credentials in the user interface or API responses once they have been saved. |
| SR-5.2 | The system shall encrypt stored Jira credentials at rest. |
| SR-5.3 | The system shall restrict creation and modification of the Jira integration configuration to authorized administrators. |
| SR-5.4 | The system shall authenticate inbound Jira webhook requests before processing them, and shall ignore webhook requests when the integration is disabled. |
| SR-5.5 | The system shall log changes to the integration configuration and integration activity, so that unauthorized requests or changes can be identified. |

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

### Scope and method

We reviewed the documentation an open-source (OS) user would follow to install, configure, and harden DefectDojo. For each security-relevant statement, we checked the shipped files and code to see whether the docs match the actual behavior. We reviewed the `master` branch of [django-DefectDojo](https://github.com/DefectDojo/django-DefectDojo) at commit `8b12d80` (28 Sep 2026).

| Documentation reviewed | Checked against |
|---|---|
| `README.md` (Quick Start, Documentation links) | `docker-compose.yml` |
| `readme-docs/DOCKER.md` | `docker-compose.override.https.yml`, `docker-compose.override.dev.yml` |
| `readme-docs/KUBERNETES.md` | `helm/defectdojo/values.yaml` |
| `readme-docs/SECURITY.md` | `nginx/nginx.conf`, `nginx/nginx_TLS.conf` |
| `docs/content/get_started/open_source/installation.md` | `dojo/settings/settings.dist.py` |
| `docs/content/get_started/open_source/configuration.md` | `dojo/management/commands/complete_initialization.py` |
| `docs/content/get_started/open_source/running-in-production.md` | |
| `docs/content/automation/api/api-v2-docs.md` and `rate_limiting.md` | |
| `docs/content/connectors/os_jira/os__jira_guide.md` | |

### What the documentation does well

- **Honest about the quick-start file.** It says clearly that `docker-compose.yml` is for evaluation and "not intended for production use without first customizing it."
- **Explains the credential encryption key.** *Running in Production* tells users to replace the default `DD_CREDENTIAL_AES_256_KEY`, gives an `openssl rand -base64 32` example, and admits the key "does not cover every credential DefectDojo stores."
- **Random first admin password.** The initializer generates a random 22-character admin password instead of shipping a fixed one.
- **Warns against the debug toolbar.** `DOCKER.md` warns not to enable the Django Debug Toolbar in production.
- **Good Jira webhook guidance.** The OS Jira guide requires a webhook secret whenever the integration is enabled and tells users to "treat the generated value as a credential."
- **Documents API token controls.** The API docs explain how to disable API tokens (`DD_API_TOKENS_ENABLED`, `DD_API_TOKEN_AUTH_ENDPOINT_ENABLED`) and how to set token expiry.
- **Clear reporting process.** `SECURITY.md` gives a private reporting channel (HackerOne), a GitHub security-advisory workflow, and a safe-harbor statement.

### Observations: what is missing or could be improved

#### A. Secrets and default credentials

| # | Observation | Where | Why it matters | Suggested improvement |
|---|---|---|---|---|
| A1 | `docker-compose.yml` has a hard-coded fallback for `DD_SECRET_KEY` (`hhZCp@D28z!n@NED*yB!ROMt+WzsY*iq`), but *Running in Production* only tells users to change the AES key. `DD_SECRET_KEY` isn't mentioned anywhere in the OS docs, only on Pro pages. | `running-in-production.md`, `docker-compose.yml` | Django uses this key to sign sessions and password-reset tokens. Every deployment that keeps the public default shares a known signing key. | Add `DD_SECRET_KEY` next to the AES key in the Security section, with a generation command and a warning that the default is public. |
| A2 | The bundled PostgreSQL uses `defectdojo` / `defectdojo` as its default user and password. No page tells users to change them. | `docker-compose.yml` | This database holds every finding and the Jira credentials. Anyone who can reach it with the default login can read everything. | Document `DD_DATABASE_PASSWORD` / `DD_DATABASE_URL` in the production guide and recommend a unique password or an external database. |
| A3 | The README and `DOCKER.md` tell users to get the first admin password with `docker compose logs initializer \| grep "Admin password:"`. Nothing says to rotate it afterward, clear the logs, or set `DD_ADMIN_PASSWORD` in advance. `DOCKER.md`'s own password-change example uses `Password123!`. | `README.md`, `DOCKER.md` | The superuser password stays in container logs, which often end up in log aggregators. The example teaches a weak password. | Recommend setting `DD_ADMIN_PASSWORD` from a secret store, or changing the password on first login and clearing the logs. Replace the example with a placeholder such as `<strong-unique-password>`. |
| A4 | *Configuration* says environment variables must be set for "three services: `uwsgi`, `celerybeat` and `celeryworker`." The `initializer` service also reads `DD_SECRET_KEY` and `DD_CREDENTIAL_AES_256_KEY`. | `configuration.md` | If a user changes the keys in three services but not the initializer, the stack runs with inconsistent secrets. That is hard to debug, and it tempts users back to the defaults. | List all four services, or recommend a single `.env` file / `x-` anchor so secrets are defined once. |
| A5 | The production guide says the AES key "does not cover every credential," but doesn't say which ones it misses. In code, the Jira integration password (`JIRA_Instance.password`) is stored as plain text. | `running-in-production.md` | Admins can't judge how sensitive their database and backups are without knowing this. | State which credentials are not encrypted, recommend a least-privilege Jira API token instead of a password, and recommend encrypting the database and backups at rest. |

#### B. Transport security and HTTP hardening

| # | Observation | Where | Why it matters | Suggested improvement |
|---|---|---|---|---|
| B1 | The HTTPS override (`docker-compose.override.https.yml`) generates a self-signed certificate by default and still publishes plain-HTTP port 8080. It sets secure cookies, but not `DD_SECURE_SSL_REDIRECT` or HSTS. The "use your own credentials" steps in `DOCKER.md` are unclear: they say to "copy your secrets into `../nginx/nginx_TLS.conf`" when they mean certificate paths. | `DOCKER.md`, `docker-compose.override.https.yml` | Users who follow the HTTPS instructions still expose an HTTP login page, and nothing redirects them to HTTPS. | Provide a production TLS example that disables the HTTP port or redirects it, sets `DD_SECURE_SSL_REDIRECT=True` and HSTS, and uses a CA-issued certificate. Rewrite the certificate steps as a numbered procedure. |
| B2 | None of the session and cookie hardening settings are documented anywhere: `DD_SESSION_COOKIE_SECURE`, `DD_CSRF_COOKIE_SECURE`, `DD_SECURE_SSL_REDIRECT`, `DD_SECURE_HSTS_SECONDS` / `_INCLUDE_SUBDOMAINS`, `DD_SESSION_COOKIE_AGE` (default 14 days), and `DD_SESSION_EXPIRE_AT_BROWSER_CLOSE`. In `settings.dist.py`, HSTS is only applied when `DD_SECURE_HSTS_INCLUDE_SUBDOMAINS=True`, which a reader would not expect. | All OS docs | Users can only find these by reading the source. The defaults are all insecure: cookies aren't HTTPS-only, there's no redirect, and sessions last 14 days. | Add a "Session and transport security" table to *Running in Production* with each variable, its default, and the recommended production value, and explain the HSTS quirk. |
| B3 | `docker-compose.yml` sets `DD_ALLOWED_HOSTS` to `*`, while `settings.dist.py` defaults to `localhost`. The OS docs never mention the variable; only Pro install pages do. | `docker-compose.yml`, OS docs | A wildcard host list allows Host-header attacks such as poisoned password-reset links. | Document `DD_ALLOWED_HOSTS` as a required production setting and change the compose default to a placeholder. |
| B4 | The Security section says only "verify the `nginx` configuration and other run-time aspects such as security headers." The shipped `nginx.conf` / `nginx_TLS.conf` add no security headers. They do set `server_tokens off`, and the TLS config limits protocols to TLS 1.2/1.3. | `running-in-production.md`, `nginx/` | Users are told to check headers but not which ones, or which ones Django already sets. | List recommended headers (HSTS, Content-Security-Policy, Referrer-Policy, Permissions-Policy), say which ones Django already provides, and give a sample `add_header` block. |
| B5 | The API code samples use `http://` URLs and include the comment "set verify to False if ssl cert is self-signed." | `api-v2-docs.md` | This treats turning off TLS certificate checks as normal for the credential users copy most: API tokens. | Use `https://` in the examples and recommend pointing `verify=` at a CA bundle instead of disabling verification. |

#### C. Authentication, authorization, and auditing

| # | Observation | Where | Why it matters | Suggested improvement |
|---|---|---|---|---|
| C1 | *Rate Limiting* lists the defaults as `DD_RATE_LIMITER_ENABLED=(bool, True)`, `..._BLOCK=(bool, True)`, and `..._ACCOUNT_LOCKOUT=(bool, True)`. In `settings.dist.py` all three default to **`False`**. The page also contradicts itself: "By default, rate limiting records offenses but does not block requests." The error is copied into all 7 translated versions. | `rate_limiting.md` (+ translations) | Readers will believe brute-force protection is on when it is not. | Fix the defaults, add a "to enable" example, and propagate the fix to the translations. |
| C2 | The same page says rate-limit counters are "not shared across processes" when uWSGI runs several workers, which is the default (4 processes). It doesn't say how to fix it. | `rate_limiting.md` | Even when enabled, an attacker gets roughly four times the configured rate. | Show how to point Django's cache at the bundled Valkey so counters are shared. |
| C3 | The API docs show `DD_API_TOKEN_DEFAULT_EXPIRY_DAYS=90` as an example but don't say the default is `0`, meaning tokens never expire. | `api-v2-docs.md` | CI/CD tokens are long-lived and easy to leak. Users may assume they expire. | State the default and recommend an expiry for production. |
| C4 | The README still links to OAuth2/SAML2 (under `archived_docs`) and LDAP pages. The 3.0 upgrade notes say SSO and remote-user authentication are Pro-only and OS supports only local username/password login. MFA is also Pro-only, and the OS install docs don't say so. | `README.md` | Teams may plan their authentication around SSO or MFA that the OS edition no longer provides. | Remove or relabel the links as "Pro only," and add an OS authentication page listing what is and isn't available. |
| C5 | OS and Pro permission docs sit side by side. Pages describing Reader/Writer/Maintainer/Owner roles are only marked Pro by a filename prefix or a toggle. The OS model is the *Authorized Users* list plus staff/superuser. | `admin/user_management/` | Readers may think role separation, such as a read-only developer, exists in OS when it does not. | Put a clear edition banner at the top of every Pro-only page, and link OS readers to the Authorized Users page. |
| C6 | `DD_ENABLE_AUDITLOG` is documented only on a Pro audit-log page and in the 2.30 upgrade notes. The retention settings (`DD_AUDITLOG_FLUSH_RETENTION_PERIOD`, etc.) aren't documented. Nothing says which objects are audited. For example, changes to Authorized Users lists and to the Jira configuration are not recorded. | OS audit docs | Admins can't tell whether the audit trail will answer "who gave this person access?" | Document the settings, list the audited models, and state known gaps. |

#### D. Deployment options, policy, and structure

| # | Observation | Where | Why it matters | Suggested improvement |
|---|---|---|---|---|
| D1 | *Installation* lists Kubernetes under "Options for the brave (not officially supported)." `KUBERNETES.md` is a single line pointing to the Helm README. The Helm `values.yaml` ships with empty `secretKey`, `credentialAes256Key` and admin password, and `createSecret: false`. *Architecture* notes that the chart still uses Redis rather than Valkey. | `installation.md`, `KUBERNETES.md`, `helm/` | Enterprise users are the most likely to use Kubernetes, and they get no guidance on managing secrets or network policy. | Add an OS Helm hardening note covering where secrets come from (external secret store), network policies, and the support status. Otherwise, state plainly that Helm is unsupported for production. |
| D2 | `SECURITY.md` describes how to report issues, but doesn't say which versions receive security fixes, how users are notified of security releases, or which edition (OS or Pro) the policy covers. It also has a typo ("coordonate"). | `readme-docs/SECURITY.md` | Operators can't tell whether they need to upgrade to get a fix, or how they'd find out. | Add a "Supported Versions" table (e.g. latest minor release only), link to the GitHub Security Advisories page and release notes, and state the edition scope. |
| D3 | There's no single security checklist for production. The settings above are spread across 6+ pages, or found only in source code. Upload-hardening limits (`DD_MAX_ZIP_MEMBER_SIZE`, `DD_MAX_ZIP_TOTAL_SIZE`, `DD_MAX_ZIP_RATIO`) appear only in `settings.dist.py`. `DD_SCAN_FILE_MAX_SIZE` is documented only on a Pro page. | Docs as a whole | Operators can't check whether an instance is hardened without reading the code. | Create an OS "Security Hardening Checklist" page (outline below). |
| D4 | Every page exists in 8 languages. Security corrections, such as C1, have to be copied into each translation, and nothing flags a translation as out of date. | `docs/content/**` | Non-English readers may get stale or wrong security guidance. | Mark translations with the English revision they're based on, or show an "English version is authoritative" note on security pages. |

### Proposed contribution: OS Security Hardening Checklist

The highest-value fix is one page, linked from *Installation* and *Running in Production*, that brings the scattered settings together:

1. **Secrets:** replace `DD_SECRET_KEY`, `DD_CREDENTIAL_AES_256_KEY`, and `DD_DATABASE_PASSWORD`, and set `DD_ADMIN_PASSWORD` before first boot. Define them once, for all four services.
2. **Network and TLS:** serve over HTTPS with a CA-issued certificate, close or redirect port 8080, and set `DD_ALLOWED_HOSTS`, `DD_SECURE_SSL_REDIRECT`, and HSTS.
3. **Sessions:** set `DD_SESSION_COOKIE_SECURE` and `DD_CSRF_COOKIE_SECURE`, and shorten `DD_SESSION_COOKIE_AGE`.
4. **Login protection:** enable `DD_RATE_LIMITER_ENABLED`, `_BLOCK`, and `_ACCOUNT_LOCKOUT`, backed by a shared cache.
5. **API:** set `DD_API_TOKEN_DEFAULT_EXPIRY_DAYS`, and disable tokens (`DD_API_TOKENS_ENABLED=False`) if they aren't used.
6. **Integrations:** use a least-privilege Jira API token, keep the webhook secret enabled, and remember that the Jira credential is not encrypted at rest.
7. **Data:** encrypt the database and media volume at rest, back them up, and protect the backups like the database itself.
8. **Auditing:** keep `DD_ENABLE_AUDITLOG=True`, set a retention period, and forward logs to a SIEM.
9. **Edition awareness:** SSO, MFA, and role-based permissions are Pro-only, so plan compensating controls (e.g. an identity-aware reverse proxy) if they're required.

Per `CONTRIBUTING.md`, this would be submitted as a pull request against the `dev` branch. Documentation-only changes don't need the `enhancement-approved` pre-approval that code enhancements need. Items A1–A5, B5, C1, C3, and D2 are small, self-contained edits that could each be a separate first-time PR.

### Summary

DefectDojo's documentation is honest that the default Docker Compose setup is for evaluation only, but it doesn't tell users how to get from evaluation to a hardened deployment. Our most significant findings:

- **Undocumented defaults.** The public default `DD_SECRET_KEY` and database password aren't mentioned anywhere in the OS docs.
- **Incorrect rate-limiting defaults.** The docs claim protection is on when it is off.
- **Missing transport settings.** None of the session, cookie, and HSTS settings are documented.
- **Stale edition information.** Links to SSO features that were removed from the OS edition are still live.

Nearly every security control DefectDojo implements is off by default, or can only be found by reading `settings.dist.py`. A single hardening checklist plus the corrections above would close most of the gap between what the software can do and what an operator following the docs actually gets.
