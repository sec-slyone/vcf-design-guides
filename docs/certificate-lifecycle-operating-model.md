# Certificate Lifecycle Operating Model

**VMware Cloud Foundation 9.1.1 and standalone vSphere estates**

*Operating model and runbook · Mohammad "Sarge" Siddiqui · Version 1.0 · 25 October 2026*

---

## 1. Purpose and scope

This document defines how certificate lifecycle is owned, operated and evidenced across a mixed VMware estate. It covers VMware Cloud Foundation 9.1.1 instances managed as a fleet, VCF 5.2 instances, and standalone vCenter Server 8.0 Update 3 and above that have been integrated with VCF Operations without being converted to VCF.

It is written to be lifted into a customer environment with the organisation name and CA details substituted. Sections marked as a decision require a named owner to confirm before the model is operable.

Out of scope: application and web server certificates not presented by the virtualisation platform, hardware management controller certificates, and certificate issuance policy itself, which is assumed to be owned by the organisation's security function.

## 2. The problem this addresses

Certificate expiry in a virtualisation platform is a known, dated, entirely predictable event that nonetheless causes unplanned outages at a rate that would be unacceptable for any other class of risk. The reason is rarely technical. It is that the record of what expires when lives in a spreadsheet maintained by individuals rather than in a system, and the spreadsheet degrades faster than the certificates do.

The specific failure that matters most is the vCenter Security Token Service certificate. When it expires, authentication stops across a platform that is otherwise completely healthy. Workloads continue running, which delays detection, and the recovery procedure is one that most teams have never rehearsed.

> ℹ️ **WHAT CHANGED, AND WHY THIS IS WORTH DOING NOW**
>
> VCF 9.1 extended VCF Operations certificate management to standalone vSphere 8.0 Update 3 and above, and to VCF 5.2, so an estate no longer has to be converted to VCF before its certificates can be managed centrally. VCF 9.1.1 then added management of the vCenter STS certificate and the VMware Certificate Authority root certificate, and added alerting on failed renewals. Those three additions together are what make a managed operating model practical rather than aspirational.

## 3. Requirements, assumptions, constraints and risks

### 3.1 Requirements

| ID | Requirement |
|---|---|
| **R-01** | Every TLS certificate presented by a platform endpoint is discoverable without reference to a manually maintained list. |
| **R-02** | Expiry is known at least 90 days in advance for every certificate in scope. |
| **R-03** | Renewal failure is detected and raised as an incident, not discovered at expiry. |
| **R-04** | Certificate issuance remains inside the organisation's existing trust chain. |
| **R-05** | A documented, rehearsed recovery procedure exists for an expired STS certificate. |
| **R-06** | Evidence of the above is producible for audit without manual collation. |

### 3.2 Assumptions

- A VCF Operations instance exists and is entitled for certificate and password management. See section 11, as this is the single largest commercial dependency in the model.
- The organisation operates an internal certificate authority, most commonly Microsoft Active Directory Certificate Services.
- Standalone vCenter instances in scope are at 8.0 Update 3 or above. Instances below that floor cannot be integrated and are handled under section 9.3.
- Forward and reverse DNS is correct for all platform endpoints, which is a prerequisite rather than a nice to have.

### 3.3 Constraints

- Certificate and password management is listed as a VCF capability rather than a VMware vSphere Foundation one. A VVF estate cannot operate this model as written.
- Change freezes apply. Certificate renewal is a change and competes for the same windows as everything else.
- Some endpoints will be owned by teams outside infrastructure, particularly load balancers and anything fronted by a reverse proxy.

### 3.4 Risks

| ID | Risk | Treatment |
|---|---|---|
| **K-01** | Automated renewal fails silently and the estate is less safe than before automation was introduced. | Enable renewal failure alerting before enabling any automated renewal. See CD-06. |
| **K-02** | VMCA root rotation is performed without understanding the blast radius and causes a platform-wide outage. | Treat as a major change with a tested rollback. Never bundle with another change. |
| **K-03** | The discovered inventory and the maintained inventory disagree and the discrepancy is ignored. | Treat the first discovery run as a finding, with a named owner to reconcile. See section 9.1. |
| **K-04** | Entitlement does not cover the capability and the model is unfundable after design effort is spent. | Confirm in writing before design sign-off. See section 11. |
| **K-05** | An endpoint outside the platform boundary is assumed to be covered and is not. | Maintain the out-of-scope register in section 1 as a live artefact, not a one-time statement. |

## 4. Certificate authority topology options

This is where most designs stall, and it stalls on organisational rather than technical grounds. The security function owns issuance and will not accept a second path to it. The options below are ordered by how often they survive that conversation.

| Option | Description | When it fits | Trade-off |
|---|---|---|---|
| **A. VMCA as subordinate CA** | VMCA issues platform certificates but is itself subordinate to the corporate root. | Default recommendation for most enterprises. Keeps issuance inside the existing trust chain while retaining platform automation. | Requires the security function to issue a subordinate CA certificate, which has a lead time measured in weeks. |
| **B. Enterprise CA direct** | Microsoft CA or OpenSSL issues certificates for VCF instances and management components directly. | Where policy prohibits any subordinate CA, or where certificate issuance must be individually approved. | More moving parts at renewal. Supported for both VCF instances and management components from 9.1; in 9.0 management components supported Microsoft CA only. |
| **C. External third-party CA** | VCF Operations generates the CSR, the external CA returns the certificate, VCF Operations imports it. DigiCert, Google and Sectigo are named examples. | Where issuance is outsourced, or where public trust is required on an externally reachable endpoint. | Renewal is only as automated as the external CA's own process allows. Cost per certificate becomes a design input. |
| **D. VMCA as default internal CA** | VMCA issues and signs with its own root, untrusted outside the platform. | Laboratory, proof of concept, and isolated environments. | Browser and client trust warnings. Rarely acceptable in production for anything a human logs into. |
| **E. API-driven integration** | The VCF Operations REST API introduced in 9.1 drives third-party CA and certificate lifecycle from the organisation's own automation. | Where a mature automation practice already owns certificate issuance end to end. | Shifts ownership to the automation team. Point them at the API rather than letting them script against the user interface. |

> ✅ **RECOMMENDATION**
>
> Lead with option A. In design reviews the stated requirement is almost always "we must use our own CA", and a VMCA subordinate under the corporate root satisfies that requirement while keeping renewal automation intact. Agree it with the security function before integration work starts, not after, because the lead time on a subordinate CA certificate will otherwise sit on the critical path.

## 5. Design decisions

**CD-01 — Certificate authority topology**

| | |
|---|---|
| **Decision** | VMCA operates as a subordinate certificate authority under the corporate root certificate authority. |
| **Justification** | Satisfies R-04 by keeping issuance inside the existing trust chain, while preserving the platform's ability to renew without a manual request per certificate. |
| **Implication** | A subordinate CA certificate must be requested from the security function during the design phase. Platform work is blocked until it is issued. |

**CD-02 — Scope of central management**

| | |
|---|---|
| **Decision** | All vCenter instances at 8.0 Update 3 or above are integrated with VCF Operations for certificate management, whether or not they are part of a VCF instance. |
| **Justification** | VCF 9.1 extended the capability to standalone instances specifically so that conversion is not a prerequisite. The estates least likely to be converted carry the most unmanaged risk. |
| **Implication** | Integration is a change per vCenter but not a conversion. Instances below 8.0 Update 3 remain on the manual process in section 9.3 until upgraded. |

**CD-03 — Order of onboarding**

| | |
|---|---|
| **Decision** | Standalone instances with no conversion plan are onboarded first. Instances already scheduled for VCF conversion are onboarded last. |
| **Justification** | Instances heading for VCF acquire the capability through conversion anyway. Onboarding them first spends effort where the risk is lowest. |
| **Implication** | The schedule will look counter-intuitive to anyone reading it as a VCF adoption plan. State explicitly that it is a risk reduction plan. |

**CD-04 — STS and VMCA root management**

| | |
|---|---|
| **Decision** | vCenter STS and VMCA root certificates are brought under VCF Operations management, which requires VCF Operations at 9.1.1 or above. |
| **Justification** | These two are the highest-consequence certificates in the platform and were the two that could not be centrally managed before 9.1.1. |
| **Implication** | VCF Operations must be at 9.1.1. An estate on 9.1.0 gets the standalone reach but not the two certificates that matter most. |

**CD-05 — Discovery precedes automation**

| | |
|---|---|
| **Decision** | A full discovery run is completed and reconciled against the existing record before any automated renewal is enabled. |
| **Justification** | The discovered inventory and the maintained inventory will disagree. That disagreement is the most valuable output of the exercise and must not be overwritten by automation that assumes the record is correct. |
| **Implication** | Adds a reconciliation activity with a named owner before go-live. Expect findings. |

**CD-06 — Alerting precedes renewal**

| | |
|---|---|
| **Decision** | Renewal failure alerting is enabled and routed to an on-call queue before any automated renewal is enabled. |
| **Justification** | Automation whose failure mode is silence replaces a known manual task with an unverified assumption, which is a net reduction in safety. |
| **Implication** | Sequencing constraint on the implementation plan. Alerting has value on its own even while renewal remains manual. |

**CD-07 — VMCA root rotation**

| | |
|---|---|
| **Decision** | VMCA root rotation is executed as a standalone major change with a tested rollback, never bundled with another change. |
| **Justification** | The blast radius of an incorrect VMCA root rotation is the entire platform trust chain. |
| **Implication** | Consumes a change window on its own. Plan it as a project activity rather than business as usual. |

**CD-08 — Non-TLS endpoint register**

| | |
|---|---|
| **Decision** | The VCF Operations view of components using non-TLS certificates is reviewed quarterly and the output retained as audit evidence. |
| **Justification** | Satisfies R-06 and answers the standing audit question about unencrypted endpoints without manual collation. |
| **Implication** | Quarterly activity with a named owner. Output is retained for the audit retention period. |

## 6. Coverage by component and version

The table below records what VCF Operations manages and from which release, so that a design is not written against a capability the deployed version does not have.

| Component or certificate | Available from | Notes |
|---|---|---|
| **vCenter STS certificate** | 9.1.1 | The expiry most commonly responsible for an authentication outage. |
| **VMCA root certificate** | 9.1.1 | Managed rather than hand-rotated. See CD-07. |
| **Renewal failure alerting** | 9.1.1 | Includes a knowledge base link on the alert. Prerequisite for CD-06. |
| **Non-TLS certificate visibility** | 9.1.1 | View, manage and filter components using non-TLS certificates. |
| **Standalone vCenter 8.0 U3 and above** | 9.1 | Integrated through the Integrations menu in VCF Operations. |
| **VCF 5.2 instances** | 9.1 | In scope without conversion. |
| **Root and TLS certificates** | Earlier | Across managed components. |
| **NSX Edge, Supervisor, License Server** | Earlier | License Server coverage matters given it is mandatory from 9.1 and carries its own DNS requirements. |
| **VCF Operations Unified Cloud Proxy** | Earlier | Covered. |
| **VCF Operations for Networks collector** | Earlier | Covered. |
| **AVI load balancer, HCX instances** | 9.1 | Require 9.1 plus the relevant management pack and adapter rather than working without additional configuration. |

## 7. Roles and responsibilities

| Activity | Accountable | Responsible | Consulted | Informed |
|---|---|---|---|---|
| **Certificate issuance policy** | Security | Security | Infrastructure | Audit |
| **Subordinate CA issuance** | Security | PKI team | Infrastructure |   |
| **Platform integration and onboarding** | Infrastructure | Platform engineering | Security | Service owners |
| **Renewal execution** | Infrastructure | Platform operations | Change | Service owners |
| **Renewal failure response** | Infrastructure | On-call | Security |   |
| **Expired STS recovery** | Infrastructure | On-call and platform engineering | Security, Major incident | Executive |
| **Quarterly non-TLS review** | Infrastructure | Platform engineering | Security | Audit |
| **Evidence retention** | Infrastructure | Platform engineering |   | Audit |

> ⚠️ **THE ROW THAT GETS ARGUED ABOUT**
>
> Renewal execution sits with infrastructure, not security. Security owns whether a certificate may be issued and under what policy. Infrastructure owns when it is installed and what breaks if it is not. Conflating the two produces a process where nobody can act inside a change window.

## 8. Cadence and alerting

### 8.1 Renewal cadence

| Certificate class | Review | Renewal trigger | Change class |
|---|---|---|---|
| **Machine TLS, managed components** | Monthly | 90 days before expiry | Standard |
| **vCenter STS** | Monthly | 120 days before expiry | Normal |
| **VMCA root or subordinate** | Quarterly | 12 months before expiry | Major, see CD-07 |
| **Externally issued, third-party CA** | Monthly | 120 days before expiry, to allow for external CA turnaround | Normal |
| **Non-TLS endpoint register** | Quarterly | Not applicable | Review only |

### 8.2 Alert routing

- Renewal failure raises a priority 2 incident to the platform on-call queue. It is not a notification and it is not an email to a shared mailbox.
- Expiry inside 30 days with no renewal scheduled raises a priority 3 to the platform queue.
- Expiry inside 7 days raises a priority 1 and is treated as an imminent outage regardless of whether anything has broken yet.
- Any alert on a VMCA root or subordinate certificate is routed to platform engineering rather than on-call, because the response is never a runbook step executed under time pressure.

## 9. Runbooks

### 9.1 Runbook A, first discovery and reconciliation

1. Run the credential-free inventory across every platform endpoint in the estate, including endpoints you do not believe are in scope. Retain the raw output.
2. Integrate the first standalone vCenter through the Integrations menu in VCF Operations and allow discovery to complete.
3. Export the discovered certificate list.
4. Reconcile the discovered list against the existing maintained record. Record every discrepancy as a finding with an owner, rather than correcting silently.
5. Classify each discrepancy: certificate present but unrecorded, recorded but absent, expiry date incorrect, owner unknown.
6. Present the findings before proceeding to automation. This output is usually the business case for the rest of the work.

### 9.2 Runbook B, planned renewal

1. Confirm the renewal trigger has fired within the window in section 8.1 and that a change record exists.
2. Confirm the issuing path is the one recorded in CD-01 and that the CA is reachable from VCF Operations.
3. Confirm a current backup or snapshot of the affected appliance exists and that its restore has been validated within the last quarter.
4. Execute renewal through VCF Operations rather than by hand on the appliance, so that the platform record stays authoritative.
5. Verify the new certificate is presented on every endpoint the component serves, not only the one checked by the renewal task.
6. Verify dependent integrations still authenticate. Backup, monitoring, automation and any service using the platform as an identity source.
7. Retain the before and after certificate detail as change evidence.

### 9.3 Runbook C, expired STS certificate

> 🛑 **READ BEFORE YOU NEED IT**
>
> This is the procedure nobody has rehearsed. Workloads keep running while authentication is down, so the first symptom is usually a flood of unrelated-looking failures from backup, monitoring and automation rather than a platform alarm. Establish early that the cause is authentication, because the time lost to misdiagnosis is typically larger than the time lost to the fix.

1. Confirm the symptom set: administrative logins fail, API authentication fails, workloads continue running normally.
2. Confirm the cause by inspecting the presented certificate and its validity dates directly from an endpoint, not from the platform's own interface, which may itself be unavailable.
3. Declare a major incident. The recovery touches the trust chain and should not be executed by a single engineer without oversight.
4. Confirm a restore point exists before making any change.
5. Follow the vendor procedure current for the deployed version. Record the knowledge base article reference in the incident, because the procedure is version-specific and the version in the runbook will date.
6. After recovery, verify every dependent integration that authenticates against the platform, in the order of business criticality.
7. Raise the cause as a finding against the inventory process rather than against the individual. An expired certificate is a process failure by definition.

### 9.4 Runbook D, onboarding a vCenter below 8.0 Update 3

Instances below the integration floor cannot be brought under central management. Until they are upgraded, they remain on a manual process, and the manual process should be the credential-free inventory run on a schedule rather than a spreadsheet.

1. Run the inventory tool against the endpoint on a monthly schedule and retain the output.
2. Record the instance on the exceptions register with the reason and the upgrade plan.
3. Review the exceptions register at the quarterly non-TLS review under CD-08, so that exceptions are visible rather than permanent.

## 10. Audit evidence

| Evidence | Produced by | Frequency | Retained |
|---|---|---|---|
| **Full certificate inventory with expiry** | Discovery export or inventory tool | Monthly | Per retention policy |
| **Renewal change records with before and after detail** | Change process, Runbook B step 7 | Per renewal | Per retention policy |
| **Renewal failure incidents and resolution** | Incident process | Per occurrence | Per retention policy |
| **Non-TLS endpoint register** | VCF Operations, CD-08 | Quarterly | Per retention policy |
| **Exceptions register** | Platform engineering | Quarterly review | Per retention policy |

## 11. Open items requiring confirmation

> ⚠️ **CONFIRM BEFORE DESIGN SIGN-OFF**
>
> Certificate and password management appears in the VCF column of Broadcom's feature comparison rather than the VMware vSphere Foundation column. That document presents the comparison as a table that does not extract reliably, so the position here is read from column placement rather than from a sentence. Before this model is funded, obtain written confirmation from the account team that certificate management across standalone vCenter instances is covered by the SKUs held.

- **E-01.** Written confirmation of entitlement for certificate management, including against integrated standalone instances.
- **E-02.** Supported scale: the maximum number of integrated standalone vCenter instances per VCF Operations instance.
- **E-03.** Capability delta between an integrated standalone instance and a converted one, stated by the vendor rather than inferred.
- **E-04.** Current port and network path requirements for integration, taken from the version-specific documentation at the time of build.
- **E-05.** Version-current knowledge base reference for STS recovery, to be embedded in Runbook C and re-verified at each platform upgrade.

## 12. References

- Managing Certificates at Scale with VMware Cloud Foundation 9.1.1, Broadcom, 14 October 2026.
- Broadcom TechDocs, Managing certificates in VMware Cloud Foundation.
- VCF 9.1.1 and VVF 9.1.1 feature comparison and upgrade paths.
- Scale, Simplify and Secure Your Private Cloud Operations with VCF 9.1, Broadcom, 5 May 2026.
- VCF 9.1 release notes index.
