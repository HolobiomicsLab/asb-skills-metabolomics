---
name: metabolomic-data-transformation-and-normalization
description: Use when a metabolomic peak table has passed filtering/imputation (S1)
  and QCS correction (S2) and must be nonlinearly transformed (e.g., LOG) and normalized
  (e.g., VAS-LIN) before performance assessment in NOREVA.
when_to_use_negative:
- Input is a raw proprietary instrument file that still needs conversion and peak-table
  generation (use Proteowizard, nmrML, XCMS or NMRProc Flow first)
- The peak table has not yet been filtered/imputed (S1) or QCS-corrected (S2) — NOREVA's
  four processing stages are ordered S1→S2→S3→S4 and cannot be run out of sequence
- The goal is performance assessment itself (S5) rather than producing the normalized
  processing outcome that assessment consumes
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: NOREVA
  role: R package providing the transformation/normalization functions (PrepareInuputFiles,
    normulticlassqcall, normulticlassnoall, normulticlassisall, nortimecourseqcall,
    nortimecoursenoall) with study-assumption flags
  repo: github.com/idrblab/NOREVA
- name: R
  role: Execution environment; R dependencies such as vsn, limma, statTarget, ropls
    and NormalizeMets back the transformation/normalization methods
  repo: https://www.r-project.org/
- name: NormalizeMets
  role: GitHub-installed dependency supplying normalization methods used by NOREVA's
    S4 stage
  repo: devtools::install_github("metabolomicstats/NormalizeMets")
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41596-021-00636-9
    title: Optimization of metabolomic data processing using NOREVA
  source_content_hash: sha256:61a7c9b88d9dbc5bc8ad067d3f5864aab72e3f9628b6fc47d19e7d441eeb9c10
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetaboLights
  - NOREVA
  - NormalizeMets
  - R
  - statTarget
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- metabolomics-data-normalization
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# metabolomic-data-transformation-and-normalization

## Summary

Applies the S3 (data transformation) and S4 (data normalization) stages of the NOREVA protocol to standardized metabolomic peak tables, correcting data heteroscedasticity and removing unwanted technical variation after the study-assumption (S4a) has been determined.

## When to use

Use when a metabolomic peak table has passed filtering/imputation (S1) and QCS correction (S2) and must be nonlinearly transformed (e.g., LOG) and normalized (e.g., VAS-LIN) before performance assessment in NOREVA.

## When NOT to use

- Input is a raw proprietary instrument file that still needs conversion and peak-table generation (use Proteowizard, nmrML, XCMS or NMRProc Flow first)
- The peak table has not yet been filtered/imputed (S1) or QCS-corrected (S2) — NOREVA's four processing stages are ordered S1→S2→S3→S4 and cannot be run out of sequence
- The goal is performance assessment itself (S5) rather than producing the normalized processing outcome that assessment consumes

## Inputs

- Standardized metabolomic peak table (rows = samples named, columns = metabolites) after S1 filtering/imputation and S2 QCS correction
- Label file for time-course/multi-class sample annotation (input to PrepareInuputFiles)
- Study-assumption settings (SAalpha, SAbeta, SAgamma as Y/N)
- Internal-standard column numbers (for normulticlassisall)

## Outputs

- Transformed and normalized peak table ready for S5 multi-criteria performance assessment
- NOREVA output files from the normulticlass*/nortimecourse* function call

## How to apply

First load the standardized peak table (rows = samples, columns = metabolites) via NOREVA's PrepareInuputFiles, specifying whether the format is NOREVA-standardized (dataformat=1) or from one of 12 external tools (dataformat=2). Apply S3 data transformation — nonlinear conversions such as LOG — to correct heteroscedasticity, then at S4a determine the type of study assumption (alpha: all metabolites equally important; beta: metabolite abundance constant among samples; gamma: intensities of the majority of metabolites unchanged) before normalization. Run the NOREVA function matching the dataset type and QCS/IS availability (normulticlassqcall, normulticlassnoall, normulticlassisall, nortimecourseqcall, nortimecoursenoall), passing the SAalpha/SAbeta/SAgamma flags (Y/N) and IS column numbers when internal standards are present. The stage order is fixed: transformation precedes normalization, as in scanned workflows such as KNN+NON+LOG+[VAS-LIN], and the normalized output feeds directly into the S5 multi-criteria assessment.

## Related tools

- **NOREVA** (R package providing the transformation/normalization functions (PrepareInuputFiles, normulticlassqcall, normulticlassnoall, normulticlassisall, nortimecourseqcall, nortimecoursenoall) with study-assumption flags) — github.com/idrblab/NOREVA
- **R** (Execution environment; R dependencies such as vsn, limma, statTarget, ropls and NormalizeMets back the transformation/normalization methods) — https://www.r-project.org/
- **NormalizeMets** (GitHub-installed dependency supplying normalization methods used by NOREVA's S4 stage) — devtools::install_github("metabolomicstats/NormalizeMets")

## Examples

```
library(NOREVA); res <- normulticlassqcall(fileName="peaktable_input", SAalpha="Y", SAbeta="Y", SAgamma="Y")
```

## Evaluation signals

- Output peak table preserves the unified format: rows remain samples (provided by name) and columns remain metabolites
- The chosen normalization matches the dataset scenario: QCS-without-IS, no-QCS-no-IS, IS-without-QCS, or time-course variants, and the SAalpha/SAbeta/SAgamma settings reflect the dataset's study assumptions
- Stage ordering is respected: transformation (e.g., LOG) is applied before normalization (e.g., VAS-LIN) as in the scanned workflow encodings
- Output files are checked against earlier NOREVA step outputs, since partial assessment functions can generate identically named files that overwrite previous results and produce misleading outputs
- The normalized outcome is consistent with the Good/Fair/Poor classification later reported in the S5 comparison for the same workflow–dataset pair

## Limitations

- Manually standardizing the peak table to the unified format is relatively time-consuming due to diverse organizations of raw data files
- Successful application depends on upfront experimental design with both QCSs and ISs adopted, which NOREVA does not enforce
- Users need preliminary R programming experience, and mastery of R is required to modify or improve the transformation/normalization functions
- For relatively large datasets (e.g., PMID28528106), >200 min are still required despite parallel computing and memory management

## Evidence

- [readme] Function interface with SA flags: [evidence span withheld — anchor status: not_found]
- [discussion] Unified input format requirement: [evidence span withheld — anchor status: not_found]
- [discussion] Output-overwrite pitfall: [evidence span withheld — anchor status: not_found]
