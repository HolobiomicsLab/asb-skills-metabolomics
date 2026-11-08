---
name: benchmark-workflow-ranking-reproduction
description: Use when a task asks to reproduce a published metabolomics workflow-optimization
  benchmark (e.g., PMID28528106 / MTBLS403) by running NOREVA's full workflow scan
  and validating the resulting overall ranking of processing workflows.
when_to_use_negative:
- The dataset has fewer than 6 samples per class or timepoint — criterion Cc of the
  S5 assessment is unsuitable for such small sample sizes
- No reference metabolites (e.g., spike-ins) are available — criterion Ce (LogFC-based)
  cannot be applied, so a faithful multi-criteria reproduction is not possible
- The goal is raw spectral processing (file conversion or peak picking) rather than
  peak-table workflow optimization — tools like ProteoWizard or XCMS are needed first,
  since NOREVA expects an already-derived peak table
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: NOREVA
  role: R package that executes the full S1–S4 processing scan across thousands of
    combinatorial workflows and performs S5 multi-criteria performance assessment
    and ranking
  repo: https://github.com/idrblab/NOREVA
- name: R
  role: Execution environment for the NOREVA protocol; NOREVA is installed via install.packages/devtools
    and requires R (>3.5)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41596-021-00636-9
    title: Optimization of metabolomic data processing using NOREVA
  source_content_hash: sha256:6ce4f643c7c0ba80101c0e4bc3b9d7cd874dbc58303d7724abd4884be2e847a7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idrblab/NOREVA
  tools:
  - NOREVA
  - R
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# benchmark-workflow-ranking-reproduction

## Summary

Reproduces the full NOREVA benchmark scan for PMID28528106 (MetaboLights MTBLS403 tea dataset): thousands of combinatorially generated peak-table processing workflows (S1 filtering/imputation → S2 QCS correction → S3 transformation → S4 normalization) are executed and collectively assessed in S5 against five criteria, producing an overall ranking that must match Figure 4 of the Nature Protocols paper.

## When to use

Use when a task asks to reproduce a published metabolomics workflow-optimization benchmark (e.g., PMID28528106 / MTBLS403) by running NOREVA's full workflow scan and validating the resulting overall ranking of processing workflows.

## When NOT to use

- The dataset has fewer than 6 samples per class or timepoint — criterion Cc of the S5 assessment is unsuitable for such small sample sizes
- No reference metabolites (e.g., spike-ins) are available — criterion Ce (LogFC-based) cannot be applied, so a faithful multi-criteria reproduction is not possible
- The goal is raw spectral processing (file conversion or peak picking) rather than peak-table workflow optimization — tools like ProteoWizard or XCMS are needed first, since NOREVA expects an already-derived peak table

## Inputs

- MetaboLights MTBLS403 raw data files (https://www.ebi.ac.uk/metabolights/MTBLS403/files)
- Peak table standardized to the unified NOREVA format (rows = samples by name, columns = metabolites)
- Label file indicating time-course/multi-class sample groups (input to PrepareInuputFiles)

## Outputs

- ranking_table.csv — overall ranking of all evaluated (5,904) processing workflows
- Per-workflow scores from S5 performance assessment across multiple criteria

## How to apply

Download the raw MTBLS403 files from MetaboLights and manually standardize the peak table into NOREVA's unified format, preserving rows (samples named) and columns — this manual step is time-consuming because raw data organizations vary. Install the NOREVA R package (v2.1.1) from GitHub via devtools, along with its Bioconductor dependencies (pcaMethods, limma, impute, statTarget, ropls, vsn, etc.) and the NormalizeMets/ggord GitHub packages. Run PrepareInuputFiles(dataformat=1, ...) on the standardized table, then invoke the appropriate multiclass/time-course assessment function (e.g., normulticlassqcall with study assumptions SAalpha/SAbeta/SAgamma set to Y/N) so that the four processing stages (S1–S4) and the S5 multi-criteria assessment scan all combinatorial workflows. Expect >200 min runtime for this large dataset even with parallel computing and memory management. Finally, collect the workflow scores, build ranking_table.csv, and confirm the ordering matches Figure 4: HAM+LLR+NON+[SUM-LEV] at rank 1 and KNN+NWE+LOG+[PAR-LIN] at the last rank.

## Related tools

- **NOREVA** (R package that executes the full S1–S4 processing scan across thousands of combinatorial workflows and performs S5 multi-criteria performance assessment and ranking) — https://github.com/idrblab/NOREVA
- **R** (Execution environment for the NOREVA protocol; NOREVA is installed via install.packages/devtools and requires R (>3.5))

## Examples

```
library(NOREVA); f <- PrepareInuputFiles(dataformat=1, rawdata="MTBLS403_peaktable.csv", label="labels.txt"); normulticlassqcall(f, SAalpha="Y", SAbeta="Y", SAgamma="Y")
```

## Evaluation signals

- ranking_table.csv reproduces the Figure 4 ordering of PMID28528106 exactly, with HAM+LLR+NON+[SUM-LEV] at rank 1 and KNN+NWE+LOG+[PAR-LIN] at the last rank
- The scan covers the full combinatorial space (5,904 workflows) rather than a subset of S1–S4 method combinations
- All five S5 criteria are evaluated (or their inapplicability documented, e.g., Ce when no reference metabolites exist) rather than a single criterion
- Runtime is in the expected range — hundreds to thousands of minutes, with >200 min for this large dataset even with parallel computing and memory management
- The input peak table preserves rows (samples provided by name) and columns after manual standardization to the unified NOREVA format

## Limitations

- For datasets of relatively large size (e.g., PMID28528106), >200 min are still required despite parallel computing and corresponding memory management
- Criterion Cc is unsuitable for datasets with fewer than 6 samples in one timepoint or class, and criterion Ce cannot be applied when no reference metabolites (e.g., spike-ins) are available
- Manual peak-table standardization to the unified format is relatively time-consuming because raw data files are organized in diverse ways
- Some partial assessment functions write output files with names identical to Step 10 outputs and can silently overwrite earlier results
- Users need preliminary R programming experience, and mastering R is required to modify or improve the protocol

## Evidence

- [discussion] Optimal workflow ranking: [evidence span withheld — anchor status: not_found]
- [intro] Five-stage pipeline definition: "(S1) data filtering and imputation, (S2) QCS correction, (S3) transformation, (S4) normalization and (S5) performance assessment by multiple criteria"
- [discussion] Manual peak table standardization: [evidence span withheld — anchor status: not_found]
- [discussion] Large dataset runtime: [evidence span withheld — anchor status: not_found]
- [readme] README capability statement: [evidence span withheld — anchor status: external_doc]
