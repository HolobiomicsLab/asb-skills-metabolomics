---
name: ppm-mass-tolerance-filtering
description: Use when you have MS1 precursor m/z values and a pre-calculated compound
  database (HDF5 with fingerprints/fragmentation patterns), and need to bound the
  candidate set by a ppm m/z tolerance before annotation.
when_to_use_negative:
- You need molecular formula-level candidate enumeration from elemental compositions
  rather than retrieval from a pre-built exact-mass compound database (use chemdistiller.filters.formula/element
  for that)
- Spectra are not yet converted to the ChemDistiller internal hierarchical format
  — the engine currently requires that format for input spectra
- You are doing MS2-only or fragmentation-pattern comparison without an MS1 precursor
  m/z to constrain the search
edam_operation: http://edamontology.org/operation_2421
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ChemDistiller
  role: Annotation engine whose first filter stage performs the ppm m/z candidate
    retrieval; run via annotate.py with --delta_mz
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: h5py
  role: Library used to read the HDF5 (*.h5) compound databases storing exact masses,
    fingerprints and fragmentation patterns
- name: NumPy
  role: Used to compute the ppm tolerance window (delta_mz = m/z * 20 / 1e6) around
    each query MS1 peak
- name: Anaconda
  role: Recommended environment for installing ChemDistiller and its dependencies
    (SciPy, NumPy, h5py)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:0d7c0700049f560c63ba307814f73cea46674d5a2a02231e19e9662b74e5d528
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Anaconda
  - ChemDistiller
  - NumPy
  - Python 2 and 3
  - h5py
  edam_operation: http://edamontology.org/operation_2421
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# ppm-mass-tolerance-filtering

## Summary

First-stage candidate generation in ChemDistiller: retrieve all compounds from a pre-built HDF5 database whose exact masses fall within a ppm tolerance window of the MS1 precursor m/z. This prunes the compound search space before fragmentation-based scoring (e.g. ~24 candidates per MS1 peak from a ~130k-compound small database at 20 ppm).

## When to use

Use when you have MS1 precursor m/z values and a pre-calculated compound database (HDF5 with fingerprints/fragmentation patterns), and need to bound the candidate set by a ppm m/z tolerance before annotation.

## When NOT to use

- You need molecular formula-level candidate enumeration from elemental compositions rather than retrieval from a pre-built exact-mass compound database (use chemdistiller.filters.formula/element for that)
- Spectra are not yet converted to the ChemDistiller internal hierarchical format — the engine currently requires that format for input spectra
- You are doing MS2-only or fragmentation-pattern comparison without an MS1 precursor m/z to constrain the search

## Inputs

- MS1 precursor m/z values (from spectra in ChemDistiller internal hierarchical format)
- Pre-built compound database in HDF5 format (*.h5) with pre-calculated fingerprints and fragmentation patterns, copied to .../ChemDistiller/DBs/
- PPM tolerance setting (e.g. delta_mz = 20)

## Outputs

- Per-MS1-peak candidate compound lists with their fingerprint and fragmentation records
- Candidate count summary (mean candidates per MS1 peak, e.g. ~24 for the small database, ~10000 for the large database), written to candidate_counts.csv

## How to apply

Load the compound database with h5py through chemdistiller.chemdb.database, which stores compounds with pre-calculated fingerprints and fragmentation patterns. For each query MS1 precursor m/z, compute the tolerance window as delta_mz = m/z * ppm / 1e6 (e.g. 20 ppm, the default --delta_mz used by annotate.py), so candidates must lie within +/- delta_mz of the peak's m/z. Retrieve matching compounds via chemdistiller.chemdb.manager, keeping their fingerprint and fragmentation records, and optionally cap the set with --max_results (default 10; increase it for large compound databases). Aggregate candidate counts over a set of MS1 peaks to sanity-check retrieval volume against the reported averages: ~24 candidates per MS1 peak for the small database (HMDB, ChEBI, MassBank, NIST14; ~130 thousand compounds) and ~10000 for the large database. Rationale: exact-mass pre-filtering keeps the per-peak candidate list small enough for downstream fingerprint/fragmentation scoring while remaining sensitive to instrument mass accuracy.

## Related tools

- **ChemDistiller** (Annotation engine whose first filter stage performs the ppm m/z candidate retrieval; run via annotate.py with --delta_mz) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **h5py** (Library used to read the HDF5 (*.h5) compound databases storing exact masses, fingerprints and fragmentation patterns)
- **NumPy** (Used to compute the ppm tolerance window (delta_mz = m/z * 20 / 1e6) around each query MS1 peak)
- **Anaconda** (Recommended environment for installing ChemDistiller and its dependencies (SciPy, NumPy, h5py))

## Examples

```
python annotate.py Input_Spectra_folder [output_folder] --ncpu 5 --delta_mz 20 --max_results 100
```

## Evaluation signals

- Mean candidates per MS1 peak matches the reported scale: ~24 for the small database (HMDB, ChEBI, MassBank, NIST14; ~130 thousand compounds), ~10000 for the large database
- Every retrieved candidate's exact mass falls within +/- delta_mz of the query MS1 peak m/z at the chosen ppm setting (e.g. 20 ppm)
- Per-query candidate lists include fingerprint and fragmentation records for each hit, not just masses
- Candidate list length never exceeds --max_results (default 10; raise for large databases)
- Database files (*.h5) are picked up automatically from .../ChemDistiller/DBs/ and loaded without errors in the run log (log.txt)

## Limitations

- The tolerance is defined around the MS1 peak m/z, so mass accuracy of the instrument directly bounds sensitivity; ppm window too tight may miss true candidates
- Large compound databases yield ~10000 candidates per MS1 peak on average, so --max_results (default 10) must be raised for large database searches
- The distributed small database/SVM set is limited by size; full databases must be downloaded separately and carry licensing caveats from the original compound sources for commercial use
- For the full annotate.py run, only simple Linear_weighted SVMs ship by default; Radial_weighted SVMs are recommended but larger and slower

## Evidence

- [readme] tolerance for the MS1 peak _m/z_ values when searching in the databases, _i.e._ candidates should be withing +/- _delta_mz_ from MS1 peak _m/z_ value (_delta_mz_ is in ppm): [evidence span withheld — anchor status: not_found]
- [readme] python annotate.py Input_Spectra_folder [output_folder] --ncpu 5 --delta_mz 20 --max_results 100: [evidence span withheld — anchor status: external_doc]
- [intro] _ChemDistiller__ supports Python 2 and 3 (64-bit version recommended) and requires _SciPy_, _NumPy_, _h5py_ libraries.: "__ChemDistiller__ supports Python 2 and 3 (64-bit version recommended) and requires _SciPy_, _NumPy_, _h5py_ libraries."
- [readme] Default: 10, but you may wish to increase it for large compound databases.: [evidence span withheld — anchor status: external_doc]
