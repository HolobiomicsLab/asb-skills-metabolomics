---
name: pubchem-formula-reference-construction
description: Use when setting up or recalibrating FSA's reference formula list — e.g.
  rebuilding the CHONSP-only list of 928,369 PubChem formulas, or extending it to
  2,613,203 formulas that include halogens (F, Cl, Br, I) for halogen-containing metabolite
  searches.
when_to_use_negative:
- When the goal is to rank MS/MS spectra themselves (use FSA's scoring step, not the
  reference-list construction).
- When formulas must include elements beyond C, H, O, N, S, P, F, Cl, Br and I — the
  article does not document a validated extraction for other elements.
- When the task requires structural information or MS/MS spectral databases — this
  skill builds a formula-only reference list and contributes no structural or spectral
  content.
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: FSA
  role: Consumes the PubChem-derived formula/mass reference list as the candidate-formula
    source for ranking molecular formulas per MS/MS spectrum.
  repo: https://github.com/kslynn128171/FSA
- name: PubChem
  role: Source database from which molecular formulas and their monoisotopic masses
    are extracted to build the reference formula list.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s11306-025-02379-0
    title: Automated metabolite formula ranking using formula subset analysis for
      LC-MS/MS-based metabolomics
  source_content_hash: sha256:3fa4491124b5fa1c9fe08da608c32f675303afd0abc9b75dd64257351fc316c7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kslynn128171/FSA
  repo_url: https://github.com/kslynn128171/FSA
  tools:
  - FSA
  - PubChem
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# pubchem-formula-reference-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs the reference formula list used by FSA by extracting molecular formulas and monoisotopic masses from the PubChem compound database, optionally extended to include halogen elements (F, Cl, Br, I) beyond the validated CHONSP scope.

## When to use

Use when setting up or recalibrating FSA's reference formula list — e.g. rebuilding the CHONSP-only list of 928,369 PubChem formulas, or extending it to 2,613,203 formulas that include halogens (F, Cl, Br, I) for halogen-containing metabolite searches.

## When NOT to use

- When the goal is to rank MS/MS spectra themselves (use FSA's scoring step, not the reference-list construction).
- When formulas must include elements beyond C, H, O, N, S, P, F, Cl, Br and I — the article does not document a validated extraction for other elements.
- When the task requires structural information or MS/MS spectral databases — this skill builds a formula-only reference list and contributes no structural or spectral content.

## Inputs

- PubChem compound database snapshot (molecular formulas + monoisotopic molecular weights)
- Element scope definition (CHONSP, optionally with halogens F, Cl, Br, I)

## Outputs

- Reference formula list of 928,369 CHONSP entries (or 2,613,203 entries when halogens are included), each with a monoisotopic mass
- Element/formula reference list consumed by FSA for candidate formula ranking

## How to apply

Download a PubChem compound database snapshot and extract each compound's molecular formula together with its monoisotopic molecular weight, retaining formulas composed of C, H, O, N, S and P for the base list (928,369 entries) or additionally retaining those containing F, Cl, Br and I for the halogen-extended list (2,613,203 entries). Verify the extracted entry count against the paper's reported counts (928,369 for CHONSP; 2,613,203 for halogen-inclusive) before feeding the list into FSA. Configure FSA to use this formula/mass reference list when ranking candidate molecular formulas per MS/MS spectrum. Note the paper used a PubChem snapshot with 109,383,767 compounds, whereas a 2022/11/27 snapshot contains 112,419,402 compounds — a larger snapshot will shift the formula counts and may alter ranking metrics. This list is purely formula-and-mass based; FSA does not require training data or MS/MS databases, which is why PubChem is sufficient as the source.

## Related tools

- **FSA** (Consumes the PubChem-derived formula/mass reference list as the candidate-formula source for ranking molecular formulas per MS/MS spectrum.) — https://github.com/kslynn128171/FSA
- **PubChem** (Source database from which molecular formulas and their monoisotopic masses are extracted to build the reference formula list.)

## Evaluation signals

- The extracted CHONSP-only formula list contains 928,369 entries.
- The halogen-extended formula list contains 2,613,203 entries.
- Every formula in the reference list carries a monoisotopic molecular weight.
- No formula in the CHONSP-only list contains elements outside C, H, O, N, S and P; no formula in the extended list contains elements outside C, H, O, N, S, P, F, Cl, Br and I.
- Feeding the list into FSA yields rank#1 accuracy of ~85.28% (CHONSP, 0.024 s/spectrum) or ~77.84% (halogen-extended, 0.16 s/spectrum) on the paper's non-lipid test datasets.

## Limitations

- The paper used a PubChem snapshot of 109,383,767 compounds; newer snapshots (e.g. 112,419,402 compounds as of 2022/11/27) will yield different formula counts and may shift ranking metrics.
- The reference list contains only formulas and masses — it provides no structural information, which limits FSA's accuracy compared with tools such as Buddy or SIRIUS.
- FSA has been validated only on compounds composed of C, H, O, N, S and P; extending to halogens is a recent, less thoroughly validated expansion (rank#1 drops from 85.28% to 77.84% and runtime increases from 0.024 s to 0.16 s per spectrum).
- Few chemical rules are implemented to rank or filter out unfeasible formulas within the reference list.

## Evidence

- [other] After including halogen elements (F, Cl, Br, I), FSA's reference formula list grew from 928,369 to 2,613,203 entries: [evidence span withheld — anchor status: not_found]
