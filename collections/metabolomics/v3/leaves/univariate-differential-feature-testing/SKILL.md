---
name: univariate-differential-feature-testing
description: Use when comparing raw vs. signal-drift-corrected feature matrices from
  LC-MS metabolomics or proteomics (e.g., XCMS or Skyline outputs) to count significant
  features via Welch's T-test with BH FDR and assess how correction improves statistical
  accuracy.
when_to_use_negative:
- Input is a single already-corrected feature matrix and the goal is only descriptive
  QC (RSD distributions) rather than raw-vs-corrected differential-feature comparison
  — QC-RFSC correction and statAnalysis on one matrix alone does not yield the enlargement
  metric.
- Data have not been peak-extracted into a statTarget-compatible feature table (e.g.,
  still vendor MS raw files); use the transX function to convert XCMS/MZmine2/SIEVE/SKYLINE
  outputs to statTarget input format first.
- The comparison targets batch-integration quality (e.g., Combat-based QC-free correction
  via shiftCor_dQC) rather than QC-injection-anchored signal drift correction; mixing
  correction modes undermines the raw-vs-QC-RFSC enlargement interpretation.
edam_operation: http://edamontology.org/operation_3565
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: statTarget
  role: R package providing shiftCor (QC-RFSC correction of raw matrices) and statAnalysis
    (Welch's T-test with BH FDR, glog transform, Pareto scaling) used to generate
    raw and corrected inputs and call differential features
  repo: github.com/13479776/statTarget
- name: XCMS
  role: Peak abundance extraction tool for metabolites; its output feature tables
    feed the raw and corrected matrices prior to differential-feature testing
- name: Skyline
  role: Peak abundance extraction tool for tryptic peptides in SRM-based proteomics;
    its output feeds the proteomics feature matrices
- name: ProteoWizard
  role: Converts MS raw files to mzXML format before peak extraction with XCMS/Skyline
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.aca.2018.08.002
    title: 'statTarget: A streamlined tool for signal drift correction and interpretations
      of quantitative mass spectrometry-based omics data'
  source_content_hash: sha256:46a2b139c27e2803628bf36ce00bfde1f1b58b4beae9284538be10628788f0f5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ProteoWizard
  - Skyline
  - XCMS
  - statTarget
  edam_operation: http://edamontology.org/operation_3565
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# univariate-differential-feature-testing

## Summary

Runs statTarget's statAnalysis function (Welch's T-test with Benjamini-Hochberg FDR adjustment, 80% rule filtering, KNN imputation, glog transform, Pareto scaling) on raw and QC-RFSC-corrected metabolomics/proteomics feature matrices to count differential features and quantify how signal drift correction enlarges the number of statistically significant features.

## When to use

Use when comparing raw vs. signal-drift-corrected feature matrices from LC-MS metabolomics or proteomics (e.g., XCMS or Skyline outputs) to count significant features via Welch's T-test with BH FDR and assess how correction improves statistical accuracy.

## When NOT to use

- Input is a single already-corrected feature matrix and the goal is only descriptive QC (RSD distributions) rather than raw-vs-corrected differential-feature comparison — QC-RFSC correction and statAnalysis on one matrix alone does not yield the enlargement metric.
- Data have not been peak-extracted into a statTarget-compatible feature table (e.g., still vendor MS raw files); use the transX function to convert XCMS/MZmine2/SIEVE/SKYLINE outputs to statTarget input format first.
- The comparison targets batch-integration quality (e.g., Combat-based QC-free correction via shiftCor_dQC) rather than QC-injection-anchored signal drift correction; mixing correction modes undermines the raw-vs-QC-RFSC enlargement interpretation.

## Inputs

- Raw feature abundance matrix (Before_shiftCor) from XCMS or Skyline peak extraction, in statTarget input format
- QC-RFSC-corrected feature matrix (After_shiftCor) from statTarget shiftCor (MLmethod = "QCRFSC", Frule = 0.8, imputeM = "KNN")
- Sample-to-class metadata table (samPeno) linking sample names to biological groups
- Dataset-specific matrices: metabolomics Dataset 1 (urine), metabolomics Dataset 2 (Giga Science ID 100108, plasma), proteomics Dataset 3 (serum, 174 transitions / 69 proteins)

## Outputs

- Per-dataset counts of differential (significant) features in raw vs. QC-RFSC-corrected data
- Fold enlargement of the differential feature count after QC-RFSC correction
- Summary table, e.g., shiftCor_vs_raw_differential_counts.csv, with raw count, corrected count, and enlargement factor per dataset

## How to apply

Load both the raw (Before_shiftCor) and QC-RFSC-corrected (After_shiftCor) feature matrices produced by statTarget's shiftCor function, run with Frule = 0.8, MLmethod = "QCRFSC", QCspan = 0, and imputeM = "KNN". Apply the 80-percent rule (a variable is kept if it has a non-zero value for at least 80% of samples in any one group) and KNN imputation identically to both matrices. Invoke statAnalysis(file, Frule = 0.8, normM = "NONE", imputeM = "KNN", glog = TRUE, scaling = "Pareto") on each matrix to perform Welch's T-test with Benjamini-Hochberg FDR adjustment and call differential features per dataset (Dataset 1 and 2 metabolomics, Dataset 3 proteomics). Count significant features in raw versus corrected data for each dataset, then compute the fold enlargement of the differential feature count after QC-RFSC. Tabulate raw count, corrected count, and enlargement factor per dataset in a CSV (e.g., shiftCor_vs_raw_differential_counts.csv). The rationale is that QC-RFSC improves precision (e.g., metabolomics dataset 2 RSD < 30% peaks rise from 52.1% to 90.9%), which propagates to a substantially larger set of statistically significant features.

## Related tools

- **statTarget** (R package providing shiftCor (QC-RFSC correction of raw matrices) and statAnalysis (Welch's T-test with BH FDR, glog transform, Pareto scaling) used to generate raw and corrected inputs and call differential features) — github.com/13479776/statTarget
- **XCMS** (Peak abundance extraction tool for metabolites; its output feature tables feed the raw and corrected matrices prior to differential-feature testing)
- **Skyline** (Peak abundance extraction tool for tryptic peptides in SRM-based proteomics; its output feeds the proteomics feature matrices)
- **ProteoWizard** (Converts MS raw files to mzXML format before peak extraction with XCMS/Skyline)

## Examples

```
Rscript -e 'library(statTarget); statAnalysis(file = "After_shiftCor/Feature_table.csv", Frule = 0.8, normM = "NONE", imputeM = "KNN", glog = TRUE, scaling = "Pareto")'
```

## Evaluation signals

- The corrected matrix must show a substantially larger number of significant features than the raw matrix for each dataset, consistent with the paper's precision gains (e.g., dataset 2 peaks with RSD < 30% rose from 52.1% raw to 90.9% after QC-RFSC).
- The 80-percent rule (Frule = 0.8) must be applied identically to both raw and corrected matrices; a feature kept in one and not the other invalidates the comparison.
- statAnalysis parameters must be held constant across both matrices (normM = "NONE", imputeM = "KNN", glog = TRUE, scaling = "Pareto") so the only variable is correction status.
- The enlargement factor computed from raw-vs-corrected differential counts should reproduce the paper's reported fold increase in significant features (e.g., a 12.7-fold increase in dataset 1 peaks within 15% RSDs).
- Output CSV must contain one row per dataset with three columns: raw count, corrected count, and enlargement factor, matching the paper's tabulated enlargement.

## Limitations

- statTarget depends on R (>= 3.3.0); on macOS, XQuartz must be installed instead of X11 for the GUI, and RGtk2 (recommended 2.20.31) is a binding for the GTK2 library that can complicate installation on Windows/Mac.
- The comparison assumes QC injections exist at regular intervals (every 5, 6, or 8 samples depending on dataset) to anchor QC-RFSC; datasets without pooled QC samples cannot be corrected this way (Combat/shiftCor_dQC is offered for QC-free datasets instead).
- The enlargement metric is specific to the Welch's T-test with BH FDR and the 80% rule used in this paper; different statistical models or feature filters will yield different counts.
- Peaks failing the Frule = 0.8 (80-percent rule) are removed before testing, so differential-feature counts are conditioned on this quality filter and may exclude low-prevalence features.

## Evidence

- [methods] statTarget deployment for signal drift correction and statistical analysis: [evidence span withheld — anchor status: not_found]
