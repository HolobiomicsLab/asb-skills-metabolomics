---
name: compound-database-mz-window-retrieval
description: Use when a tandem-MS metabolite annotation pipeline needs a candidate
  compound list per MS1 precursor peak from a pre-built database (e.g. HMDB, ChEBI,
  MassBank, NIST14 in HDF5), before any MS2 scoring.
when_to_use_negative:
- The input is already an MS2-level scoring problem (FingerScorer/SVM scoring of candidates)
  — this skill only performs the mass-window candidate retrieval stage.
- The compound data is not a pre-built HDF5 database with pre-calculated fingerprints
  and fragmentation patterns; rebuilding fingerprints/fragmentation is a separate
  database-compilation step, not part of this retrieval skill.
- Only isotope/adduct assignment or peak annotation of spectra is needed (chemdistiller.annotations.peakannotator),
  with no database search.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: ChemDistiller
  role: The annotation engine whose first filter stage implements the m/z-window candidate
    retrieval from the compound database
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: h5py
  role: Reads the HDF5 (*.h5) compound database containing pre-calculated fingerprints
    and fragmentation patterns
  repo: http://www.h5py.org/
- name: chemdb.database / chemdb.manager
  role: chemdistiller.chemdb submodules that expose the stored compounds and perform
    exact-mass window queries
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: NumPy
  role: Computes the 20 ppm tolerance window (delta_mz = m/z * 20 / 1e6) and aggregates
    candidate counts
- name: Python 2 and 3
  role: Runtime environment for ChemDistiller (64-bit recommended)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:3937573d53475e87d690ad9c1b863c911c6cea05e888abf34d0ce9c06be9f633
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDistiller
  - NumPy
  - Python 2 and 3
  - chemdb.database / chemdb.manager
  - h5py
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# compound-database-mz-window-retrieval

## Summary

Reconstructs ChemDistiller's first (MS1-level) filter stage: retrieving candidate compounds from a pre-built HDF5 compound database by exact-mass window around the precursor m/z, which seeds all downstream tandem-MS scoring.

## When to use

Use when a tandem-MS metabolite annotation pipeline needs a candidate compound list per MS1 precursor peak from a pre-built database (e.g. HMDB, ChEBI, MassBank, NIST14 in HDF5), before any MS2 scoring.

## When NOT to use

- The input is already an MS2-level scoring problem (FingerScorer/SVM scoring of candidates) — this skill only performs the mass-window candidate retrieval stage.
- The compound data is not a pre-built HDF5 database with pre-calculated fingerprints and fragmentation patterns; rebuilding fingerprints/fragmentation is a separate database-compilation step, not part of this retrieval skill.
- Only isotope/adduct assignment or peak annotation of spectra is needed (chemdistiller.annotations.peakannotator), with no database search.

## Inputs

- MS1 precursor m/z values for the peaks to annotate (from spectra in ChemDistiller internal hierarchical format)
- Pre-built chemical compound database in HDF5 format (*.h5) with pre-calculated fingerprints and fragmentation patterns, e.g. HMDB, ChEBI, MassBank, NIST14
- m/z tolerance in ppm (default delta_mz = 20)

## Outputs

- Per-query candidate compound lists (compounds whose exact mass lies within +/- delta_mz of the MS1 peak m/z, retaining fingerprint and fragmentation records)
- candidate_counts.csv with per-peak and mean candidates per MS1 peak (expected ~24 for the ~130k-compound small database, ~10000 for the large database)

## How to apply

Load the HDF5 compound database via h5py through the chemdistiller.chemdb.database module; entries carry pre-calculated fingerprints and fragmentation patterns. For each query MS1 precursor m/z, compute the tolerance window delta_mz = m/z * 20 / 1e6 (20 ppm, the annotate.py default) and use chemdistiller.chemdb.manager to pull every compound whose exact mass falls inside the window, keeping fingerprint and fragmentation records. Aggregate retrieved candidate counts across a set of MS1 peaks to characterize retrieval volume: the paper reports ~24 candidates per MS1 peak for the small database (HMDB, ChEBI, MassBank, NIST14, ~130 thousand compounds) and ~10000 for the large tens-of-millions-compound build. When running the full engine, cap the list with --max_results (default 10, increased for large databases) since this candidate set feeds the downstream FingerScorer/SVM scoring stage.

## Related tools

- **ChemDistiller** (The annotation engine whose first filter stage implements the m/z-window candidate retrieval from the compound database) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **h5py** (Reads the HDF5 (*.h5) compound database containing pre-calculated fingerprints and fragmentation patterns) — http://www.h5py.org/
- **chemdb.database / chemdb.manager** (chemdistiller.chemdb submodules that expose the stored compounds and perform exact-mass window queries) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **NumPy** (Computes the 20 ppm tolerance window (delta_mz = m/z * 20 / 1e6) and aggregates candidate counts)
- **Python 2 and 3** (Runtime environment for ChemDistiller (64-bit recommended))

## Examples

```
python annotate.py Input_Spectra_folder [output_folder] --ncpu 5 --delta_mz 20 --max_results 100
```

## Evaluation signals

- Mean candidates per MS1 peak is ~24 for the small database build (HMDB, ChEBI, MassBank, NIST14; ~130 thousand compounds) — values far from this indicate a wrong ppm window or database load failure.
- Every retrieved candidate's exact mass falls within +/- delta_mz of the query MS1 peak m/z (verify delta_mz = m/z * 20 / 1e6 per query).
- Retrieved records carry non-empty fingerprint and fragmentation-pattern fields from the HDF5 database.
- Count summary matches the reported large-database regime (~10000 candidates per peak) when the tens-of-millions-compound database is used.
- candidate_counts.csv contains one row per queried MS1 peak with consistent per-peak and mean counts.

## Limitations

- The candidate count grows with database size — the large tens-of-millions-compound database yields ~10000 candidates per MS1 peak, so --max_results (default 10) must be raised for large databases.
- Only a small set of chemical compound databases and simple SVM models ships with the default install; larger HDF5 databases must be downloaded separately into .../ChemDistiller/DBs/.
- Spectra must be converted into ChemDistiller's internal hierarchical format; conversion from other formats was not yet implemented at the time of the README.

## Evidence

- [readme] delta_mz tolerance controls retrieval width: [evidence span withheld — anchor status: not_found]
- [abstract] HDF5 databases with pre-calculated fingerprints and fragmentation: "a compiled database containing tens of millions of compounds with pre-calculated 'fingerprints' and fragmentation"
- [intro] h5py dependency for the compound database: "__ChemDistiller__ supports Python 2 and 3 (64-bit version recommended) and requires _SciPy_, _NumPy_, _h5py_ libraries."
- [methods] chemdb modules implementing the database retrieval: "chemdistiller.chemdb.manager module
-----------------------------------"
- [readme] candidate count is limited by --max_results: [evidence span withheld — anchor status: not_found]
