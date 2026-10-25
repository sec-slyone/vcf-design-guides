# Standalone vCenter Integration Pattern

**Bringing unconverted vCenter estates under VCF Operations management**

*Architecture pattern · Mohammad "Sarge" Siddiqui · Version 1.0 · 25 October 2026*

---

## 1. Intent

Bring vCenter Server instances that are not part of any VMware Cloud Foundation instance under central management by VCF Operations, without converting them, so that operational capability reaches the parts of an estate that a conversion programme will not reach for years and in some cases never.

The pattern is useful because the distinction it rests on is a commercial and organisational one rather than a technical one. Converting fourteen vCenter instances is a programme with a business case, a project manager and a multi-year timeline. Integrating fourteen vCenter instances is a change.

## 2. Applicability

| Condition | Requirement |
|---|---|
| **Source vCenter version** | 8.0 Update 3 or above for a standalone instance. VCF 5.2 instances are also in scope. |
| **VCF Operations version** | 9.1 for the extended reach. 9.1.1 for vCenter STS and VMCA root certificate management. |
| **Entitlement** | VCF rather than VMware vSphere Foundation. See section 9, this is the principal commercial dependency. |
| **Network** | Routable path from the VCF Operations collection point to each target vCenter. See section 5. |
| **Identity** | A service account per target, or a federated identity path. See section 6. |

> 🛑 **WHERE THIS PATTERN DOES NOT APPLY**
>
> An estate entitled to VMware vSphere Foundation rather than VCF cannot operate this pattern as written. Certificate and password management, lifecycle management, centralised tag management and fleet management are listed as VCF capabilities. Establish entitlement before design effort is spent, because this is the failure mode that wastes the most work.

## 3. Context and forces

Every large estate that has been running vSphere for more than a decade contains vCenter instances that exist for reasons nobody would choose today. One supports an application whose vendor certifies a specific version. One belongs to a business unit acquired four years ago. One sits in a facility that is scheduled for exit and therefore cannot justify investment. Two are in a DMZ and are managed by a different team under a different change process.

The forces in tension are these. The operational risk in these instances is high, because they receive the least attention. The appetite to invest in them is low, because they are all transitional. And a conversion to VCF, which would bring them operational capability, is exactly the kind of investment that cannot be justified for an instance that is leaving.

The pattern resolves that tension by separating the operational capability from the platform conversion. The capability arrives through integration, which is cheap and reversible. The conversion, if it ever happens, happens on its own timeline.

## 4. Logical architecture

The architecture has three tiers. A VCF Operations instance provides the management function and holds the configuration. A collection tier reaches the targets, which for instances outside the management network means a cloud proxy. The targets are the vCenter instances themselves, integrated through a workflow under the Integrations menu in VCF Operations.

| Tier | Component | Placement consideration |
|---|---|---|
| **Management** | VCF Operations 9.1.1 | Sized per the management plane reference design. The integration load from standalone targets is an input to that sizing. See E-01. |
| **Collection** | Cloud proxy | One per network zone that cannot reach the management network directly. Treat each DMZ, each acquired-business network and each remote site as a candidate zone. |
| **Target** | vCenter 8.0 U3 or above, or VCF 5.2 | Integrated, not converted. The instance continues to be operated by whoever operates it today. |

### 4.1 Cloud proxy placement options

| Option | Description | When it fits | Trade-off |
|---|---|---|---|
| **A. One proxy per network zone** | A proxy in each zone that cannot route to the management network. | Default. Segmented enterprise networks, DMZs, acquired estates. | Proxy count grows with zone count. Each proxy is itself a managed object with its own certificate and lifecycle. |
| **B. Shared proxy via firewall rules** | A smaller number of proxies reaching multiple zones through permitted paths. | Where the network team will permit the paths and the zones are not security boundaries. | Firewall rule sprawl, and a change in network policy can sever collection without warning. |
| **C. Direct collection, no proxy** | Targets on the management network reached directly. | Instances already inside the management boundary. | Only applicable to targets that are already there. |

> ℹ️ **DESIGN THE PROXY TIER AGAINST THE NETWORK, NOT THE VCENTER COUNT**
>
> The common error is to size the collection tier against the number of target instances. The determining factor is the number of network zones that cannot reach the management network, because that sets the proxy count. Fourteen vCenter instances in one zone need less collection infrastructure than four instances in four zones.

## 5. Design decisions

**PT-01 — Scope of integration**

| | |
|---|---|
| **Decision** | All vCenter instances at 8.0 Update 3 or above are integrated, including those with no conversion plan and those scheduled for decommission. |
| **Justification** | An instance scheduled for decommission in eighteen months still has to survive eighteen months, and it is the instance least likely to have a maintained certificate record. |
| **Implication** | The integration schedule will not resemble the conversion roadmap. State explicitly that it is an operational risk reduction activity rather than a VCF adoption activity. |

**PT-02 — Onboarding order**

| | |
|---|---|
| **Decision** | Instances with no conversion plan are onboarded first. Instances with a near-term conversion plan are onboarded last or skipped. |
| **Justification** | An instance converting within the next two quarters acquires the capability through conversion. Spending integration effort on it duplicates work. |
| **Implication** | Requires the conversion roadmap as an input to the integration schedule. Where no roadmap exists, treat every instance as having no conversion plan. |

**PT-03 — Collection tier**

| | |
|---|---|
| **Decision** | One cloud proxy per network zone that cannot reach the management network, per option A in section 4.1. |
| **Justification** | Keeps the collection path inside an existing network boundary rather than creating new permitted paths across one, which is both easier to approve and more stable over time. |
| **Implication** | Each proxy is a managed object with its own lifecycle, certificate and patching requirement. Account for them in operational effort, not just in build effort. |

**PT-04 — Credential handling**

| | |
|---|---|
| **Decision** | A dedicated service account per target instance, with the minimum privilege required, held in the organisation's existing secret store. |
| **Justification** | A shared account across integrated instances makes attribution impossible in an audit and turns a single credential rotation into an estate-wide change. |
| **Implication** | Increases account administration. Requires the privilege set to be established and documented. See E-03. |

**PT-05 — Discovery before action**

| | |
|---|---|
| **Decision** | The first integration of each instance is treated as a discovery exercise. Findings are reconciled before any managed action is enabled against that instance. |
| **Justification** | The discovered state and the documented state will differ. The difference is the most valuable output of the exercise and must not be silently overwritten. |
| **Implication** | Adds a reconciliation step per instance with a named owner. Expect this to take longer than the integration itself. |

**PT-06 — Operational ownership unchanged**

| | |
|---|---|
| **Decision** | Integration does not transfer operational ownership of the target instance. The team that operates it today continues to operate it. |
| **Justification** | The pattern delivers visibility and managed certificate lifecycle. Presenting it as a transfer of ownership turns a low-friction change into a negotiation about headcount. |
| **Implication** | Requires a clear statement in the service description of what central management does and does not do for an integrated instance. |

## 6. Network and connectivity

Port and path requirements change between releases and the version-current documentation is the only reliable source. Rather than reproduce a port table that will be wrong by the next release, this pattern records the connectivity classes that must be permitted, so that a firewall request can be raised in the right shape and then populated from current documentation.

| Class | From | To | Purpose |
|---|---|---|---|
| **Collection** | Cloud proxy | Target vCenter | Inventory, metrics and certificate operations against the target. |
| **Proxy registration** | Cloud proxy | VCF Operations | Registration and data return. Typically outbound from the proxy. |
| **Name resolution** | Cloud proxy and VCF Operations | Corporate DNS | Forward and reverse resolution for every participating endpoint. |
| **Time** | All participants | NTP sources | Certificate validation is time sensitive. Clock skew presents as an authentication fault. |
| **Certificate authority** | VCF Operations | Issuing CA | Required where the certificate operating model issues through an enterprise or subordinate CA. |

> ⚠️ **RAISE THE FIREWALL REQUEST AS CLASSES, THEN POPULATE FROM CURRENT DOCUMENTATION**
>
> A rejected or partially implemented firewall request is one of the most common causes of a stalled integration, and the cause is usually that the request was raised late and against an out-of-date port list. Raise the classes above during design so the request is in the queue, and populate the specific ports from the documentation for the version being deployed. See E-04.

## 7. Identity

- A target instance keeps its own identity source. Integration does not federate authentication for the target.
- The service account used for integration is a platform credential, not an administrator's named account, and is subject to the organisation's rotation policy.
- Where the target sits in a different identity domain, as is common with acquired estates, the service account is local to that domain. Do not create a trust relationship purely to simplify integration.
- Record the privilege set required by the integration in the design, so that a security review has something to assess. See E-03.

## 8. Capability delta: integrated versus converted

The honest position is that the public material establishes which capabilities exist and against which versions, but does not state cleanly which of them operate identically against an integrated standalone instance compared with a converted one. This table records what is established and marks the rest for confirmation rather than asserting it.

| Capability | Integrated standalone | Notes |
|---|---|---|
| **Certificate discovery and lifecycle** | Established | The explicit purpose of the 9.1 extension to standalone 8.0 U3 and above. |
| **vCenter STS and VMCA root management** | Established at 9.1.1 | The two additions that make the pattern worth operating. |
| **Renewal failure alerting** | Established at 9.1.1 | Includes a knowledge base reference on the alert. |
| **Non-TLS component visibility** | Established at 9.1.1 | View, manage and filter. |
| **Lifecycle management of the target** | Confirm | Listed as a VCF capability. Whether it operates against an integrated standalone target is not established here. See E-02. |
| **Password management** | Confirm | Listed alongside certificate management as a VCF capability. Scope against standalone targets not established. See E-02. |
| **Centralised tag management** | Confirm | See E-02. |
| **Supported instance count per VCF Operations instance** | Confirm | No figure located. This is a sizing input and therefore a design gate. See E-01. |

## 9. Failure modes

| Failure | Presentation | Mitigation |
|---|---|---|
| **Proxy loses its path to the target** | Collection gaps, stale data that still looks like data | Alert on collection staleness, not only on collection failure. Stale data that renders normally is worse than an obvious gap. |
| **Service account expires or is rotated out of band** | Integration fails silently at the next collection | Bring the account into the standard rotation process with the platform as a declared consumer. |
| **Target upgraded below or above a supported boundary** | Integration degrades or stops | Make platform integration a checklist item in the target's own upgrade change. |
| **Clock skew between proxy and target** | Authentication faults that read as credential problems | Monitor time sync across all participants. Check skew before troubleshooting credentials. |
| **Entitlement lapses or is found not to cover the capability** | The capability is withdrawn after the estate depends on it | Confirm entitlement in writing before go-live. See E-05. |
| **Proxy itself expires a certificate** | Collection stops across an entire zone | Include every proxy in the certificate inventory. A proxy is a managed object, not infrastructure plumbing. |

## 10. Implementation sequence

1. Confirm entitlement in writing. Do not proceed without it.
2. Confirm VCF Operations is at 9.1.1. At 9.1.0 the pattern delivers reach without the two certificates that justify it.
3. Inventory the estate: every vCenter, its version, its network zone, its identity domain, its conversion plan if any.
4. Classify targets. Above or below the 8.0 Update 3 floor, and in or out of a zone requiring a proxy.
5. Derive the proxy count from the zone count, not the target count.
6. Raise network requests as connectivity classes early, populated from version-current documentation.
7. Create service accounts with the documented privilege set.
8. Integrate one target in the least critical zone. Treat it as discovery, reconcile the findings, and record the elapsed effort as the estimate for the rest.
9. Enable alerting before enabling any managed action, in that order.
10. Proceed by zone rather than by criticality, so that each proxy is proven once and then reused.

## 11. Open questions requiring vendor confirmation

These are the questions this pattern could not answer from public material. They are listed here deliberately rather than guessed at, because each one is a design gate and a wrong assumption on any of them changes the architecture.

- **E-01.** Supported number of integrated standalone instances per VCF Operations instance, and the effect of integration load on management plane sizing.
- **E-02.** Authoritative statement of the capability delta between an integrated standalone instance and a converted one, covering lifecycle, password and tag management.
- **E-03.** Minimum privilege set required by the integration service account on the target instance.
- **E-04.** Version-current port and path requirements for cloud proxy to target and proxy to VCF Operations.
- **E-05.** Written confirmation that the entitlement held covers certificate and password management against integrated standalone instances.
- **E-06.** Behaviour when a target is upgraded past a boundary while integrated, and whether re-integration is required.

## 12. References

- Managing Certificates at Scale with VMware Cloud Foundation 9.1.1, Broadcom, 14 October 2026.
- Broadcom TechDocs, Managing certificates in VMware Cloud Foundation.
- VCF 9.1.1 and VVF 9.1.1 feature comparison and upgrade paths.
- Scale, Simplify and Secure Your Private Cloud Operations with VCF 9.1, Broadcom, 5 May 2026.
- Digital Thought Disruption, VCF Operations fleet management.
