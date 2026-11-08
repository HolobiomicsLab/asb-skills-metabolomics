---
name: fragment-ion-cross-collision-energy-alignment
description: Use when MS/MS spectra for the same compound were acquired at multiple
  collision energies and you need to link identical fragment ions across CEs to construct
  per-fragment longitudinal intensity vectors before similarity scoring.
when_to_use_negative:
- Only a single collision energy per compound is available — alignment across the
  CE series is impossible and cross-sectional spectral matching should be used instead
- Input is already a per-fragment intensity table or feature matrix rather than raw/filtered
  MS/MS peak lists — proceed directly to similarity calculations
- The analysis goal is cross-sectional spectrum similarity (e.g. classic cosine, dot-product,
  or optimal entropy scoring), which does not require fragment tracking across collision
  energies
edam_topics:
- http://edamontology.org/topic_3521
tools:
- name: mzbatch.py
  role: Python library of mass spectrometry data processing methods (file operations,
    data validation, data generation, data calculation, data extraction) used to load
    and process the multi-NCE spectra for alignment
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: 001-007.ipynb
  role: Jupyter notebook series executed cell-by-cell that contains the fragment alignment
    / invalid fragment removal step (i.e. LFP construction) ahead of isomer similarity
    calculations
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: Python 3.8.13
  role: Runtime environment ('longprof' conda environment) in which the alignment
    and RII computation are run
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c01414
    title: Longitudinal Fragment Profiles Based on Multi-Collision Energy Tandem Mass
      Spectra Improve the Accuracy of Metabolite Identification in Untargeted Metabolomics
  source_content_hash: sha256:5d31c524410826f3faeeb482338cce8c753e0a72e958120ca34f18147986a038
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
  tools:
  - 001-007.ipynb
  - Jupyter Notebook
  - Python
  - Python 3.8.13
  - mzbatch.py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3521
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# fragment-ion-cross-collision-energy-alignment

## Summary

Aligns fragment ion m/z values of a reference compound across the full multi-NCE MS/MS acquisition series (13 distinct CEs plus stepped NCE 20/40/60%) so the same physical fragment is tracked longitudinally, then drops fragments whose relative ion intensity (RII) is below 2% to build LFP vectors for downstream spectral matching.

## When to use

Use when MS/MS spectra for the same compound were acquired at multiple collision energies and you need to link identical fragment ions across CEs to construct per-fragment longitudinal intensity vectors before similarity scoring.

## When NOT to use

- Only a single collision energy per compound is available — alignment across the CE series is impossible and cross-sectional spectral matching should be used instead
- Input is already a per-fragment intensity table or feature matrix rather than raw/filtered MS/MS peak lists — proceed directly to similarity calculations
- The analysis goal is cross-sectional spectrum similarity (e.g. classic cosine, dot-product, or optimal entropy scoring), which does not require fragment tracking across collision energies

## Inputs

- MS/MS spectra per reference compound acquired at 13 distinct NCEs plus stepped NCE 20/40/60% (spectrum-filtered, invalid-fragment-removed)
- mzbatch.py mass spectrometry data processing library
- 001-007.ipynb notebook series (Python 3.8.13 environment, requirements.txt)

## Outputs

- lfp_profiles.csv: one aligned relative-ion-intensity vector per retained fragment per compound spanning the full collision-energy series (13 NCEs + stepped NCE 20/40/60%)

## How to apply

Load the spectrum-filtered, invalid-fragment-removed MS/MS spectra for each reference compound at the 13 NCEs and stepped NCE 20/40/60% using the mzbatch.py mass spectrometry processing library in Python (run within the 001-007.ipynb notebook series). For each compound, align fragment ion m/z values across the full collision-energy series using lowest-m/z matching with a 0.01 m/z tolerance so the same physical fragment is linked across all acquisition energies. Compute the relative ion intensity (RII) of each aligned fragment at every collision energy, then zero out (treat as invalid) fragments whose RII falls below the 2% threshold so only fragments with meaningful longitudinal behavior are retained. Assemble the retained fragments into per-compound intensity vectors spanning the complete CE series and write the result to lfp_profiles.csv. The rationale is that longitudinal profiling considers predominant and minute fragments independently, unlike cross-sectional scoring that weighs predominant fragments heavily and diminishes minute fragments reflecting subtle structural differences.

## Related tools

- **mzbatch.py** (Python library of mass spectrometry data processing methods (file operations, data validation, data generation, data calculation, data extraction) used to load and process the multi-NCE spectra for alignment) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **001-007.ipynb** (Jupyter notebook series executed cell-by-cell that contains the fragment alignment / invalid fragment removal step (i.e. LFP construction) ahead of isomer similarity calculations) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **Python 3.8.13** (Runtime environment ('longprof' conda environment) in which the alignment and RII computation are run)

## Examples

```
conda create -n longprof python=3.8.13; conda activate longprof; pip install -r requirements.txt; jupyter notebook 001-007.ipynb
```

## Evaluation signals

- lfp_profiles.csv contains one intensity vector per retained fragment per compound, each spanning the complete 13-NCE + stepped NCE 20/40/60% series
- Every removed fragment satisfies the RII < 2% exclusion criterion (no retained fragment has RII below 2%)
- Lowest-m/z matching with 0.01 m/z tolerance links each m/z value to the lowest matching m/z across all CEs — alignment correctness is accepted by expert review of representative compounds
- Each aligned fragment row carries its highest RII within the 0.01 m/z match window for every collision energy
- Notebook run follows the prescribed sequence of cells in 001-007.ipynb with data stored at the path specified in the script

## Limitations

- Fragments with RII below 2% are zeroed as invalid, so fragments below this threshold cannot contribute to the longitudinal profile
- The 0.01 m/z alignment tolerance may mislink near-isobaric fragments in crowded low-m/z regions (lowest-m/z matching assigns to the lowest m/z within the window)
- The in-house CQMD/CQMU spectral data (over 4500 metabolites, ~180,000 MS/MS spectra) are described as in-house with no public accession; alignment validation was accepted by expert review rather than an automated check, and no changelog is available for the repository

## Evidence

- [methods] Peak removal threshold precedent in the article: [evidence span withheld — anchor status: not_found]
