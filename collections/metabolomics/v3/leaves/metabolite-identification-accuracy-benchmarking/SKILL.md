---
name: metabolite-identification-accuracy-benchmarking
description: Use when comparing two MS/MS spectral matching strategies (e.g., longitudinal
  fragment profiles vs cross-sectional entropy) on annotation accuracy and FDR per
  mode, using ROC-derived thresholds and a labeled cohort like CKD plasma.
when_to_use_negative:
- You have only raw MS/MS spectra and no per-mode annotation results or ground-truth
  labels — preprocess via 001-007.ipynb first (spectrum filtering, invalid fragment
  removal, CQMU database construction) before benchmarking.
- Your comparison is between database search engines (Progenesis QI, Compound Discoverer,
  MS-DIAL) rather than between spectral-matching scoring algorithms (Jaccard longitudinal
  vs optimal-entropy cross-sectional).
- You need to establish the identification thresholds themselves — this skill consumes
  the ROC-derived 0.5/0.75 thresholds, it does not derive them.
tools:
- name: Jaccard similarity algorithm
  role: Optimized similarity metric underlying the longitudinal fragment profiling
    matcher scored at the 0.5 ROC-derived threshold
- name: mzbatch.py
  role: Library of mass spectrometry data-processing helpers (file operations, data
    validation/generation/calculation/extraction) used to assemble the annotation
    tables
  repo: https://github.com/academicCQMD/LongitudinalProf-MSMS-Method
- name: 001-007.ipynb
  role: Notebook series producing the MS/MS extraction, filtering, CQMU database,
    and isomer similarity calculations that feed the benchmark
  repo: https://github.com/academicCQMD/LongitudinalProf-MSMS-Method
- name: Python 3.8.13 / pandas / Jupyter Notebook
  role: Environment in which the Excel S8–S13 tables are loaded and accuracies/FDRs
    are computed and written to CSV
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c01414
    title: Longitudinal Fragment Profiles Based on Multi-Collision Energy Tandem Mass
      Spectra Improve the Accuracy of Metabolite Identification in Untargeted Metabolomics
  source_content_hash: sha256:70576f30dbb3b25116604c30d6d03d16763ef74e474c65e0772b2acae6f886bc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/academicCQMD/LongitudinalProf-MSMS-Method
  tools:
  - 001-007.ipynb
  - Jaccard similarity algorithm
  - Jupyter Notebook
  - Python
  - Python 3.8.13 / pandas / Jupyter Notebook
  - mzbatch.py
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# metabolite-identification-accuracy-benchmarking

## Summary

Benchmarks metabolite annotation accuracy and false discovery rate of spectral matching methods — here longitudinal Jaccard-similarity profiling of multi-CE MS/MS fragments (0.5 threshold) versus cross-sectional optimal-entropy matching (0.75 threshold) — across ionization modes using a real plasma cohort.

## When to use

Use when comparing two MS/MS spectral matching strategies (e.g., longitudinal fragment profiles vs cross-sectional entropy) on annotation accuracy and FDR per mode, using ROC-derived thresholds and a labeled cohort like CKD plasma.

## When NOT to use

- You have only raw MS/MS spectra and no per-mode annotation results or ground-truth labels — preprocess via 001-007.ipynb first (spectrum filtering, invalid fragment removal, CQMU database construction) before benchmarking.
- Your comparison is between database search engines (Progenesis QI, Compound Discoverer, MS-DIAL) rather than between spectral-matching scoring algorithms (Jaccard longitudinal vs optimal-entropy cross-sectional).
- You need to establish the identification thresholds themselves — this skill consumes the ROC-derived 0.5/0.75 thresholds, it does not derive them.

## Inputs

- Supplementary Excel annotation result tables S8–S13 (CKD plasma cohort, CP/CN/AP/AN modes)
- ROC-derived score thresholds (0.5 for longitudinal Jaccard profiling; 0.75 for cross-sectional optimal entropy)
- Ground-truth correct-annotation labels for top-ranked candidates per mode

## Outputs

- Per-mode accuracy for both methods (CP/CN/AP/AN) matching Figure 5c,d values
- Per-mode FDR for both methods with derived reduction percentages
- figure5cd_reproduction.csv containing accuracy, improvement, and FDR values

## How to apply

Load the per-mode annotation result tables (supplementary Excel S8–S13 for CKD plasma CP, CN, AP, AN) from the academicCQMD/LongitudinalProf-MSMS-Method repository into pandas (Python 3.8.13, Jupyter Notebook). For each mode, rank candidate metabolites twice: once with the optimized Jaccard-similarity longitudinal fragment profiling matcher at its ROC-derived 0.5 threshold, and once with the cross-sectional optimal-entropy matcher at its ROC-derived 0.75 threshold. Count correct top-ranked annotations per mode per method and compute accuracy, confirming totals of 518 correct (longitudinal) versus 485 (cross-sectional). Then compute per-mode FDR and derive accuracy improvements (25.9/12.6/8.7/16.9%) and FDR reductions (41.3/39.8/28.6/41.7%) across CP/CN/AP/AN. Write the reproduced values to figure5cd_reproduction.csv and verify they match the reported accuracies (77.4/85.5/83.3/83.2% longitudinal vs 61.5/75.9/76.6/71.2% entropy).

## Related tools

- **Jaccard similarity algorithm** (Optimized similarity metric underlying the longitudinal fragment profiling matcher scored at the 0.5 ROC-derived threshold)
- **mzbatch.py** (Library of mass spectrometry data-processing helpers (file operations, data validation/generation/calculation/extraction) used to assemble the annotation tables) — https://github.com/academicCQMD/LongitudinalProf-MSMS-Method
- **001-007.ipynb** (Notebook series producing the MS/MS extraction, filtering, CQMU database, and isomer similarity calculations that feed the benchmark) — https://github.com/academicCQMD/LongitudinalProf-MSMS-Method
- **Python 3.8.13 / pandas / Jupyter Notebook** (Environment in which the Excel S8–S13 tables are loaded and accuracies/FDRs are computed and written to CSV)

## Examples

```
conda create -n longprof python=3.8.13 && conda activate longprof && pip install -r requirements.txt && python -c "import pandas as pd; df = pd.read_excel('Excel_S8.xlsx')"
```

## Evaluation signals

- Correct top-ranked annotation totals must be 518 for longitudinal profiling and 485 for cross-sectional entropy matching.
- Longitudinal accuracies must reproduce 77.4/85.5/83.3/83.2% and entropy accuracies 61.5/75.9/76.6/71.2% for CP/CN/AP/AN, matching Figure 5c,d.
- Per-mode accuracy improvements must equal 25.9/12.6/8.7/16.9% and FDR reductions 41.3/39.8/28.6/41.7% across CP/CN/AP/AN.
- Thresholds must be the ROC-derived values: 0.5 for the longitudinal Jaccard matcher and 0.75 for the cross-sectional entropy matcher; using either threshold for both methods invalidates the comparison.
- Output CSV (figure5cd_reproduction.csv) must contain one row per mode with accuracy, improvement, and FDR columns for both methods.

## Limitations

- The in-house CQMD database used for ROC threshold derivation has no stated public accession, so exact threshold re-derivation may not be possible outside the original lab.
- The repository's clone URL in the README points to 'yourusername/LongitudinalProf-MSMS-Method'; the canonical repo is academicCQMD/LongitudinalProf-MSMS-Method.
- The benchmark reflects the CKD plasma cohort only; results should not be directly interpreted or used to guide human medicine per the repository license terms.
- No changelog is available for the repository, so code-version drift between reproduction runs cannot be ruled out.

## Evidence

- [methods] Peaks with relative intensity below 1% were removed from the MS/MS spectra.: [evidence span withheld — anchor status: not_found]
