---
name: lc-ms-feature-filtering
description: Use when a PATBS-processed LC-MS feature table with m/z, MS1/MS2, and
  retention information must be filtered to link in vivo metabolites to prototypes,
  especially when metabolites follow unpredictable (non-obvious) metabolic pathways.
when_to_use_negative:
- When metabolites follow predictable metabolic responses already covered by tools
  such as Metworks 1.3.0.200 rather than unpredictable pathways
- When only secondary mass-spectrometry map similarity is needed for one-to-one matching
  against a prototype database (the MTSF approach), without multi-dimensional mass-defect
  filtering
- When no prototype (parent) structures or reference masses are available to bridge
  metabolites against
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MDRB
  role: R-based mass dataset relevance bridging platform that executes the multi-dimensional
    mass-defect screening, scoring, and N-count filtering of metabolite–prototype
    pairs
  repo: https://github.com/933ZhangDD/MDRB
- name: R
  role: Implementation language in which the MDRB data post-processing technique is
    run on the PATBS feature table
  repo: https://github.com/933ZhangDD/MDRB
- name: PATBS
  role: Upstream precise-and-thorough background-subtraction technique that produces
    the non-targeted feature table fed into MDRB screening
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13020-024-01031-8
    title: 'An integrated approach for studying exposure, metabolism, and disposition
      of traditional Chinese medicine using PATBS and MDRB tools: a case study of
      semen Armeniacae Amarum'
  source_content_hash: sha256:a918cd4d85dfd3f372841b2841d73fe5126f76cb5f2109d4fcb7d4f5937e3cc4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/933ZhangDD/MDRB
  tools:
  - MDRB
  - PATBS
  - R
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# lc-ms-feature-filtering

## Summary

MDRB-based multi-dimensional LC-MS feature filtering that scores and retains metabolite–prototype pairs using m/z proximity, MS1/MS2 mass-defect windows, MS2 fragment matching, and atom-count constraints, producing a scored screening table for unpredictable metabolic pathways.

## When to use

Use when a PATBS-processed LC-MS feature table with m/z, MS1/MS2, and retention information must be filtered to link in vivo metabolites to prototypes, especially when metabolites follow unpredictable (non-obvious) metabolic pathways.

## When NOT to use

- When metabolites follow predictable metabolic responses already covered by tools such as Metworks 1.3.0.200 rather than unpredictable pathways
- When only secondary mass-spectrometry map similarity is needed for one-to-one matching against a prototype database (the MTSF approach), without multi-dimensional mass-defect filtering
- When no prototype (parent) structures or reference masses are available to bridge metabolites against

## Inputs

- PATBS-processed feature table containing m/z, MS1, MS2, and retention information for prototypes and candidate metabolites
- Prototype database of reference component structures/masses

## Outputs

- mass_defect_screening_results.csv — scored metabolite–prototype pairs that pass all screening thresholds

## How to apply

Load the PATBS-processed feature table (m/z, MS1, MS2, retention information for prototypes and candidate metabolites) into R using the MDRB platform. For unpredictable-metabolite matching, apply MS1 mass-defect screening: keep prototype–metabolite pairs whose MS1 m/z difference falls within ±100 amu and whose mass defect falls within ±25 mmu, awarding a score of 50 for qualifying pairs. Then perform one-to-one MS2 fragment matching between prototype and metabolite within a ±50 amu / ±25 mmu window, awarding 5 points per matched fragment, and assign a retention score of 50 to pairs consistent in retention behavior. Apply the N-count filter, discarding pairs whose atom-count (N) change between prototype and metabolite exceeds 3. Aggregate the dimension scores and write the retained, scored pairs to mass_defect_screening_results.csv.

## Related tools

- **MDRB** (R-based mass dataset relevance bridging platform that executes the multi-dimensional mass-defect screening, scoring, and N-count filtering of metabolite–prototype pairs) — https://github.com/933ZhangDD/MDRB
- **R** (Implementation language in which the MDRB data post-processing technique is run on the PATBS feature table) — https://github.com/933ZhangDD/MDRB
- **PATBS** (Upstream precise-and-thorough background-subtraction technique that produces the non-targeted feature table fed into MDRB screening)

## Examples

```
Rscript MDRB_mass_defect_screening.R --input PATBS_feature_table.csv --output mass_defect_screening_results.csv
```

## Evaluation signals

- Every retained pair in mass_defect_screening_results.csv satisfies the ±100 amu / ±25 mmu MS1 mass-defect window
- Every retained pair satisfies the ±50 amu / ±25 mmu MS2 fragment-matching window
- No retained pair has an atom-count (N) change between prototype and metabolite exceeding 3
- All qualifying pairs carry the expected dimension scores: 50 for MS1 mass-defect screening, 5 points per matched MS2 fragment, and 50 for retention consistency
- Only threshold-compliant pairs appear in the output table (qualitative check; no quantitative acceptance metric reported in the paper)

## Limitations

- No statistical validation, replicate counts, or quantitative performance metrics are reported for the screening strategy
- Identification of in vivo components remains difficult despite MS data post-processing techniques
- No software versions stated for the MDRB technique or its data-processing platform, and no changelog is available for the source repository

## Evidence

- [results] MS1 mass-defect window and N-count rule: [evidence span withheld — anchor status: not_found]
- [abstract] Multi-dimensional scoring rationale: "The current method integrates three dimensions of mass spectral data including m/z, mass defect of MS1 and MS2, and the similarity of MS2 fragments"
- [intro] MS2 fragment bridging rationale: "the mass defect of MS2 fragments of prototypes and its metabolites is within a certain range, which can likewise provide critical information for bridging"
- [full_text] Output artifact: [evidence span withheld — anchor status: not_found]
- [discussion] Limitation: "there are still more difficulties in the identification of components."
