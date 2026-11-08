---
name: ppm-to-dalton-mass-tolerance-conversion
description: Use when setting precursor-mass search tolerances for tandem MS annotation
  (e.g. ChemDistiller's --delta_mz) or when quantifying how much a high-accuracy ppm
  window narrows database candidate lists.
when_to_use_negative:
- The analysis requires MS2-level discrimination (fragmentation patterns, fingerprints,
  FingerScorer SVM scoring) rather than precursor-mass pre-filtering of candidates
- The MS1 data are low-resolution so that a 20 ppm window is unrealistic; the delta_mz
  tolerance should be set to match the instrument's actual mass accuracy
- The comparison database differs fundamentally in composition from PubChem, so the
  absolute counts (250,079 / 3,229) and the ~77.5x ratio would not transfer
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3318
tools:
- name: NumPy
  role: Loads the PubChem compound-mass dump and performs windowed mass filtering
    (+/- 0.5 Da vs +/- 0.008 Da)
- name: ChemDistiller
  role: Annotation engine whose candidate filtering applies the ppm tolerance (--delta_mz)
    against pre-calculated compound databases
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: chemdistiller.filters.formula
  role: Formula-filter module used in the candidate-filtering pipeline the ppm window
    feeds into
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: chemdistiller.chemdb.database
  role: Database module providing the compound records (with monoisotopic masses)
    that the mass windows are evaluated against
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:2a05556d64489d460d456f04ba310e04cb50d55509563dc3030facf66d6a1e78
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDistiller
  - NumPy
  - Python 2 and 3
  - chemdistiller.chemdb.database
  - chemdistiller.filters.formula
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# ppm-to-dalton-mass-tolerance-conversion

## Summary

Converts an MS1 m/z tolerance expressed in ppm (e.g. 20 ppm) into an absolute Dalton window (e.g. +/-0.008 Da at 400 Da) and quantifies the resulting candidate-molecule search-space reduction against a 1 Da window in a compound database such as PubChem.

## When to use

Use when setting precursor-mass search tolerances for tandem MS annotation (e.g. ChemDistiller's --delta_mz) or when quantifying how much a high-accuracy ppm window narrows database candidate lists.

## When NOT to use

- The analysis requires MS2-level discrimination (fragmentation patterns, fingerprints, FingerScorer SVM scoring) rather than precursor-mass pre-filtering of candidates
- The MS1 data are low-resolution so that a 20 ppm window is unrealistic; the delta_mz tolerance should be set to match the instrument's actual mass accuracy
- The comparison database differs fundamentally in composition from PubChem, so the absolute counts (250,079 / 3,229) and the ~77.5x ratio would not transfer

## Inputs

- Monoisotopic molecular-mass array for all compounds in a chemical database (e.g. PubChem dump loaded via chemdistiller.chemdb.database)
- Target precursor mass in Da (e.g. 400 Da)
- MS1 m/z tolerance in ppm (e.g. 20 ppm) and a comparison tolerance in Da (e.g. 1 Da)

## Outputs

- Counts of candidate molecules within the 1 Da window and within the 20 ppm window (e.g. 250,079 vs 3,229)
- Search-space reduction ratio (~77.5x)
- mass_window_candidate_counts.csv containing the computed counts and ratio

## How to apply

Load a monoisotopic molecular-mass dump of the compound database (e.g. PubChem) with NumPy. For a target mass (e.g. 400 Da), count compounds within a 1 Da window (400.0 +/- 0.5 Da) and within a ppm window computed as mass * ppm * 1e-6 (400.0 +/- 400*20e-6 = +/-0.008 Da). Compute the ratio of the two counts (reported ~77.5x reduction: 250,079 vs 3,229 candidates for PubChem at 400 Da). The rationale is that ChemDistiller filters database candidates to those within +/- delta_mz (in ppm) of the MS1 peak m/z, so the ppm-to-Da conversion determines the effective absolute search width at any given mass; the ratio is mass-dependent because a fixed ppm window shrinks in Dalton terms at low mass.

## Related tools

- **NumPy** (Loads the PubChem compound-mass dump and performs windowed mass filtering (+/- 0.5 Da vs +/- 0.008 Da))
- **ChemDistiller** (Annotation engine whose candidate filtering applies the ppm tolerance (--delta_mz) against pre-calculated compound databases) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **chemdistiller.filters.formula** (Formula-filter module used in the candidate-filtering pipeline the ppm window feeds into) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **chemdistiller.chemdb.database** (Database module providing the compound records (with monoisotopic masses) that the mass windows are evaluated against) — https://bitbucket.org/iAnalytica/chemdistillerpython

## Examples

```
python annotate.py Input_Spectra_folder output_folder --ncpu 5 --delta_mz 20 --max_results 100
```

## Evaluation signals

- The 1 Da window count at 400 Da in PubChem matches ~250,079 candidates
- The 20 ppm window count at 400 Da matches ~3,229 candidates (window half-width 400*20e-6 = 0.008 Da)
- The ratio of the two counts equals approximately 77.5
- The computed counts and ratio are written to mass_window_candidate_counts.csv
- In ChemDistiller runs, candidates returned for an MS1 peak lie within +/- delta_mz ppm of the peak m/z

## Limitations

- The ppm-to-Da window width and the search-space reduction ratio are mass-dependent: a fixed ppm tolerance yields a smaller absolute window at lower masses, so the ~77.5x figure holds specifically for a 400 Da compound in PubChem
- ChemDistiller ships with only a small set of compound databases and simple SVM models due to size limitations; full-scale counts require downloading the large HDF5 (*.h5) databases separately
- The generated compound databases carry licensing caveats from their original sources and are provided AS IS, so commercial use requires checking the source databases' disclaimers

## Evidence

- [readme] ppm tolerance as candidate filter: [evidence span withheld — anchor status: not_found]
- [intro] NumPy dependency: [evidence span withheld — anchor status: not_found]
- [abstract] database scale: "combines automated large-scale annotation of metabolites using tandem MS data with a compiled database containing tens of millions of compounds with pre-calculated 'fingerprints'"
