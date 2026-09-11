# Contributing to PPE-REX

## License

This project is licensed under **Apache License 2.0** (see `LICENSE`). By submitting a contribution (a pull request, issue with proposed schema text, etc.), you agree it's licensed under the same terms per License §5 ("Submission of Contributions"), unless you state otherwise explicitly and in writing.

This project is explicitly meant to be implemented by commercial, competing vendors (department software platforms, ISPs, manufacturers). Apache 2.0's patent grant (§3) means anyone contributing schema text, type-vocabulary entries, or code also grants implementers a license to any patent claims they'd necessarily need to infringe to use that contribution, and that grant terminates for anyone who turns around and sues an implementer over it. 
## Goal

The goal stated for this project is eventual submission to NFPA; likely as an informative annex to NFPA 1850, or via a neutral standards-body host (IJIS Institute, OASIS/NIEMOpen) as an intermediate step. **NFPA's own IP process may require a separate rights grant from contributors**, on top of this repository's Apache 2.0 license, before it will incorporate or reference this material formally. That typically takes the form of a Contributor License Agreement (CLA) specific to the target standards body, executed once the project is far enough along to pursue that path seriously.

This isn't in place yet. If/when it is, it will be documented here, and it will not retroactively change the license on contributions already made under Apache 2.0. A CLA at that point would need to be negotiated for existing contributions or would apply prospectively, and that's a decision for whoever is stewarding the project at that stage, ideally with actual legal counsel involved rather than guidance from this document.

## Scope of contributions

Useful contributions at this stage:
- Clause-verification work — mapping additional NFPA 1851/1850 chapters (Inspection, Cleaning, Repair, Storage, Retirement) against actual clause text, the way §4.3 has been done
- Schema review — identifying gaps, ambiguities, or edge cases in the core entity models
- Reference implementations — a validator, an OpenAPI spec for the exchange layer, example integrations
- Domain expertise — particularly on the `defect-code.yaml` vocabulary, which is explicitly marked as illustrative rather than complete, and would benefit from input from people who actually perform advanced inspections

Please open an issue before a large schema-restructuring PR. Module/type file changes ripple through the clause-mapping table and examples, and any change in direction needs review and consensus.
