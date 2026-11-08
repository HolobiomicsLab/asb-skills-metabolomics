---
name: differential-feature-cross-referencing
description: Use when you have a list of theoretical biotransformation candidate m/z
  values and a separate table of dysregulated/DE features from untargeted LC-HRMS
  data, and need to preannotate which candidates map to observed features.
when_to_use_negative:
- Input is already an annotated feature table with compound identities — cross-referencing
  adds no information
- Only intact parent compounds (no biotransformation) are of interest; a reference-library
  bottom-up search is more appropriate
- The feature table comes from low-resolution MS where 5 ppm mass-error matching is
  not meaningful
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: CMDN
  role: Framework implementing the RCS layer that applies 290 reaction rules and performs
    5 ppm matching of theoretical candidates against DMS-flagged dysregulated features
  repo: https://github.com/LeaveMeNotTonight/CMDN
- name: Biotransformer 3.0
  role: Compatible in silico biotransformation prediction tool for enumerating candidate
    metabolites
- name: MSDial
  role: Compatible software (ver. 4.80) for LC-HRMS feature deconvolution producing
    the feature table that feeds the differential metabolite space
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.est.5c08558
    title: Reaction-Guided Metabolomics Accelerates High-Throughput Annotation of
      Xenobiotic Metabolites for Human Exposome
  source_content_hash: sha256:d899e9236062b82256a182ef0478787bebeeb2a8233b4672301e864c03f7dfe9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/LeaveMeNotTonight/CMDN
  tools:
  - Biotransformer 3.0
  - CMDN
  - MSDial
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# differential-feature-cross-referencing

## Summary

Cross-references reaction-rule-derived theoretical metabolite candidates (pseudo-MS1 candidature space) against dysregulated features flagged in a differential metabolite space by accurate-mass matching at ≤5 ppm, producing preannotated candidate features with parent, rule, and mass-error provenance.

## When to use

Use when you have a list of theoretical biotransformation candidate m/z values and a separate table of dysregulated/DE features from untargeted LC-HRMS data, and need to preannotate which candidates map to observed features.

## When NOT to use

- Input is already an annotated feature table with compound identities — cross-referencing adds no information
- Only intact parent compounds (no biotransformation) are of interest; a reference-library bottom-up search is more appropriate
- The feature table comes from low-resolution MS where 5 ppm mass-error matching is not meaningful

## Inputs

- Structure library of parent compounds (e.g., 1,021-pesticide library)
- Reaction rule set (e.g., 290 nonisomeric phase-I rules from KEGG and curated literature)
- DMS-flagged dysregulated feature list from untargeted LC-HRMS data (feature m/z values)

## Outputs

- rcs_candidate_features.csv with per-feature theoretical m/z, parent pesticide identity, applied reaction rule, and m/z error in ppm
- Nonredundant preannotated candidate set (scale consistent with reported outcome: 2,886 nonredundant derivatives of 1,021 pesticides)

## How to apply

Enumerate theoretical biotransformed derivatives by applying reaction rules (e.g., CMDN's 290 nonisomeric phase-I rules sourced from KEGG and curated literature) in silico to each parent compound (e.g., the 1,021-pesticide structure library), computing monoisotopic m/z values to form the pseudo-MS1 candidature space (RCS). Load dysregulated features flagged by the differential metabolite space (DMS) layer from the untargeted LC-HRMS feature table. Match each theoretical candidate m/z against DMS-flagged features using a mass-error tolerance of 5 ppm; retain only matches satisfying this criterion. Emit one row per preannotated feature carrying the theoretical m/z, parent compound identity, applied reaction rule, and m/z error in ppm (rcs_candidate_features.csv). The rationale is a top-down strategy: reaction-derived candidates constrain the search space before MS2-level confirmation (e.g., via the spectrum similarity network layer).

## Related tools

- **CMDN** (Framework implementing the RCS layer that applies 290 reaction rules and performs 5 ppm matching of theoretical candidates against DMS-flagged dysregulated features) — https://github.com/LeaveMeNotTonight/CMDN
- **Biotransformer 3.0** (Compatible in silico biotransformation prediction tool for enumerating candidate metabolites)
- **MSDial** (Compatible software (ver. 4.80) for LC-HRMS feature deconvolution producing the feature table that feeds the differential metabolite space)

## Examples

```
Rscript cmdn_rcs_match.R --rules reaction_rules_290.xlsx --parents pesticide_library_1021.csv --dms_features dms_flagged_features.csv --ppm 5 --out rcs_candidate_features.csv
```

## Evaluation signals

- Every cross-referenced entry satisfies the 5 ppm mass-error criterion
- The enumerated nonredundant candidate set is consistent in scale with the reported 2,886 nonredundant biotransformed derivatives of 1,021 pesticides
- rcs_candidate_features.csv schema includes theoretical m/z, parent pesticide identity, applied reaction rule, and m/z error in ppm for every row
- All candidate m/z values are traceable to a specific parent compound and reaction rule from the 290-rule library
- Matched features are a subset of the DMS-flagged dysregulated features (no unmatched features emitted)

## Limitations

- Preannotation is based on accurate mass only at this stage; it provides partial insight and requires downstream MS2 similarity-network and multichannel validation (standards, RT prediction, time-course modeling, in vivo studies)
- Reaction-rule enumeration is restricted to the adopted rule library (290 nonisomeric phase-I rules), so metabolites outside these transformation types will be missed
- In silico biotransformation approaches often fall short of providing direct evidence under real-world, omics-scale conditions without orthogonal validation

## Evidence

- [results] Theoretical candidate features generated by the RCS reaction rules were cross-referenced with dysregulated features identified in the DMS layer and preannotated by accurate mass matching at an m/z error tolerance of ≤ 5 ppm.: [evidence span withheld — anchor status: not_found]
