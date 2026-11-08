---
name: noreva-workflow-scan-configuration
description: Use when the input is a time-course metabolomic peak table (e.g. MTBLS656
  plasma samples labelled T0–T48) and the goal is to identify the study-specific optimal
  processing workflow by exhaustively scoring all S1–S4 method combinations.
when_to_use_negative:
- The input is a case–control or multi-class (not time-course) dataset — use NOREVA's
  normulticlassqcall/noall/isall functions instead
- The data are still raw instrument files (e.g. vendor LC-MS/GC-MS formats); peak-table
  generation must first be done with upstream tools such as Proteowizard, XCMS or
  NMRProc Flow
- Only a single predefined processing workflow needs to be applied and no comparative
  optimization across S1–S4 combinations is required
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3272
tools:
- name: NOREVA
  role: R package whose PrepareInuputFiles and nortimecourseqcall/nortimecoursenoall
    functions build the input peak table, generate the S1–S4 workflow combinations,
    scan all of them and rank them by multi-criteria scores
  repo: https://github.com/idrblab/NOREVA
- name: R
  role: Execution environment for the R-based NOREVA protocol; required to install
    dependencies (Biobase, pcaMethods, limma, ropls, statTarget, NormalizeMets, ggord,
    etc.) and run the local workflow scan
  repo: https://www.r-project.org/
- name: MetaboLights MTBLS656
  role: Source of the LC-MS quadrupole plasma time-course benchmark peak table used
    to configure and validate the workflow scan
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41596-021-00636-9
    title: Optimization of metabolomic data processing using NOREVA
  source_content_hash: sha256:4d5c4418f773313c808402453283a581230c7508626a23a118a6780a8085beeb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idrblab/NOREVA
  tools:
  - MetaboLights MTBLS656
  - NOREVA
  - NOREVA_exampledata
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# noreva-workflow-scan-configuration

## Summary

Configure the NOREVA R package to combinatorially generate and comprehensively scan the candidate peak-table processing workflows for a time-course metabolomics dataset, ranking every workflow (e.g. 1,968 combinations of S1 filtering/imputation, S2 QCS-correction, S3 transformation and S4 normalization methods) with five independent performance-assessment criteria.

## When to use

Use when the input is a time-course metabolomic peak table (e.g. MTBLS656 plasma samples labelled T0–T48) and the goal is to identify the study-specific optimal processing workflow by exhaustively scoring all S1–S4 method combinations.

## When NOT to use

- The input is a case–control or multi-class (not time-course) dataset — use NOREVA's normulticlassqcall/noall/isall functions instead
- The data are still raw instrument files (e.g. vendor LC-MS/GC-MS formats); peak-table generation must first be done with upstream tools such as Proteowizard, XCMS or NMRProc Flow
- Only a single predefined processing workflow needs to be applied and no comparative optimization across S1–S4 combinations is required

## Inputs

- Time-course metabolomic peak table in NOREVA's unified format (sample rows, feature columns; e.g. MTBLS656 plasma data or NOREVA_exampledata)
- Label file mapping samples to time-point labels (T0–T48)
- Study-assumption settings SAalpha, SAbeta, SAgamma ('Y'/'N' each)
- Indicator of whether quality-control samples and/or internal-standard columns are present (IS column numbers, e.g. 2,6,9,n)

## Outputs

- Ranked list of all candidate processing workflows (e.g. 1,968) with multi-criteria performance scores
- Optimal workflow string (expected: MEI+NON+CUT+[QUA-LEV] for MTBLS656) and worst-ranked workflow (KNN+NON+LOG+[PAR-SUM])
- Processed peak table produced by the selected workflow, from which per-metabolite time series (e.g. cortisol across T0–T48) can be extracted

## How to apply

First standardize the peak table to NOREVA's unified format (sample rows, feature columns) with a separate label file carrying the time-point labels, using PrepareInuputFiles. Then call the time-course assessment function (nortimecourseqcall when quality-control samples are present but no internal standards; nortimecoursenoall when neither is available), specifying the study assumptions SAalpha (all metabolites equally important), SAbeta (metabolite abundance constant among samples) and SAgamma (majority of metabolites unchanged) as 'Y'/'N'. The function generates the combinatorial space of workflows — e.g. 1,968 combinations such as MEI/KNN/MIN/CUT (S1) x NON (S2) x LOG/NON (S3) x QUA-LEV/PAR-SUM (S4) — and evaluates each with the multi-criteria assessment, ranking them. On the MTBLS656 benchmark the expected optimum is MEI+NON+CUT+[QUA-LEV] (rank 1) and KNN+NON+LOG+[PAR-SUM] (last). Rationale: workflow performance is study-dependent, so the article insists on assessing as many combinations as possible for the dataset at hand rather than importing a workflow that worked elsewhere.

## Related tools

- **NOREVA** (R package whose PrepareInuputFiles and nortimecourseqcall/nortimecoursenoall functions build the input peak table, generate the S1–S4 workflow combinations, scan all of them and rank them by multi-criteria scores) — https://github.com/idrblab/NOREVA
- **R** (Execution environment for the R-based NOREVA protocol; required to install dependencies (Biobase, pcaMethods, limma, ropls, statTarget, NormalizeMets, ggord, etc.) and run the local workflow scan) — https://www.r-project.org/
- **MetaboLights MTBLS656** (Source of the LC-MS quadrupole plasma time-course benchmark peak table used to configure and validate the workflow scan)

## Examples

```
library(NOREVA); res <- nortimecourseqcall(fileName = "MTBLS656_prepared", SAalpha = "Y", SAbeta = "Y", SAgamma = "Y")
```

## Evaluation signals

- The output ranking file enumerates the full combinatorial space (e.g. 1,968 workflows) and places MEI+NON+CUT+[QUA-LEV] first and KNN+NON+LOG+[PAR-SUM] last for the MTBLS656 time-course benchmark
- Each workflow carries scores from the multiple independent assessment criteria (not a single criterion), consistent with the five-criteria assessment stage (S5)
- Extracting the cortisol series from the first-ranked workflow's processed table reproduces the reported circadian rise/fall across T0–T48, while the last-ranked workflow's trace visibly distorts it
- The scan was run with explicit study-assumption settings (SAalpha/SAbeta/SAgamma) and the correct time-course function variant (with vs. without QCS/IS)
- The run is reproducible from the NOREVA GitHub release (NOREVA.2.1.1.tar.gz, GPL-3.0) with all declared dependencies installed

## Limitations

- The scan is very time-consuming: 'the time spent applying the NOREVA protocol varies from hundreds to thousands of minutes', and datasets of relatively large size (e.g. PMID28528106) need >200 min even with parallel computing and memory management
- Criterion Cc is unsuitable for datasets with fewer than 6 samples per timepoint/class; criterion Ce (LogFC) cannot be applied when no reference metabolites (spike-ins) are available
- Some partial assessment functions produce output files with names identical to earlier steps, which can overwrite and invalidate previous outputs
- The peak table must be manually standardized to the unified row/column format, which is time-consuming due to diverse raw-data organizations; successful application also presupposes good experimental design with both QCSs and ISs
- Users need preliminary R programming experience, and mastery of R is required to modify or improve the protocol

## Evidence

- [readme] This function enables the performance assessment of metabolomic data processing for the time-course dataset (with quality control sample but without internal standard): [evidence span withheld — anchor status: external_doc]
- [intro] characterized by its combinatorial diversity (thousands of processing workflows can be generated by combining various methods among stages) and study dependency: [evidence span withheld — anchor status: not_found]
- [readme] nortimecourseqcall(fileName, SAalpha="Y", SAbeta="Y", SAgamma="Y"): [evidence span withheld — anchor status: external_doc]
