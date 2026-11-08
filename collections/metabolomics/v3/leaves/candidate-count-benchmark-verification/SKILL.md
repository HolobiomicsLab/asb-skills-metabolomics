---
name: candidate-count-benchmark-verification
description: Use when you have an HDF5 ChemDistiller compound DB and a set of MS1
  precursor m/z values, and you need to quantify how many candidates the exact-mass
  filter retrieves per peak (delta_mz in ppm).
when_to_use_negative:
- Input is raw MS2 tandem spectra needing full annotation scoring rather than a first-stage
  candidate count check
- No pre-built ChemDistiller HDF5 compound database is available and fingerprints/fragmentation
  patterns would have to be generated first
- Spectra are not in the ChemDistiller internal hierarchical format and no conversion
  exists yet
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: ChemDistiller
  role: Engine whose chemdb.database and chemdb.manager modules perform the exact-mass
    candidate retrieval from the HDF5 compound database
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: h5py
  role: Library required to open and read the pre-built HDF5 compound database
- name: NumPy
  role: Computes the ppm m/z tolerance windows and aggregates candidate counts
- name: annotate.py
  role: Command-line entry point whose --delta_mz and --max_results parameters control
    the candidate retrieval being benchmarked
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:fae918d735de64aa0a04ea9a883eee7c8857925de7e3f5fcb3af70678a40b956
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDistiller
  - NumPy
  - Python 2 and 3
  - annotate.py
  - h5py
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# candidate-count-benchmark-verification

## Summary

Benchmarks the first ChemDistiller filter stage by counting exact-mass candidate compounds retrieved per MS1 precursor peak from the pre-built HDF5 compound database, checking the retrieved averages (about 24 per peak for the ~130k-compound small database; ~10000 for the large build) against expectation.

## When to use

Use when you have an HDF5 ChemDistiller compound DB and a set of MS1 precursor m/z values, and you need to quantify how many candidates the exact-mass filter retrieves per peak (delta_mz in ppm).

## When NOT to use

- Input is raw MS2 tandem spectra needing full annotation scoring rather than a first-stage candidate count check
- No pre-built ChemDistiller HDF5 compound database is available and fingerprints/fragmentation patterns would have to be generated first
- Spectra are not in the ChemDistiller internal hierarchical format and no conversion exists yet

## Inputs

- ChemDistiller compound database in HDF5 format (*.h5) placed in .../ChemDistiller/DBs/
- MS1 precursor m/z values (one or more peaks, in ChemDistiller internal format or a peak list)
- delta_mz tolerance in ppm (default run uses 20 ppm)
- max_results cap on candidates per MS1 peak (annotate.py default: 10; raise for large databases)

## Outputs

- Per-query candidate compound lists (with fingerprint and fragmentation records)
- Count summary CSV (e.g. candidate_counts.csv) with mean candidates per MS1 peak per database build

## How to apply

Load the HDF5 database (compounds carry pre-calculated fingerprints and fragmentation patterns) with h5py via the chemdistiller.chemdb.database module, then use chemdistiller.chemdb.manager to retrieve all compounds whose exact mass falls within the ppm tolerance window around each MS1 precursor m/z; for the paper's default run the window is delta_mz = 20 ppm. Aggregate retrieved candidate counts over the MS1 peak set and compute the mean candidates per peak for both the small database (HMDB, ChEBI, MassBank, NIST14; ~130 thousand compounds, expected mean ~24) and the large database (expected mean ~10000). The rationale is that the first filter stage determines the downstream annotation load, so verifying per-peak candidate counts validates both the database build and the m/z tolerance setting before any scoring. Write the per-query candidate lists and a count summary (e.g. candidate_counts.csv) so means can be compared against the reported averages.

## Related tools

- **ChemDistiller** (Engine whose chemdb.database and chemdb.manager modules perform the exact-mass candidate retrieval from the HDF5 compound database) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **h5py** (Library required to open and read the pre-built HDF5 compound database)
- **NumPy** (Computes the ppm m/z tolerance windows and aggregates candidate counts)
- **annotate.py** (Command-line entry point whose --delta_mz and --max_results parameters control the candidate retrieval being benchmarked) — https://bitbucket.org/iAnalytica/chemdistillerpython

## Examples

```
python annotate.py Input_Spectra_folder output_folder --delta_mz 20 --max_results 100
```

## Evaluation signals

- Mean candidates per MS1 peak for the small database (HMDB, ChEBI, MassBank, NIST14; ~130k compounds) matches the reported average of ~24
- Mean candidates per MS1 peak for the large database build is on the order of ~10000, within the reported range
- Every retrieved candidate has exact mass within +/- delta_mz ppm of the query MS1 m/z
- Per-query candidate lists retain fingerprint and fragmentation records from the HDF5 store
- Count summary CSV row counts equal the number of queried MS1 peaks

## Limitations

- Candidate counts scale with database size, so the ~24-per-peak benchmark only holds for the small database; --max_results (default 10) must be increased for large compound databases or retrieval is truncated
- Spectra must be in ChemDistiller internal format; format conversion was not yet implemented at v0.1
- Large databases and more complex SVMs must be downloaded separately due to size limitations of the shipped package

## Evidence

- [readme] delta_mz ppm tolerance: [evidence span withheld — anchor status: external_doc]
- [readme] max_results cap on candidates: [evidence span withheld — anchor status: external_doc]
- [abstract] database with pre-calculated fingerprints and fragmentation patterns: "automated large-scale annotation of metabolites using tandem MS data with a compiled database containing tens of millions of compounds"
- [intro] ChemDistiller is the tandem MS annotation engine being benchmarked: "__ChemDistiller__ is the high-throughput annotation engine for tandem MS spectra annotation."
- [readme] required libraries for loading the HDF5 database: "requires _SciPy_, _NumPy_, _h5py_ libraries."
