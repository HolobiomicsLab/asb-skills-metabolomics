---
name: ionization-mode-stratified-sensitivity-specificity-evaluation
description: Use when evaluating identification-criteria performance of a spectral-matching
  score (e.g., I/N Jaccard similarity) per ionization mode, requiring binned ROC analysis
  and per-mode sensitivity/specificity values.
when_to_use_negative:
- Spectra were acquired at a single collision energy — longitudinal fragment profiles
  require multiple distinct CEs, so LFP-number binning is undefined
- The goal is metabolite identification itself rather than evaluating identification
  criteria; use the longitudinal profiling matching workflow (CQMU database construction,
  isomer similarity calculations) instead
- No labeled isomer pairs (identical vs constitutional isomer) are available to define
  ROC-positive and ROC-negative classes
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_3172
tools:
- name: 001-007.ipynb
  role: Notebook series covering spectrum extraction through isomer similarity calculations
    that supply the scores for ROC analysis
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: mzbatch.py
  role: Library of mass spectrometry data-processing methods (file operations, data
    validation, calculation, extraction) used to prepare similarity data
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: 'conda (env: longprof, Python 3.8.13)'
  role: Reproducible environment for running the ROC workflow notebooks
- name: Jupyter Notebook
  role: Execution environment for the 001-007.ipynb pipeline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5c01414
    title: Longitudinal Fragment Profiles Based on Multi-Collision Energy Tandem Mass
      Spectra Improve the Accuracy of Metabolite Identification in Untargeted Metabolomics
  source_content_hash: sha256:85356e93cb1ad056a952e00d5940f187cdf3fca494f7842628e253a5969f6776
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
  tools:
  - 001-007.ipynb
  - Jupyter Notebook
  - Python
  - conda
  - 'conda (env: longprof, Python 3.8.13)'
  - mzbatch.py
  - pip
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.631578947368421
n_claims: 19
---

# ionization-mode-stratified-sensitivity-specificity-evaluation

## Summary

Derives I/N-ratio ROC cutoffs for isomer identification and reports sensitivity/specificity separately for each ionization mode (CP, CN, AP, AN) of CQMD using multi-collision-energy MS/MS spectra (NCE 1–100%), yielding average sensitivity 98.89% and specificity 92.66%.

## When to use

Use when evaluating identification-criteria performance of a spectral-matching score (e.g., I/N Jaccard similarity) per ionization mode, requiring binned ROC analysis and per-mode sensitivity/specificity values.

## When NOT to use

- Spectra were acquired at a single collision energy — longitudinal fragment profiles require multiple distinct CEs, so LFP-number binning is undefined
- The goal is metabolite identification itself rather than evaluating identification criteria; use the longitudinal profiling matching workflow (CQMU database construction, isomer similarity calculations) instead
- No labeled isomer pairs (identical vs constitutional isomer) are available to define ROC-positive and ROC-negative classes

## Inputs

- Isomer pair similarity scores from the in-house CQMD database (identical isomer pairs vs constitutional isomer pairs)
- I/N-ratio Jaccard similarity values for MS/MS spectra acquired at NCE 1–100%
- Longitudinal fragment profile (LFP) counts per isomer pair for binning

## Outputs

- roc_analysis_results.csv with per-bin AUCs and optimal cutoffs
- Per-mode (CP, CN, AP, AN) sensitivity and specificity values comparable to Figures 5e–h, S6, and Excel S9–S13

## How to apply

Load isomer pair similarity data from the in-house CQMD database via the 001-007.ipynb notebooks and mzbatch.py in the Python 3.8.13 conda environment (longprof). Compute the I/N ratio (intensity-based over number-based Jaccard similarity) for identical isomer pairs versus constitutional isomer pairs, then stratify pairs into bins by the number of longitudinal fragment profiles (LFP number). Run ROC analysis within each LFP-number bin to obtain AUC values (expected 0.874–0.991) and the optimal similarity score cutoff per bin. Apply the derived cutoffs separately for the four ionization modes at NCE 1–100% to obtain per-mode sensitivity (100, 98.35, 99.56, 99.69%) and specificity (92.98, 91.67, 94.17, 91.91%), writing results to roc_analysis_results.csv.

## Related tools

- **001-007.ipynb** (Notebook series covering spectrum extraction through isomer similarity calculations that supply the scores for ROC analysis) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **mzbatch.py** (Library of mass spectrometry data-processing methods (file operations, data validation, calculation, extraction) used to prepare similarity data) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **conda (env: longprof, Python 3.8.13)** (Reproducible environment for running the ROC workflow notebooks)
- **Jupyter Notebook** (Execution environment for the 001-007.ipynb pipeline)

## Examples

```
conda create -n longprof python=3.8.13; conda activate longprof; pip install -r requirements.txt; jupyter notebook 001-007.ipynb
```

## Evaluation signals

- Per-bin ROC AUCs fall within the reported range 0.874–0.991
- Per-mode sensitivity values reproduce 100, 98.35, 99.56, 99.69% for the four ionization modes at NCE 1–100%
- Per-mode specificity values reproduce 92.98, 91.67, 94.17, 91.91%
- Averages across the four modes match the reported sensitivity of 98.89% and specificity of 92.66%
- roc_analysis_results.csv is produced and its per-bin AUCs, cutoffs, and per-mode values agree with Figures 5e–h, S6, and Excel S9–S13

## Limitations

- The in-house database CQMD is not publicly available (no accession given), so the exact isomer pair set cannot be re-obtained independently
- No changelog exists for the associated repository, so code-version drift in the 001-007.ipynb/mzbatch.py pipeline cannot be tracked
- Source results (Figures 5e–h, S6, Excel S9–S13) are referenced but their availability/location is not confirmed in the article section
- README states the data and findings are for experimental reference only and should not be used to guide human medicine

## Evidence
