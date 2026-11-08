---
name: roc-criterion-threshold-application
description: Use when similarity scores from longitudinal fragment profiling (multi-collision-energy
  MS/MS matched against an in-house spectral library) must be converted into identification
  calls via ROC-derived thresholds, or when verifying match rates under a loosened
  criterion.
when_to_use_negative:
- Spectra are cross-sectional (single collision energy), so no longitudinal fragment
  profile can be built for Jaccard-based scoring against the multi-CE library.
- The goal is de novo annotation of unknowns with no reference MS/MS library — the
  ROC criteria only threshold similarity scores against the in-house ~180,000-spectrum
  library.
- Input is already a curated, identified feature/annotation table rather than raw
  QC MS/MS spectra needing match-rate evaluation.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Jaccard similarity algorithm (optimized for longitudinal profiling)
  role: Computes the similarity scores between experimental QC spectra and library
    spectra to which the ROC-derived criteria are applied
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: 001-007.ipynb
  role: Notebook series executing spectrum extraction through isomer similarity calculation
    and metabolite identification, where thresholds are applied and match rates computed
- name: mzbatch.py
  role: Python library for mass spectrometry data processing used to extract QC spectra
    and apply peak (<1% relative intensity) and neutral-loss filters before scoring
- name: conda / Python 3.8.13
  role: Execution environment (env 'longprof', python=3.8.13) required to run the
    notebooks reproducibly
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c01414
    title: Longitudinal Fragment Profiles Based on Multi-Collision Energy Tandem Mass
      Spectra Improve the Accuracy of Metabolite Identification in Untargeted Metabolomics
  source_content_hash: sha256:c97d61734e124e698d8291663d32ab4cde3de2f00a6911932e90144b145b8c41
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
  tools:
  - 001-007.ipynb
  - Jaccard similarity algorithm (optimized for longitudinal profiling)
  - Jupyter Notebook
  - Python
  - conda
  - conda / Python 3.8.13
  - mzbatch.py
  - pip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# roc-criterion-threshold-application

## Summary

Applies ROC-derived identification criteria (and a 0.7-fold loosened variant) to similarity scores produced by the optimized Jaccard longitudinal fragment profiling method, converting spectral-match scores into per-metabolite accept/reject calls and per-class correct-match rates for spiked plasma QC samples.

## When to use

Use when similarity scores from longitudinal fragment profiling (multi-collision-energy MS/MS matched against an in-house spectral library) must be converted into identification calls via ROC-derived thresholds, or when verifying match rates under a loosened criterion.

## When NOT to use

- Spectra are cross-sectional (single collision energy), so no longitudinal fragment profile can be built for Jaccard-based scoring against the multi-CE library.
- The goal is de novo annotation of unknowns with no reference MS/MS library — the ROC criteria only threshold similarity scores against the in-house ~180,000-spectrum library.
- Input is already a curated, identified feature/annotation table rather than raw QC MS/MS spectra needing match-rate evaluation.

## Inputs

- MS1 and MS/MS spectra of spiked pooled-plasma QC samples (e.g., 2 uM standards, 150 CP / 134 CN metabolites) from an Exploris 120
- Longitudinal fragment profiles (relative fragment ion intensities across 13 CEs / stepped NCE 20/40/60%)
- In-house spectral library of ~180,000 MS/MS spectra (CQMD/CQMU) with isomer similarity scores
- ROC-derived identification criteria thresholds and the 0.7-fold loosened variant

## Outputs

- Per-metabolite correct/incorrect match calls under standard and 0.7-fold loosened criteria
- Correct-match rates per ionization mode and metabolite class (CP vs CN), e.g., 92.96% CP / 88.57% CN, >97% under loosening
- Results table spiked_qc_match_rates.csv

## How to apply

Run the 001-007.ipynb notebooks in sequence in a conda env (python=3.8.13, pip install -r requirements.txt) after cloning LongitudinalProf-MSMS-Method. Extract MS1 and MS/MS spectra from the QC runs (e.g., Exploris 120 spiked pooled plasma) with mzbatch.py, then preprocess before scoring: remove peaks with relative intensity below 1% and filter fragments corresponding to common neutral losses. Build longitudinal fragment profiles (relative ion intensities of each fragment across the 13 collision energies / stepped NCE 20/40/60%) and match them against the ~180,000-spectrum in-house library using the optimized Jaccard similarity algorithm. Apply the identification criteria derived by ROC analysis of thousands of isomers in the in-house CQMD database to call matches, then repeat with the 0.7-fold loosened variant. Compute correct-match rates separately per ionization mode and metabolite class (e.g., 150 CP and 134 CN spiked metabolites) and judge success against the reported benchmarks: 92.96% (CP) and 88.57% (CN) correct matching, rising above 97% for both classes under the 0.7-fold loosening; write the per-class rates to spiked_qc_match_rates.csv.

## Related tools

- **Jaccard similarity algorithm (optimized for longitudinal profiling)** (Computes the similarity scores between experimental QC spectra and library spectra to which the ROC-derived criteria are applied) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **001-007.ipynb** (Notebook series executing spectrum extraction through isomer similarity calculation and metabolite identification, where thresholds are applied and match rates computed)
- **mzbatch.py** (Python library for mass spectrometry data processing used to extract QC spectra and apply peak (<1% relative intensity) and neutral-loss filters before scoring)
- **conda / Python 3.8.13** (Execution environment (env 'longprof', python=3.8.13) required to run the notebooks reproducibly)

## Examples

```
conda create -n longprof python=3.8.13 && conda activate longprof && pip install -r requirements.txt && jupyter notebook
```

## Evaluation signals

- Correct-match rates on the Exploris 120 spiked-plasma data reproduce 92.96% (CP) and 88.57% (CN) under the ROC-derived criteria.
- Applying the 0.7-fold loosened criterion raises match rates above 97% for both CP and CN classes.
- Rates are computed separately for the 150 CP and 134 CN spiked metabolites, not pooled.
- The output file spiked_qc_match_rates.csv is produced with per-class rates under both criterion variants.
- Thresholded calls retain sensitivity/specificity reported as better than existing cross-sectional matching methods.

## Limitations

- The identification criteria were derived from ROC analysis of the in-house CQMD database, which has no stated public accession, so thresholds may not transfer to other spectral libraries or instruments.
- Loosening the criterion by 0.7-fold raises match rates above 97% but relaxes the ROC-derived cutoff, so the loosened calls should not be treated as equivalent in stringency to the standard criteria.
- Some validation results (Figures 5e–h, S6, Excel S9–S13) are referenced in supplementary artifacts whose availability/location is not confirmed.

## Evidence

- [methods] Peaks with relative intensity below 1% were removed from the MS/MS spectra.: [evidence span withheld — anchor status: not_found]
