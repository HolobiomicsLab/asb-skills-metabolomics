---
name: gcms-internal-standard-dose-gradient-analysis
description: Use when a multiclass metabolomic peak table contains internal standards
  spiked at known levels (e.g. 6/12/25/50 µL classes) and you need to select or validate
  a NOREVA processing workflow by whether it preserves the true dose gradient.
when_to_use_negative:
- The dataset has no reference (spiked internal-standard) metabolites — NOREVA states
  criterion e (LogFC) cannot be applied in that case, so the dose-gradient validation
  is infeasible.
- The study is case–control or time-course rather than multiclass — use NOREVA's corresponding
  time-course/case-control functions instead of the multiclass IS workflow.
- Input is raw instrument data rather than a peak table — peak tables must first be
  generated with tools like XCMS or NMRProc Flow and converted (e.g. via Proteowizard
  to an open format) before NOREVA can be applied.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NOREVA
  role: Runs the multiclass IS-aware workflow scan (normulticlassisall), scores/ranks
    processing workflows, and evaluates spike-in gradient preservation via criterion
    e
  repo: github.com/idrblab/NOREVA
- name: R
  role: Local execution environment for the NOREVA protocol, including dose-response
    fitting and Figure 6-style plotting
- name: XCMS
  role: Preparation of the metabolomic peak table from raw GC-MS data prior to NOREVA
    input
- name: ggord
  role: R plotting utility for the dose-response figure mirroring the paper's Figure
    6 layout
  repo: fawda123/ggord
- name: NormalizeMets
  role: Dependency of the NOREVA R package providing normalization methods among the
    scanned workflow combinations
  repo: metabolomicstats/NormalizeMets
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41596-021-00636-9
    title: Optimization of metabolomic data processing using NOREVA
  source_content_hash: sha256:3d9463958c21789f7a5b33ca48a31152b6c1959831933baa3f8fb8784330daec
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NOREVA
  - NormalizeMets
  - R
  - XCMS
  - ggord
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# gcms-internal-standard-dose-gradient-analysis

## Summary

Validates metabolomic data-processing workflows against the known dose gradient of spiked internal-standard metabolites in a GC-MS multiclass dataset (fetal calf serum, MetaboLights MTBLS6), reproducing the NOREVA Figure 6 comparison in which the top-ranked workflow HAM+NON+CUT+[CON-RAN] preserves the 6/12/25/50 µL dose gradient of spiked aspartic acid and malic acid while the last-ranked KNN+NON+LOG+[VAS-LIN] does not.

## When to use

Use when a multiclass metabolomic peak table contains internal standards spiked at known levels (e.g. 6/12/25/50 µL classes) and you need to select or validate a NOREVA processing workflow by whether it preserves the true dose gradient.

## When NOT to use

- The dataset has no reference (spiked internal-standard) metabolites — NOREVA states criterion e (LogFC) cannot be applied in that case, so the dose-gradient validation is infeasible.
- The study is case–control or time-course rather than multiclass — use NOREVA's corresponding time-course/case-control functions instead of the multiclass IS workflow.
- Input is raw instrument data rather than a peak table — peak tables must first be generated with tools like XCMS or NMRProc Flow and converted (e.g. via Proteowizard to an open format) before NOREVA can be applied.

## Inputs

- GC-MS peak table from MetaboLights MTBLS6 (fetal calf serum), samples as rows, metabolites as columns, in NOREVA unified input format
- Label file assigning each sample to a multiclass dose group (6, 12, 25, 50 µL)
- Column indices of the spiked internal-standard metabolites (aspartic acid, malic acid) in the peak table

## Outputs

- Ranked list of candidate processing workflows scored by NOREVA's multiple criteria (HAM+NON+CUT+[CON-RAN] first, KNN+NON+LOG+[VAS-LIN] last)
- Processed intensity tables of the spiked aspartic acid and malic acid per dose class under top- and bottom-ranked workflows
- Dose-response curves (Figure 6-style side-by-side R/ggord plot) showing gradient preservation vs. failure

## How to apply

Download the GC-MS quadrupole fetal calf serum benchmark from MetaboLights (MTBLS6) and manually standardize the peak table into NOREVA's unified format, preserving rows as named samples and columns as metabolites, plus a label file for the multiclass dose levels. Identify the spiked internal standards (aspartic acid and malic acid, 6/12/25/50 µL per class) and pass their peak-table column numbers to the IS-aware NOREVA function `normulticlassisall`, which scans the combinatorial three-to-five-stage pipeline — (S1) data filtering and imputation, (S2) QCS correction, (S3) transformation, (S4) normalization — and ranks thousands of candidate workflows by multiple criteria including the spike-in-based criterion e (LogFC). Confirm the ranking places HAM+NON+CUT+[CON-RAN] first and KNN+NON+LOG+[VAS-LIN] last, then extract the processed intensities of the two spiked metabolites per dose class under each workflow and fit a dose-response to test monotonicity against the 6/12/25/50 µL reference. Finally, plot dose-response curves for both metabolites under both workflows side by side (Figure 6 layout) using R/ggord plotting utilities; the IS-based criterion is precisely what makes this validation possible, since datasets without reference metabolites cannot be assessed with criterion e.

## Related tools

- **NOREVA** (Runs the multiclass IS-aware workflow scan (normulticlassisall), scores/ranks processing workflows, and evaluates spike-in gradient preservation via criterion e) — github.com/idrblab/NOREVA
- **R** (Local execution environment for the NOREVA protocol, including dose-response fitting and Figure 6-style plotting)
- **XCMS** (Preparation of the metabolomic peak table from raw GC-MS data prior to NOREVA input)
- **ggord** (R plotting utility for the dose-response figure mirroring the paper's Figure 6 layout) — fawda123/ggord
- **NormalizeMets** (Dependency of the NOREVA R package providing normalization methods among the scanned workflow combinations) — metabolomicstats/NormalizeMets

## Examples

```
library(NOREVA); normulticlassisall("MTBLS6_peaktable", IS=2,6)
```

## Evaluation signals

- Top-ranked workflow HAM+NON+CUT+[CON-RAN] yields processed intensities of both spiked metabolites that increase monotonically across the 6/12/25/50 µL dose classes.
- Last-ranked workflow KNN+NON+LOG+[VAS-LIN] produces non-monotonic or dose-discordant intensities for aspartic acid and malic acid under the same dose-response fit.
- The workflow ranking reproduces the qualitative ordering reported in the paper's Figure 6 (first-ranked preserves, last-ranked fails).
- The peak table conforms to NOREVA's unified format: rows are samples by name, columns are metabolites, and IS column indices match the spiked metabolites.
- All candidate combinations spanning the S1–S4 stages are evaluated so that both endpoint workflows (HAM+NON+CUT+[CON-RAN] and KNN+NON+LOG+[VAS-LIN]) appear among the ranked results.

## Limitations

- Scanning thousands of workflows is time-consuming — hundreds to thousands of minutes even with parallel computing, and >200 min for large datasets despite parallel computing and memory management.
- Manual standardization of the peak table into the unified format is relatively time-consuming because raw data files are organized in diverse ways.
- The IS-based criterion e requires that internal standards were deliberately spiked at known doses during experimental design; NOREVA does not enforce this, so missing or mis-annotated IS columns invalidate the gradient validation.
- Users need preliminary R programming experience, and mastery of R is required to modify or improve the protocol.

## Evidence

- [intro] Five-stage pipeline definition: "(S1) data filtering and imputation, (S2) QCS correction, (S3) transformation, (S4) normalization and (S5) performance assessment by multiple criteria"
- [discussion] Criterion e requires reference metabolites: [evidence span withheld — anchor status: not_found]
- [discussion] Unified peak-table format: [evidence span withheld — anchor status: not_found]
- [discussion] MTBLS6 is the GC-MS quadrupole benchmark: "GC-MS quadrupole (MTBLS6)"
- [readme] IS column specification for multiclass function: [evidence span withheld — anchor status: external_doc]
- [discussion] ggord installation for plotting: "devtools::install_github("fawda123/ggord")"
