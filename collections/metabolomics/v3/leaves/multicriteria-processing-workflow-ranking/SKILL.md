---
name: multicriteria-processing-workflow-ranking
description: Use when a metabolomics peak table (e.g., MTBLS656 plasma time-course
  LC-MS data with T0–T48 labels) must be processed and the optimal combination of
  filtering/imputation, QCS correction, transformation, and normalization methods
  is unknown and study-dependent.
when_to_use_negative:
- Datasets with fewer than 6 samples per class or timepoint, since criterion Cc is
  unsuitable at small sample size
- Studies with no reference metabolites (e.g., spike-ins/internal standards available),
  since criterion Ce (LogFC) cannot be applied
- Raw spectral files needing peak detection first — the skill expects a peak table;
  convert/peak-pick with tools such as Proteowizard or XCMS beforehand
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: NOREVA
  role: R package that combinatorially generates thousands of processing workflows,
    scans them all, and ranks them via five independent criteria (S5)
  repo: https://github.com/idrblab/NOREVA
- name: R
  role: Runtime environment for the R-based NOREVA protocol enabling local optimization
    of data processing
  repo: https://www.r-project.org/
- name: MetaboLights MTBLS656
  role: 'Source benchmark dataset: LC-MS quadrupole plasma time-course peak table
    (T0–T48) used to demonstrate ranking and cortisol circadian-pattern validation'
  repo: https://www.ebi.ac.uk/metabolights/MTBLS656/files
- name: NOREVA_exampledata
  role: All publication data made available by the authors for reproducing the workflow
    scan and Figure 5 comparison
  repo: https://idrblab.org/noreva/NOREVA_exampledata.zip
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41596-021-00636-9
    title: Optimization of metabolomic data processing using NOREVA
  source_content_hash: sha256:e5f3ed355511732d08263d3e008b602eeb4219f4d0c94799d7cbefccc1737743
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/idrblab/NOREVA
  tools:
  - MetaboLights MTBLS656
  - NOREVA
  - NOREVA_exampledata
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# multicriteria-processing-workflow-ranking

## Summary

Ranks thousands of metabolomic peak-table processing workflows (S1 filtering/imputation through S4 normalization) using NOREVA's five independent assessment criteria, so that the study-specific optimal workflow is discovered instead of an arbitrarily chosen one. Grounded in the Nature Protocols NOREVA protocol demonstrated on the MTBLS656 time-course dataset.

## When to use

Use when a metabolomics peak table (e.g., MTBLS656 plasma time-course LC-MS data with T0–T48 labels) must be processed and the optimal combination of filtering/imputation, QCS correction, transformation, and normalization methods is unknown and study-dependent.

## When NOT to use

- Datasets with fewer than 6 samples per class or timepoint, since criterion Cc is unsuitable at small sample size
- Studies with no reference metabolites (e.g., spike-ins/internal standards available), since criterion Ce (LogFC) cannot be applied
- Raw spectral files needing peak detection first — the skill expects a peak table; convert/peak-pick with tools such as Proteowizard or XCMS beforehand

## Inputs

- MetaboLights MTBLS656 plasma time-course peak table (sample rows, feature columns, time-point labels T0–T48)
- Label file mapping samples to time-course/multi-class classes
- NOREVA unified-format peak table (output of PrepareInuputFiles)
- NOREVA example data bundle (NOREVA_exampledata.zip)

## Outputs

- Ranked table of all scanned processing workflows with multi-criteria (five-criterion) performance scores
- First-ranked workflow (MTBLS656: MEI+NON+CUT+[QUA-LEV]) and last-ranked workflow (KNN+NON+LOG+[PAR-SUM])
- Processed cortisol intensity time series (T0–T48) from first- and last-ranked workflows
- Comparison panel figure (figure5_cortisol_pattern.png) plus accompanying CSV of processed values

## How to apply

Standardize the peak table into NOREVA's unified format (samples as rows, features as columns, plus a label file for time-course/multi-class classes). Run the NOREVA R package's comprehensive scan, which combinatorially generates all candidate workflows (e.g., 1,968 for MTBLS656, combining S1 methods such as MEI, KNN, MIN, CUT; S2 options such as NON; S3 options such as LOG, NON; and S4 options such as QUA-LEV, PAR-SUM) and assesses each with five independent criteria in the final performance-assessment stage (S5). Rank all workflows by their multi-criteria scores, treating the top-ranked combination as the study-specific optimal processing. For MTBLS656, verify the scan reproduces MEI+NON+CUT+[QUA-LEV] first-ranked and KNN+NON+LOG+[PAR-SUM] last-ranked. Finally, extract the processed intensity series from the ranked outputs (e.g., cortisol across T0–T48) to confirm the top workflow preserves expected biological patterns while the bottom one distorts them.

## Related tools

- **NOREVA** (R package that combinatorially generates thousands of processing workflows, scans them all, and ranks them via five independent criteria (S5)) — https://github.com/idrblab/NOREVA
- **R** (Runtime environment for the R-based NOREVA protocol enabling local optimization of data processing) — https://www.r-project.org/
- **MetaboLights MTBLS656** (Source benchmark dataset: LC-MS quadrupole plasma time-course peak table (T0–T48) used to demonstrate ranking and cortisol circadian-pattern validation) — https://www.ebi.ac.uk/metabolights/MTBLS656/files
- **NOREVA_exampledata** (All publication data made available by the authors for reproducing the workflow scan and Figure 5 comparison) — https://idrblab.org/noreva/NOREVA_exampledata.zip

## Examples

```
result <- nortimecourseqcall(fileName, SAalpha="Y", SAbeta="Y", SAgamma="Y")
```

## Evaluation signals

- The produced ranking file lists MEI+NON+CUT+[QUA-LEV] first and KNN+NON+LOG+[PAR-SUM] last among the 1,968 scanned workflows
- Every workflow carries scores from the five independent criteria used in NOREVA's S5 assessment stage
- The first-ranked workflow's extracted cortisol trace across T0–T48 qualitatively matches the reported circadian rise/fall pattern
- The last-ranked workflow's cortisol trace visibly distorts the circadian pattern, confirming ranking discriminates workflows
- The figure5_cortisol_pattern.png panel and accompanying CSV contain consistent processed values for both traces

## Limitations

- Assessing thousands of workflows is time-consuming: hundreds to thousands of minutes, and >200 min for large datasets (e.g., PMID28528106) even with parallel computing and memory management
- Criterion Cc is unsuitable for datasets with fewer than 6 samples per class/timepoint, and criterion Ce (LogFC) requires reference metabolites (e.g., spike-ins)
- Some partial assessment functions generate output files with names identical to those from Step 10 and can overwrite existing outputs, producing misleading results
- Users need preliminary R programming experience, and mastery of R is required to modify or improve the protocol; successful application also depends on upfront experimental design including both QCSs and ISs

## Evidence

- [intro] Peak table processing workflows have combinatorial diversity (thousands of workflows from combining methods) and study dependency: [evidence span withheld — anchor status: not_found]
- [readme] NOREVA time-course assessment function scans and ranks workflows using four independent criteria: [evidence span withheld — anchor status: external_doc]
