---
name: r-data-frame-na-manipulation
description: Use when you have a complete (NA-free) scaled data matrix and need to
  simulate missing-not-at-random, left-censored values at controlled missing proportions
  (e.g., 0.1–0.8) for benchmarking imputation methods.
when_to_use_negative:
- The input data already contains missing values — MNAR_gen_imp expects a complete
  matrix; use imputation wrappers (e.g., pre_processing_GS_wrapper) instead
- You need to simulate MCAR/MAR missingness — left-censored quantile masking only
  models MNAR/LOD-type missingness
- Values above the quantile cut-offs must be preserved untouched — masking is strictly
  one-sided (left-censoring) and will not remove high-value entries
edam_operation: http://edamontology.org/operation_3595
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: GSimp
  role: Provides GSimp_evaluation.R containing MNAR_gen_imp and the missing-generation/evaluation
    pipeline functions used to mask the complete matrix
  repo: https://github.com/WandeRum/GSimp
- name: R
  role: Language in which the masking procedure (quantile cut-off per variable, NA
    assignment) and CSV I/O are executed
  repo: https://github.com/WandeRum/GSimp
- name: MVI-evaluation
  role: Companion repository whose Imputation evaluations.R holds the full imputation
    evaluation pipeline (missing generation, imputations, evaluations)
  repo: https://github.com/WandeRum/MVI-evaluation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1005973
    title: 'GSimp: A Gibbs sampler based left-censored missing value imputation approach
      for metabolomics studies'
  source_content_hash: sha256:599675c761a228886c72ac2173f75513c9bc283ef0a183e4c3e5050cfb369117
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WandeRum/GSimp
  tools:
  - GSimp
  - MVI-evaluation
  - R
  edam_operation: http://edamontology.org/operation_3595
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# r-data-frame-na-manipulation

## Summary

Generate MNAR (left-censored) masked matrices in R by setting elements below per-variable quantile cut-offs to NA, using the GSimp evaluation pipeline's MNAR_gen_imp function (GSimp_evaluation.R) on a complete scaled data matrix.

## When to use

Use when you have a complete (NA-free) scaled data matrix and need to simulate missing-not-at-random, left-censored values at controlled missing proportions (e.g., 0.1–0.8) for benchmarking imputation methods.

## When NOT to use

- The input data already contains missing values — MNAR_gen_imp expects a complete matrix; use imputation wrappers (e.g., pre_processing_GS_wrapper) instead
- You need to simulate MCAR/MAR missingness — left-censored quantile masking only models MNAR/LOD-type missingness
- Values above the quantile cut-offs must be preserved untouched — masking is strictly one-sided (left-censoring) and will not remove high-value entries

## Inputs

- Complete (NA-free) scaled data matrix in R, e.g., data_sim_sc derived from data_sim.csv
- Vector of target missing proportions, e.g., mis_var_prop = seq(.1, .8, .1)
- Source CSV files: real_data.csv, targeted_data.csv, untargeted_data.csv, or data_sim.csv

## Outputs

- List of MNAR-masked data matrices (sim_MNAR_list), one per missing-proportion level, with left-tail values set to NA
- Per-matrix CSV files written to the output directory

## How to apply

Load the complete matrix (real_data.csv, targeted_data.csv, untargeted_data.csv, or data_sim.csv) into R and source GSimp_evaluation.R. For each target missing proportion (seq(0.1, 0.8, 0.1) for the simulation dataset; 0.1–0.6 by 0.05 for real-world datasets), determine the fraction of variables made missing and draw a per-variable quantile cut-off from a uniform distribution — U(0.1, 0.5) for real-world datasets, U(0.3, 0.6) for the simulation dataset. Set every element falling below its variable's cut-off to NA, i.e., use MNAR_gen_imp(data_c=data_sim_sc, mis_var_prop=seq(.1, .8, .1), ...) which returns a list of MNAR-masked matrices, one per requested missing proportion. Save each generated matrix as a CSV in the output directory. Rationale: left-tail removal mimics LOD/LOQ-driven MNAR missingness typical of MS-based targeted metabolomics, providing masked ground-truth matrices against which imputation accuracy can be evaluated.

## Related tools

- **GSimp** (Provides GSimp_evaluation.R containing MNAR_gen_imp and the missing-generation/evaluation pipeline functions used to mask the complete matrix) — https://github.com/WandeRum/GSimp
- **R** (Language in which the masking procedure (quantile cut-off per variable, NA assignment) and CSV I/O are executed) — https://github.com/WandeRum/GSimp
- **MVI-evaluation** (Companion repository whose Imputation evaluations.R holds the full imputation evaluation pipeline (missing generation, imputations, evaluations)) — https://github.com/WandeRum/MVI-evaluation

## Examples

```
source('GSimp_evaluation.R'); sim_MNAR_list <- MNAR_gen_imp(data_c=data_sim_sc, mis_var_prop=seq(.1, .8, .1))
```

## Evaluation signals

- For each output MNAR matrix, the realized proportion of missing values matches the requested missing-proportion level
- No elements above the per-variable quantile cut-offs were removed — missingness is confined to the left tail of each variable's distribution
- Non-missing entries of the masked matrix are identical to the corresponding entries of the original complete matrix (masking only introduces NA, never alters values)
- The number of output matrices equals the length of mis_var_prop (e.g., 8 matrices for seq(.1, .8, .1))
- Every CSV output file loads back into R with the same dimensions and NA positions as the in-memory masked matrix

## Limitations

- The masking procedure assumes left-censored MNAR missingness (LOD/LOQ-type); it does not model MCAR or MAR patterns
- Cut-off distributions are dataset-specific: U(0.1, 0.5) for real-world datasets vs U(0.3, 0.6) for the simulation dataset — mixing them up changes the missingness severity
- Reaching an exact target missing proportion depends on the realized data distribution; validation of the achieved proportion per matrix is required

## Evidence

- [methods] MNAR generation call: "sim_MNAR_list <- MNAR_gen_imp(data_c=data_sim_sc, mis_var_prop=seq(.1, .8, .1),"
- [readme] README pipeline description: "**GSimp_evaluation.R** contains MNAR generation and evaluation functions which are part of our missing value imputation evaluation pipeline."
- [abstract] MNAR rationale: "Missing values caused by the limit of detection/quantification (LOD/LOQ) were widely observed in mass spectrometry (MS)-based targeted metabolomics studies and could be recognized as missing not at"
- [full_text] MNAR finding from source card: [evidence span withheld — anchor status: not_found]
- [readme] Evaluation repository pointer: [evidence span withheld — anchor status: external_doc]
