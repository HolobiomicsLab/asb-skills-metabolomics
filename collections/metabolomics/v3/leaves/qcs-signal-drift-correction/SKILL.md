---
name: qcs-signal-drift-correction
description: Use when a metabolomic peak table (multi-class or time-course) contains
  quality control samples (QCS) measured across the run, and instrumental signal drift
  must be corrected before transformation, normalization, and workflow ranking.
when_to_use_negative:
- The dataset lacks quality control samples (QCS) — use the no-QCS variants (normulticlassnoall
  / nortimecoursenoall) instead of the qcall functions
- Classes or time points contain fewer than 6 samples, since criterion Cc is unsuitable
  for small-sample datasets and will distort S5 ranking
- The goal is exploratory statistics or biomarker discovery rather than optimizing
  the peak-table processing workflow itself (NOREVA does not replace MetaboAnalyst/W4M-style
  downstream analysis)
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NOREVA
  role: R package executing the S1–S5 workflow scan including QCS-based signal drift
    correction (stage S2) and multi-criteria ranking; web service also at idrblab.org/noreva
  repo: https://github.com/idrblab/NOREVA
- name: R
  role: Runtime environment for the NOREVA protocol (requires BiocManager packages
    such as statTarget, pcaMethods, impute, vsn, and devtools-installed NormalizeMets
    and ggord)
- name: statTarget
  role: Bioconductor dependency installed for NOREVA that supports signal drift correction
    based on QCS within the workflow scan
- name: XCMS
  role: Upstream tool for preparing metabolomic peak tables from raw files before
    NOREVA standardization
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1038/s41596-021-00636-9
    title: Optimization of metabolomic data processing using NOREVA
  source_content_hash: sha256:30c0f7c99922895c415efba1a36454885ca492a43cc7c335116cfca9b496293d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idrblab/NOREVA
  tools:
  - NOREVA
  - R
  - XCMS
  - statTarget
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# qcs-signal-drift-correction

## Summary

Stage S2 of the NOREVA metabolomic peak-table processing pipeline: correct instrumental signal drift using repeatedly measured quality control samples (QCS) before transformation (S3) and normalization (S4), as part of a combinatorial workflow scan whose results are ranked by multiple criteria in S5.

## When to use

Use when a metabolomic peak table (multi-class or time-course) contains quality control samples (QCS) measured across the run, and instrumental signal drift must be corrected before transformation, normalization, and workflow ranking.

## When NOT to use

- The dataset lacks quality control samples (QCS) — use the no-QCS variants (normulticlassnoall / nortimecoursenoall) instead of the qcall functions
- Classes or time points contain fewer than 6 samples, since criterion Cc is unsuitable for small-sample datasets and will distort S5 ranking
- The goal is exploratory statistics or biomarker discovery rather than optimizing the peak-table processing workflow itself (NOREVA does not replace MetaboAnalyst/W4M-style downstream analysis)

## Inputs

- Metabolomic peak table standardized to the unified NOREVA format (rows = samples by name, columns = metabolites)
- Label file assigning samples to classes/time points (for time-course/multi-class analysis)
- Raw benchmark data from MetaboLights (e.g., MTBLS403, MTBLS6, MTBLS656) or NOREVA_exampledata.zip from idrblab.org/noreva
- Study assumption flags SAalpha, SAbeta, SAgamma (Y/N) for alpha/beta/gamma assumptions

## Outputs

- QCS-corrected peak tables (input to S3 transformation and S4 normalization)
- Workflow performance scores and ranking table across all evaluated workflow combinations (S5 criteria)
- Ranked workflow identifiers such as HAM+LLR+NON+[SUM-LEV] (rank 1 for PMID28528106)

## How to apply

Order the peak table into the unified NOREVA format (rows = samples by name, columns = metabolites) and load it via PrepareInuputFiles. Place QCS-based signal drift correction as stage S2, strictly after S1 (data filtering and imputation) and before S3 (transformation) and S4 (normalization), since NOREVA orders the stages S1→S2→S3→S4 with S5 assessing all combinatorially generated workflows by multiple criteria. For a multi-class dataset with QCS but no internal standards, call normulticlassqcall (for time-course data, nortimecourseqcall), declaring study assumptions alpha (all metabolites equally important), beta (metabolite abundance constant among samples), and gamma (majority of metabolite intensities unchanged) as Y/N flags so the scan evaluates thousands of three-to-five-calculation workflow combinations. The scan executes all workflow combinations and ranks them, e.g. for PMID28528106 5,904 workflows were scanned with HAM+LLR+NON+[SUM-LEV] ranked first; expect runtimes of hundreds to thousands of minutes even with NOREVA's parallel computing and memory management (v2.1.1). Confirm success by checking that the ranking_table output reproduces the paper's reported ordering and that the corrected peak tables feed cleanly into S5 assessment without overwriting prior output files.

## Related tools

- **NOREVA** (R package executing the S1–S5 workflow scan including QCS-based signal drift correction (stage S2) and multi-criteria ranking; web service also at idrblab.org/noreva) — https://github.com/idrblab/NOREVA
- **R** (Runtime environment for the NOREVA protocol (requires BiocManager packages such as statTarget, pcaMethods, impute, vsn, and devtools-installed NormalizeMets and ggord))
- **statTarget** (Bioconductor dependency installed for NOREVA that supports signal drift correction based on QCS within the workflow scan)
- **XCMS** (Upstream tool for preparing metabolomic peak tables from raw files before NOREVA standardization)

## Examples

```
library(NOREVA); d <- PrepareInuputFiles(dataformat=1, rawdata="peaktable.csv", label="labels.csv"); normulticlassqcall(d, SAalpha="Y", SAbeta="Y", SAgamma="Y")
```

## Evaluation signals

- The stage ordering S1→S2→S3→S4→S5 is respected: QCS correction is applied after filtering/imputation and before transformation and normalization
- The resulting ranking_table reproduces the published ordering, e.g. HAM+LLR+NON+[SUM-LEV] at rank 1 and KNN+NWE+LOG+[PAR-LIN] last for PMID28528106 (Figure 4), or HAM+NON+CUT+[CON-RAN] first and KNN+NON+LOG+[VAS-LIN] last for the MTBLS6 spike-in validation (Figure 6)
- All thousands of combinatorially generated workflows spanning three-to-five calculations are collectively assessed in S5, not a single hand-picked workflow
- The top-ranked workflow preserves known biology, e.g. monotonic dose response across 6/12/25/50 µL spiked aspartic acid and malic acid classes
- Output files from repeated assessment commands are not silently overwritten (NOREVA can overwrite Step 10 outputs with identically named files)

## Limitations

- Assessing thousands of workflows is very time-consuming: hundreds to thousands of minutes, with >200 min still required for large datasets (e.g., PMID28528106) despite parallel computing and memory management
- Successful QCS correction depends on upfront experimental design that adopts both QCSs and internal standards (ISs); NOREVA does not enforce this, and manual peak-table standardization is time-consuming due to diverse raw file organizations
- Only commonly applied processing methods are covered; extending the method set cannot be done automatically and requires manual updates by bioinformaticians

## Evidence

- [discussion] NOREVA enabled the signal drift correction based on QCS, which had been successfully integrated into the metabolomic processing workflows: [evidence span withheld — anchor status: not_found]
- [readme] NOREVA R package scope and version capabilities: [evidence span withheld — anchor status: external_doc]
