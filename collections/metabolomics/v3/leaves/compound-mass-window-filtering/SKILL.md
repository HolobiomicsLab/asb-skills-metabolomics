---
name: compound-mass-window-filtering
description: Use when retrieving candidate molecules from a large chemical database
  (e.g. PubChem) by precursor m/z and you need to quantify or exploit how a high-accuracy
  ppm mass window narrows the candidate list.
when_to_use_negative:
- When precursor masses are low-accuracy (ppm-level windows are meaningless; fall
  back to a coarse Da window or no m/z filter)
- When you need structural or MS2 spectral candidates rather than mass-based candidate
  lists — this skill only filters by monoisotopic molecular mass and does not score
  tandem MS spectra or fingerprints
- When the compound database lacks pre-calculated monoisotopic masses and must be
  mass-annotated first
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3331
tools:
- name: ChemDistiller
  role: Annotation engine whose --delta_mz (ppm) MS1 candidate filter this skill reproduces;
    supplies chemdb.database and chemdistiller.filters.formula for candidate retrieval
    and formula-level filtering
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: NumPy
  role: Vectorized selection and counting of database compounds whose monoisotopic
    masses fall within the specified Da/ppm window
- name: Python 2 and 3
  role: Runtime for the window-selection script and for ChemDistiller itself (64-bit
    recommended)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:50695d989b2075a754ecd9893a275b6823f1acde5c68e2c79bbcdb4c3c92d4e6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDistiller
  - NumPy
  - Python 2 and 3
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3331
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# compound-mass-window-filtering

## Summary

Filters compound-database candidates by monoisotopic molecular mass using a narrow ppm-bounded m/z window (e.g. ±20 ppm = ±0.008 Da at 400 Da) instead of a coarse 1 Da window, quantifying the resulting search-space reduction (~77.5x in PubChem at 400 Da) before downstream tandem MS annotation.

## When to use

Use when retrieving candidate molecules from a large chemical database (e.g. PubChem) by precursor m/z and you need to quantify or exploit how a high-accuracy ppm mass window narrows the candidate list.

## When NOT to use

- When precursor masses are low-accuracy (ppm-level windows are meaningless; fall back to a coarse Da window or no m/z filter)
- When you need structural or MS2 spectral candidates rather than mass-based candidate lists — this skill only filters by monoisotopic molecular mass and does not score tandem MS spectra or fingerprints
- When the compound database lacks pre-calculated monoisotopic masses and must be mass-annotated first

## Inputs

- PubChem compound-mass dump: monoisotopic molecular masses for all compounds, loadable as a NumPy array
- Target precursor m/z value (e.g. 400.0 Da) and mass-window specification (ppm value such as 20 ppm, and/or a nominal window such as 1 Da)

## Outputs

- Count of candidate compounds within the coarse mass window (reported: 250,079 at 400 ± 0.5 Da)
- Count of candidate compounds within the ppm mass window (reported: 3,229 at 400 ± 0.008 Da)
- Search-space reduction ratio (reported: ~77.5x)
- CSV file (mass_window_candidate_counts.csv) recording window definitions, counts, and ratio

## How to apply

Load a pre-computed database dump of monoisotopic molecular masses (e.g. the PubChem mass dump used by ChemDistiller) as a NumPy array. Select compounds within a 1 Da window centered at the target mass (400.0 ± 0.5 Da) and count them (reported: 250,079), then select compounds within a 20 ppm window (400.0 ± 400*20e-6 = ±0.008 Da) and count them (reported: 3,229). Compute the ratio of the two counts to quantify the search-space reduction (~77.5x). This mirrors how ChemDistiller's --delta_mz parameter in ppm constrains MS1 candidates before MS2 scoring, formula filters (chemdistiller.filters.formula), and SVM fingerprint scoring. Note the window width scales with mass because delta_mz is specified in ppm, so the absolute Da tolerance must be recomputed for each target m/z. Write the per-window counts and ratio to a CSV (e.g. mass_window_candidate_counts.csv) for traceability.

## Related tools

- **ChemDistiller** (Annotation engine whose --delta_mz (ppm) MS1 candidate filter this skill reproduces; supplies chemdb.database and chemdistiller.filters.formula for candidate retrieval and formula-level filtering) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **NumPy** (Vectorized selection and counting of database compounds whose monoisotopic masses fall within the specified Da/ppm window)
- **Python 2 and 3** (Runtime for the window-selection script and for ChemDistiller itself (64-bit recommended))

## Examples

```
python annotate.py Input_Spectra_folder [output_folder] --ncpu 5 --delta_mz 20 --max_results 100
```

## Evaluation signals

- 1 Da window count centered at 400 Da equals 250,079 candidates
- 20 ppm window count centered at 400 Da equals 3,229 candidates
- Ratio of the two counts is ~77.5 (250,079 / 3,229)
- Window bounds are symmetric and correctly mass-scaled: 20 ppm at 400 Da corresponds to ±0.008 Da, not ±0.008 fixed across masses
- All selected candidates' masses satisfy |mass - 400.0| <= delta (assert on the boolean mask)

## Limitations

- The ppm window's absolute width scales with mass (±20 ppm = ±0.008 Da at 400 Da), so reduction ratios measured at one mass do not transfer directly to other masses
- Counts depend on the specific database snapshot (PubChem grows over time), so exact numbers like 250,079 and 3,229 may drift with database version
- Full-scale ChemDistiller databases and SVMs are not bundled (size limitations) and must be downloaded separately as HDF5 (*.h5) files; input spectra must be converted to ChemDistiller's internal format
- Provided databases 'are provided AS IS' with licensing caveats inherited from the original compound databases, which may restrict commercial use

## Evidence

- [readme] MS1 ppm tolerance parameter: [evidence span withheld — anchor status: external_doc]
- [abstract] Large pre-computed candidate database rationale: "combines automated large-scale annotation of metabolites using tandem MS data with a compiled database containing tens of millions of compounds"
- [intro] NumPy dependency for the window computation: "requires _SciPy_, _NumPy_, _h5py_ libraries."
- [intro] Engine purpose context: "__ChemDistiller__ is the high-throughput annotation engine for tandem MS spectra annotation."
