# VCF Design Guides

Design documents for VMware Cloud Foundation 9.1.1 and the standalone vSphere estates around it.

Not summaries of what changed, and not checklists of what to look at. These record the decision itself: what to do, why, and what it costs you, in a form somebody can disagree with.

A design review does not need a restatement of the release notes. It needs a line that reads *we are deploying Small HA, because availability no longer requires a step up in profile, and the implication is that VCF Automation remains a single appliance and the service description has to say so.* That is what is in here.

## The documents

| Document | What it is |
|---|---|
| [Certificate Lifecycle Operating Model](docs/certificate-lifecycle-operating-model.md) | CA topology options with trade-offs, eight design decisions, RACI, renewal cadence, alert routing, and four runbooks including recovery from an expired vCenter STS certificate. |
| [VCF 9.1.1 Management Plane Reference Design](docs/vcf-9.1.1-management-plane-reference-design.md) | Ten design decisions on profile selection, Day 0 versus Day N components, Small HA, addressing, and the post-upgrade right-sizing task most estates never run. |
| [Standalone vCenter Integration Pattern](docs/standalone-vcenter-integration-pattern.md) | Bringing unconverted vCenter instances under VCF Operations. Architecture, cloud proxy placement, connectivity classes, six design decisions, failure modes, implementation sequence. |
| [VCF Upgrade Sequencing Assessment](docs/vcf-upgrade-sequencing-assessment.md) | A workbook, not a reference. Estate inventory, constraint capture, latest-start arithmetic, an eight-step decision path, and a sequencing template that maps onto change records. |

Word versions of all four are attached to the [latest release](../../releases/latest), for anyone who needs to hand one to a customer or track changes in review.

## How to use them

Take them. Change the organisation name, change the decisions you disagree with, delete the sections that do not apply, and put your own name on the front. They are written in a form that survives that treatment, because a design document that only works for the person who wrote it is not a design document.

The upgrade sequencing assessment in particular is meant to be completed in a room with the people who hold the constraints, including at least one application owner who will say no to something. It takes about ninety minutes.

## Two conventions worth knowing

**Open items are marked, not guessed.** Each document ends with a list of things that could not be established from public material: supported instance counts per VCF Operations instance, the capability delta between an integrated and a converted vCenter, Small HA resource figures, entitlement scope. These are left as questions on purpose. A design document that quietly guesses is worse than no design document, because the guess acquires authority from the format. If you take one of these to a customer, take the open items with it and get them answered in writing.

**Sources are attributed by type.** Where a figure comes from a vendor page it is cited. Where it comes from community material, the document says so in the text rather than only in the references. The management network address requirement and the internal service runtime range are both in that category: plan against them, verify before you build.

## Lifecycle dates used throughout

Confirmed by Broadcom in October 2026. Re-verify against the Product Lifecycle Matrix for your own SKUs before putting any of these in a steering pack.

| Release | End of general support |
|---|---|
| vSphere 8.0 and VCF 5.x | 11 October 2027 |
| VCF 9.0 | 17 September 2027 |
| VCF 9.1.1 | August 2028 |

VCF 9.0 goes out of general support 24 days before vSphere 8.0 does. A plan that moves to 9.0 now and 9.1 later therefore arrives at the cliff sooner, having paid for two upgrades. If a 9.0-first sequence appears in your roadmap, that is the question to ask before it is funded.

## Related

- [vcertscan](https://github.com/sec-slyone/vcertscan) — credential-free TLS certificate inventory. The tool that produces the "before" picture for the certificate operating model.
- [The Architect's Field Guide](https://www.mohammadsiddiqui.com) — the reasoning behind these documents, written up at length.

## Contributing

If you use one of these and it is wrong somewhere, open an issue. That is considerably more useful than a star.

## Licence

[Creative Commons Attribution 4.0 International](LICENSE). Use them commercially, adapt them, put your name on the derivative. Just do not claim the original was yours.

## Author

Mohammad "Sarge" Siddiqui — [mohammadsiddiqui.com](https://www.mohammadsiddiqui.com) · [LinkedIn](https://www.linkedin.com/in/sargesiddiqui)
