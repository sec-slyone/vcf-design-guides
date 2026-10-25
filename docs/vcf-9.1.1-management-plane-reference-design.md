# VCF 9.1.1 Management Plane Reference Design

**Sizing, availability, addressing and the Day 2 right-sizing task**

*Reference design · Mohammad "Sarge" Siddiqui · Version 1.0 · 25 October 2026*

---

## 1. Purpose and scope

This document is a reference design for the VMware Cloud Foundation Management Services plane at version 9.1.1. It covers deployment profile selection, the Day 0 versus Day N component decision, availability, network addressing, and the post-upgrade right-sizing task that an upgraded estate must perform explicitly in order to receive the 9.1.1 footprint reduction.

It is written so that the decisions can be lifted into a high level design with the organisation's own requirements substituted. Each decision is stated with its justification and its implication, so that a reviewer can disagree with a decision without having to reconstruct the reasoning behind it.

Out of scope: workload domain design, NSX design, vSAN design, and the VCF Automation functional design beyond its effect on management plane availability.

> ℹ️ **WHY THIS DESIGN EXISTS SEPARATELY FROM THE VCF DESIGN**
>
> The management plane is where VCF business cases are most often lost. The objection is rarely about capability and almost always about what the platform costs to run before a single workload is placed on it. 9.1.1 changes the arithmetic in two ways, and both of them need a decision recorded rather than a default accepted.

## 2. Requirements, assumptions, constraints and risks

### 2.1 Requirements

| ID | Requirement |
|---|---|
| **R-01** | The management plane survives the loss of a single node without loss of management function. |
| **R-02** | The management plane footprint is the minimum that satisfies R-01 and the functional requirements actually in use. |
| **R-03** | Management plane addressing is allocated before build and does not conflict with existing corporate addressing. |
| **R-04** | Footprint is reviewed after every platform upgrade rather than assumed to be current. |
| **R-05** | The design states explicitly which functional components are enabled, and the cost of each. |

### 2.2 Assumptions

- The target version is VCF 9.1.1. Several decisions here do not apply below it.
- The deployment is a management cluster rather than a consumption cluster.
- A corporate DNS service capable of forward and reverse records is available to the management network.
- The organisation has an existing IP address management process with a lead time that must be respected.

### 2.3 Constraints

- The right-sizing behaviour described in this design applies to VCF Management Services and not to VMware vSphere Foundation.
- VCF Automation deploys as a single appliance and has no high availability form factor at this release.
- An upgraded deployment does not inherit the reduced footprint automatically, which constrains the benefit realisation plan rather than the design itself.

### 2.4 Risks

| ID | Risk | Treatment |
|---|---|---|
| **K-01** | The estate upgrades to 9.1.1 and never adopts the reduced footprint, so the upgrade cost is paid and the benefit is not received. | MD-05. Put the right-sizing task in the same change record as the upgrade. |
| **K-02** | The internal service runtime range overlaps corporate addressing and is discovered during build. | MD-08. Raise with the network team during design, not during build. |
| **K-03** | Availability is assumed across the whole management plane when VCF Automation is a single appliance. | MD-07. State the gap in the design and in the service description. |
| **K-04** | A functional component is enabled during a proof of concept, never disabled, and silently blocks the right-sizing task. | MD-03. Record enabled components as a decision with an owner. |
| **K-05** | The right-sizing script is run outside a change window and interrupts management operations during the rolling node replacement. | MD-06. Classify as a normal change with a window. |

## 3. Sizing model

### 3.1 Day 0 and Day N

VCF 9.1.1 right-sizes Management Services components for a Day 0 deployment. Day 0 means the platform is deployed and the optional functional components have not been enabled. The two that matter to this design are Log Management and Real-time Metrics. Enabling either one changes the sizing requirement and makes the Day 0 right-sizing unavailable.

This is a design decision rather than an operational detail, because the cost of each component is paid continuously and the decision to enable one is usually taken once, informally, by whoever was building the environment that week.

| Component state | Right-sizing available | Design treatment |
|---|---|---|
| **Day 0, neither component enabled** | Yes | Default. Record as MD-03. |
| **Log Management enabled** | No | Justify the component against a requirement, or disable it. |
| **Real-time Metrics enabled** | No | As above. |
| **Both enabled** | No | Expect the largest footprint. Ensure both are traceable to a requirement. |

### 3.2 Profiles and the reduction

In a Simple, non-high-availability deployment the documented reduction is one fewer worker node. That node is 12 vCPU and 24 GB of memory. A three worker deployment becomes two.

> ⚠️ **ON PERCENTAGE FIGURES**
>
> Larger percentage reductions circulate around this release. A before and after total for vCPU, memory and storage stated directly by the vendor could not be located at the time of writing, so this design quotes the node count and the node size only. If a business case needs a percentage, derive it from your own measured before state rather than from a figure quoted second hand.

### 3.3 Small HA

Small HA is a deployment option introduced in 9.1.1 that provides high availability without moving to the next size profile. Before it existed, an availability requirement on the management plane forced a step up in profile, and a step up in profile forced a conversation about hosts. Small HA removes that step, which is the more consequential of the two 9.1.1 sizing changes even though it attracts less attention than the footprint reduction.

- In the VCF Installer JSON API it is a deployment size value of small_ha in the cluster specification.
- A Small non-HA deployment can be scaled to Small HA as a Day N operation, under Build, then Lifecycle, VCF Management, Components, VCF Services Runtime, Action, Scale.
- VCF Automation has no Small HA form factor and deploys as a single appliance. Small HA does not make the whole management plane highly available.

## 4. Design decisions

**MD-01 — Deployment profile**

| | |
|---|---|
| **Decision** | Small HA, unless a measured capacity requirement justifies a larger profile. |
| **Justification** | Satisfies R-01 and R-02 together. Before 9.1.1 these two requirements were in tension because availability forced a profile increase. Small HA resolves that tension and the previous reasoning for sizing up no longer holds. |
| **Implication** | Any design produced before 9.1.1 that sized up to achieve availability should be re-costed. An earlier decision not to proceed on footprint grounds was made against a different set of options. |

**MD-02 — High availability model**

| | |
|---|---|
| **Decision** | High availability is provided at the VCF Management Services layer. It is not claimed for components that deploy as a singleton. |
| **Justification** | Overstating availability in a design is worse than a documented gap, because the gap is at least visible to the service owner and to the risk register. |
| **Implication** | The service description must state that VCF Automation is a single appliance. See MD-07. |

**MD-03 — Functional component enablement**

| | |
|---|---|
| **Decision** | Log Management and Real-time Metrics are disabled unless each is traceable to a stated requirement with a named owner. |
| **Justification** | Each carries a continuous footprint cost and each blocks the Day 0 right-sizing. The default position should therefore be off, with enablement as an explicit decision. |
| **Implication** | Requires a register of enabled components with owners, reviewed at each upgrade. Components enabled during a proof of concept must be explicitly re-justified or disabled. |

**MD-04 — Target version**

| | |
|---|---|
| **Decision** | VCF Management Services is deployed at or upgraded to 9.1.1. |
| **Justification** | Small HA, the Day 0 right-sizing and the certificate management additions all require 9.1.1. An estate at 9.1.0 receives none of them. |
| **Implication** | Version becomes a prerequisite on the implementation plan rather than a preference. |

**MD-05 — Right-sizing on upgrade**

| | |
|---|---|
| **Decision** | For an upgraded deployment, the right-sizing procedure in Broadcom KB 455842 is executed as part of the same change record as the upgrade. |
| **Justification** | The right-sizing is applied automatically only to new 9.1.1 deployments. An upgraded deployment retains its previous worker sizing indefinitely and nothing reports this. Raising it as a separate change three months later competes against a lower appetite for touching the management plane again. |
| **Implication** | The upgrade change record acquires an additional task and an additional verification step. The change window must accommodate a rolling node replacement. |

**MD-06 — Right-sizing change classification**

| | |
|---|---|
| **Decision** | The right-sizing procedure is classified as a normal change executed within a maintenance window, not as a standard change or a routine task. |
| **Justification** | The script triggers a rolling replacement of worker nodes underneath a running management plane. Nodes are replaced one at a time, the script runs synchronously and waits for the rollout, and on a larger cluster this takes a meaningful period. |
| **Implication** | Requires a window, an approver and a verification step. Verification is the package deployment status reading Successful. |

**MD-07 — VCF Automation availability gap**

| | |
|---|---|
| **Decision** | The single-appliance nature of VCF Automation is recorded on the risk register and reflected in the service description and recovery time objective. |
| **Justification** | An availability requirement expressed as covering the management plane will be read as covering automation. It does not. |
| **Implication** | Recovery for VCF Automation depends on backup and restore rather than on platform availability, which sets its recovery time objective independently of the rest of the plane. |

**MD-08 — Internal service runtime addressing**

| | |
|---|---|
| **Decision** | The internal service runtime range is reviewed against corporate addressing during the design phase and a non-conflicting range is agreed before build. |
| **Justification** | The default internal range of 198.18.0.0/15 is documented in community material as a potential collision with customer addressing. A collision discovered during build stops the build. |
| **Implication** | Requires a network team review as a design gate. Record the agreed range in the design rather than leaving it at default. |

**MD-09 — Management network address allocation**

| | |
|---|---|
| **Decision** | Management network addresses are requested at the upper end of the published range during the design phase. |
| **Justification** | VCF 9.1 Management Services is described in community material as requiring between 12 and 30 addresses on the management network. Requesting the lower figure and discovering the upper one mid-build costs more than the addresses do. |
| **Implication** | Consumes a larger allocation than strictly required. Validate the actual figure against version-current documentation at build time. See E-02. |

**MD-10 — License Server name resolution**

| | |
|---|---|
| **Decision** | Forward and reverse DNS records for the License Server are created and verified before deployment begins. |
| **Justification** | The centralised License Server is described as mandatory from 9.1 for both VCF and vSphere Foundation environments and requires A and PTR records. Reverse records are the ones most often missing in an enterprise DNS estate. |
| **Implication** | Adds a DNS prerequisite with its own lead time. Verify the reverse record explicitly rather than assuming it follows the forward record. |

## 5. Network and addressing design

| Item | Design position | Owner |
|---|---|---|
| **Management network addresses** | Allocate to the upper end of the published range. See MD-09. | Network |
| **Internal service runtime range** | Agreed non-conflicting range confirmed in design. See MD-08. | Network and platform |
| **License Server forward DNS** | A record created and verified. | DNS |
| **License Server reverse DNS** | PTR record created and verified. Verify separately. | DNS |
| **Depot access** | Connected or offline depot decided in design. Offline requires its own artefact handling plan. | Platform |
| **Time synchronisation** | Consistent NTP sources across the management plane. | Platform |

> ⚠️ **THE ADDRESSING FIGURES IN THIS SECTION ARE COMMUNITY SOURCED**
>
> The 12 to 30 address range and the 198.18.0.0/15 internal range come from community documentation rather than from a vendor page located at the time of writing. The design treats them as planning figures that require confirmation against version-current documentation before build. See section 8.

## 6. Day 2 procedure: adopting the reduced footprint

This procedure applies only to a deployment that reached 9.1.1 by upgrade. A new 9.1.1 deployment is already right-sized and requires none of it.

### 6.1 Preconditions

| Precondition | Effect if not met |
|---|---|
| **Deployment is at 9.1.1 or above, reached by patching from 9.1 or 9.1.0.x** | Procedure not applicable. |
| **Log Management is not installed** | Script aborts with no changes made. |
| **Real-time Metrics is not installed** | Script aborts with no changes made. |
| **Cluster type is management, not consumption** | Script aborts with no changes made. |
| **Environment is VCF, not VVF** | Out of scope entirely. |
| **SSH access is enabled on the Services Runtime cluster** | Procedure cannot be executed. |
| **Change record exists with an approved window** | Procedure should not be executed. |

### 6.2 Execution

1. Confirm every precondition in section 6.1 and record the confirmation in the change.
2. Confirm a current backup of the management plane exists and that its restore has been validated.
3. Obtain the right-sizing script from Broadcom KB 455842 and copy it to a control plane node.
4. Connect to the control plane node over SSH as the vmware-system-user account and elevate.
5. Execute the script. It will present a warning and require explicit confirmation before proceeding.
6. Allow the rolling replacement to complete. The script runs synchronously. Do not interrupt it.
7. Verify the package deployment status reads Successful before closing the change.
8. Record the before and after node count and sizing as change evidence, which is also the benefit realisation evidence.

> ℹ️ **WHAT THE SCRIPT DETERMINES FOR ITSELF**
>
> The script reads the size profile, the high availability setting and whether this is a fleet or an instance deployment from the platform's own state rather than requiring the operator to describe the environment. It then applies the matching worker size, machine type and minimum replica count. Fleet deployments carry a higher minimum replica count than instance deployments, which is precisely the kind of distinction that would be missed if this were done by hand.

## 7. Validation checklist

| Check | Expected result |
|---|---|
| **Deployment profile** | Matches MD-01, or a recorded exception with justification. |
| **High availability** | Management Services highly available. VCF Automation singleton documented. |
| **Enabled functional components** | Matches the register under MD-03. |
| **Worker node count and size** | Matches a new 9.1.1 deployment of the same profile. |
| **Package deployment status** | Successful. |
| **Management network addresses** | Allocated, documented, no conflict. |
| **Internal service runtime range** | Agreed range in use, no overlap with corporate addressing. |
| **License Server DNS** | Forward and reverse records both resolve. |
| **Risk register** | MD-07 availability gap recorded. |

## 8. Open items requiring confirmation

- **E-01.** A vendor-stated before and after total for the management plane footprint, if one exists, to replace the node-count figure used in section 3.2.
- **E-02.** Version-current management network address requirement, to replace the community figure used in MD-09.
- **E-03.** Version-current default internal service runtime range and the supported method of changing it.
- **E-04.** Resource figures for the Small HA profile, which were not published at the time of writing.
- **E-05.** Whether a future release applies the Day 0 right-sizing to upgraded deployments automatically, which would retire MD-05.

## 9. References

- Broadcom KB 455842, Reducing footprint of VCF management services cluster that is only hosting Day 0 component.
- Customer Favorite Capabilities: Accelerating Your Path to VCF 9.1.1, Broadcom, 13 October 2026.
- William Lam, VCF 9.1.1: adopting the reduced VCF Management Services footprint after an upgrade.
- William Lam, 10 exciting enhancements in VMware Cloud Foundation 9.1.1.
- Broadcom TechDocs, VCF Management Services models.
- Digital Thought Disruption, VCF Operations fleet management, for the addressing figures noted as community sourced.
