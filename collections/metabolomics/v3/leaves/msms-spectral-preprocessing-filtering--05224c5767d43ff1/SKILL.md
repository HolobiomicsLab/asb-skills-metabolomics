---
name: msms-spectral-preprocessing-filtering
description: Use when raw MS/MS spectra acquired across multiple collision energies
  must be cleaned (noise peak removal, invalid fragment removal) before computing
  per-fragment LFP intensity vectors.
when_to_use_negative:
- Input is a single-collision-energy spectrum — longitudinal filtering presumes an
  acquired CE series across which fragment RII behavior can be assessed
- The spectra are already filtered (i.e., an existing lfp_profiles.csv or cleaned
  spectral table is available) — re-running would drop valid fragments against the
  2% RII criterion redundantly
- The task is downstream similarity scoring or metabolite identification — this skill
  only covers the preprocessing stages, not isomer similarity calculations or identification
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mzbatch.py
  role: Python library supplying the file operations, data validation, data generation,
    data calculation, and data extraction functions used to load, filter, and clean
    the multi-CE MS/MS spectra
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: 001-007.ipynb
  role: 'Notebook series implementing the sequential preprocessing steps: MS1 and
    MS/MS spectrum extraction, spectrum filtering, invalid fragment removal, and fragment
    rationality assessment'
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: conda
  role: Creates the pinned Python 3.8.13 environment ('longprof') in which the filtering
    notebooks are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c01414
    title: Longitudinal Fragment Profiles Based on Multi-Collision Energy Tandem Mass
      Spectra Improve the Accuracy of Metabolite Identification in Untargeted Metabolomics
  source_content_hash: sha256:b01a4afa6f2509811ee4d977e6b17383dd43d868e9caf1a304a3ffea03afb50d
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# msms-spectral-preprocessing-filtering

## Summary

Filters and cleans multi-collision-energy MS/MS spectra (13 NCEs plus stepped NCE 20/40/60%) prior to longitudinal fragment profile (LFP) construction, removing noise peaks and invalid fragments so only fragments with meaningful longitudinal behavior are retained.

## When to use

Use when raw MS/MS spectra acquired across multiple collision energies must be cleaned (noise peak removal, invalid fragment removal) before computing per-fragment LFP intensity vectors.

## When NOT to use

- Input is a single-collision-energy spectrum — longitudinal filtering presumes an acquired CE series across which fragment RII behavior can be assessed
- The spectra are already filtered (i.e., an existing lfp_profiles.csv or cleaned spectral table is available) — re-running would drop valid fragments against the 2% RII criterion redundantly
- The task is downstream similarity scoring or metabolite identification — this skill only covers the preprocessing stages, not isomer similarity calculations or identification

## Inputs

- Raw multi-collision-energy MS/MS spectra per reference compound (13 NCEs + stepped NCE 20/40/60%)
- mzbatch.py library of mass spectrometry data-processing methods
- 001-007.ipynb notebook chain (spectrum extraction through fragment rationality assessment)
- requirements.txt package specification (Python 3.8.13)

## Outputs

- Spectrum-filtered, invalid-fragment-removed MS/MS spectra for each compound across the full collision-energy series
- Fragment-rationality-assessed spectra ready for longitudinal fragment profile (LFP) vector construction

## How to apply

Load the MS1 and MS/MS spectra for each reference compound across the 13 NCEs and stepped NCE 20/40/60% using the mzbatch.py mass spectrometry processing library in Python (Python 3.8.13 conda env 'longprof'). First apply spectrum filtering, removing peaks with relative intensity below 1% from each MS/MS spectrum. Next perform invalid fragment removal: drop fragments whose relative ion intensity (RII) falls below the 2% threshold, since these fragments carry no meaningful longitudinal behavior across the collision-energy series. Then carry out fragment rationality assessment on the surviving peaks before the cleaned spectra are passed downstream to LFP alignment and CQMU/CQMU-style database construction. These steps are implemented as the early stages of the 001-007.ipynb notebook chain (spectrum extraction → spectrum filtering → invalid fragment removal → fragment rationality assessment), so they must run in sequence before any similarity scoring.

## Related tools

- **mzbatch.py** (Python library supplying the file operations, data validation, data generation, data calculation, and data extraction functions used to load, filter, and clean the multi-CE MS/MS spectra) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **001-007.ipynb** (Notebook series implementing the sequential preprocessing steps: MS1 and MS/MS spectrum extraction, spectrum filtering, invalid fragment removal, and fragment rationality assessment) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **conda** (Creates the pinned Python 3.8.13 environment ('longprof') in which the filtering notebooks are executed)

## Examples

```
conda create -n longprof python=3.8.13 && conda activate longprof && pip install -r requirements.txt && jupyter notebook 001-007.ipynb
```

## Evaluation signals

- QC-check that removed fragments satisfy the RII < 2% invalid-fragment exclusion criterion
- QC-check that peaks with relative intensity below 1% have been removed from every MS/MS spectrum prior to downstream alignment
- Verify that each compound's cleaned spectra retain the complete collision-energy series (13 NCEs plus stepped NCE 20/40/60%) so LFP vectors can span all energies
- Confirm the notebook chain 001-007 was executed in sequence, with filtering completed before fragment rationality assessment and CQMU database construction
- Expert review of representative compounds' fragment rationality assessment to confirm no physically implausible fragments survive filtering

## Limitations

- No changelog is available for the associated repository/artifact, so filtering-code provenance must be established manually from the notebooks
- The study data, including datasets and findings, are stated to be for experimental reference only and not to be used to guide human medicine; the project is provided 'as is' without warranty
- Results referenced in supplementary artifacts (Figures 5e–h, S6, Excel S9–S13) have availability/location not confirmed, complicating end-to-end verification of the filtering pipeline's outputs

## Evidence

- [methods] Concrete peak-filtering threshold applied to MS/MS spectra: [evidence span withheld — anchor status: not_found]
