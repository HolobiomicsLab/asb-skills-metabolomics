---
name: lc-ms-background-subtraction
description: Use when you have paired pre-dosing (blank) and post-dosing LC-MS/MS
  data from biological matrices (plasma, lung, urine, feces) and need to strip endogenous/background
  peaks to retain exogenous, dosage-related components non-targeted.
when_to_use_negative:
- The input is already a background-subtracted, exogenous-only feature table — re-running
  subtraction would discard components
- The goal is bridging prototypes to metabolites in already-identified components
  — MDRB data post-processing (m/z, MS1/MS2 mass defect, MS2 fragment similarity)
  is the appropriate next step, not background subtraction
- A targeted approach is preferred that requires prior prediction of the structures
  of herbal components and their metabolites
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: PATBS
  role: Self-developed technique performing non-targeted background subtraction of
    pre-dosing vs. post-dosing LC-MS/MS feature lists to identify exogenous metabolites
- name: R
  role: Implementation environment for the R-based PATBS/MDRB data processing pipeline
  repo: https://github.com/933ZhangDD/MDRB
- name: MDRB
  role: Downstream data post-processing that takes PATBS-retained exogenous components
    and bridges them to prototypes via mass-defect and MS2 fragment similarity
  repo: https://github.com/933ZhangDD/MDRB
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13020-024-01031-8
    title: 'An integrated approach for studying exposure, metabolism, and disposition
      of traditional Chinese medicine using PATBS and MDRB tools: a case study of
      semen Armeniacae Amarum'
  source_content_hash: sha256:9505adaa4de3892e7373a16add3bddd2ec477550b741777662050ef939a1aa42
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/933ZhangDD/MDRB
  tools:
  - MDRB
  - PATBS
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# lc-ms-background-subtraction

## Summary

Non-targeted LC-MS/MS background subtraction (PATBS) that identifies exogenous, dose-related components by comparing post-dosing sample feature lists against pre-dosing blank matrices, without requiring prior prediction of herbal component structures.

## When to use

Use when you have paired pre-dosing (blank) and post-dosing LC-MS/MS data from biological matrices (plasma, lung, urine, feces) and need to strip endogenous/background peaks to retain exogenous, dosage-related components non-targeted.

## When NOT to use

- The input is already a background-subtracted, exogenous-only feature table — re-running subtraction would discard components
- The goal is bridging prototypes to metabolites in already-identified components — MDRB data post-processing (m/z, MS1/MS2 mass defect, MS2 fragment similarity) is the appropriate next step, not background subtraction
- A targeted approach is preferred that requires prior prediction of the structures of herbal components and their metabolites

## Inputs

- Pre-dosing (blank) LC-MS/MS feature lists per biological matrix (plasma, lung, urine, feces)
- Post-dosing LC-MS/MS feature lists per biological matrix
- CKXR prototype component list for prototype/metabolite annotation

## Outputs

- Annotated component table (patbs_exogenous_components.csv) with prototype/metabolite labels and per-matrix assignments
- Per-matrix exogenous component counts (expected: plasma 87, lung 53, urine 342, feces 42)

## How to apply

Load the pre-dosing and post-dosing LC-MS/MS data files for each biological matrix (plasma, lung, urine, feces) into the R-based PATBS processing environment. Run PATBS background subtraction by comparing each post-dosing sample feature list against the corresponding pre-dosing (blank) matrix feature list, removing endogenous/background components and retaining exogenous, dosage-related peaks. Annotate the retained exogenous features against the CKXR (Semen Armeniacae Amarum) prototype component list to separate prototypes from their metabolites. Tabulate component occurrences per biological matrix; in the reference application this yielded 351 CKXR-related exogenous components in mice — 41 prototypes and 310 metabolites — distributed as 87 in plasma, 53 in lung, 342 in urine, and 42 in feces. The rationale is that PATBS is a non-targeted technique that does not require prediction of herbal component structures, unlike targeted metabolite-prediction approaches. Retained components can be passed downstream to MDRB for prototype–metabolite bridging via m/z, MS1/MS2 mass defect, and MS2 fragment similarity scoring.

## Related tools

- **PATBS** (Self-developed technique performing non-targeted background subtraction of pre-dosing vs. post-dosing LC-MS/MS feature lists to identify exogenous metabolites)
- **R** (Implementation environment for the R-based PATBS/MDRB data processing pipeline) — https://github.com/933ZhangDD/MDRB
- **MDRB** (Downstream data post-processing that takes PATBS-retained exogenous components and bridges them to prototypes via mass-defect and MS2 fragment similarity) — https://github.com/933ZhangDD/MDRB

## Evaluation signals

- Output table contains exactly 351 exogenous components (41 prototypes + 310 metabolites) in the reference CKXR mouse dataset
- Per-matrix counts match the reported PATBS results: 87 plasma, 53 lung, 342 urine, 42 feces
- Every retained component is absent from (or not attributable to) the corresponding pre-dosing blank matrix feature list
- Each retained component carries a prototype or metabolite label derived from the CKXR prototype component list
- Output CSV (patbs_exogenous_components.csv) includes per-matrix assignments for all components

## Limitations

- No statistical validation, replicate counts, or quantitative performance metrics reported for the strategy
- No specific mass spectrometry instrument model or data acquisition parameters named, limiting reproducibility of feature-list generation upstream of PATBS
- Identification of in vivo components remains difficult despite MS data post-processing techniques
- MDRB has no changelog and no software versions stated for the technique or data-processing platform

## Evidence

- [intro] First, we realized the non-target identification of metabolites by our self-developed PATBS technique.: "First, we realized the non-target identification of metabolites by our self-developed PATBS technique."
- [intro] Non-targeted techniques do not require structural prediction: "Non-targeted techniques, such as precise-and-thorough background-subtraction (PATBS) [15, 16], or metabolomics techniques [17, 18], do not require prediction of the structure of herbal components."
- [abstract] MDRB+PATBS combined strategy: "By combining MDRB with PATBS through this non-targeted identification technology, we developed a comprehensive strategy for identification, characterization and bridging analysis of TCM metabolite in"
