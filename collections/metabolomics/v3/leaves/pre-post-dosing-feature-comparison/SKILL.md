---
name: pre-post-dosing-feature-comparison
description: Use when paired LC-MS/MS feature lists exist for pre-dosing (blank) and
  post-dosing biological matrices (e.g., plasma, lung, urine, feces) and the goal
  is non-targeted isolation of dosage-related exogenous components for prototype/metabolite
  annotation.
when_to_use_negative:
- When no matched pre-dosing (blank) matrix is available for the post-dosing sample,
  since background subtraction requires the blank comparison.
- When the goal is targeted, structure-predicted quantification of a predefined metabolite
  panel rather than non-targeted discovery of exogenous components.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: PATBS
  role: Performs the non-targeted background subtraction of post-dosing features against
    pre-dosing blank matrices to retain exogenous, dosage-related peaks
- name: R
  role: Processing environment hosting the PATBS comparison and the R-based MDRB data
    post-processing of retained features
  repo: https://github.com/933ZhangDD/MDRB
- name: MDRB
  role: Post-processing step that characterizes retained exogenous components and
    bridges prototypes to metabolites via m/z, mass defect, and MS2 fragment similarity
  repo: https://github.com/933ZhangDD/MDRB
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13020-024-01031-8
    title: 'An integrated approach for studying exposure, metabolism, and disposition
      of traditional Chinese medicine using PATBS and MDRB tools: a case study of
      semen Armeniacae Amarum'
  source_content_hash: sha256:94c7f11b057c38e656c70c73ed14135d57b768cfb6b4d17d322d5875a2d7d8ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/933ZhangDD/MDRB
  tools:
  - MDRB
  - PATBS
  - R
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# pre-post-dosing-feature-comparison

## Summary

A non-targeted LC-MS/MS background-subtraction workflow (PATBS) that compares post-dosing sample feature lists against matched pre-dosing (blank) matrices to retain exogenous, dosage-related components, then labels them as prototypes or metabolites of a model herb (here, semen Armeniacae Amarum, CKXR).

## When to use

Use when paired LC-MS/MS feature lists exist for pre-dosing (blank) and post-dosing biological matrices (e.g., plasma, lung, urine, feces) and the goal is non-targeted isolation of dosage-related exogenous components for prototype/metabolite annotation.

## When NOT to use

- When no matched pre-dosing (blank) matrix is available for the post-dosing sample, since background subtraction requires the blank comparison.
- When the goal is targeted, structure-predicted quantification of a predefined metabolite panel rather than non-targeted discovery of exogenous components.

## Inputs

- Pre-dosing (blank) LC-MS/MS feature lists per biological matrix (plasma, lung, urine, feces)
- Post-dosing LC-MS/MS feature lists for the same matrices
- CKXR prototype component list for prototype/metabolite annotation

## Outputs

- Annotated exogenous component table (patbs_exogenous_components.csv) with prototype/metabolite labels and per-matrix assignments
- Per-matrix exogenous component counts (e.g., plasma 87, lung 53, urine 342, feces 42; 351 total = 41 prototypes + 310 metabolites)

## How to apply

Load pre-dosing and post-dosing LC-MS/MS data files for each biological matrix (plasma, lung, urine, feces) into the R-based PATBS processing environment. Run PATBS background subtraction by comparing the post-dosing sample feature list against the corresponding pre-dosing blank matrix, which removes endogenous/background components and retains exogenous, dosage-related peaks without needing to predict herbal component structures in advance. Annotate the retained exogenous features against the CKXR prototype component list, separating prototypes from metabolites, and optionally pass results through the R-based MDRB post-processing step that bridges prototypes and metabolites via m/z, MS1/MS2 mass defect, and MS2 fragment similarity. Tabulate component occurrences per matrix and write the annotated component table (e.g., patbs_exogenous_components.csv) with prototype/metabolite labels and per-matrix assignments.

## Related tools

- **PATBS** (Performs the non-targeted background subtraction of post-dosing features against pre-dosing blank matrices to retain exogenous, dosage-related peaks)
- **R** (Processing environment hosting the PATBS comparison and the R-based MDRB data post-processing of retained features) — https://github.com/933ZhangDD/MDRB
- **MDRB** (Post-processing step that characterizes retained exogenous components and bridges prototypes to metabolites via m/z, mass defect, and MS2 fragment similarity) — https://github.com/933ZhangDD/MDRB

## Examples

```
git clone https://github.com/933ZhangDD/MDRB  # then load pre/post-dosing LC-MS/MS feature lists per matrix into the R-based PATBS/MDRB environment
```

## Evaluation signals

- Output table contains exactly 351 exogenous components (41 prototypes + 310 metabolites) for the CKXR mouse case study.
- Per-matrix component counts match the reported results: plasma 87, lung 53, urine 342, feces 42.
- Every retained component is labeled as either prototype or metabolite and assigned to at least one biological matrix.
- Retained features should be absent from (or negligible in) the corresponding pre-dosing blank matrix, confirming they are exogenous/dosage-related rather than endogenous background.

## Limitations

- No specific mass spectrometry instrument model or data acquisition parameters are stated for the PATBS/MDRB pipeline, limiting reproducibility.
- No statistical validation, replicate counts, or quantitative performance metrics are reported for the strategy.
- Identification of in vivo components remains difficult despite MS data post-processing techniques; MTSF-style matching alone only uses MS2 similarity of single metabolites against the prototype database.

## Evidence

- [intro] Non-targeted techniques do not require structure prediction: "Non-targeted techniques, such as precise-and-thorough background-subtraction (PATBS) [15, 16], or metabolomics techniques [17, 18], do not require prediction of the structure of herbal components."
- [abstract] MDRB + PATBS combined strategy in mice: "By combining MDRB with PATBS through this non-targeted identification technology, we developed a comprehensive strategy for identification, characterization and bridging analysis of TCM metabolite in"
- [abstract] R-based MDRB post-processing: "which was sequentially analyzed by a R-based mass dataset relevance bridging (MDRB) data post-processing technique"
- [intro] Stepwise identification of exposed CKXR components: "the identification of the exposed components of CKXR was completed in a stepwise manner through MDRB"
- [abstract] Case-study scope: exposure, metabolism, disposition in mice: "we adopted the proposed strategy to investigate the process of exposure, metabolism, and disposition of Semen Armeniacae Amarum (CKXR) in mice"
