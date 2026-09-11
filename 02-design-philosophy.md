# PPE-REX Design Philosophy: Structured Core, Open Everywhere Else

This is a short statement of a design decision that shapes most of the schema choices in this repo, worth writing down explicitly rather than leaving implicit in individual field definitions — so it can be agreed with, or argued against, as a decision rather than reverse-engineered from the JSON.

## The NFIRS lesson

NFIRS's data dictionary tried to enumerate a code for nearly every possible value across incident types, causes, property uses, detector types, and so on — a genuinely enormous controlled-vocabulary surface. That approach made sense for its era: if a value wasn't a code, it wasn't reliably queryable or aggregable across thousands of departments' worth of records. But it also meant the schema was perpetually behind reality (new causes, new equipment, new scenarios), and "OTHER — see narrative" became a very heavily used escape hatch in practice.

PPE-REX takes a deliberately different position, made possible partly by the source of the requirements being much narrower — this standard doesn't need to describe every kind of fire-service incident, only the record-keeping obligations of one standard (NFPA 1851 §4.3 and its neighboring chapters) — and partly by how much AI-assisted search and extraction has changed what unstructured text is actually worth. A well-written free-text note is now something a system can reliably search, summarize, and cross-reference, in a way that wasn't true when NFIRS's vocabulary was designed. Closed enums bought reliable querying at the cost of expressiveness and maintenance burden; that trade is worth revisiting now that querying unstructured text is no longer the weak link it used to be.

## The resulting rule of thumb

**Structured (closed enum, validated, required for conformance) when:**
- The field drives compliance logic that has to be machine-checkable — "does this element have a complete record per §4.3.3" needs `event_type` to be a known, closed set of values, or that query can't be answered reliably.
- The field is an identifier or foreign key — `element_id`, `org_id`, `event_id` relationships need to resolve exactly, not fuzzily.
- The field distinguishes record profiles that have genuinely different required content — `rotation_program` true/false changes which event types apply, so it has to be a clean boolean, not a description.

**Open (free text, or a documented-but-non-enforced "suggested values" list) when:**
- The field is descriptive rather than structural — defect findings, cleaning methods, repair descriptions. These vary by manufacturer, technology, and department practice in ways that would make a closed list either immediately incomplete or a maintenance burden that outpaces the standard's revision cycle (CO2 cleaning is the example already raised — a rigid vocabulary written today would need amending as cleaning technology keeps evolving).
- The field is department-specific convention with no compliance meaning attached — asset-tag aliases, informal pairing between elements, general notes.

## What this means concretely in this schema

- `event_type`, `element_type`, `org_type`, `identifier_type`, `element_status`, `protection_type` — closed enums (`type: string, enum: [...]` in JSON Schema, referencing the corresponding type file).
- `defect_codes`, `method` (cleaning method), `disposal_method` — open strings, with a type file documenting common/suggested values for consistency, but never enforced as a closed list. A validator checking "Core Conformant" status should not reject a record for using a value outside the suggested list here.
- `notes` fields exist at both the element and event level, treated as first-class rather than an afterthought — not a fallback for "the schema didn't anticipate this," but the intended place for anything that doesn't need to be queried structurally.
- `extensions` and `aliases` exist for the same underlying reason from a different angle: rather than the standard trying to anticipate every vendor's or department's needs up front, it defines a stable place for that variation to live without threatening interoperability of the core record.

## The tradeoff, stated honestly

This approach makes cross-department statistical aggregation on the open fields weaker than a fully-coded system would provide — you can't reliably produce a clean bar chart of "defect types nationwide" from free text the way you could from a closed enum, at least not without an extraction/normalization step. That's a real cost, and it's worth the committee weighing explicitly rather than PPE-REX assuming the tradeoff is obviously correct. The bet here is that (a) NFPA 1851's actual record-keeping requirement is about *departmental* compliance and auditability, not national statistical aggregation the way NFIRS/NERIS's mission is, so this tradeoff matters less for this standard's core purpose than it would for an incident-reporting system, and (b) where aggregation is wanted later, AI-assisted extraction over the free-text fields is a more realistic path to it than trying to get every vendor and department to agree on a closed vocabulary up front.
