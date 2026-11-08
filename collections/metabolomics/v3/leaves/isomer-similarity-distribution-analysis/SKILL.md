---
name: isomer-similarity-distribution-analysis
description: Use when MS/MS spectra acquired at multiple collision energies (e.g.,
  NCE 1–100%) must be converted into quantitative identification thresholds separating
  identical isomer pairs from constitutional isomer pairs.
when_to_use_negative:
- Only single-collision-energy MS/MS spectra are available, since longitudinal fragment
  profiles require relative ion intensities of a fragment across multiple collision
  energies.
- The goal is general compound annotation against public libraries (MoNA, GNPS, METLIN)
  rather than establishing quantitative identification criteria for isomer discrimination
  in the CQMD database.
- Candidate ranking only is needed — this skill derives acceptance cutoffs via ROC
  analysis, not candidate lists.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: Jupyter Notebook
  role: Runs the 001-007.ipynb notebooks that perform isomer similarity calculations
    and the ROC-based cutoff derivation
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: mzbatch.py
  role: Library of mass spectrometry data processing methods (file operations, data
    validation, calculation, extraction) used within the notebooks
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: 001-007.ipynb
  role: Notebook series covering spectrum extraction through CQMU database construction,
    isomer similarity calculations, and metabolite identification
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: conda
  role: Creates the python=3.8.13 'longprof' environment required to reproduce the
    analysis
- name: Jaccard similarity algorithm
  role: Optimized similarity metric whose scores (intensity-based over number-based,
    I/N) feed the per-bin ROC analysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5c01414
    title: Longitudinal Fragment Profiles Based on Multi-Collision Energy Tandem Mass
      Spectra Improve the Accuracy of Metabolite Identification in Untargeted Metabolomics
  source_content_hash: sha256:cb64129c3af1549a1577725f1c487ef52bfc43706cee60d5c20dc2dd52d6e02e
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
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# isomer-similarity-distribution-analysis

## Summary

Establishes identification criteria for isomer-level metabolite identification by computing intensity-based over number-based (I/N) Jaccard similarity ratios for identical versus constitutional isomer pairs, then running per-bin ROC analysis of longitudinal fragment profile (LFP) data to derive optimal similarity cutoffs with sensitivity/specificity.

## When to use

Use when MS/MS spectra acquired at multiple collision energies (e.g., NCE 1–100%) must be converted into quantitative identification thresholds separating identical isomer pairs from constitutional isomer pairs.

## When NOT to use

- Only single-collision-energy MS/MS spectra are available, since longitudinal fragment profiles require relative ion intensities of a fragment across multiple collision energies.
- The goal is general compound annotation against public libraries (MoNA, GNPS, METLIN) rather than establishing quantitative identification criteria for isomer discrimination in the CQMD database.
- Candidate ranking only is needed — this skill derives acceptance cutoffs via ROC analysis, not candidate lists.

## Inputs

- Isomer pair similarity data from the in-house CQMD spectral database (~1,80,000 MS/MS spectra, >4500 metabolites)
- Jaccard similarity algorithm scores for identical vs. constitutional isomer pairs
- MS/MS spectra acquired at NCE 1–100% (13 distinct CEs and stepped NCE 20/40/60%)
- Longitudinal fragment profile (LFP) counts per isomer pair for binning

## Outputs

- roc_analysis_results.csv containing per-bin AUCs, optimal similarity cutoffs, and per-mode sensitivity/specificity
- I/N-ratio ROC curves per LFP-number bin (AUC 0.874–0.991) corresponding to Figures 5e–h and S6, Excel S9–S13
- Per-mode sensitivity (100, 98.35, 99.56, 99.69%) and specificity (92.98, 91.67, 94.17, 91.91%) values

## How to apply

Load isomer pair similarity data from the in-house CQMD database (~1,80,000 MS/MS spectra of over 4500 metabolites acquired at 13 distinct CEs and stepped NCE 20/40/60%) using the 001-007.ipynb notebooks and mzbatch.py utilities in the Python 3.8.13 'longprof' conda environment. Compute the I/N ratio (intensity-based over number-based Jaccard similarity) for identical isomer pairs versus constitutional isomer pairs, then stratify pairs into bins by the number of longitudinal fragment profiles (LFP number) so that thresholds account for profile-count-dependent score behavior. Run ROC analysis within each LFP-number bin to obtain AUC values (expected range 0.874–0.991) and the optimal similarity score cutoff per bin. Finally, apply the derived cutoffs per ionization mode at NCE 1–100% to evaluate sensitivity and specificity (reported values: average sensitivity 98.89%, average specificity 92.66% across the four CQMD ionization modes), writing per-bin AUCs, cutoffs, and per-mode metrics to roc_analysis_results.csv.

## Related tools

- **Jupyter Notebook** (Runs the 001-007.ipynb notebooks that perform isomer similarity calculations and the ROC-based cutoff derivation) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **mzbatch.py** (Library of mass spectrometry data processing methods (file operations, data validation, calculation, extraction) used within the notebooks) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **001-007.ipynb** (Notebook series covering spectrum extraction through CQMU database construction, isomer similarity calculations, and metabolite identification) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **conda** (Creates the python=3.8.13 'longprof' environment required to reproduce the analysis)
- **Jaccard similarity algorithm** (Optimized similarity metric whose scores (intensity-based over number-based, I/N) feed the per-bin ROC analysis)

## Examples

```
conda create -n longprof python=3.8.13 && conda activate longprof && pip install -r requirements.txt && jupyter notebook 001-007.ipynb
```

## Evaluation signals

- roc_analysis_results.csv is produced and per-bin AUC values fall in the expected 0.874–0.991 range.
- Per-mode sensitivity values match 100, 98.35, 99.56, 99.69% and specificity values match 92.98, 91.67, 94.17, 91.91% across the four CQMD ionization modes at NCE 1–100%.
- Average sensitivity/specificity across modes equal 98.89% and 92.66%, matching Figures 5e–h, S6, and Excel S9–S13.
- Isomer pairs are correctly stratified into bins by LFP number before ROC fitting, with one optimal cutoff per bin rather than a single global cutoff.
- The environment reproduces with python=3.8.13 and requirements.txt (conda env 'longprof') and 001-007.ipynb cells run in sequence without errors.

## Limitations

- The in-house database CQMD is described as in-house with no accession or public access given, restricting full reproduction of the underlying isomer pairs.
- Results referenced in supplementary artifacts (Figures 5e–h, S6, Excel S9–S13) have availability/location not confirmed in the section.
- README states the data are 'intended for experimental reference only and should not be directly interpreted or used to guide human medicine' and are provided 'as is', without warranty of any kind.
- Criteria are mode- and NCE-dependent: cutoffs were derived within LFP-number bins for the four ionization modes of CQMD at NCE 1–100% and may not transfer to other acquisition settings.

## Evidence
