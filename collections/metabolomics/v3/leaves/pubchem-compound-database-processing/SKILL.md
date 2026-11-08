---
name: pubchem-compound-database-processing
description: Use when you need to measure or justify database pruning for MS1 candidate
  retrieval in PubChem, e.g. comparing ppm-scale mass tolerances (--delta_mz) against
  absolute Da windows before annotation.
when_to_use_negative:
- Input is already an annotated tandem MS spectrum or feature table — use the full
  ChemDistiller annotation pipeline (annotate.py) with scoring/SVM fingerprint scoring
  instead of raw mass-window counting.
- The instrument's MS1 accuracy is too low for ppm-scale tolerance (poor calibration),
  where a wide 1 Da window is the realistic operating point and the 'reduction' is
  not meaningful.
- You need formula-level or element-level candidate filtering beyond mass (chemdistiller.filters.formula
  / filters.element), which this mass-window skill does not perform.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3473
tools:
- name: ChemDistiller
  role: Annotation engine whose MS1 candidate search uses the ppm m/z tolerance quantified
    by this skill; its chemdb.database and filters.formula modules hold the compound
    database and formula filters
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: NumPy
  role: Loads and vectorizes the PubChem compound-mass dump for windowed selection
    and counting
- name: Python 2 and 3
  role: Runtime for ChemDistiller and the NumPy-based mass-window analysis (64-bit
    recommended)
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:68d8d6f647f6a9741ff75995088af57dab64d466b52c3aecc84c5ed677ac7949
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDistiller
  - NumPy
  - Python 2 and 3
  - chemdistiller.chemdb.database
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# pubchem-compound-database-processing

## Summary

Loads a PubChem compound monoisotopic-mass dump with NumPy and quantifies how much a high-accuracy 20 ppm m/z window (+/-0.008 Da at 400 Da) shrinks the candidate compound search space versus a 1 Da window, as used upstream of ChemDistiller's tandem-MS annotation engine.

## When to use

Use when you need to measure or justify database pruning for MS1 candidate retrieval in PubChem, e.g. comparing ppm-scale mass tolerances (--delta_mz) against absolute Da windows before annotation.

## When NOT to use

- Input is already an annotated tandem MS spectrum or feature table — use the full ChemDistiller annotation pipeline (annotate.py) with scoring/SVM fingerprint scoring instead of raw mass-window counting.
- The instrument's MS1 accuracy is too low for ppm-scale tolerance (poor calibration), where a wide 1 Da window is the realistic operating point and the 'reduction' is not meaningful.
- You need formula-level or element-level candidate filtering beyond mass (chemdistiller.filters.formula / filters.element), which this mass-window skill does not perform.

## Inputs

- PubChem compound monoisotopic molecular mass dump (all compounds, NumPy-loadable array)
- Target MS1 peak m/z value (e.g. 400 Da)
- ppm mass tolerance (e.g. 20 ppm) and/or absolute mass window (e.g. 1 Da)

## Outputs

- Candidate compound counts per window type (1 Da window vs ppm window)
- Search-space reduction ratio (1 Da count / ppm count)
- mass_window_candidate_counts.csv summary table

## How to apply

Load the PubChem compound-mass dump (monoisotopic molecular masses for all compounds) into NumPy, as used by chemdistiller.chemdb.database for its pre-compiled compound databases. Select compounds whose mass falls within a 1 Da window centered at the target mass (400.0 +/- 0.5 Da) and count them; then repeat with a ppm-scaled window computed as +/- mass*delta_mz*1e-6 (400 Da at 20 ppm gives +/- 0.008 Da). The ppm window is the same tolerance ChemDistiller exposes as the --delta_mz argument (in ppm) for MS1 peak candidate search, so the ratio is directly interpretable as the search-space cost of MS1 tolerance. Compute the ratio of 1 Da-window count to ppm-window count (expected ~77.5x: 250,079 vs 3,229 candidates for a 400 Da compound in PubChem) and write counts and ratio to mass_window_candidate_counts.csv for downstream annotation-budget planning.

## Related tools

- **ChemDistiller** (Annotation engine whose MS1 candidate search uses the ppm m/z tolerance quantified by this skill; its chemdb.database and filters.formula modules hold the compound database and formula filters) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **NumPy** (Loads and vectorizes the PubChem compound-mass dump for windowed selection and counting)
- **Python 2 and 3** (Runtime for ChemDistiller and the NumPy-based mass-window analysis (64-bit recommended)) — https://bitbucket.org/iAnalytica/chemdistillerpython

## Examples

```
python annotate.py Input_Spectra_folder [output_folder] --ncpu 5 --delta_mz 20 --max_results 100
```

## Evaluation signals

- 1 Da window centered at 400 Da must yield 250,079 PubChem candidate molecules
- 20 ppm window centered at 400 Da must yield 3,229 candidates
- Computed count ratio must be approximately 77.5 (250079 / 3229)
- Window bounds are exact: 400.0 +/- 0.5 Da for the 1 Da window and 400.0 +/- 0.008 Da (= 400*20e-6) for the 20 ppm window
- Output file mass_window_candidate_counts.csv contains both window counts and the reduction ratio

## Limitations

- Counts depend on the exact PubChem database version and its monoisotopic mass dump; re-running against a different database release will change the absolute candidate counts and the ratio.
- The 20 ppm window width scales with the target mass (+/- 0.008 Da at 400 Da), so the reduction factor is mass-dependent and cannot be extrapolated unchanged to other m/z values.
- ChemDistiller's shipped compound databases are size-limited subsets; full-scale PubChem databases must be downloaded separately (HDF5 *.h5 format) and licensing of derivative databases must be checked for commercial use.

## Evidence

- [abstract] ChemDistiller pairs large-scale annotation with a massive compound database whose MS1 tolerance is ppm-scaled: "combines automated large-scale annotation of metabolites using tandem MS data with a compiled database containing tens of millions of compounds with pre-calculated 'fingerprints' and fragmentation"
- [intro] ChemDistiller requires SciPy, NumPy and h5py: "requires _SciPy_, _NumPy_, _h5py_ libraries."
- [readme] README defines --delta_mz as the ppm tolerance for MS1 candidate search: [evidence span withheld — anchor status: external_doc]
