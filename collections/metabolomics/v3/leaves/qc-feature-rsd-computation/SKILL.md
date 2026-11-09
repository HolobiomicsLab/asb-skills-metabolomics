---
name: qc-feature-rsd-computation
description: Use when you have a QC-labeled LC-MS feature table (e.g. statTarget input
  from XCMS/Skyline peak abundances) and need to quantify precision by computing RSD%
  per feature across QC injections and plotting cumulative RSD distributions.
when_to_use_negative:
- Datasets without repeated pooled QC injections interleaved with real samples (e.g.
  QC-free designs); use Combat via shiftCor_dQC instead, since no QC RSDs can be computed
- Peak tables not yet formatted to statTarget input layout (XCMS/MZmine/SIEVE/Skyline
  output must first be converted with the transX function)
- When the goal is drift correction itself rather than precision assessment — RSD
  computation only evaluates correction quality, it does not correct signal
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: statTarget
  role: R package providing the shiftCor pipeline whose RSDresult output produces
    the per-feature QC RSD values and cumulative RSD distribution used by this skill
  repo: github.com/13479776/statTarget
- name: shiftCor
  role: The signal drift correction entry point (MLmethod='QCRFSC' by default, Frule=0.8,
    imputeM='KNN') whose corrected output is compared against raw data via RSD computation
- name: transX
  role: Converts XCMS, MZmine2, SIEVE and SKYLINE outputs into statTarget input files
    so RSD computation can run on standard peak-extraction formats
- name: XCMS
  role: Used upstream to extract peak abundances of metabolites that become the feature
    matrix on which QC RSDs are calculated
- name: Skyline
  role: Used upstream to extract peak abundances (e.g. of tryptic peptides in the
    proteomics dataset) feeding the QC RSD calculation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.aca.2018.08.002
    title: 'statTarget: A streamlined tool for signal drift correction and interpretations
      of quantitative mass spectrometry-based omics data'
  source_content_hash: sha256:c95953e7c340e4ebfc793b3e8b7701326da729f2550aedc113f30314c98b4139
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - QC-RLSC
  - QC-SVR
  - Skyline
  - XCMS
  - shiftCor
  - statTarget
  - transX
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# qc-feature-rsd-computation

## Summary

Computes the relative standard deviation (RSD/CV%) of every feature across pooled QC injections in LC-MS metabolomics peak tables, then bins features into cumulative RSD intervals to quantify data precision before and after signal-drift correction (e.g. QC-RFSC vs QC-RLSC vs QC-SVR).

## When to use

Use when you have a QC-labeled LC-MS feature table (e.g. statTarget input from XCMS/Skyline peak abundances) and need to quantify precision by computing RSD% per feature across QC injections and plotting cumulative RSD distributions.

## When NOT to use

- Datasets without repeated pooled QC injections interleaved with real samples (e.g. QC-free designs); use Combat via shiftCor_dQC instead, since no QC RSDs can be computed
- Peak tables not yet formatted to statTarget input layout (XCMS/MZmine/SIEVE/Skyline output must first be converted with the transX function)
- When the goal is drift correction itself rather than precision assessment — RSD computation only evaluates correction quality, it does not correct signal

## Inputs

- statTarget-format sample description CSV (samPeno) mapping sample name to class (QC vs subject) and batch/injection order
- Feature-by-sample peak abundance matrix CSV (samFile) from XCMS, Skyline, MZmine2 or SIEVE, including QC injection columns
- Optional: corrected output tables from shiftCor runs (QCRFSC, QCRLSC, QCSVr) and the uncorrected raw peak table as baseline

## Outputs

- Per-feature RSD (CV%) values across QC injections
- Cumulative frequency table of RSD bins (e.g. % of features with RSD < 15%, < 30%) per correction condition
- Cumulative RSD distribution plot (analogous to the statTarget 'RSD distribution.pdf' QC plot) suitable for PNG/CSV export

## How to apply

Load the QC sample metadata (samPeno) and the feature-intensity matrix (samFile) whose columns include the pooled QC injections analyzed every five (Dataset 1) or six (Dataset 2) real samples. For each feature, compute the RSD (standard deviation / mean × 100) across the QC injections only, excluding subject samples. Bin features into cumulative frequency classes (e.g. RSD < 5%, < 10%, < 15%, < 30%, < 100%) as the statTarget RSDresult output does; report the proportion of peaks within a given RSD cutoff such as 15% or 30%. Apply the Frule = 0.8 quality filter (keep variables with non-zero values in at least 80% of samples in any group) before computing RSDs so that low-quality peaks do not inflate the distribution. Compare the resulting cumulative RSD distributions between raw data and corrected data (QC-RFSC, QC-RLSC, QC-SVR) to judge which correction improved precision; in dataset 1 raw data only 1.97% of peaks fell within 15% RSD, rising to 25.1% (12.7-fold) after QC-RFSC correction.

## Related tools

- **statTarget** (R package providing the shiftCor pipeline whose RSDresult output produces the per-feature QC RSD values and cumulative RSD distribution used by this skill) — github.com/13479776/statTarget
- **shiftCor** (The signal drift correction entry point (MLmethod='QCRFSC' by default, Frule=0.8, imputeM='KNN') whose corrected output is compared against raw data via RSD computation)
- **transX** (Converts XCMS, MZmine2, SIEVE and SKYLINE outputs into statTarget input files so RSD computation can run on standard peak-extraction formats)
- **XCMS** (Used upstream to extract peak abundances of metabolites that become the feature matrix on which QC RSDs are calculated)
- **Skyline** (Used upstream to extract peak abundances (e.g. of tryptic peptides in the proteomics dataset) feeding the QC RSD calculation)

## Examples

```
library(statTarget); shiftCor(samPeno='samPeno.csv', samFile='samFile.csv', Frule=0.8, MLmethod='QCRFSC', QCspan=0, imputeM='KNN')
```

## Evaluation signals

- RSD values are computed only on QC injection columns, verified by cross-checking against the samPeno class labels
- Features failing Frule = 0.8 (non-zero in <80% of samples in any group) are excluded before RSD aggregation
- The proportion of features with RSD < 30% should fall in a plausible range; e.g. raw dataset 1 ~43.9%, QC-RFSC ~86.2%, QC-RLSC ~65.5%, QC-SVR ~61.9%
- Cumulative RSD distribution is monotonically non-decreasing across bin thresholds (5%, 10%, 15%, ..., 100%)
- Corrected data should show a higher fraction of low-RSD features than the raw baseline (e.g. 12.7-fold increase at the 15% RSD cutoff for QC-RFSC in dataset 1)

## Limitations

- RSD computation requires interleaved QC injections; the article reports QC samples analyzed every five (Dataset 1), six (Dataset 2) or eight (Dataset 3) subject samples, so datasets without such periodic QC runs cannot support this metric
- The metric reflects precision of pooled QC material only and may not capture batch effects specific to biological groups; the article also notes Combat is provided for QC-free datasets, implying RSD-based assessment is inapplicable there
- Low-quality peaks can distort the RSD distribution if the Frule (default 0.8) filter is bypassed

## Evidence

- [results] RSD threshold comparison for corrected vs raw data: [evidence span withheld — anchor status: not_found]
