# NFPA 1850 Clause → PPE-REX Field Mapping

**Status:** Draft. The §4.3 (Records) section below is mapped directly against the actual NFPA 1851 clause text (§4.3.1–4.3.5), which the current 1850 consolidation carries forward for structural/proximity ensembles. All other chapters (Selection, Inspection, Cleaning, Repair, Storage, Retirement) remain topic-mapped pending the same clause-level verification — flagged per section below. Nothing here reproduces NFPA's text; requirements are paraphrased and mapped to schema fields, with the clause number cited for traceability.

**How to read this table:** each row ties a record-keeping obligation to the specific PPE-REX entity/field that satisfies it. A validator can walk this table mechanically to confirm a given implementation covers every required data point — this is what makes a "Core Conformant" claim defensible.

---

## §4.3 Records (clause-verified)

### §4.3.1–4.3.2 — Scope

The organization must compile and maintain records on its structural and proximity ensembles/elements, **including rental or loaner elements** (§4.3.1, §4.3.2).

| PPE-REX entity | Field(s) |
|---|---|
| `EnsembleElement` | Exists as a record regardless of `owning_organization_id` relationship type — no schema distinction between owned and rental/loaner elements. Rental/loaner status, if tracked, belongs in `extensions` (department-specific business logic) rather than core, since §4.3.2 just says the *same* records apply, not that a different record type is needed. |

### §4.3.3 — Minimum record content (individually assigned elements)

| Clause | Requirement (paraphrased) | PPE-REX entity | Field(s) |
|---|---|---|---|
| (1)–(2) | Person to whom the element is issued; date and condition at issuance | `ServiceEvent` | **New:** `event_type = ISSUANCE`, `performed_by_person_id` *(recipient, not issuer — see note below)*, `performed_date`, `result.outcome` (condition at issue) |
| (3) | Manufacturer and model name or design | `EnsembleElement` | `manufacturer_id`, `model` |
| (4) | Manufacturer's identification number, lot number, **or** serial number | `EnsembleElement` | **Revised:** `identifier.identifier_type` (enum: `SERIAL_NUMBER`, `LOT_NUMBER`, `IDENTIFICATION_NUMBER`) + `identifier.value` — see identity note below. The standard explicitly allows any of the three; the schema no longer assumes a serial number always exists. |
| (5) | Month and year of manufacture | `EnsembleElement` | **Revised:** `manufacture_date` format changed to `YYYY-MM` (month/year only) — the standard doesn't require day-level precision, so the schema shouldn't imply false precision. |
| (6) | Date(s) and findings of advanced inspection(s) | `ServiceEvent` | `event_type = ADVANCED_INSPECTION`, `performed_date`, `result` |
| (7) | Date(s) and findings of advanced cleaning, disinfection/sanitization, or specialized cleaning | `ServiceEvent` | `event_type` ∈ `{ADVANCED_CLEANING, DISINFECTION_SANITIZATION, SPECIALIZED_CLEANING}` — **split into three distinct event types**, matching the standard treating these as related but separate actions rather than one bucket |
| (8) | Reason(s) for, and who performed, advanced cleaning/disinfection/sanitization/specialized cleaning | `ServiceEvent` | **New:** `trigger.reason` (free text — *why* cleaning was performed, distinct from `trigger.trigger_type`, the structured category) + `performed_by_org_id`/`performed_by_person_id` (*who*) |
| (9) | Date(s) of repair(s), who performed them, brief description | `ServiceEvent` | `event_type = REPAIR`, `performed_date`, `performed_by_org_id`, `result.notes` |
| (10) | Date of retirement | `EnsembleElement` + `ServiceEvent` | `retirement_date` (element-level summary) + `event_type = RETIREMENT` (event-level record) |
| (11) | Date and method of disposal | `ServiceEvent` | `event_type = DISPOSITION`, `performed_date`, `result.disposal_method` |

**Note on (1)–(2):** the standard requires recording who an element was issued *to*; it's silent on tracking who performed the issuance transaction. `ServiceEvent.performed_by_person_id` is repurposed here to mean "the recipient" for `ISSUANCE` events specifically — worth flagging as a naming inconsistency to resolve before this goes further (an `ISSUANCE` event may need its own `issued_to_person_id` field rather than overloading `performed_by_person_id`, to avoid ambiguity between "who did the action" and "who received the item").

### §4.3.4 — Contaminant-protection ensembles (liquid/particulate)

Elements with liquid and particulate contaminant protection require §4.3.3's full record set, **plus** a documented list of the specific required elements and interface components that make up that contaminant-protected ensemble (e.g., a particulate hood is only rated as part of a specific coat/pant/interface combination).

| PPE-REX entity | Field(s) |
|---|---|
| **New entity:** `EnsembleConfiguration` | Groups a set of `element_id`s that together constitute a certified contaminant-protection configuration, plus the protection type (`LIQUID`, `PARTICULATE`, `BOTH`). See `schemas/modules/ensemble-configuration.schema.json`. This is a relationship record, not a property of any single element — a coat doesn't "know" which hood it's certified with; the configuration record does. |

### §4.3.5 — Rotating exchange program (new clause; unassigned elements)

Where elements are kept in a rotating pool rather than assigned to an individual, the organization must still run a clean/inspect program and keep records — same core content as §4.3.3, **minus** the person-issued fields, **plus** a record of when the element is returned to rotating inventory.

| PPE-REX entity | Field(s) |
|---|---|
| `EnsembleElement` | **New:** `rotation_program` (boolean) — flags that this element follows the §4.3.5 profile rather than §4.3.3's assigned-member profile. When `true`, `assigned_to_person_id` is expected to remain `null` between uses. |
| `ServiceEvent` | Same event types as §4.3.3 items (3)–(9) above, **minus** `ISSUANCE`, **plus new:** `event_type = RETURNED_TO_ROTATION_INVENTORY`, `performed_date` — satisfies §4.3.5(6). |

---

## Selection (topic-mapped, clause numbers not yet verified)

| Topic area | Record-keeping obligation | PPE-REX entity | Field(s) |
|---|---|---|---|
| Certification compliance at selection | Selected elements must meet applicable NFPA product standard (1970, etc.) | `EnsembleElement` | `certification_refs` |
| Risk assessment basis | Selection informed by documented risk assessment (informative annex guidance) | *Out of core scope* | Org-level SOP attachment, not per-element |

## Inspection (topic-mapped, clause numbers not yet verified)

| Topic area | Record-keeping obligation | PPE-REX entity | Field(s) |
|---|---|---|---|
| Routine inspection | Performed by wearer at/after each use | `ServiceEvent` | `event_type = ROUTINE_INSPECTION` — **note:** not explicitly named in §4.3.3's minimum record list; included in PPE-REX as a recommended-core event type since departments generally track it and Chapter 6's substantive requirements likely call for it, but it isn't §4.3-mandated the way advanced inspection is |
| Inspection escalation | Failed routine inspection should trigger advanced inspection | `ServiceEvent` | `trigger.trigger_type = ROUTINE_INSPECTION_FAILURE`, `trigger.related_event_id` |
| Defect documentation | Specific defect categories documented when found | `ServiceEvent` | `result.defect_codes[]` |

## Repair, Storage, Retirement/Disposition, ISP Verification

Unchanged from prior draft — topic-mapped, not yet clause-verified. Repair and Retirement/Disposition content is now largely redundant with §4.3.3(9)–(11) above, which takes precedence as the clause-verified source.

---

## Identity note (revised again)

`EnsembleElement.identifier` is a structured object, per §4.3.3(4)'s "identification number, lot number, or serial number" — now extended with a fourth, PPE-REX-specific option for items the manufacturer doesn't individually number at all:

```json
"identifier": {
  "identifier_type": "ORGANIZATION_ASSIGNED",
  "value": "77102-G-014",
  "assigned_by": "ORGANIZATION"
}
```

Four `identifier_type` values (see `schemas/types/identifier-type.yaml`):

- `SERIAL_NUMBER`, `LOT_NUMBER`, `IDENTIFICATION_NUMBER` — all manufacturer-assigned (`assigned_by = MANUFACTURER`)
- `ORGANIZATION_ASSIGNED` — the department assigns its own number, typically for items like gloves that ship with no individual manufacturer identifier (`assigned_by = ORGANIZATION`)

**`assigned_by` changes `element_id`'s namespace, not just its metadata.** `element_id` is constructed as `"{namespace}:{identifier.value}"`, where `namespace` = `manufacturer_id` when manufacturer-assigned, or `owning_organization_id` when organization-assigned:

```
MSA:AB-2024-88213                    ← manufacturer-namespaced, globally unique
org_dept_springfield_fd:C104         ← organization-namespaced, unique only within that org
```

This is the mechanism that actually resolves the uniqueness problem you flagged: two departments can both use "C104" as a department-assigned number without collision, because the org_id prefix disambiguates them — the number itself only needs to be unique *within* the assigning department's own scheme, not globally. The tradeoff worth flagging to the committee: if gear with an organization-assigned identifier is ever transferred to a different department (mutual aid, surplus transfer), the receiving department inherits a foreign namespace prefix it doesn't control — worth deciding whether transfer should trigger re-identification or whether the original namespace simply persists as a historical artifact.

## Aliases and pairing (non-normative, but first-class)

NFPA 1851 doesn't require any of this — it's included because the practice (department asset tags like "C104"/"P104," often encoding garment type and an informal pairing) is common enough that leaving it out would just push every vendor to reinvent it differently in their own `extensions` namespace, defeating some of the point of having a shared standard. `EnsembleElement.aliases[]` carries these as explicitly non-normative, department-scoped values; `paired_element_ids[]` carries informal "issued as a set" relationships. Both are distinct from `EnsembleConfiguration` (§4.3.4), which is a *certified* grouping with compliance weight — aliases and pairing carry none.

## Specialized cleaning methods (§4.3.3(7)-(8))

The standard requires recording the reason for and who performed specialized cleaning, but doesn't name specific methods — appropriately, since methods evolve (CO2 cleaning is increasingly treated as a preferred approach for certain contamination levels, alongside established wet-extraction methods). `ServiceEvent.result.method` is free text with documented common examples rather than a closed list, consistent with the design philosophy in `02-design-philosophy.md` — a fixed vocabulary here would need revising every time cleaning technology shifted.
