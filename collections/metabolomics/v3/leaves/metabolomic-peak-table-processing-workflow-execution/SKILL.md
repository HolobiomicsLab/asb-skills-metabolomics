---
name: metabolomic-peak-table-processing-workflow-execution
description: Use when you have metabolomic peak tables (e.g., MetaboLights MTBLS6,
  MTBLS583, MTBLS403, MTBLS370, MTBLS59, MTBLS656, MTBLS319, MTBLS1569, MTBLS665)
  and must run a specific method combination across the four processing stages to
  produce a processed output for performance assessment.
when_to_use_negative:
- You only need to interpret existing Good/Fair/Poor rankings already reported (e.g.,
  in NOREVA's Table 3) rather than run the processing stages yourself
- The dataset has fewer than 6 samples in one timepoint or class — criterion Cc is
  unsuitable, so the downstream assessment, not the execution itself, must be adapted
- The goal is raw-file conversion or peak picking (Proteowizard, nmrML, XCMS) rather
  than post-peak-table processing of an existing metabolite feature table
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: NOREVA
  role: R package that executes the four processing stages (S1 filtering/imputation,
    S2 QCS correction, S3 transformation, S4 normalization) and scans thousands of
    workflow combinations
  repo: https://github.com/idrblab/NOREVA
- name: R
  role: Runtime environment for the R-based protocol enabling local optimization;
    preliminary R programming experience is needed
- name: MetaboLights
  role: Source of the nine benchmark peak-table datasets downloaded for workflow execution
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41596-021-00636-9
    title: Optimization of metabolomic data processing using NOREVA
  source_content_hash: sha256:7e17034ba950356323e240ea24d9507ed5d588b7055f4d006bc4b1c70ebfaeb1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idrblab/NOREVA
  tools:
  - MetaboLights
  - NOREVA
  - R
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# metabolomic-peak-table-processing-workflow-execution

## Summary

Executes candidate metabolomic peak-table processing workflows (e.g., MDI+NON+NON+[CON-AUT], HAM+LLR+NON+[SUM-LEV], KNN+NON+LOG+[VAS-LIN]) through the NOREVA R package, running the four ordered stages S1 filtering/imputation, S2 QCS correction, S3 transformation, and S4 normalization on standardized MetaboLights benchmark tables so their outputs can be scored in stage S5.

## When to use

Use when you have metabolomic peak tables (e.g., MetaboLights MTBLS6, MTBLS583, MTBLS403, MTBLS370, MTBLS59, MTBLS656, MTBLS319, MTBLS1569, MTBLS665) and must run a specific method combination across the four processing stages to produce a processed output for performance assessment.

## When NOT to use

- You only need to interpret existing Good/Fair/Poor rankings already reported (e.g., in NOREVA's Table 3) rather than run the processing stages yourself
- The dataset has fewer than 6 samples in one timepoint or class — criterion Cc is unsuitable, so the downstream assessment, not the execution itself, must be adapted
- The goal is raw-file conversion or peak picking (Proteowizard, nmrML, XCMS) rather than post-peak-table processing of an existing metabolite feature table

## Inputs

- Metabolomic peak table (e.g., from MetaboLights MTBLS6/583/403/370/59/656/319/1569/665 or NOREVA_exampledata.zip) standardized to NOREVA's unified row-sample/column-metabolite format
- Label file naming classes or time points for multi-class/time-course datasets
- Column indices of internal standards (IS) when ISs are present
- A candidate workflow specification as a method combination, e.g., MDI+NON+NON+[CON-AUT] or HAM+LLR+NON+[SUM-LEV]

## Outputs

- Processed peak table (filtered, QCS-corrected, transformed, normalized) per workflow per dataset
- Numeric performance-ready outputs that feed NOREVA's S5 assessment against criteria Ca–Cd and the Good/Fair/Poor classification

## How to apply

First standardize each peak table manually into NOREVA's unified format, preserving rows as samples (by name) and columns as metabolites, optionally passing a label file for time-course/multi-class design via PrepareInuputFiles. Select the correct assessment entry function based on the dataset's data availability (QCS present/absent, internal standards present/absent; multi-class vs time-course) and set the study-assumption flags SAalpha, SAbeta, SAgamma (metabolites equally important; abundance constant among samples; majority of metabolites unchanged). Execute the workflow's method combination in the fixed stage order: S1 data filtering and imputation (removing technical/mathematical uninformative features by a prespecified cutoff), S2 QCS-based signal-drift correction, S3 nonlinear transformation correcting heteroscedasticity, and S4 normalization chosen according to the study assumption. Run via the NOREVA 2.1.1 R package, which applies parallel computing, memory management, and I/O optimization; be prepared for long runtimes (hundreds to thousands of minutes for large datasets like PMID28528106). Repeat for every workflow under the combinatorial scan (e.g., 5,904 workflows for PMID28528106) so that the resulting processed tables can be scored against criteria Ca–Cd in stage S5.

## Related tools

- **NOREVA** (R package that executes the four processing stages (S1 filtering/imputation, S2 QCS correction, S3 transformation, S4 normalization) and scans thousands of workflow combinations) — https://github.com/idrblab/NOREVA
- **R** (Runtime environment for the R-based protocol enabling local optimization; preliminary R programming experience is needed)
- **MetaboLights** (Source of the nine benchmark peak-table datasets downloaded for workflow execution)

## Examples

```
library(NOREVA); d <- PrepareInuputFiles(dataformat="1", rawdata="MTBLS403_peaktable.csv", label="labels.csv"); normulticlassqcall(d, SAalpha="Y", SAbeta="Y", SAgamma="Y")
```

## Evaluation signals

- The output peak table preserves NOREVA's unified structure: rows are samples (by name) and columns are metabolites
- Executed stage order matches the fixed sequence S1 filtering/imputation → S2 QCS correction → S3 transformation → S4 normalization for every workflow
- The set of processed outputs covers all workflows in the combinatorial scan (e.g., 5,904 for PMID28528106) before any S5 assessment is attempted
- Feeding each output into S5 yields numeric criterion scores (Ca–Cd) that convert to Good/Fair/Poor ratings consistent with those in Table 3 for the same workflow–dataset pair
- Study-assumption flags (SAalpha, SAbeta, SAgamma) were set deliberately and the correct entry function was chosen for the dataset's QCS/IS availability

## Limitations

- Manual standardization to the unified format is time-consuming because raw data files are organized in diverse ways
- Assessment of thousands of workflows takes hundreds to thousands of minutes even with parallel computing; large datasets (e.g., PMID28528106) still require >200 min
- Users need preliminary R programming experience, and mastery of R is required to modify or improve the protocol
- Only commonly applied processing methods are covered; adding new methods cannot be done automatically and requires manual updates by bioinformaticians
- Successful application depends on upfront experimental design adopting both QCSs and ISs, which NOREVA does not enforce

## Evidence

- [methods] Four stages of metabolomic data processing are ordered from (S1) data filtering and imputation, to (S2) QCS correction, to (S3) data transformation and to (S4) data normalization: [evidence span withheld — anchor status: not_found]
- [discussion] Manually standardize a peak table file of the unified format by preserving the rows (samples provided by name) and columns: [evidence span withheld — anchor status: not_found]
- [discussion] the workflow 'HAM + LLR + NON + [SUM-LEV]' is discovered as the optimal (ranked the first) one: [evidence span withheld — anchor status: not_found]
- [readme] This function enables the preparation and input of peak table which facilitate the subsequent application of other NOREVA functions.: [evidence span withheld — anchor status: external_doc]
- [readme] NOREVA version 2.1.1 realizes (1) the parallel computing together with memory management and (2) the optimization of I/O efficiency.: [evidence span withheld — anchor status: external_doc]
