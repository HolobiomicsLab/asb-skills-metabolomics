---
name: biotransformation-reaction-rule-matching
description: Use when untargeted LC-HRMS feature tables lack library MS2 matches but
  samples are enzymatic incubations (e.g., human liver S9) of known parent xenobiotics,
  so candidates can be derived in silico from reaction rules rather than reference
  spectra.
when_to_use_negative:
- Input data are targeted assays of known compounds with authentic standards already
  available — rule-based mass matching adds no benefit over direct confirmation.
- No parent structure library exists (e.g., truly unknown exposure mixtures) — without
  parent identities, reaction-rule enumeration cannot proceed.
- MS2 spectral libraries already give high-confidence library matches for all features
  — the RCS layer is redundant when bottom-up annotation succeeds.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: CMDN
  role: Framework housing the reaction-rule enumeration and RCS matching layer
  repo: https://github.com/LeaveMeNotTonight/CMDN
- name: LC-HRMS
  role: Source of the accurate-mass feature data being preannotated against rule-derived
    candidates
- name: biotransformation prediction
  role: General in-silico rule application referenced for candidate generation context
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1021/acs.est.5c08558
    title: Reaction-Guided Metabolomics Accelerates High-Throughput Annotation of
      Xenobiotic Metabolites for Human Exposome
  source_content_hash: sha256:77b2aaeef41b9d640fa2b676279e2aee06c8a4d1bcac13e6e67b82bf2c1e4e62
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/LeaveMeNotTonight/CMDN
  tools:
  - CMDN
  - LC-HRMS
  - biotransformation prediction
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# biotransformation-reaction-rule-matching

## Summary

Enumerates theoretical biotransformed metabolite candidates by applying curated phase-I reaction rules (e.g., 290 KEGG-sourced rules) to parent compound structures, then preannotates dysregulated LC-HRMS features by accurate-mass matching within a 5 ppm m/z tolerance.

## When to use

Use when untargeted LC-HRMS feature tables lack library MS2 matches but samples are enzymatic incubations (e.g., human liver S9) of known parent xenobiotics, so candidates can be derived in silico from reaction rules rather than reference spectra.

## When NOT to use

- Input data are targeted assays of known compounds with authentic standards already available — rule-based mass matching adds no benefit over direct confirmation.
- No parent structure library exists (e.g., truly unknown exposure mixtures) — without parent identities, reaction-rule enumeration cannot proceed.
- MS2 spectral libraries already give high-confidence library matches for all features — the RCS layer is redundant when bottom-up annotation succeeds.

## Inputs

- Parent compound structure library (e.g., 1,021 pesticides as SMILES/structures)
- Curated nonisomeric phase-I biotransformation reaction rule set (n = 290)
- DMS-flagged dysregulated LC-HRMS feature table with accurate m/z values

## Outputs

- rcs_candidate_features.csv listing theoretical m/z, parent pesticide, applied reaction rule, and m/z error (ppm)
- Preannotated feature set carrying metabolite candidates for downstream SSN layering
- Nonredundant candidate space (e.g., 2,886 derivatives from 1,021 parents)

## How to apply

Load the parent compound structure library (e.g., 1,021 pesticides) and a curated set of nonisomeric phase-I reaction rules (290 rules in CMDN, sourced from KEGG and literature). Apply each rule in silico to every parent to enumerate derivatives and compute their monoisotopic m/z values, forming a pseudo-MS1 candidature space (RCS). Cross-reference these theoretical m/z values against dysregulated features flagged by a differential metabolite space (DMS) layer, retaining matches within an m/z error tolerance of ≤ 5 ppm. Emit a candidate table (e.g., rcs_candidate_features.csv) recording theoretical m/z, parent identity, applied reaction rule, and m/z error. The resulting preannotated features then feed downstream MS2 spectrum-similarity (SSN) validation: in the 1,021-pesticide S9 study, RCS contributed candidates consistent with 2,886 nonredundant annotations and an 87.3% MS1–MS2 match rate.

## Related tools

- **CMDN** (Framework housing the reaction-rule enumeration and RCS matching layer) — https://github.com/LeaveMeNotTonight/CMDN
- **LC-HRMS** (Source of the accurate-mass feature data being preannotated against rule-derived candidates)
- **biotransformation prediction** (General in-silico rule application referenced for candidate generation context)

## Examples

```
Rscript cmdn_rcs_pipeline.R --rules 290_kegg_rules.csv --parents pesticides_1021.smi --features dms_features.csv --ppm 5 --out rcs_candidate_features.csv
```

## Evaluation signals

- Every cross-referenced entry must satisfy the ≤ 5 ppm m/z error criterion
- Enumerated nonredundant candidate set should match the reported scale (2,886 derivatives of 1,021 pesticides)
- Output CSV rows must carry theoretical m/z, parent identity, reaction rule, and ppm error columns
- Downstream SSN layer should confirm a large majority of RCS candidates (87.3% MS1–MS2 match rate in the reference study)

## Limitations

- Rule enumeration is nonisomeric phase-I only — phase-II conjugates and isomer-specific products are outside this RCS scope
- Mass matching alone gives preannotations, not identities; confirmation requires downstream MS2, standards, or other validation channels
- Candidate space depends on rule coverage (290 rules here); gaps in rule libraries under-annotate exotic transformations

## Evidence

- [methods] Theoretical candidate features generated by the RCS reaction rules were cross-referenced with dysregulated features identified in the DMS layer and preannotated by accurate mass matching at an m/z error tolerance of ≤ 5 ppm.: [evidence span withheld — anchor status: not_found]
- [intro] a triple-layered architecture comprising: (1) a differential metabolite space (DMS), (2) a reaction-based pseudo-MS1 candidature space (RCS) and (3) an MS2 spectrum similarity network (SSN): "a triple-layered architecture comprising: (1) a differential metabolite space (DMS), (2) a reaction-based pseudo-MS1 candidature space (RCS) and (3) an MS2 spectrum similarity network (SSN)"
- [supplementary] CMDN-annotated PMDB with manual curation (N = 492) (Table S2; sheet 1); reaction rules adopted in CMDN (n = 290): "CMDN-annotated PMDB with manual curation (N = 492) (Table S2; sheet 1); reaction rules adopted in CMDN (n = 290)"
- [intro] In silico tools such as biotransformation prediction27 and fragmentation simulation28 platforms can provide partial insights: "In silico tools such as biotransformation prediction27 and fragmentation simulation28 platforms can provide partial insights"
- [abstract] CMDN nonredundantly annotated 2,886 biotransformed derivatives of 1,021 pesticides.: "Coupled with enzymatic biotransformation assays involving 1,021 pesticides, CMDN nonredundantly annotated 2,886 biotransformed derivatives."
