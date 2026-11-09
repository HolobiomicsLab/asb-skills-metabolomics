---
name: batch-integration-of-ms-omics-data
description: Use when raw MS-based metabolomics or proteomics feature intensities
  from multiple injection batches must be corrected for intra- and inter-batch drift
  and merged into one table for statistical analysis.
when_to_use_negative:
- When data lack an interleaved QC injection series (no QC samples after every N real
  samples) — use Combat via shiftCor_dQC for QC-free datasets instead
- When input is already a fully merged, drift-corrected feature table with no batch
  structure to remove
- When peaks fail the 80 percent non-zero threshold or carry more than 20 percent
  missing values — these are filtered rather than integrated
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3360
tools:
- name: statTarget
  role: Main R package implementing the QC-RFSC signal drift correction and batch
    integration workflow; provides shiftCor, shiftCor_dQC, statAnalysis, and transX
    functions
  repo: github.com/13479776/statTarget
- name: random forest
  role: Ensemble-learning engine used inside QC-RFSC to fit QC-sample intensities
    vs. injection order and predict correction factors per feature (500 trees, ntree
    = 500)
- name: ProteoWizard
  role: Converts vendor MS raw data to mzXML format before peak extraction
- name: XCMS
  role: Extracts peak abundances from metabolomics mzXML files to produce the input
    feature table
- name: Skyline
  role: Extracts peak abundances of tryptic peptides for proteomics feature tables
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.aca.2018.08.002
    title: 'statTarget: A streamlined tool for signal drift correction and interpretations
      of quantitative mass spectrometry-based omics data'
  source_content_hash: sha256:b262d27510732c72adafe3fb2378008e4df70f054e55ff0a5aa7d1b96e595139
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ProteoWizard
  - Skyline
  - XCMS
  - random forest
  - statTarget
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# batch-integration-of-ms-omics-data

## Summary

Integrates signal-drift-corrected QC and biological sample intensities from multiple injection batches into a single normalized feature table using the statTarget QC-RFSC random forest correction workflow for LC/GC-MS metabolomics and SRM proteomics data.

## When to use

Use when raw MS-based metabolomics or proteomics feature intensities from multiple injection batches must be corrected for intra- and inter-batch drift and merged into one table for statistical analysis.

## When NOT to use

- When data lack an interleaved QC injection series (no QC samples after every N real samples) — use Combat via shiftCor_dQC for QC-free datasets instead
- When input is already a fully merged, drift-corrected feature table with no batch structure to remove
- When peaks fail the 80 percent non-zero threshold or carry more than 20 percent missing values — these are filtered rather than integrated

## Inputs

- Feature intensity table from XCMS, MZmine2, SIEVE, or Skyline (converted to statTarget format via the transX function)
- Sample annotation file mapping samples to batches and injection order (samPeno)
- Raw mzXML files converted with ProteoWizard
- Dataset 1 urinary metabolic profiles, Dataset 2 (Giga Science ID: 100108) plasma metabolomics, Dataset 3 SRM-based proteomics (174 transitions, 69 proteins)

## Outputs

- shift_all_cor.csv — integrated corrected table for QC and biological samples across batches
- shift_QC_cor.csv — corrected QC-only feature table
- shift_sample_cor.csv — corrected sample-only feature table
- RSD distribution metrics before and after correction for quality assessment

## How to apply

Order feature intensities by injection sequence, fitting a 500-tree random forest regression on QC-sample intensities versus injection order per feature with the randomForest package to learn unwanted variation. Predict the correction factor Fij at each sample's injection-order position (eq 2) and divide each feature intensity xij by Fij (eq 1) to obtain corrected values for both QC and biological samples. Apply the Frule = 0.8 filter keeping variables with a non-zero value in at least 80 percent of samples in any one group, remove features with more than 20 percent missing values, and impute remaining NAs with KNN. Invoke statTarget's shiftCor function with samPeno and samFile inputs, MLmethod = "QCRFSC", QCspan = 0, and imputeM = "KNN" to integrate batches into a single output; for QC-free datasets use shiftCor_dQC with MLmethod = "Combat" instead. Rationale: random forest learns the drift surface from the injected QC series (every five, six, or eight real samples depending on the dataset) and predicts correction factors in neighboring real samples, which proved superior to QC-RLSC and QC-SVR for increasing the proportion of peaks below 30 percent RSD.

## Related tools

- **statTarget** (Main R package implementing the QC-RFSC signal drift correction and batch integration workflow; provides shiftCor, shiftCor_dQC, statAnalysis, and transX functions) — github.com/13479776/statTarget
- **random forest** (Ensemble-learning engine used inside QC-RFSC to fit QC-sample intensities vs. injection order and predict correction factors per feature (500 trees, ntree = 500))
- **ProteoWizard** (Converts vendor MS raw data to mzXML format before peak extraction)
- **XCMS** (Extracts peak abundances from metabolomics mzXML files to produce the input feature table)
- **Skyline** (Extracts peak abundances of tryptic peptides for proteomics feature tables)

## Examples

```
shiftCor(samPeno,samFile, Frule = 0.8, MLmethod = "QCRFSC", QCspan = 0,imputeM = "KNN")
```

## Evaluation signals

- Relative standard deviation (RSD) of corrected QC peaks should shift significantly upward in the low-RSD tail: dataset 1 raw 43.9% → 86.2% below 30% RSD; dataset 2 raw 52.1% → 90.9%
- Proportion of peaks within 15% RSD in dataset 1 should rise ~12.7-fold (1.97% → 25.1%) after QC-RFSC
- RF prediction accuracy (R-squared at median and first quartile) should outperform QC-RLSC and QC-SVR baselines on the same data
- Output CSV row/column count must match the input sample and feature counts after the 80% Frule filter and NA removal
- No remaining batch offsets when comparing corrected QC peaks across batches in shift_QC_cor.csv

## Limitations

- Requires a QC injection series interleaved with real samples (every 5, 6, or 8 samples depending on the study design); datasets without QC samples must use Combat instead of QC-RFSC
- Performance is dataset-dependent — QC-RLSC and QC-SVR still recovered 65.5%/61.9% (dataset 1) and 80.1%/72.3% (dataset 2) of peaks below 30% RSD, so RF is not universally dominant in every feature
- Requires R >= 3.3.0 and, on macOS, XQuartz instead of X11; RGtk2 installation can be problematic on some platforms
- statTarget does not document explicit data/code availability for the QC-RFSC algorithm itself beyond the linked GitHub page

## Evidence

- [methods] QC based random forest signal correction (QC-RFSC): "A variable will be kept if it has a non-zero value for at least n precent of samples in any one group. (Default: 0.8)"
- [results] RF prediction accuracy: [evidence span withheld — anchor status: not_found]
- [methods] Combat fallback for QC-free data: "shiftCor_dQC(samPeno2,samFile, Frule = 0.8, MLmethod = "Combat")"
- [methods] QC injection cadence: "The aliquoted QC samples were analyzed after every five aliquoted real samples in the entire batch"
- [readme] Platform requirements: [evidence span withheld — anchor status: external_doc]
