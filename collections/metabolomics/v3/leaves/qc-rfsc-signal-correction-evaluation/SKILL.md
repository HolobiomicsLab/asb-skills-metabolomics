---
name: qc-rfsc-signal-correction-evaluation
description: Use when assessing whether batch correction (QC-RFSC, QC-RLSC, QC-SVR,
  Combat) improved data precision on LC-MS metabolomics or proteomics feature tables,
  comparing raw vs corrected RSD distributions and differential feature counts.
when_to_use_negative:
- Input is a single-sample feature table with no pooled QC injections to estimate
  batch drift (QC-RFSC requires QC reference points)
- Goal is removing known biological covariates rather than signal-drift/batch variation;
  Combat (shiftCor_dQC) is better suited for QC-free datasets
- Data has already been batch-corrected and you only need downstream statistics (statAnalysis)
  rather than a re-evaluation of correction performance
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: statTarget
  role: Implements shiftCor() for QC-RFSC batch correction and statAnalysis() for
    differential feature testing used in the evaluation
  repo: github.com/13479776/statTarget
- name: QC-RFSC
  role: The correction algorithm being evaluated (QC-based random forest signal correction)
    compared against QC-RLSC and QC-SVR baselines
  repo: github.com/13479776/statTarget
- name: XCMS
  role: Peak extraction for metabolomics feature intensities feeding the raw matrix
    being evaluated
- name: Skyline
  role: Peak extraction of tryptic peptide transitions for SRM proteomics matrix (Dataset
    3)
- name: ProteoWizard
  role: Converts LC-MS raw files to mzXML before feature extraction
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.aca.2018.08.002
    title: 'statTarget: A streamlined tool for signal drift correction and interpretations
      of quantitative mass spectrometry-based omics data'
  source_content_hash: sha256:61e6713c87c3e9df5c912aedca8ebf26cdcc9d99072da4d00294f8486b34b588
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ProteoWizard
  - QC-RFSC
  - Skyline
  - XCMS
  - statTarget
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# qc-rfsc-signal-correction-evaluation

## Summary

Evaluate and compare QC-based random forest signal correction (QC-RFSC) against other correction methods by measuring relative standard deviation (RSD) distributions and differential feature counts before and after correction in metabolomics and proteomics datasets.

## When to use

Use when assessing whether batch correction (QC-RFSC, QC-RLSC, QC-SVR, Combat) improved data precision on LC-MS metabolomics or proteomics feature tables, comparing raw vs corrected RSD distributions and differential feature counts.

## When NOT to use

- Input is a single-sample feature table with no pooled QC injections to estimate batch drift (QC-RFSC requires QC reference points)
- Goal is removing known biological covariates rather than signal-drift/batch variation; Combat (shiftCor_dQC) is better suited for QC-free datasets
- Data has already been batch-corrected and you only need downstream statistics (statAnalysis) rather than a re-evaluation of correction performance

## Inputs

- Sample phenotype table (samPeno) mapping injection order to sample/QC group
- Feature-by-sample intensity matrix (samFile) from XCMS, Skyline, MZmine2, SIEVE or transX-converted format
- Raw (Before_shiftCor) and QC-RFSC corrected (After_shiftCor) feature matrices
- QC injection metadata (QC samples analyzed every 5, 6 or 8 real samples depending on dataset)

## Outputs

- Cumulative RSD distribution (proportion of peaks with RSD < 15% and < 30%) for raw and corrected data
- Count of significant/differential features per dataset before and after correction, with fold-enlargement factor
- CSV table (e.g. shiftCor_vs_raw_differential_counts.csv) comparing raw vs corrected feature counts per dataset
- Quality assessment plots comparing QC-RFSC to QC-RLSC, QC-SVR and Combat

## How to apply

Run statTarget's shiftCor() on the sample phenotype and feature intensity matrices with Frule = 0.8 (80-percent rule), MLmethod = "QCRFSC", QCspan = 0, and imputeM = "KNN" to obtain corrected (After_shiftCor) alongside raw (Before_shiftCor) matrices. For each dataset compute the cumulative RSD distribution of all features (e.g. proportion of peaks within 15% or 30% RSDs in QC samples) in raw vs corrected data. Then run statAnalysis() (Welch's T-test with Benjamini-Hochberg FDR adjustment, normM = "NONE", imputeM = "KNN", glog = TRUE, scaling = "Pareto") on each matrix and count significant/differential features per condition. Tabulate raw vs corrected RSD percentages and differential feature counts (with fold enlargement) per dataset to judge whether correction improved precision and statistical accuracy.

## Related tools

- **statTarget** (Implements shiftCor() for QC-RFSC batch correction and statAnalysis() for differential feature testing used in the evaluation) — github.com/13479776/statTarget
- **QC-RFSC** (The correction algorithm being evaluated (QC-based random forest signal correction) compared against QC-RLSC and QC-SVR baselines) — github.com/13479776/statTarget
- **XCMS** (Peak extraction for metabolomics feature intensities feeding the raw matrix being evaluated)
- **Skyline** (Peak extraction of tryptic peptide transitions for SRM proteomics matrix (Dataset 3))
- **ProteoWizard** (Converts LC-MS raw files to mzXML before feature extraction)

## Examples

```
shiftCor(samPeno,samFile, Frule = 0.8, MLmethod = "QCRFSC", QCspan = 0, imputeM = "KNN")
```

## Evaluation signals

- Proportion of peaks with RSD < 30% should increase substantially after QC-RFSC (e.g. 43.9% → 86.2% in dataset 1; 52.1% → 90.9% in dataset 2)
- Proportion of peaks within 15% RSD in dataset 1 should show ~12.7-fold enlargement (1.97% → 25.1%)
- Corrected data should yield a larger count of significant features (Benjamini-Hochberg FDR via Welch's T-test) than raw data across datasets 1–3
- QC-RFSC should outperform QC-RLSC and QC-SVR at the same RSD cutoffs (e.g. 30% RSD proportions of 65.5% and 61.9% respectively)
- RSD distribution must be computed on QC sample intensities, not the entire sample set, to measure precision rather than biological variation

## Limitations

- QC-RFSC requires regularly interleaved pooled QC injections (e.g. every 5, 6 or 8 real samples); datasets lacking this design cannot be corrected this way
- Comparison is limited to the algorithms benchmarked in the paper (QC-RLSC, QC-SVR, Combat); other correction methods are not covered
- RSD thresholds (15%, 30%) are dataset-dependent; results in dataset 1 vs 2 differ even for the same algorithm

## Evidence

- [results] RSD improvement from 43.9% to 86.2% after QC-RFSC: [evidence span withheld — anchor status: not_found]
