---
name: jaccard-similarity-spectrum-scoring
description: Use when experimental and reference MS/MS spectra are available at multiple
  collision energies (e.g., stepped NCE 20/40/60%) and you must identify metabolites
  or quantify correct-match rates against an isomer-rich library.
when_to_use_negative:
- Only single-collision-energy MS/MS spectra are available — without multi-CE acquisition,
  longitudinal fragment profiles cannot be built and cross-sectional dot-product/entropy
  scoring is the fallback
- The input is already a feature/annotation table with no raw MS/MS spectra to re-score
- No reference library of spectra at matching collision energies exists, so Jaccard
  scores would be computed against incomparable profiles
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Jaccard similarity algorithm
  role: Optimized similarity metric for scoring experimental spectra against the reference
    library via longitudinal fragment profiles
- name: mzbatch.py
  role: 'Python library for MS data processing: MS1/MS2 extraction, spectrum filtering,
    neutral-loss fragment removal'
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: 001-007.ipynb
  role: Notebook series executing isomer similarity calculations and metabolite identification
    steps
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: Python 3.8.13
  role: Runtime environment created via conda (env 'longprof') with dependencies from
    requirements.txt
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c01414
    title: Longitudinal Fragment Profiles Based on Multi-Collision Energy Tandem Mass
      Spectra Improve the Accuracy of Metabolite Identification in Untargeted Metabolomics
  source_content_hash: sha256:3db959d6c81e7676ca12b4df90a2f43171ab4844dfa16203dbf3f35324cf83cf
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
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# jaccard-similarity-spectrum-scoring

## Summary

Scores experimental MS/MS spectra against a reference library by optimizing the Jaccard similarity algorithm for longitudinal fragment profiles (relative ion intensities of each fragment across multiple collision energies), treating predominant and minute fragments independently to reduce isomer misidentification.

## When to use

Use when experimental and reference MS/MS spectra are available at multiple collision energies (e.g., stepped NCE 20/40/60%) and you must identify metabolites or quantify correct-match rates against an isomer-rich library.

## When NOT to use

- Only single-collision-energy MS/MS spectra are available — without multi-CE acquisition, longitudinal fragment profiles cannot be built and cross-sectional dot-product/entropy scoring is the fallback
- The input is already a feature/annotation table with no raw MS/MS spectra to re-score
- No reference library of spectra at matching collision energies exists, so Jaccard scores would be computed against incomparable profiles

## Inputs

- Experimental MS/MS spectra acquired at multiple collision energies (e.g., Exploris 120, stepped NCE 20/40/60%)
- Reference MS/MS spectral library (in-house ~180,000 spectra of >4500 metabolites; also validated against NIST MS/MS, MoNA, GNPS, METLIN)
- QC sample spectra, e.g., spiked pooled plasma with 2 uM standards (150 CP / 134 CN metabolites)

## Outputs

- Metabolite identifications with optimized-Jaccard longitudinal-profile match scores
- Correct-match rates per mode (e.g., written to spiked_qc_match_rates.csv)
- ROC-derived identification criteria and the 0.7-fold loosened variant

## How to apply

Extract MS1/MS2 spectra (e.g., with mzbatch.py from the 001-007.ipynb notebooks), remove peaks with relative intensity below 1%, and filter fragments corresponding to common neutral losses before scoring. Build longitudinal fragment profiles (LFPs) of each fragment's relative ion intensity across the 13 distinct collision energies, then match against the in-house library (~180,000 MS/MS spectra of >4500 metabolites) with the optimized Jaccard similarity algorithm rather than classic dot-product or optimal entropy cross-sectional scoring. Apply identification criteria derived by ROC analysis of thousands of isomers in the in-house CQMD database, and optionally a 0.7-fold loosened criterion variant. Rationale: cross-sectional scoring weighs predominant fragments heavily and diminishes minute fragments that carry subtle structural differences, whereas longitudinal profiling considers both independently, improving accuracy by 8.7–25.9% and cutting FDR by 28.6–41.7%.

## Related tools

- **Jaccard similarity algorithm** (Optimized similarity metric for scoring experimental spectra against the reference library via longitudinal fragment profiles)
- **mzbatch.py** (Python library for MS data processing: MS1/MS2 extraction, spectrum filtering, neutral-loss fragment removal) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **001-007.ipynb** (Notebook series executing isomer similarity calculations and metabolite identification steps) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **Python 3.8.13** (Runtime environment created via conda (env 'longprof') with dependencies from requirements.txt)

## Examples

```
conda create -n longprof python=3.8.13 && conda activate longprof && pip install -r requirements.txt  # then run 001-007.ipynb cells in Jupyter Notebook
```

## Evaluation signals

- Reproduce spiked-plasma correct-match rates of 92.96% (CP) and 88.57% (CN) on Exploris 120 data
- Match rates rise above 97% for both CP and CN when the identification criteria are loosened by 0.7-fold
- Annotation accuracy improves by 8.7–25.9% and FDR drops by 28.6–41.7% versus cross-sectional profiling with the optimal entropy algorithm
- Confirm all peaks with relative intensity below 1% and common neutral-loss fragments were removed before scoring
- Results are written to spiked_qc_match_rates.csv after running 001-007.ipynb in sequence

## Limitations

- The in-house CQMD database (~180,000 spectra) is described as in-house with no public accession, so exact reproduction may require contacting the authors (<maintainer e-mail, see the repository README>)
- Criteria were ROC-derived from thousands of isomers in CQMD; performance on libraries with different collision-energy coverage is not established
- No changelog is available for the repository, and no code availability is stated in the discussion section

## Evidence

- [methods] Peaks with relative intensity below 1% were removed from the MS/MS spectra.: [evidence span withheld — anchor status: not_found]
- [methods] Fragments corresponding to common neutral losses were filtered out before scoring.: [evidence span withheld — anchor status: not_found]
