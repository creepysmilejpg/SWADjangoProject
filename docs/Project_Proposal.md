# Software Assurance Project Proposal

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

## 1. Selected Open-Source Software

**Project:** DefectDojo
**Repository:** [github.com/DefectDojo/django-DefectDojo](https://github.com/DefectDojo/django-DefectDojo)

DefectDojo is an open-source application security, DevSecOps, and ASPM (Application Security Posture Management) platform. It acts as a central location where an organization can collect, organize, deduplicate, track, and remediate security findings produced by many different security tools. Throughout the remainder of this proposal, DefectDojo is referred to simply as "the software."

---

## 2. Hypothetical Operational Environment

**Setting:** A large enterprise with over 5,000 employees operating a mix of internal and public-facing applications — employee portals, public websites, financial systems, cloud applications, and network infrastructure.

The organization's cybersecurity department is responsible for identifying and correcting vulnerabilities across these numerous and dense platforms. DefectDojo allows the department to reduce the volume of raw findings that must be manually investigated. Its core functionality is a vulnerability tracker that ingests, organizes, and standardizes reports from other security tools. It identifies duplicates, tracks remediation progress, enforces remediation deadlines, and generates its own concise reports — producing a simplified database of information that may range from vulnerability details to sensitive information about internal applications, or even credentials.

### Systems Engineering View

The diagram below identifies DefectDojo as the system of interest, the enabling systems it depends on to operate, the interoperating systems it exchanges data with, and the stakeholders acting on it — all within the organization's enterprise operational environment.

![Systems Engineering View of DefectDojo](assets/systems_engineering_diagram.png)

### Perceived Threats

DefectDojo's threat profile is particularly significant because it is a security management system that itself holds sensitive information capable of compromising the security of the organization using it:

- **Unauthorized access / cross-application disclosure** — A developer responsible for Application A should not be able to view vulnerabilities belonging to Application B. Unauthorized access allows vulnerability information to be seen and leveraged by parties who should not have visibility into it.
- **Privilege escalation** — Administrators can modify and access large amounts of security information across products. In the wrong hands, this data essentially draws a map of how to attack the organization's systems, directly affecting confidentiality.
- **Integrity compromise** — Modifying findings (deleting them, changing severity, etc.) compromises the integrity of the remediation record and can be used to hide active vulnerabilities.
- **Credential theft** — Because the platform stores integration credentials (API keys, tokens) for the tools it ingests from, compromise of DefectDojo can cascade into compromise of connected systems.
- **Denial of service** — Disruption of the platform interrupts an organization's ability to track and remediate vulnerabilities across all connected products simultaneously.

### Security Features

Security features implemented in DefectDojo include:

- User authentication
- Authorization and access control
- Role and permission management
- Tracking
- Deduplication
- Risk acceptance
- SLA tracking
- Security reporting
- REST API
- Audit history
- Credential encryption
- SSO support
- Backups

---

## 3. Team Motivation

Our motivation for selecting this project comes from prior team experience working with vulnerability management tools in professional settings. DefectDojo is a widely used resource for aggregating discovered vulnerabilities, deduplicating them, tracking remediation, and reporting metrics — making it directly relevant to the kind of security tooling several of us already work with or expect to work with.

---

## 4. Open-Source Project Description

**What it is:** DefectDojo is an open-source application security, DevSecOps, and ASPM (application security posture management) platform. It acts as a central location where an organization can collect, organize, deduplicate, track, and remediate security findings produced by many different security tools. It began as an internal tool at Rackspace used to track testing efforts, was later open-sourced, and became an OWASP Flagship project. It is now also maintained commercially by DefectDojo, Inc., which offers a Pro/SaaS edition alongside the OWASP Community Edition.

**Contributors:** The project is maintained by Greg Anderson (`@devGregA`) and Matt Tesauro (`@mtesauro`), with core moderator Cody Maffucci (`@Maffooch`) and moderator Blake Owens (`@blakeaowens`). The repository's Hall of Fame recognizes long-time contributors including Valentijn Scholten, Jannik Jürgens, Fred Blaise, Aaron Weaver, Jay Paz, and Charles Neill, and the contributor graph shows hundreds of community contributors overall.

**Activity:** Very active. As of September 2026, the master branch has over 14,500 commits and 298 releases; the latest release, 3.0.100, was published June 22, 2026. Releases follow a regular monthly cadence with automatically generated changelogs. The repository has roughly 200 open issues and 30–45 open pull requests at any given time, with CI running unit and integration test suites on every change.

**Popularity:** ~4.8k GitHub stars, ~1.9k forks, ~200 watchers. DefectDojo appears on the Open Source Security Index as one of the fastest-growing open-source security projects and holds a CII (OpenSSF) Best Practices badge.

**Use:** Deployed by security teams to ingest scanner output from CI/CD pipelines, manage remediation across products, push findings to issue trackers like Jira, and report on compliance mandates such as PCI-DSS.

**Languages:** Python (Django framework) for the backend; HTML/Django templates and JavaScript for the UI; Go templates (Helm), CSS, and shell scripts for deployment. Approximate breakdown: HTML 50%, Python 39%, JavaScript 9%.

**Platform:** Linux containers. Officially supported installations are Docker Compose (Community Edition) and Kubernetes via Helm chart, or SaaS (Pro). Backed by PostgreSQL, Redis, Celery, nginx, and uWSGI.

**Documentation sources:**
- Official docs: [docs.defectdojo.com](https://docs.defectdojo.com/)
- REST API v2 docs: [docs.defectdojo.com/en/open_source/api-v2-docs](https://docs.defectdojo.com/en/open_source/api-v2-docs/)
- Repository `readme-docs/` (Docker setup, contributing guide, security policy)
- Community Slack and Community Portal (linked from the README)
- OpenHub: [openhub.net/p/django-DefectDojo](https://www.openhub.net/p/django-DefectDojo)

---

## 5. License, Contribution Procedures, and Contributor Agreements

**License:** DefectDojo is released under the **BSD 3-Clause License** (see `LICENSE.md` and `NOTICE`). This permissive license allows use, modification, and redistribution in source or binary form, provided the copyright notice and disclaimer are retained and the project's name is not used to endorse derived products without permission.

**Contribution procedures** (from `readme-docs/CONTRIBUTING.md`):
- Bug reports must include the OS name/version and the DefectDojo install type (Docker, Kubernetes, etc.); reports missing this information are closed.
- Enhancements require **pre-approval** — contributors open an issue before starting work, and maintainers add an `enhancement-approved` label if the idea fits. New parsers for unsupported tools, bug fixes, security vulnerability resolutions, and added tests are always acceptable. New API routes for third-party integrations, new data models, and new UI pages for metadata are not approved.
- Pull requests must be based against the `dev` or `bugfix` branch, pass all tests in `tests/`, conform to PEP8 (via `ruff`/`flake8`), and be Python 3.13 compliant. Database model changes require a migration committed alongside `max_migration.txt` to preserve a linear migration history.
- Code review requires discussion — reviewer comments must be addressed rather than silently resolved, and a contributor declining a suggestion must reply with their reasoning.
- Security issues are **not** filed as public issues. They are routed through the HackerOne disclosure program, after which maintainers open a private GitHub security advisory and a temporary private fork to coordinate the fix.

**Contributor agreements:** DefectDojo does not require a signed Contributor License Agreement (CLA) or Developer Certificate of Origin (DCO). Contributions are accepted under the project's BSD 3-Clause license by virtue of submitting a pull request. Community conduct is governed through the OWASP Slack channel and Community Portal.

---

## 6. Security-Related History

DefectDojo has a well-documented security history, with more than 30 published GitHub Security Advisories across four pages. Recurring themes are **broken access control** and **unsafe handling of uploaded scan data**, which map directly to the threats identified in Section 2.

| Advisory / CVE | Date | Severity | Summary |
|---|---|---|---|
| GHSA-8q8j-7wc4-vjg5 | Nov 2020 (≤1.9.2) | — | Jira/Tool Configuration credentials (passwords, SSH keys, API keys) exposed in plaintext via the Django admin portal and API v1/v2 GET requests. Fixed in 1.9.3; drove the later decision to encrypt stored integration credentials. |
| GHSA-9jr7-2hgp-vhp8 | Feb 2021 (<1.12.1) | High | API v1/v2 lacked authorization checks, allowing retrieval of findings, endpoints, notes, and reports for unauthorized products. API v2 was fixed; API v1 was deprecated and never fixed — a deliberate decision to retire the older surface. |
| GHSA-96vq-gqr9-vf2c | 2021 | — | Metrics and report pages leaked product, product-type, and finding details to unauthorized users. |
| GHSA-fwg9-752c-qh8w | Nov 2021 (<2.4.0) | — | Stored XSS via the File Path and SAST Source File Path fields on the finding view page (discovered via HackerOne). |
| GHSA-f82x-m585-gj24 / GHSA-v7fv-g69g-x7p2 | Jan 2022 (<2.6.0) | — | Stored XSS when viewing uploaded files (fixed by forcing `Content-Disposition: attachment`); improper access control allowed any staff user to view/edit Jira configurations and product tracking files by guessing URLs. |
| GHSA-hfp4-q5pg-2p7r | Feb 2023 (<2.19.4) | — | Documented GitHub OAuth2 SSO setup authenticated at the service level rather than org/team level, allowing any GitHub account to log in. v2.19.4 shipped an intentional breaking change to force correct configuration. |
| GHSA-6cv6-rq35-rq67, GHSA-jwr5-h452-j325, GHSA-9mg2-8x24-9j62 | Aug 2025 (v2.4x) | High (×3) | MFA bypass; IDOR through the user profile; improper access control on settings objects. |
| GHSA-4859-jfm8-m5xp | Jan 2026 | — | Arbitrary file read via code injection, available to admin users. |
| CVE-2026-3816 | Mar 2026 (≤2.55.4) | 6.5 | Denial of service via zip bomb in the SonarQube and Microsoft Defender parsers; the parser did not limit decompressed size. Fixed in 2.56.0. |
| GHSA-qx2r-p3pg-q6h2 | Jul 2026 | Low | CSRF allowing state-changing actions on findings and engagements. |
| (unnamed) | 2026 | — | Improper authorization allowing use of stored integration credentials across products — again tied to the credential-storage design. |

**Notable security engineering decisions:**
- A formal HackerOne disclosure program with private security advisories.
- A major authorization overhaul in the 2.x series, replacing the legacy staff/superuser model with product- and product-type-scoped roles.
- Deprecation and removal of the vulnerable API v1 surface rather than patching it.
- Encryption of stored integration credentials.
- Adoption of `ruff` linting, a `.dryrunsecurity.yaml` automated security review configuration, and CII Best Practices compliance.
- The recent 3.0 release line introduced a new importer (v3) and a locations model that will require continued security scrutiny.

---

## 7. Team Reflection

This assignment shifted how we, as a team, evaluate open-source security software — not just by its feature list, but by the maturity of the processes behind it. Working through DefectDojo's documentation showed us that it is far more than a report-condensing tool: it also functions as a platform for assigning ownership of vulnerabilities and tracking remediation progress across teams, which several of us hadn't realized going in. That capability is directly relevant to our own professional experience, since a few of us already work with vulnerability management tools and dashboards that face the same challenge of parsing inconsistent report formats from different scanners and removing duplicates.

Reading DefectDojo's security advisory history in chronological order was one of the most valuable parts of the assignment. Many of us primarily interact with tools like this as consumers of their output, so it was eye-opening to see how many of DefectDojo's own vulnerabilities were broken access control and IDOR issues — the exact class of problem the tool exists to help other teams track. That pattern, from the 2020 plaintext credential leak, through the 2021 API authorization gaps, to the 2025 MFA bypass and IDOR advisories, made the systems engineering view click for us as a group: the threats to DefectDojo aren't really about DefectDojo, they're about every product whose findings live inside it. That risk is compounded by the sensitive nature of the data itself; giving users complete control and confidence that their data will stay safe is one of the hardest problems this kind of software has to solve, and it's one we continue to see play out in our own workplaces.

We also learned that an open-source project signals its maturity through process as much as through code. The pre-approval labels for enhancements, the HackerOne-to-private-advisory disclosure pipeline, and the deliberate decision to deprecate API v1 rather than patch it are all software assurance decisions, not just engineering ones — and being able to read them out of a public repository is a skill we expect to reuse well beyond this course. Overall, this project gave us a clearer picture of not only how DefectDojo approaches these problems, but what the broader industry considers best practice for building trustworthy security tooling.
