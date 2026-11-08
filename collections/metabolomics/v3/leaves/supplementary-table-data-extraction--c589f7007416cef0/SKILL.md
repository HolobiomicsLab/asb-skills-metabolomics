---
name: supplementary-table-data-extraction
description: Use when a validation task requires comparing article-reported intermediate
  precision or agreement metrics (CVs, median Pearson r) against values recomputed
  from supplementary replicate data tables.
when_to_use_negative:
- The task requires raw peak annotation or integration of mzML files — use the PeakMeister
  pipeline itself rather than extracting already-reported metrics
- The article provides no supplementary replicate-level data, so reported values cannot
  be cross-checked against recomputed ones
- Input is already a pre-computed feature/CV table needing statistical analysis rather
  than metric extraction
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3272
tools:
- name: PeakMeister
  role: Source of the validation study whose supplementary QC replicate peak areas
    and reported precision metrics are extracted
  repo: github.com/PBM-Group/PeakMeister
- name: R
  role: Environment used (with pandas in Python) to load the supplementary QC replicate
    table and recompute CVs and correlations
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c03513
    title: A Software Tool for Rapid and Automated Preprocessing of Large-Scale Serum
      Metabolomic Data by Multisegment Injection-Capillary Electrophoresis-Mass Spectrometry
  source_content_hash: sha256:063e2ebb7b23b5a9ab911e18e250947e9b01e5c3b75d9077348b8c1e408495ac
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PBM-Group/PeakMeister
  tools:
  - MSI-CE-MS
  - PeakMeister
  - R
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# supplementary-table-data-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts reported quantitative validation metrics (e.g., median CV = 16.0%, F-Phe control-chart CVs, Pearson r = 0.98) from supplementary tables (e.g., Table S1 QC replicate peak areas) and article text, then tabulates them for reproducibility checks against independently recomputed values.

## When to use

Use when a validation task requires comparing article-reported intermediate precision or agreement metrics (CVs, median Pearson r) against values recomputed from supplementary replicate data tables.

## When NOT to use

- The task requires raw peak annotation or integration of mzML files — use the PeakMeister pipeline itself rather than extracting already-reported metrics
- The article provides no supplementary replicate-level data, so reported values cannot be cross-checked against recomputed ones
- Input is already a pre-computed feature/CV table needing statistical analysis rather than metric extraction

## Inputs

- Supplementary table file containing validation-study QC replicate peak areas (e.g., Table S1, 420 pooled QCs)
- Article-reported precision metrics (median CV = 16.0%; F-Phe CVs 9.2%/8.5% and 7.8%/7.5%; median r = 0.98)

## Outputs

- Summary CSV (e.g., intermediate_precision_summary.csv) tabulating extracted and reproduced CV values and the PeakMeister-vs-manual Pearson correlation

## How to apply

Load the supplementary table of QC replicate peak areas (e.g., Table S1 from the PeakMeister validation, n = 420 pooled QCs from the ENANI-2019 study) using pandas in Python or R. Extract the article-reported precision metrics verbatim: median CV = 16.0% for both PeakMeister and manual preprocessing, F-Phe control-chart CVs of 9.2% vs 8.5% before and 7.8% vs 7.5% after QC-based batch correction. Recompute the control-chart CVs and per-metabolite Pearson correlations across all 47 serum metabolites from the loaded replicate areas, then tabulate reported and reproduced values together (e.g., intermediate_precision_summary.csv). Accept the extraction if the reproduced median r matches the reported r = 0.98 (within rounding) and all extracted CVs match the reported 16.0% median and F-Phe CVs exactly.

## Related tools

- **PeakMeister** (Source of the validation study whose supplementary QC replicate peak areas and reported precision metrics are extracted) — github.com/PBM-Group/PeakMeister
- **R** (Environment used (with pandas in Python) to load the supplementary QC replicate table and recompute CVs and correlations)

## Examples

```
import pandas as pd; qc = pd.read_excel('TableS1_QC_replicates.xlsx', sheet_name='peak_areas'); cv = qc.std(axis=0, ddof=1)/qc.mean(axis=0)*100; print(cv.median())
```

## Evaluation signals

- Reproduced median Pearson r between PeakMeister and manual peak areas across all 47 serum metabolites equals the reported r = 0.98 within rounding
- Extracted median CV for both methods equals the reported 16.0% (n = 420 pooled QCs)
- F-Phe control-chart CVs reproduce as 9.2% vs 8.5% before and 7.8% vs 7.5% after QC batch correction
- Summary output CSV contains one row per metric with matching reported and extracted values and no missing entries

## Limitations

- Late migrating metabolites lacking two flanking or a stable-isotope internal standard were prone to peak misannotation, which can distort replicate-derived CVs
- Low abundance metabolites with signals near the method detection limit were prone to peak misannotation, affecting extracted precision metrics
- Metrics extracted from the abstract may lack explicit DOI/year/journal context and must be verified against the full text or SI

## Evidence
