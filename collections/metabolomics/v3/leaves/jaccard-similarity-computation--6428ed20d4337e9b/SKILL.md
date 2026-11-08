---
name: jaccard-similarity-computation
description: Use when scoring isomer-pair MS/MS similarity from multi-collision-energy
  spectra (e.g., NCE 1–100%) to build I/N-ratio-based identification criteria, or
  when comparing Jaccard vs dot product/entropy sensitivity-specificity on spectral
  libraries like CQMD.
when_to_use_negative:
- You only have single-CE (stepped NCE 20/40/60% or one NCE) reference spectra — cross-sectional
  dot product/entropy matching is the appropriate baseline, and longitudinal profiling
  gains require multi-CE acquisitions
- Input spectra have not passed the upstream pipeline (spectrum filtering, invalid
  fragment removal, fragment rationality assessment); the CQMD/CQMU database must
  be constructed first
- Candidate compounds lack isomer counterparts in the reference library, so longitudinal
  fragment-profile matching cannot distinguish them
edam_operation: http://edamontology.org/operation_3765
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: mzbatch.py
  role: Python library of mass spectrometry data-processing methods (file operations,
    data validation/generation/calculation/extraction) used to compute isomer similarity
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: 001-007.ipynb
  role: Notebooks covering CQMU database construction and isomer similarity calculations
    that feed the Jaccard scoring
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: Python 3.8.13
  role: Runtime environment (conda env 'longprof') for the similarity and ROC analyses
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.5c01414
    title: Longitudinal Fragment Profiles Based on Multi-Collision Energy Tandem Mass
      Spectra Improve the Accuracy of Metabolite Identification in Untargeted Metabolomics
  source_content_hash: sha256:fae3e125015ced7fb3085d9425a69d75f043820b7e478ced6d2be8886424ba7b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
  tools:
  - 001-007.ipynb
  - Jaccard similarity algorithm
  - Jupyter Notebook
  - Python
  - Python 3.8.13
  - conda
  - mzbatch.py
  - pip
  edam_operation: http://edamontology.org/operation_3765
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# jaccard-similarity-computation

## Summary

An optimized Jaccard similarity algorithm for scoring MS/MS spectral matches using longitudinal fragment profiles (LFPs) acquired at multiple collision energies, treating predominant and minute fragments independently rather than weighting them together as in cross-sectional dot product or entropy scoring.

## When to use

Use when scoring isomer-pair MS/MS similarity from multi-collision-energy spectra (e.g., NCE 1–100%) to build I/N-ratio-based identification criteria, or when comparing Jaccard vs dot product/entropy sensitivity-specificity on spectral libraries like CQMD.

## When NOT to use

- You only have single-CE (stepped NCE 20/40/60% or one NCE) reference spectra — cross-sectional dot product/entropy matching is the appropriate baseline, and longitudinal profiling gains require multi-CE acquisitions
- Input spectra have not passed the upstream pipeline (spectrum filtering, invalid fragment removal, fragment rationality assessment); the CQMD/CQMU database must be constructed first
- Candidate compounds lack isomer counterparts in the reference library, so longitudinal fragment-profile matching cannot distinguish them

## Inputs

- Preprocessed MS/MS spectra of isomer pairs from the in-house CQMD database (~180,000 MS/MS spectra of >4,500 metabolites, four ionization modes CP/CN/AP/AN, NCE 1–100%)
- Isomer-pair labels (identical vs constitutional isomer pairs)
- Longitudinal fragment profiles (fragment relative-intensity vectors across 13 distinct CEs)
- Similarity score threshold (0.95 for cross-sectional binarization)

## Outputs

- Per-pair Jaccard similarity scores and I/N ratios
- Per-LFP-number-bin ROC AUCs and optimal similarity cutoffs (roc_analysis_results.csv)
- Per-ionization-mode sensitivity (e.g., 100, 98.35, 99.56, 99.69%) and specificity (e.g., 92.98, 91.67, 94.17, 91.91%) values
- Algorithm × NCE metrics table of sensitivity/specificity (cross_sectional_similarity_metrics.csv)

## How to apply

After upstream preprocessing (spectrum filtering, invalid fragment removal, fragment rationality assessment, CQMU database construction via the 001-007.ipynb notebooks), compute longitudinal fragment profiles — the relative ion intensities of each fragment across multiple distinct collision energies — and apply the optimized Jaccard similarity algorithm to each isomer pair, considering predominant and minute fragments independently. Derive the I/N ratio (intensity-based over number-based Jaccard similarity) for identical vs constitutional isomer pairs, stratify pairs into bins by LFP number, and run ROC analysis per bin to obtain AUCs (expected 0.874–0.991) and optimal similarity cutoffs. For a cross-sectional benchmark, compute dot product, modified dot product, entropy, and Jaccard scores at each of 13 individual NCEs plus stepped NCE 20/40/60%, binarize at the 0.95 threshold, and compute sensitivity/specificity per algorithm × NCE condition. Rationale: cross-sectional evaluation weights predominant fragments heavily and diminishes minute fragments that reflect subtle structural differences, producing comparable scores for isomers with similar predominant but different minute fragments.

## Related tools

- **mzbatch.py** (Python library of mass spectrometry data-processing methods (file operations, data validation/generation/calculation/extraction) used to compute isomer similarity) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **001-007.ipynb** (Notebooks covering CQMU database construction and isomer similarity calculations that feed the Jaccard scoring) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **Python 3.8.13** (Runtime environment (conda env 'longprof') for the similarity and ROC analyses)

## Examples

```
conda create -n longprof python=3.8.13 && conda activate longprof && pip install -r requirements.txt && jupyter notebook 001-007.ipynb
```

## Evaluation signals

- Reproduced I/N-ratio ROC AUCs fall in the 0.874–0.991 range across LFP-number bins
- Per-mode sensitivity values match 100 / 98.35 / 99.56 / 99.69% and specificity 92.98 / 91.67 / 94.17 / 91.91% (average 98.89% / 92.66%) within tolerance
- Cross-sectional benchmark at NCE 70% with 0.95 threshold reproduces 93.25% sensitivity / 87.96% specificity, and stepped NCE 20/40/60% reproduces 92.25% / 77.38%
- Per-bin AUCs and cutoffs match Figures 5e–h, S6, and Excel S9–S13
- Output files roc_analysis_results.csv and cross_sectional_similarity_metrics.csv are populated with schema-complete algorithm × condition rows

## Limitations

- The CQMD in-house database availability is not stated publicly (no accession given), limiting external reuse
- Results are tied to Figure 5e–h / S6 / Excel S9–S13 supplementary artifacts whose availability is not confirmed
- Data are intended for experimental reference only and must not be used to guide human medicine (per repository license disclaimer)
- Identification criteria were derived by ROC analysis on CQMD isomers; cutoffs may not transfer to other instruments or libraries without revalidation (inter-instrument, spiked, and human plasma validation was performed)

## Evidence
