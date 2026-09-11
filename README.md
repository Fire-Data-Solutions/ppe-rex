# PPE-REX Schema (Draft v0.1)

Open data standard for exchanging NFPA 1850 protective ensemble compliance records between fire departments, Independent Service Providers (ISPs), and manufacturers.

This layout deliberately mirrors the structure used by [NERIS's framework repo](https://github.com/ulfsri/neris-framework): **module files** define the record structures (the "questions"), **type files** define the controlled vocabularies those fields draw from (the "choice lists"). If you've worked with NERIS's schemas, this should be immediately familiar.

## Repo structure

```
ppe-rex/
├── README.md                          ← this file
├── 01-nfpa-1850-clause-mapping.md     ← compliance justification, clause-verified against NFPA 1851 §4.3
├── 02-design-philosophy.md            ← structured-core-vs-open-elsewhere rationale, contrasted with NFIRS
├── schemas/
│   ├── modules/                       ← core entity schemas (the "questions")
│   │   ├── ensemble-element.schema.json
│   │   ├── ensemble-configuration.schema.json  ← §4.3.4 contaminant-protection element groupings
│   │   ├── service-event.schema.json
│   │   ├── organization.schema.json
│   │   ├── person.schema.json
│   │   └── chain-of-custody-event.schema.json
│   └── types/                         ← controlled vocabularies (the "choice lists")
│       ├── element-type.yaml
│       ├── element-status.yaml
│       ├── event-type.yaml            ← annotated with mandated_by clause citations
│       ├── identifier-type.yaml       ← manufacturer- vs. organization-assigned identifiers
│       ├── alias-type.yaml            ← non-normative asset-tag/alias categories
│       ├── defect-code.yaml           ← suggested values, NOT a closed enum
│       ├── org-type.yaml
│       └── trigger-type.yaml
└── examples/
    ├── advanced-inspection-event.json
    ├── specialized-cleaning-co2-event.json  ← CO2 cleaning method, linked to a NERIS incident
    ├── ensemble-element-instance.json       ← manufacturer-assigned serial number
    ├── issuance-event.json                  ← §4.3.3(1)-(2)
    ├── rotating-exchange-element.json       ← §4.3.5 profile (lot-tracked, unassigned)
    ├── asset-tag-paired-coat.json           ← organization-assigned identifier + alias + pairing
    └── asset-tag-paired-trousers.json       ← the paired half of the above
```

## Conformance levels

- **Core Conformant** — implements every field in `schemas/modules/` marked `"required"`, using the exact field names, types, and enum values defined here. This is the level that maps 1:1 to NFPA 1850's record-keeping requirements (see the clause mapping doc).
- **Extended** — Core Conformant, plus vendor-specific fields under the `extensions` object, namespaced per vendor (see `service-event.schema.json` for the pattern). A Core Conformant reader can ignore any `extensions` block it doesn't recognize and still get a fully compliant record.

## Identity

`EnsembleElement.element_id` is the compound key `"{manufacturer_id}:{serial_number}"`. See the clause mapping doc's "Identity note" for the reasoning and known edge cases (unmarked legacy gear, post-repair serial changes).

## Governance & stewardship

This project is currently hosted under [Fire Data Solutions](https://github.com/Fire-Data-Solutions)'s GitHub organization, which is acting as **initial steward**, not owner. Fire Data Solutions makes TrackMyPPE, one of several commercial platforms in the current NFPA 1851/1850 PPE-tracking landscape this project is trying to build a neutral interchange format for.

What that means in practice:

- **Copyright is held by "PPE-REX Project Contributors,"** not by Fire Data Solutions — see `NOTICE`. Anyone who contributes retains standing alongside the initial steward, not underneath it.
- **The license is Apache 2.0** (see `LICENSE`, `CONTRIBUTING.md`), chosen specifically so no single contributor — steward included — can use a patent claim to restrict how competing vendors implement the standard.
- **The intended trajectory is away from single-vendor hosting**. See `CONTRIBUTING.md` and the architecture proposal's adoption path (pilot → neutral-body hosting via IJIS Institute or OASIS/NIEMOpen → NFPA 1850 informative annex submission).
- **Decisions about schema direction will be documented and discussable in the open** (issues, discussions), not made unilaterally by the hosting org.
  
## Status

This is a **draft proposal**, not a ratified standard. It has not been submitted to or endorsed by NFPA, USFA, FSRI, or any standards body. See `02-design-philosophy.md` for the design rationale and the architecture proposal doc for the intended path to formal adoption (build → pilot → neutral-body hosting → NFPA 1850 informative annex).

## License

Apache License 2.0 — see `LICENSE` and `NOTICE`. See `CONTRIBUTING.md` for the reasoning behind that choice and a note on how contributions may need to be re-licensed or covered by a CLA if/when this is pursued for formal NFPA submission.
