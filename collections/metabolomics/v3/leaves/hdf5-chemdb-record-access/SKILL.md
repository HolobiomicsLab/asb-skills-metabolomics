---
name: hdf5-chemdb-record-access
description: Use when you need to query ChemDistiller's pre-compiled compound databases
  in HDF5 (*.h5) format for candidate compounds matching an MS1 precursor m/z within
  a ppm tolerance, rather than rebuilding formula/fragment predictions from scratch.
when_to_use_negative:
- When the input spectra are not yet converted to ChemDistiller's internal hierarchical
  format (conversions from other formats were not yet implemented at the time of the
  README)
- When you need de novo formula prediction or candidate generation without a pre-built
  compound database
- When a full end-to-end annotation (scoring, FingerScorer SVM prediction, report
  generation) is required rather than raw candidate retrieval
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: ChemDistiller
  role: Annotation engine whose chemdb subpackage (database and manager modules) implements
    the HDF5 compound database and exact-mass candidate retrieval
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: h5py
  role: Library used to read the *.h5 compound database files; installed in Anaconda
    via conda install -c anaconda h5py
- name: NumPy
  role: Numerical library required by ChemDistiller, used for m/z tolerance-window
    arithmetic on exact masses
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:9ff02cf7c02eea786bf5f4d19d34107308da21545fbbbebc6bbdb52e27f5373f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDistiller
  - NumPy
  - Python 2 and 3
  - h5py
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# hdf5-chemdb-record-access

## Summary

Access ChemDistiller's pre-built HDF5 compound databases (compounds with pre-calculated fingerprints and fragmentation patterns) via h5py and the chemdistiller.chemdb.database/manager modules, retrieving candidate records by exact mass within an m/z tolerance window (e.g. 20 ppm delta_mz).

## When to use

Use when you need to query ChemDistiller's pre-compiled compound databases in HDF5 (*.h5) format for candidate compounds matching an MS1 precursor m/z within a ppm tolerance, rather than rebuilding formula/fragment predictions from scratch.

## When NOT to use

- When the input spectra are not yet converted to ChemDistiller's internal hierarchical format (conversions from other formats were not yet implemented at the time of the README)
- When you need de novo formula prediction or candidate generation without a pre-built compound database
- When a full end-to-end annotation (scoring, FingerScorer SVM prediction, report generation) is required rather than raw candidate retrieval

## Inputs

- MS1 precursor m/z values (query peaks from tandem MS spectra in ChemDistiller internal hierarchical text format)
- ChemDistiller compound database in HDF5 format (*.h5), e.g. built from HMDB, ChEBI, MassBank and NIST14 (~130 thousand compounds for the small build)
- ppm m/z tolerance (delta_mz) and max_results limits

## Outputs

- Candidate compound records matching the m/z window, each with pre-calculated fingerprints and fragmentation patterns
- Per-peak candidate lists and candidate count statistics (mean candidates per MS1 peak)

## How to apply

Load the HDF5 database file with h5py (installed via `conda install -c anaconda h5py`) through the chemdistiller.chemdb.database module, which stores compounds with pre-calculated fingerprints and fragmentation patterns; databases downloaded from the project's DBs folder are unpacked into .../ChemDistiller/DBs/ and picked up automatically at runtime. For a query MS1 precursor m/z, define the tolerance window as delta_mz in ppm (e.g. --delta_mz 20 gives +/- 20 ppm around the MS1 peak m/z) and retrieve all compounds whose exact mass falls inside it via chemdistiller.chemdb.manager, keeping each record's fingerprint and fragmentation data. Optionally cap the number of returned candidates per MS1 peak with --max_results (default 10; increase it for large compound databases, where ~10000 candidates per peak are expected versus ~24 for the small database). Access is read-only record retrieval feeding downstream annotation scoring, so the rationale is speed: exact-mass lookup against pre-calculated records replaces on-the-fly candidate enumeration.

## Related tools

- **ChemDistiller** (Annotation engine whose chemdb subpackage (database and manager modules) implements the HDF5 compound database and exact-mass candidate retrieval) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **h5py** (Library used to read the *.h5 compound database files; installed in Anaconda via conda install -c anaconda h5py)
- **NumPy** (Numerical library required by ChemDistiller, used for m/z tolerance-window arithmetic on exact masses)

## Examples

```
python annotate.py Input_Spectra_folder [output_folder] --ncpu 5 --delta_mz 20 --max_results 100
```

## Evaluation signals

- Retrieved candidates all fall within the specified ppm window: |candidate exact mass - MS1 m/z| <= m/z * delta_mz / 1e6
- For the small database (HMDB, ChEBI, MassBank, NIST14; ~130 thousand compounds), mean candidates per MS1 peak is ~24; ~10000 for the large database — large deviations indicate a wrong database or tolerance
- Each returned record carries fingerprint and fragmentation pattern data (not just a formula/mass), confirming the pre-calculated HDF5 records were used
- The test mode run (python annotate.py --test) on the 863 MassBank test spectra completes and retrieval statistics are printed
- Respecting --max_results caps the number of candidates per MS1 peak (default 10) when limiting is enabled

## Limitations

- Candidate count scales with database size: the large database yields ~10000 candidates per MS1 peak, so --max_results limiting is advised; more complex SVMs (Radial_weighted) are larger and slower
- Databases are derivatives of source databases with possibly restricted commercial use; consult readme.txt in the DBs distribution and the source databases' legal disclaimers before commercial use
- At the time of the README, spectra had to be in ChemDistiller's internal hierarchical format; conversions from other formats were not yet implemented

## Evidence

- [abstract] combines automated large-scale annotation of metabolites using tandem MS data with a compiled database containing tens of millions of compounds with pre-calculated 'fingerprints' and fragmentation: "combines automated large-scale annotation of metabolites using tandem MS data with a compiled database containing tens of millions of compounds with pre-calculated 'fingerprints' and fragmentation"
- [methods] chemdb.database module: "chemdistiller.chemdb.database module
------------------------------------"
- [readme] delta_mz tolerance semantics: [evidence span withheld — anchor status: external_doc]
- [readme] max_results guidance: [evidence span withheld — anchor status: external_doc]
- [readme] h5py installation: "[h5py](http://www.h5py.org/) can be installed in [Anaconda](https://www.anaconda.com/download/) environment by calling: 

```
conda install -c anaconda h5py 
```"
- [readme] DB deployment: [evidence span withheld — anchor status: external_doc]
