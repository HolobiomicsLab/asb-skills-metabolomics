---
name: spiked-standard-qc-validation-metabolomics
description: Use when a spectral-matching or annotation method needs quantitative
  validation on samples of known composition — spiked pooled-plasma QC with defined
  standard concentrations (e.g., 2 uM) acquired at multiple collision energies on
  a Thermo Exploris 120.
when_to_use_negative:
- The spectra were not acquired at multiple distinct collision energies, so longitudinal
  fragment profiles (relative ion intensities across CEs) cannot be built — use a
  cross-sectional matching method instead.
- No samples of known composition (spiked standards or QC pools) are available, so
  correct-match rates cannot be computed against ground truth.
- The goal is de novo metabolite identification or annotation of unknowns rather than
  validation of a matching method against a known-composition benchmark.
edam_operation: http://edamontology.org/operation_0322
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: mzbatch.py
  role: Extracts MS1 and MS/MS spectra for the spiked-pool QC samples and provides
    data processing routines (filtering, neutral-loss removal) before scoring
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: 001-007.ipynb
  role: Notebook series executed sequentially to run spectrum extraction, filtering,
    similarity calculations, and metabolite identification on the QC data
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: conda
  role: Creates the pinned Python environment (longprof, python=3.8.13) required for
    reproducible QC validation
- name: pip
  role: Installs the pinned dependencies from requirements.txt into the longprof environment
- name: Jupyter Notebook
  role: Runtime for executing the 001-007.ipynb notebooks that carry out the QC matching
    workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c01414
    title: Longitudinal Fragment Profiles Based on Multi-Collision Energy Tandem Mass
      Spectra Improve the Accuracy of Metabolite Identification in Untargeted Metabolomics
  source_content_hash: sha256:682e37254d5075aca740e905189d3530e9c855b2b8282a746d3d8c0f1423a0bd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
  tools:
  - 001-007.ipynb
  - Jupyter Notebook
  - Python
  - conda
  - mzbatch.py
  - pip
  edam_operation: http://edamontology.org/operation_0322
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# spiked-standard-qc-validation-metabolomics

## Summary

Validates a metabolite identification method by matching spike-in pooled-plasma QC samples (known metabolites at known concentrations) against a spectral library and computing correct-match rates per ionization mode. Here it is used to reproduce the longitudinal fragment profiling (LFP) benchmark on the in-house Exploris 120: 92.96% (CP) and 88.57% (CN) correct matches, rising above 97% under a 0.7-fold loosened criterion.

## When to use

Use when a spectral-matching or annotation method needs quantitative validation on samples of known composition — spiked pooled-plasma QC with defined standard concentrations (e.g., 2 uM) acquired at multiple collision energies on a Thermo Exploris 120.

## When NOT to use

- The spectra were not acquired at multiple distinct collision energies, so longitudinal fragment profiles (relative ion intensities across CEs) cannot be built — use a cross-sectional matching method instead.
- No samples of known composition (spiked standards or QC pools) are available, so correct-match rates cannot be computed against ground truth.
- The goal is de novo metabolite identification or annotation of unknowns rather than validation of a matching method against a known-composition benchmark.

## Inputs

- Raw MS1 and MS/MS spectra of spiked pooled-plasma QC samples (2 uM standards) acquired on a Thermo Exploris 120 at multiple collision energies (stepped NCE 20/40/60%)
- In-house spectral library of ~180,000 MS/MS spectra of over 4500 metabolites
- mzbatch.py utility library and 001-007.ipynb notebooks from the LongitudinalProf-MSMS-Method repository
- List of spiked metabolites per mode (150 in CP, 134 in CN) as ground-truth labels

## Outputs

- spiked_qc_match_rates.csv containing correct-match rates per ionization mode and criterion
- Correct-match rates under the ROC-derived criteria and the 0.7-fold loosened variant (expected: 92.96% CP / 88.57% CN; >97% both classes when loosened)

## How to apply

Set up the environment per the README: create a conda env with python=3.8.13, pip install -r requirements.txt, and run 001-007.ipynb notebooks sequentially in Jupyter Notebook. Extract MS1 and MS/MS spectra for the spiked-pool plasma QC samples (2 uM standards) with mzbatch.py, filter peaks with relative intensity below 1%, and remove fragments corresponding to common neutral losses before scoring. Build longitudinal fragment profiles (relative ion intensities of each fragment across multiple collision energies, e.g., stepped NCE 20/40/60%) for the QC spectra, then match them against the in-house spectral library of ~180,000 MS/MS spectra using the optimized Jaccard similarity algorithm for longitudinal profiling. Apply the ROC-derived identification criteria plus the 0.7-fold loosened variant, computing correct-match rates separately for the 150 CP and 134 CN spiked metabolites, and write results to spiked_qc_match_rates.csv.

## Related tools

- **mzbatch.py** (Extracts MS1 and MS/MS spectra for the spiked-pool QC samples and provides data processing routines (filtering, neutral-loss removal) before scoring) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **001-007.ipynb** (Notebook series executed sequentially to run spectrum extraction, filtering, similarity calculations, and metabolite identification on the QC data) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **conda** (Creates the pinned Python environment (longprof, python=3.8.13) required for reproducible QC validation)
- **pip** (Installs the pinned dependencies from requirements.txt into the longprof environment)
- **Jupyter Notebook** (Runtime for executing the 001-007.ipynb notebooks that carry out the QC matching workflow)

## Examples

```
conda create -n longprof python=3.8.13 && conda activate longprof && pip install -r requirements.txt
```

## Evaluation signals

- Correct-match rates must reproduce 92.96% in CP mode and 88.57% in CN mode for detected metabolites in the spiked QC samples.
- With the 0.7-fold loosened criteria, correct-match rates must rise above 97% for both the 150 CP and 134 CN spiked metabolites.
- spiked_qc_match_rates.csv is produced with rates reported separately per ionization mode and per criterion variant.
- Preprocessing invariants hold: peaks with relative intensity below 1% removed and fragments corresponding to common neutral losses filtered out before similarity scoring.
- The run used the pinned environment (python=3.8.13, requirements.txt) and all cells of 001-007.ipynb executed in sequence without skipped steps.

## Limitations

- Match rates were established on the in-house Exploris 120 and the in-house spectral library (~180,000 MS/MS spectra); no public accession for the library is given, which limits portability.
- The spiked-plasma benchmark covers 2 uM standards only (150 CP / 134 CN metabolites), so rates do not generalize to other concentrations or matrices without further validation.
- The README states the data are 'intended for experimental reference only and should not be directly interpreted or used to guide human medicine'.

## Evidence

- [methods] methods: 1% relative-intensity peak filter: [evidence span withheld — anchor status: not_found]
- [methods] methods: neutral-loss fragment filtering: [evidence span withheld — anchor status: not_found]
