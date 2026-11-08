---
name: longitudinal-fragment-profile-construction
description: Use when you have multi-collision-energy MS/MS spectra per reference
  compound (e.g. 13 NCEs and stepped NCE 20/40/60%) and need per-fragment intensity
  vectors across the CE series before similarity scoring.
when_to_use_negative:
- You only have a single-collision-energy MS/MS spectrum — no longitudinal behavior
  across CEs can be profiled, so cross-sectional spectral matching is the appropriate
  choice
- Input spectra have not yet been spectrum-filtered and had invalid fragments removed
  (a prerequisite handled by the earlier 001-007.ipynb preprocessing steps)
- Your goal is general cross-sectional peak matching against public libraries (MoNA,
  GNPS, METLIN) rather than building longitudinal fragment profiles for the in-house
  CQMD/CQMU workflow
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: mzbatch.py
  role: Library of commonly used methods for mass spectrometry data processing used
    to load and process the filtered multi-CE MS/MS spectra for LFP construction
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: Jupyter Notebook
  role: Environment in which the 001-007.ipynb cells implementing MS/MS extraction,
    filtering, and LFP assembly are run in sequence
- name: Python 3.8.13
  role: Runtime environment (conda env 'longprof') for the LFP construction code
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c01414
    title: Longitudinal Fragment Profiles Based on Multi-Collision Energy Tandem Mass
      Spectra Improve the Accuracy of Metabolite Identification in Untargeted Metabolomics
  source_content_hash: sha256:674d5a3a6e2a09eddb2f77199e0e4f06a00c10707860d7396c3fdcf11d39f0e0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
  tools:
  - Jupyter Notebook
  - Python
  - Python 3.8.13
  - mzbatch.py
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# longitudinal-fragment-profile-construction

## Summary

Constructs longitudinal fragment profile (LFP) vectors from MS/MS spectra acquired across 13 NCEs plus stepped NCE 20/40/60% by aligning fragment m/z values across collision energies (lowest-m/z matching, 0.01 m/z tolerance) and zeroing fragments with RII below 2%, producing the per-compound intensity matrix used for longitudinal similarity scoring.

## When to use

Use when you have multi-collision-energy MS/MS spectra per reference compound (e.g. 13 NCEs and stepped NCE 20/40/60%) and need per-fragment intensity vectors across the CE series before similarity scoring.

## When NOT to use

- You only have a single-collision-energy MS/MS spectrum — no longitudinal behavior across CEs can be profiled, so cross-sectional spectral matching is the appropriate choice
- Input spectra have not yet been spectrum-filtered and had invalid fragments removed (a prerequisite handled by the earlier 001-007.ipynb preprocessing steps)
- Your goal is general cross-sectional peak matching against public libraries (MoNA, GNPS, METLIN) rather than building longitudinal fragment profiles for the in-house CQMD/CQMU workflow

## Inputs

- Spectrum-filtered, invalid-fragment-removed MS/MS spectra per reference compound acquired at 13 distinct NCEs
- MS/MS spectra acquired at stepped NCE 20/40/60%
- Python environment (Python 3.8.13, conda env 'longprof', requirements.txt dependencies)

## Outputs

- lfp_profiles.csv containing one intensity vector per retained fragment per compound spanning the complete collision-energy series

## How to apply

Load the spectrum-filtered, invalid-fragment-removed MS/MS spectra for each reference compound at the 13 NCEs and stepped NCE 20/40/60% using the mzbatch.py mass spectrometry processing library in Python (run within the 001-007.ipynb notebook series). For each compound, align fragment ion m/z values across the full collision-energy series by lowest-m/z matching with a 0.01 m/z tolerance so the same physical fragment is linked across all acquisition energies. Compute the relative ion intensity (RII) of each aligned fragment at every collision energy in the series, then drop fragments whose RII is below the 2% threshold so only fragments with meaningful longitudinal behavior are retained. Assemble retained fragments into per-compound intensity vectors spanning the 13 NCEs plus stepped NCE 20/40/60% and write the LFP table to lfp_profiles.csv. The rationale is that longitudinal profiles capture how each fragment — including minute ones that cross-sectional profiles diminish — varies with collision energy, treating predominant and minute fragments independently.

## Related tools

- **mzbatch.py** (Library of commonly used methods for mass spectrometry data processing used to load and process the filtered multi-CE MS/MS spectra for LFP construction) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **Jupyter Notebook** (Environment in which the 001-007.ipynb cells implementing MS/MS extraction, filtering, and LFP assembly are run in sequence)
- **Python 3.8.13** (Runtime environment (conda env 'longprof') for the LFP construction code)

## Examples

```
conda create -n longprof python=3.8.13; conda activate longprof; pip install -r requirements.txt; then run cells of 001-007.ipynb in Jupyter Notebook to write lfp_profiles.csv
```

## Evaluation signals

- lfp_profiles.csv contains one intensity vector per retained fragment per compound spanning the complete collision-energy series (13 NCEs plus stepped NCE 20/40/60%)
- Every removed fragment satisfies the RII < 2% exclusion criterion; no fragment above 2% RII is dropped
- Lowest-m/z 0.01 m/z tolerance alignment is verified by expert review of representative compounds — the same physical fragment must be linked across all acquisition energies
- Downstream check: the LFP vectors support isomer similarity calculations that reproduce the paper's reported 8.7–25.9% accuracy improvement over cross-sectional profiling
- Repository environment check: conda env 'longprof' with Python 3.8.13 and requirements.txt installed resolves all dependencies in 001-007.ipynb

## Limitations

- The in-house spectral library (~180,000 MS/MS spectra of over 4500 metabolites at 13 CEs) and CQMD database are in-house; no public accession or access is stated, limiting reproduction outside the authors' data
- The 0.01 m/z alignment tolerance and 2% RII threshold are tuned for this acquisition scheme; other instruments or CE series may require revalidation
- The repository dataset must be stored at the exact path specified in the scripts; path mismatches will break the notebook run

## Evidence

- [other] LFP construction aligns fragment ions across CEs to the lowest m/z within a 0.01 m/z tolerance (assigning each CE its highest RII in that range), and fragments with RII below 2% are treated as invalid (zeroed).: [evidence span withheld — anchor status: not_found]
