# Assurance Cases for Software Security Engineering

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
## 1. Develop five top-level claims
**Claim 1. DefectDojo preserves the integrity of vulnerability scan results that are imported through CI/CD pipelines**

**Claim 2. DefectDojo ensures that vulnerability finding changes and risk acceptances are appropriately authorized, auditable, and subject to an expiration review**

**Claim 3. DefectDojo prevents users from accessing or modifying vulnerability information belonging to applications they are not authorized to**

**Claim 4. DefectDojo restricts administrative privilege changes to authorized administrators and provides auditable record of access control changes**

**Claim 5. DefectDojo protects Jira integration credentials and prevents unauthorized configuration changes and webhook interactions**


## 2. Prepare an argument for each top-level claim



# Part 2
## 1. Assess the alignment of the evidence

### Claim 1: DefectDojo preserves the integrity of vulnerability scan results that are imported through CI/CD pipelines

![Claim 1 overview](./diagrams/Claim1_Overview.png)

<details>
<summary><b>R1.1: Unless the pipeline API token is stolen</b></summary>

![Claim 1, R1.1 branch](./diagrams/Claim1_R1.1.png)

</details>

<details>
<summary><b>R1.2: Unless a token can import into unauthorized Assets</b></summary>

![Claim 1, R1.2 branch](./diagrams/Claim1_R1.2.png)

</details>

<details>
<summary><b>R1.3: Unless the scan file is malformed or malicious</b></summary>

![Claim 1, R1.3 branch](./diagrams/Claim1_R1.3.png)

</details>

<details>
<summary><b>R1.4 and R1.5: Unless a reimport closes valid findings / imports cannot be traced</b></summary>

![Claim 1, R1.4 and R1.5 branches](./diagrams/Claim1_R1.4_R1.5.png)

</details>

### Claim 2: DefectDojo ensures that vulnerability finding changes and risk acceptances are appropriately authorized, auditable, and subject to an expiration review

![Claim 2 overview](./diagrams/Claim2_Overview.png)

<details>
<summary><b>R2.1: Unless any user on the Asset can change finding status</b></summary>

![Claim 2, R2.1 branch](./diagrams/Claim2_R2.1.png)

</details>

<details>
<summary><b>R2.2: Unless audit logging is disabled</b></summary>

![Claim 2, R2.2 branch](./diagrams/Claim2_R2.2.png)

</details>

<details>
<summary><b>R2.3 and R2.4: Unless a risk acceptance has no meaningful expiry / the expiration job does not run</b></summary>

![Claim 2, R2.3 and R2.4 branches](./diagrams/Claim2_R2.3_R2.4.png)

</details>

<details>
<summary><b>R2.5: Unless findings are deleted without a trace</b></summary>

![Claim 2, R2.5 branch](./diagrams/Claim2_R2.5.png)

</details>
