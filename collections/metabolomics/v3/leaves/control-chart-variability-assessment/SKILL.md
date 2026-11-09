---
name: control-chart-variability-assessment
description: Use when validating automated preprocessing software against a manual
  analyst by comparing metabolite peak-area variability in repeat pooled QC samples
  (e.g., 420 QCs in ENANI-2019), before and after QC-based batch correction.
when_to_use_negative:
- Input is already a normalized feature table without QC replicate structure — CVs
  cannot be computed without repeat pooled QCs.
- Data were not acquired in batches with QC-based correction available; pre-/post-correction
  comparison is not meaningful.
- Matrix is not serum/plasma MSI-CE-MS data — PeakMeister was not yet validated for
  urine, saliva, or stool matrices.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: PeakMeister
  role: Automated R-based pre-processing of targeted full-scan MSI-CE-MS mzML files
    producing the peak-area tables whose QC variability is assessed
  repo: github.com/PBM-Group/PeakMeister
- name: R
  role: Statistical environment used to run PeakMeister and compute CVs, control-chart
    statistics, and Pearson correlations in the precision comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c03513
    title: A Software Tool for Rapid and Automated Preprocessing of Large-Scale Serum
      Metabolomic Data by Multisegment Injection-Capillary Electrophoresis-Mass Spectrometry
  source_content_hash: sha256:dfd6dfdc4e99a0abf0316ef3cc9a9dea381afb6b4855b6e0038d9d46afb5ce9b
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
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# control-chart-variability-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assess intermediate precision of automated versus manual MSI-CE-MS data preprocessing by tracking metabolite CVs across repeat pooled QC samples and F-Phe control-chart CVs before and after QC-based batch correction.

## When to use

Use when validating automated preprocessing software against a manual analyst by comparing metabolite peak-area variability in repeat pooled QC samples (e.g., 420 QCs in ENANI-2019), before and after QC-based batch correction.

## When NOT to use

- Input is already a normalized feature table without QC replicate structure — CVs cannot be computed without repeat pooled QCs.
- Data were not acquired in batches with QC-based correction available; pre-/post-correction comparison is not meaningful.
- Matrix is not serum/plasma MSI-CE-MS data — PeakMeister was not yet validated for urine, saliva, or stool matrices.

## Inputs

- Peak-area table for repeat pooled QC samples from PeakMeister-preprocessed mzML files (e.g., Table S1)
- Peak-area table for the same QC samples preprocessed manually by an experienced analyst
- Batch/QC run-order metadata for QC-based batch correction
- List of targeted serum metabolites (47 metabolites in ENANI-2019; F-Phe used for control charts)

## Outputs

- Per-metabolite CV table for each preprocessing method, before and after QC-based batch correction
- Median intermediate-precision CV per method (target ~16.0%)
- Control-chart mean CV for F-Phe per method (manual vs PeakMeister, pre- and post-correction)
- intermediate_precision_summary.csv tabulating extracted CVs and median Pearson r between methods

## How to apply

Load the repeat pooled QC peak areas (e.g., Table S1, n = 420 pooled QCs from ENANI-2019) for both PeakMeister-preprocessed and manually preprocessed data. Compute per-metabolite CVs across QC replicates and summarize with a median CV (article reports median CV = 16.0% for both methods, indicating acceptable intermediate precision). For a representative analyte such as F-Phe, compute control-chart mean CVs for each method (manual 9.2% vs PeakMeister 8.5%), then recompute after QC-based batch correction (7.8% vs 7.5%) to quantify precision gain. Optionally compute per-metabolite Pearson correlations between the two preprocessing routes across all 47 serum metabolites and take the median r (reported 0.98) as a mutual-agreement check. Tabulate all CVs and the correlation in a summary CSV; accept the comparison if reproduced values match reported metrics within rounding.

## Related tools

- **PeakMeister** (Automated R-based pre-processing of targeted full-scan MSI-CE-MS mzML files producing the peak-area tables whose QC variability is assessed) — github.com/PBM-Group/PeakMeister
- **R** (Statistical environment used to run PeakMeister and compute CVs, control-chart statistics, and Pearson correlations in the precision comparison)

## Examples

```
Rscript peakmeister_code.R  # run PeakMeister pre-processing on mzML Files/, then in R: cv <- function(x) sd(x, na.rm=TRUE)/mean(x, na.rm=TRUE)*100; write.csv(summary, 'intermediate_precision_summary.csv')
```

## Evaluation signals

- Median CV across pooled QC samples matches the reported 16.0% for both PeakMeister and manual preprocessing (n = 420 QCs).
- F-Phe control-chart CVs reproduce 9.2% (manual) vs 8.5% (PeakMeister) before correction, attenuating to 7.8% vs 7.5% after QC-based batch correction.
- Median per-metabolite Pearson correlation between PeakMeister and manual peak areas across 47 serum metabolites equals 0.98 within rounding.
- Batch correction reduces (does not increase) control-chart CVs for both methods.
- intermediate_precision_summary.csv exists and contains all extracted CV values plus the correlation, with no missing metabolites.

## Limitations

- Low abundance metabolites with signals near the method detection limit were prone to peak misannotation, which can inflate apparent CVs.
- Certain late migrating serum metabolites lacking two flanking or a stable-isotope internal standard were prone to peak misannotation.
- Real-time preprocessing output for QC review was not yet implemented; results were reviewed retrospectively weeks or months after acquisition.

## Evidence

- [abstract] Intermediate precision CVs from abstract: "acceptable intermediate precision (median CV = 16.0%)"
- [abstract] ENANI-2019 QC sample counts: "5,000 serum and 420 quality control samples from the Brazilian National Survey on Child Nutrition (ENANI-2019)"
- [intro] Manual preprocessing comparison scope from intro: "We performed an extensive validation study to demonstrate acceptable intermediate precision and good mutual agreement when using PeakMeister as compared to manual data preprocessing by an experienced"
- [readme] PeakMeister output includes peak area tables: "5. A table containing the peak areas"
- [methods] SNR threshold defines below-LOD calls: "PeakMeister will compute their S/N to determine if they should be recorded as "<LOD"."
