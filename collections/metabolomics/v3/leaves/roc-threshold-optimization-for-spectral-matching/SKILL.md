---
name: roc-threshold-optimization-for-spectral-matching
description: Use when establishing identification criteria (similarity-score thresholds)
  for MS/MS spectral matching of isomer pairs, e.g., computing I/N-ratio ROC cutoffs
  and sensitivity/specificity across CP, CN, AP, AN ionization modes in a spectral
  database like CQMD.
when_to_use_negative:
- The goal is cross-sectional spectral matching with a single collision-energy spectrum
  — the cutoffs here are derived from multi-CE longitudinal fragment profiles (I/N
  ratio), not single-spectrum similarity scores
- Comparing metabolites against public libraries (MoNA, GNPS, METLIN) without isomer-pair
  labels — ROC analysis requires known identical vs constitutional isomer pairs to
  define positives and negatives
- Quantifying metabolite abundance rather than establishing identification criteria
  — this skill only optimizes similarity-score thresholds, it does not perform identification
  itself
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Jaccard similarity algorithm
  role: Optimized similarity metric whose scores are the input to ROC cutoff optimization;
    I/N ratio = intensity-based over number-based Jaccard
- name: 001-007.ipynb
  role: Notebooks covering isomer similarity calculations and data processing steps
    feeding the ROC analysis
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: mzbatch.py
  role: Python library of MS data processing methods (file operations, data validation,
    calculation, extraction) used to load and process CQMD isomer similarity data
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: conda
  role: 'Environment management: conda create -n longprof python=3.8.13'
- name: Jupyter Notebook
  role: Execution environment for the 001-007.ipynb pipeline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5c01414
    title: Longitudinal Fragment Profiles Based on Multi-Collision Energy Tandem Mass
      Spectra Improve the Accuracy of Metabolite Identification in Untargeted Metabolomics
  source_content_hash: sha256:f8943e804049a55fabcbbe6732f811e4ba3987856fda48fb884a24bb6a8f327e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
  tools:
  - 001-007.ipynb
  - Jaccard similarity algorithm
  - Jupyter Notebook
  - Python
  - conda
  - mzbatch.py
  - pip
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# roc-threshold-optimization-for-spectral-matching

## Summary

Derives isomer-identification similarity-score cutoffs for the longitudinal fragment profiling (LFP) method by running ROC analysis on I/N (intensity-based over number-based Jaccard) similarity ratios within LFP-number bins, then reports per-bin AUCs and per-ionization-mode sensitivity/specificity.

## When to use

Use when establishing identification criteria (similarity-score thresholds) for MS/MS spectral matching of isomer pairs, e.g., computing I/N-ratio ROC cutoffs and sensitivity/specificity across CP, CN, AP, AN ionization modes in a spectral database like CQMD.

## When NOT to use

- The goal is cross-sectional spectral matching with a single collision-energy spectrum — the cutoffs here are derived from multi-CE longitudinal fragment profiles (I/N ratio), not single-spectrum similarity scores
- Comparing metabolites against public libraries (MoNA, GNPS, METLIN) without isomer-pair labels — ROC analysis requires known identical vs constitutional isomer pairs to define positives and negatives
- Quantifying metabolite abundance rather than establishing identification criteria — this skill only optimizes similarity-score thresholds, it does not perform identification itself

## Inputs

- Isomer pair similarity data (identical vs constitutional isomer pairs) from the in-house CQMD database of ~180,000 MS/MS spectra acquired at 13 distinct CEs and stepped NCE 20/40/60%
- Jaccard similarity algorithm scores for isomer pairs
- LFP-number (longitudinal fragment profile count) per isomer pair
- MS/MS spectra acquired at NCE 1–100% across four ionization modes (CP, CN, AP, AN)

## Outputs

- Per-LFP-number-bin ROC AUC values (expected range 0.874–0.991) and optimal similarity-score cutoffs (roc_analysis_results.csv)
- Per-ionization-mode sensitivity (100, 98.35, 99.56, 99.69%) and specificity (92.98, 91.67, 94.17, 91.91%) at NCE 1–100%
- Values verifiable against Figures 5e–h, S6, and Excel S9–S13

## How to apply

Load isomer pair similarity data from the in-house CQMD database using the 001-007.ipynb notebooks and mzbatch.py in a Python 3.8.13 conda environment (longprof). Compute the I/N ratio — intensity-based over number-based Jaccard similarity — for identical isomer pairs versus constitutional isomer pairs, then stratify pairs into bins by the number of longitudinal fragment profiles (LFP number). Run ROC analysis within each LFP-number bin on the Jaccard similarity algorithm scores to obtain AUC values (expected range 0.874–0.991) and the optimal per-bin similarity-score cutoff. Evaluate sensitivity and specificity for the four ionization modes at NCE 1–100% using the derived cutoffs, writing per-bin AUCs, cutoffs, and per-mode values to roc_analysis_results.csv. The rationale is that longitudinal profiles treat predominant and minute fragments independently, so I/N-ratio cutoffs yield better sensitivity/specificity than cross-sectional matching methods.

## Related tools

- **Jaccard similarity algorithm** (Optimized similarity metric whose scores are the input to ROC cutoff optimization; I/N ratio = intensity-based over number-based Jaccard)
- **001-007.ipynb** (Notebooks covering isomer similarity calculations and data processing steps feeding the ROC analysis) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **mzbatch.py** (Python library of MS data processing methods (file operations, data validation, calculation, extraction) used to load and process CQMD isomer similarity data) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **conda** (Environment management: conda create -n longprof python=3.8.13)
- **Jupyter Notebook** (Execution environment for the 001-007.ipynb pipeline)

## Examples

```
conda create -n longprof python=3.8.13 && conda activate longprof && pip install -r requirements.txt; jupyter nbconvert --to notebook --execute 001-007.ipynb
```

## Evaluation signals

- roc_analysis_results.csv per-bin AUC values fall in the reported range 0.874–0.991
- Per-mode sensitivity values match 100, 98.35, 99.56, 99.69% and specificity values match 92.98, 91.67, 94.17, 91.91% for CP, CN, AP, AN at NCE 1–100%
- Overall average sensitivity ≈ 98.89% and average specificity ≈ 92.66% across the four ionization modes
- Per-bin AUCs, cutoffs, and per-mode sensitivity/specificity agree with Figures 5e–h, S6, and Excel S9–S13
- ROC analysis is stratified by LFP number — verify each bin has its own optimal cutoff rather than a single global threshold

## Limitations

- Cutoffs and AUCs are derived from the in-house CQMD database, which is not publicly accessible (no accession or public access given); results may not transfer to other spectral libraries
- Results referenced in supplementary artifacts (Figures 5e–h, S6, Excel S9–S13) whose availability/location is not confirmed
- The dataset and findings are intended for experimental reference only and should not be directly interpreted or used to guide human medicine (per repository license)
- Performance is characterized for MS/MS spectra acquired at NCE 1–100%; sensitivity/specificity may differ at other acquisition settings

## Evidence

- [results] Sensitivity/specificity performance: [evidence span withheld — anchor status: not_found]
