# VCF Upgrade Sequencing Assessment

**A structured way to reach a defensible upgrade sequence and defend it in a steering forum**

*Assessment workbook · Mohammad "Sarge" Siddiqui · Version 1.0 · 25 October 2026*

---

## 1. How to use this

This is a working document, not a reference document. It is designed to be completed in a room with the people who actually hold the constraints: the platform team, the change function, and at least one application owner who will say no to something.

Completed properly it takes about ninety minutes and produces three things. A sequence with dates. A statement of the latest possible start date before the estate is out of support. And a list of the constraints that drove the sequence, which is what you need when somebody senior asks why you are not doing it faster or why you are not waiting.

Sections A and B are filled in. Section C is arithmetic. Section D is a decision path. Sections E and F are the output.

## 2. What this adds over the VCF Upgrade Planner

The VCF Upgrade Planner maps current state to 9.1.1, flags prerequisites and interoperability, and produces a sequenced roadmap with safe stopping points, exportable into Jira or ServiceNow. Use it. It does the part that is tedious and error prone by hand.

What it does not know is everything in section B of this document.

| The Planner covers | This assessment adds |
|---|---|
| **Component versions and interoperability** | Change freezes, including the ones not yet published |
| **Prerequisites for the target version** | Third-party agents whose vendor has not certified the target |
| **A sequenced technical roadmap** | Application owners who will not accept an outage this half |
| **Safe stopping points** | Which stopping point is acceptable to the business, which is a different question |
| **Export into Jira or ServiceNow** | Mapping onto your actual change classes and approval lead times |
|   | Entitlement and licensing position |
|   | Who is available. Upgrades are executed by people with leave, projects and notice periods |

> ℹ️ **USE BOTH, IN THIS ORDER**
>
> Run the Planner first so the technical sequence is established and correct. Then run this assessment against it, because every constraint in section B either moves a date or removes an option. A roadmap that has not been tested against those constraints is a proposal, not a plan.

## 3. Section A: estate inventory

One row per vCenter instance or VCF instance. Complete every column. An unknown is a finding, not a blank.

| Instance | Current version | Build / patch level | Hosts | Workload criticality | Conversion plan |
|---|---|---|---|---|---|
|   |   |   |   |   |   |
|   |   |   |   |   |   |
|   |   |   |   |   |   |
|   |   |   |   |   |   |
|   |   |   |   |   |   |
|   |   |   |   |   |   |
|   |   |   |   |   |   |
|   |   |   |   |   |   |

Supplementary questions, answered once for the estate:

| Question | Answer |
|---|---|
| **Are any instances below vSphere 8.0 Update 3?** |   |
| **Are any instances on a build affected by a back-in-time restriction? See section 6.** |   |
| **Is VCF Operations deployed, and at what version?** |   |
| **Is the estate entitled to VCF or to VMware vSphere Foundation?** |   |
| **Which instances share a vCenter Single Sign-On domain?** |   |
| **Are there deferred security patches, and why were they deferred?** |   |

## 4. Section B: constraint capture

This is the section that changes the answer. Capture it before anyone proposes a date.

### 4.1 Calendar constraints

| Constraint | Period | Hard or negotiable | Owner |
|---|---|---|---|
| Published change freeze |   |   |   |
| Financial year end |   |   |   |
| Peak trading or seasonal period |   |   |   |
| Audit or regulatory examination window |   |   |   |
| Other platform programme competing for windows |   |   |   |
| Key personnel unavailability |   |   |   |

### 4.2 Technical constraints

| Constraint | Detail | Blocking? | Owner |
|---|---|---|---|
| Backup product certification for target version |   |   |   |
| Monitoring and agent certification |   |   |   |
| Security tooling certification |   |   |   |
| Hardware compatibility for target version |   |   |   |
| Driver and firmware baseline |   |   |   |
| Storage array interoperability |   |   |   |
| Network or load balancer dependency |   |   |   |

### 4.3 Organisational constraints

| Constraint | Detail | Owner |
|---|---|---|
| Application owner who will not accept an outage |   |   |
| Vendor-certified version requirement for an application |   |   |
| Change class and approval lead time for this work |   |   |
| Entitlement or commercial dependency |   |   |
| Resource availability and competing commitments |   |   |

> ⚠️ **THE QUESTION THAT FINDS THE REAL CONSTRAINT**
>
> Ask each application owner what would have to be true for them to accept an outage, rather than whether they will accept one. The first question produces a no. The second produces a condition, and a condition can be planned against.

## 5. Section C: the date arithmetic

Lifecycle dates confirmed by Broadcom in October 2026. Re-verify against the Product Lifecycle Matrix for the exact SKUs held before putting any of these in a steering pack, because entitlements differ and the account team's date is the one that counts.

| Release | End of general support | Months remaining at 25 Oct 2026 |
|---|---|---|
| **vSphere 8.0 and VCF 5.x** | 11 October 2027 | 11.5 |
| **VCF 9.0** | 17 September 2027 | 10.7 |
| **VCF 9.1.1** | August 2028 | Approximately 22 |

> 🛑 **THE INVERSION, AND WHY A 9.0-FIRST PLAN FAILS**
>
> VCF 9.0 goes out of general support 24 days before vSphere 8.0 does. A plan that moves to 9.0 now and 9.1 later therefore moves off a release supported for roughly another year onto one that expires sooner, and then requires the whole exercise again. Broadcom's own position is that stopping at VCF 9.0 does not extend the support runway. If a 9.0-first sequence appears in your plan, this is the question to ask before it is funded.

### 5.1 Latest start calculation

Work backwards. Complete this for the estate, then for the single most constrained instance, because they will differ.

| Step | Input | Value |
|---|---|---|
| **1** | End of general support date for your SKU |   |
| **2** | Contingency buffer, recommended one full change cycle |   |
| **3** | Number of instances to upgrade |   |
| **4** | Elapsed time per instance including soak period |   |
| **5** | Instances that can run in parallel |   |
| **6** | Total elapsed time required, step 3 divided by step 5, times step 4 |   |
| **7** | Blackout weeks from section 4.1 falling inside that period |   |
| **8** | Latest start date, step 1 minus steps 2, 6 and 7 |   |

If step 8 produces a date that has already passed, that is the finding. It is also the most useful sentence this assessment can produce, and it is considerably better to produce it now than in nine months.

## 6. Section D: decision path

Work through in order. Stop at the first outcome that applies.

| # | Question | If yes | If no |
|---|---|---|---|
| **D1** | Is the estate entitled to VCF rather than VMware vSphere Foundation? | Continue to D2. | The 9.x target and several capabilities in the business case need re-examining. Resolve before continuing. |
| **D2** | Are all instances at 8.0 Update 3 or above? | Continue to D3. | Instances below the floor need an interim upgrade. Add to the sequence as a separate stage. |
| **D3** | Are any instances on 8.0U3j, 8.0U3k, VCF 9.0.2 EP2 or VCF 5.2.4? | These now have a supported path forward to 9.1.1. Continue to D4, and see the note below. | Continue to D4. |
| **D4** | Were security patches deferred to protect an upgrade path? | That reason has been removed. Revisit the deferral now, and see the note below. | Continue to D5. |
| **D5** | Does any plan on the table stop at 9.0? | Challenge it using section 5. Then continue to D6. | Continue to D6. |
| **D6** | Does the latest start date from 5.1 fall in the past? | Escalate. The sequence cannot be delivered within support and the decision is now about accepting risk or adding resource. | Continue to D7. |
| **D7** | Is there a blocking constraint in section 4.2? | Sequence around it and record the dependency with an owner and a date. | Continue to D8. |
| **D8** | Can the most constrained instance meet the sequence? | Proceed to section 7. | Split it out as its own workstream rather than letting it set the pace for everything else. |

> ⚠️ **ON THE BACK-IN-TIME RESTRICTION**
>
> Broadcom lifted the restriction in October 2026, so estates on 8.0U3j, 8.0U3k, VCF 9.0.2 EP2 and VCF 5.2.4 can now move forward to 9.1.1 directly, and patching no longer costs the upgrade path. Broadcom's own post warns that a future out-of-band patch could introduce a new restriction. If your estate benefits from the lift, that is an argument for moving sooner rather than later, and it is worth stating in the plan so the urgency is traceable to a reason rather than to a preference.

## 7. Section E: risk position

Score each line one to five, where five is the worst. This is not a model. It is a way of making the conversation about the same things in the same order every time it is held.

| Dimension | Score | Evidence |
|---|---|---|
| Time remaining against latest start date |   |   |
| Number of instances and parallelism available |   |   |
| Third-party certification readiness |   |   |
| Change window availability |   |   |
| Application owner readiness |   |   |
| Resource availability |   |   |
| Entitlement and commercial clarity |   |   |

| Total | Position | Action |
|---|---|---|
| **7 to 13** | Comfortable | Proceed on the sequence in section 8. Review quarterly. |
| **14 to 23** | Tight | Proceed, but the contingency buffer in 5.1 step 2 is not real. Remove a constraint or add resource. |
| **24 to 35** | Not deliverable as planned | Escalate now with the latest start date and the two constraints scoring highest. Do not escalate with a status colour. |

## 8. Section F: sequencing template

One row per stage. A stage is something that produces a supported, stoppable state. If a stage cannot be stopped at, it is not a stage, it is a step inside one.

| Stage | Scope | Window | Change ref | Stoppable state | Rollback | Owner |
|---|---|---|---|---|---|---|
|   |   |   |   |   |   |   |
|   |   |   |   |   |   |   |
|   |   |   |   |   |   |   |
|   |   |   |   |   |   |   |
|   |   |   |   |   |   |   |
|   |   |   |   |   |   |   |
|   |   |   |   |   |   |   |
|   |   |   |   |   |   |   |
|   |   |   |   |   |   |   |

### 8.1 Mapping to change records

- One change record per stage, not one per instance. A change record that covers a stoppable state is reviewable. One that covers a single host is noise.
- Reference the stage in the change title so the sequence is visible to an approver who has not read this document.
- Record the rollback position as the previous stoppable state, not as "restore from backup". The two are different and only one of them is a plan.
- Where the Planner has produced an export, attach it to the change rather than retyping its content, so that a reviewer can see the technical dependency chain.
- Carry the latest start date from 5.1 in every change record in the sequence. It is the only thing that makes a delay visible as a delay rather than as a reschedule.

## 9. Section G: outputs

| Output | Goes to | Format |
|---|---|---|
| **Latest start date with working** | Steering forum | One slide, section 5.1 reproduced |
| **Sequence with stages and dates** | Change function and delivery | Section 8 table |
| **Constraint register with owners** | Programme risk register | Section 4, lines marked blocking |
| **Risk position and escalation items** | Sponsor | Section 7, with the two highest-scoring lines named |
| **Open commercial questions** | Account team | Written request, written answer |

## 10. Reference dates and sources

Confirmed by Broadcom in October 2026. Re-verify before use.

- vSphere 8.0 and VCF 5.x end of general support: 11 October 2027.
- VCF 9.0 end of general support: 17 September 2027.
- VCF 9.1.1 general support to August 2028.
- Back-in-time restriction lifted for 8.0U3j, 8.0U3k, VCF 9.0.2 EP2 and VCF 5.2.4, with a vendor caution that a future out-of-band patch could reintroduce it.

- Customer Favorite Capabilities: Accelerating Your Path to VCF 9.1.1, Broadcom, 13 October 2026.
- VCF 9.1.1 and VVF 9.1.1 feature comparison and upgrade paths.
- VCF 9.1 release notes index and VCF 9.1.1.0 release notes.
- Broadcom Product Lifecycle Matrix, for SKU-specific dates.
