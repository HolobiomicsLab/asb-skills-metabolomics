---
name: matrix-specific-component-profiling
description: Use when you have paired pre-dosing (blank) and post-dosing LC-MS/MS
  acquisitions for multiple mouse biological matrices (e.g., plasma, lung, urine,
  feces) and need to count and classify dosage-related exogenous components per matrix.
when_to_use_negative:
- No corresponding pre-dosing (blank) matrix exists for a sample, so PATBS background
  subtraction cannot separate endogenous from exogenous peaks.
- Input is already an annotated component/feature table requiring prototype-metabolite
  bridging or pathway mapping rather than blank-subtracted exogenous profiling per
  matrix.
- Data are in vitro component lists rather than in vivo biological matrices, where
  the goal is exposure/metabolism bridging via MDRB rather than matrix-level disposition
  profiling.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: PATBS
  role: Non-targeted background subtraction tool that removes endogenous/pre-dosing
    features to retain exogenous, dose-related components per matrix
- name: R
  role: R-based processing environment in which PATBS subtraction and per-matrix tabulation
    of exogenous components are executed
- name: MDRB
  role: Downstream data post-processing that characterizes metabolites and bridges
    prototypes to metabolites among the retained exogenous components
  repo: https://github.com/933ZhangDD/MDRB
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13020-024-01031-8
    title: 'An integrated approach for studying exposure, metabolism, and disposition
      of traditional Chinese medicine using PATBS and MDRB tools: a case study of
      semen Armeniacae Amarum'
  source_content_hash: sha256:8e1c9ea0fd787fd2e907be3f5db572a35b5001ee9b35f8adbe9d04f9ea8f98ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/933ZhangDD/MDRB
  tools:
  - MDRB
  - PATBS
  - R
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# matrix-specific-component-profiling

## Summary

Profiles exogenous (dose-related) components per biological matrix by running PATBS non-targeted background subtraction of post-dosing LC-MS/MS data against matched pre-dosing blanks, then annotating retained features as prototypes vs metabolites. Applied to CKXR (Semen Armeniacae Amarum) in mice, it yielded 351 exogenous components (41 prototypes, 310 metabolites) distributed across plasma, lung, urine, and feces.

## When to use

Use when you have paired pre-dosing (blank) and post-dosing LC-MS/MS acquisitions for multiple mouse biological matrices (e.g., plasma, lung, urine, feces) and need to count and classify dosage-related exogenous components per matrix.

## When NOT to use

- No corresponding pre-dosing (blank) matrix exists for a sample, so PATBS background subtraction cannot separate endogenous from exogenous peaks.
- Input is already an annotated component/feature table requiring prototype-metabolite bridging or pathway mapping rather than blank-subtracted exogenous profiling per matrix.
- Data are in vitro component lists rather than in vivo biological matrices, where the goal is exposure/metabolism bridging via MDRB rather than matrix-level disposition profiling.

## Inputs

- Pre-dosing (blank) LC-MS/MS feature lists per biological matrix (plasma, lung, urine, feces)
- Post-dosing LC-MS/MS feature lists per biological matrix
- CKXR prototype component list for prototype/metabolite annotation

## Outputs

- Annotated component table (patbs_exogenous_components.csv) with prototype/metabolite labels and per-matrix assignments
- Per-matrix exogenous component counts (e.g., plasma 87, lung 53, urine 342, feces 42)

## How to apply

Load the pre-dosing and post-dosing LC-MS/MS feature lists for each biological matrix into the R-based PATBS processing environment, and for each matrix run PATBS background subtraction comparing post-dosing features against the corresponding pre-dosing blank to remove endogenous/background peaks and retain exogenous, dosage-related peaks. Annotate the retained exogenous features against the CKXR prototype component list, labeling each as a prototype or a metabolite (in the mouse CKXR case: 41 prototypes and 310 metabolites of 351 total). Tabulate component occurrences per matrix (plasma 87, lung 53, urine 342, feces 42) and write the annotated table with prototype/metabolite labels and per-matrix assignments to patbs_exogenous_components.csv. The rationale is that non-targeted PATBS does not require prediction of herbal component structures, and the stepwise identification can be followed by MDRB bridging of prototypes to metabolites.

## Related tools

- **PATBS** (Non-targeted background subtraction tool that removes endogenous/pre-dosing features to retain exogenous, dose-related components per matrix)
- **R** (R-based processing environment in which PATBS subtraction and per-matrix tabulation of exogenous components are executed)
- **MDRB** (Downstream data post-processing that characterizes metabolites and bridges prototypes to metabolites among the retained exogenous components) — https://github.com/933ZhangDD/MDRB

## Evaluation signals

- Output table contains exactly 351 exogenous components (41 prototypes + 310 metabolites) for the mouse CKXR case.
- Per-matrix counts match the reported values: 87 plasma, 53 lung, 342 urine, 42 feces.
- Every retained feature is flagged exogenous by comparing post-dosing features against the corresponding pre-dosing blank matrix.
- Each component carries both a prototype/metabolite label and a per-matrix assignment, and the CSV schema is complete.
- Stepwise identification traceable: blank subtraction precedes annotation against the CKXR prototype list.

## Limitations

- Identification of in vivo components remains difficult despite MS data post-processing techniques.
- No specific mass spectrometry instrument model or data acquisition parameters were named, limiting reproducibility across instruments.
- No statistical validation, replicate counts, or quantitative performance metrics are reported for the strategy.
- The source MDRB repository provides no changelog and no software versions are stated for MDRB or the data-processing platform.

## Evidence

- [intro] First, we realized the non-target identification of metabolites by our self-developed PATBS technique.: "First, we realized the non-target identification of metabolites by our self-developed PATBS technique."
- [intro] the identification of the exposed components of CKXR was completed in a stepwise manner through MDRB: "the identification of the exposed components of CKXR was completed in a stepwise manner through MDRB"
- [abstract] By combining MDRB with PATBS through this non-targeted identification technology, we developed a comprehensive strategy: "By combining MDRB with PATBS through this non-targeted identification technology, we developed a comprehensive strategy"
- [discussion] we established a technique, MDRB, for in vivo metabolite characterization in TCM and its bridging with in vitro prototypes: "we established a technique, MDRB, for in vivo metabolite characterization in TCM and its bridging with in vitro prototypes"
- [intro] Non-targeted techniques, such as precise-and-thorough background-subtraction (PATBS), do not require prediction of the structure of herbal components.: "Non-targeted techniques, such as precise-and-thorough background-subtraction (PATBS) [15, 16], or metabolomics techniques [17, 18], do not require prediction of the structure of herbal components."
