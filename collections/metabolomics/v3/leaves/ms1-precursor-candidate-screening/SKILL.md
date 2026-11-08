---
name: ms1-precursor-candidate-screening
description: Use when annotating tandem MS spectra and you need the initial MS1 precursor
  m/z candidate screen (e.g. 20 ppm delta_mz window) against a compound database in
  HDF5 format, or when measuring candidates per peak.
when_to_use_negative:
- Input is a full tandem MS/MS spectrum needing scoring or SVM-based fingerprint comparison
  rather than the initial exact-mass candidate retrieval
- Spectra are not yet converted to the ChemDistiller internal hierarchical format,
  which the current version requires (format conversions were still being implemented)
- You need to search raw online chemical databases directly instead of the pre-built
  HDF5 compound databases supplied with ChemDistiller
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ChemDistiller
  role: Annotation engine whose first filter stage performs the MS1 precursor m/z
    candidate screening against compound databases
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: h5py
  role: Python library used to load the HDF5 compound database containing pre-calculated
    fingerprints and fragmentation patterns
- name: NumPy
  role: Used to compute the ppm tolerance window (delta_mz = m/z * 20 / 1e6) around
    the query precursor m/z
- name: chemdistiller.chemdb.manager
  role: Module through which compounds whose exact mass falls inside the tolerance
    window are retrieved
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:e8926e77894a068341943b4efe2873ce76eec1fd90eed147507423661fe61a0e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDistiller
  - NumPy
  - Python 2 and 3
  - chemdistiller.chemdb.manager
  - h5py
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# ms1-precursor-candidate-screening

## Summary

Generates the first-filter candidate compound list for ChemDistiller by retrieving compounds from pre-built HDF5 databases whose exact mass falls within a ppm tolerance of each MS1 precursor m/z, and reports how many candidates are retrieved per peak.

## When to use

Use when annotating tandem MS spectra and you need the initial MS1 precursor m/z candidate screen (e.g. 20 ppm delta_mz window) against a compound database in HDF5 format, or when measuring candidates per peak.

## When NOT to use

- Input is a full tandem MS/MS spectrum needing scoring or SVM-based fingerprint comparison rather than the initial exact-mass candidate retrieval
- Spectra are not yet converted to the ChemDistiller internal hierarchical format, which the current version requires (format conversions were still being implemented)
- You need to search raw online chemical databases directly instead of the pre-built HDF5 compound databases supplied with ChemDistiller

## Inputs

- MS1 precursor m/z values from tandem MS spectra in ChemDistiller internal hierarchical format
- Pre-built compound database in HDF5 format (*.h5) with pre-calculated fingerprints and fragmentation patterns (e.g. HMDB, ChEBI, MassBank, NIST14 small build; larger builds from the provided mediafire DBs folder)
- delta_mz tolerance setting in ppm (e.g. 20 ppm)

## Outputs

- Per-MS1-peak candidate compound lists with fingerprints and fragmentation records
- Mean candidates-per-peak summary (e.g. ~24 for the small database, ~10000 for the large database), e.g. written to candidate_counts.csv

## How to apply

Load the compound database (e.g. HMDB, ChEBI, MassBank, NIST14 for the small ~130-thousand-compound build) in HDF5 format via h5py through the chemdistiller.chemdb.database module; the database stores compounds with pre-calculated fingerprints and fragmentation patterns. For each query MS1 precursor m/z, compute the tolerance window delta_mz = m/z * 20 / 1e6 (20 ppm), matching the --delta_mz command-line parameter of annotate.py. Retrieve via chemdistiller.chemdb.manager all compounds whose exact mass falls inside the window, keeping fingerprint and fragmentation records; optionally cap results with --max_results (default 10, increase for large databases). Aggregate candidate counts over a set of MS1 peaks and compute the mean; the small database yields ~24 candidates per MS1 peak versus ~10000 for the large database, which informs sensible --max_results settings and downstream scoring load.

## Related tools

- **ChemDistiller** (Annotation engine whose first filter stage performs the MS1 precursor m/z candidate screening against compound databases) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **h5py** (Python library used to load the HDF5 compound database containing pre-calculated fingerprints and fragmentation patterns)
- **NumPy** (Used to compute the ppm tolerance window (delta_mz = m/z * 20 / 1e6) around the query precursor m/z)
- **chemdistiller.chemdb.manager** (Module through which compounds whose exact mass falls inside the tolerance window are retrieved) — https://bitbucket.org/iAnalytica/chemdistillerpython

## Examples

```
python annotate.py Input_Spectra_folder output_folder --ncpu 5 --delta_mz 20 --max_results 100
```

## Evaluation signals

- Mean candidates per MS1 peak for the small database (HMDB, ChEBI, MassBank, NIST14; ~130 thousand compounds) is ~24, matching the reported average
- Every retrieved candidate has exact mass within +/- delta_mz ppm (e.g. 20 ppm) of the query MS1 peak m/z
- Retrieved candidates carry fingerprint and fragmentation records consistent with the HDF5 database schema
- annotate.py --test run on the 863 MassBank test spectra in testlogs/ prints retrieval statistics for known compounds
- Candidate counts for the large database are on the order of ~10000 per peak, so --max_results (default 10) must be raised to avoid truncation

## Limitations

- Candidate count grows dramatically with database size (~10000 per peak for the large database), so the default --max_results of 10 truncates candidates and must be increased
- Spectra must be in ChemDistiller's internal hierarchical format; conversions from other formats were not yet implemented at v0.1
- Only a small set of compound databases and simple SVM models ships with v0.1 due to size limitations; larger databases must be downloaded separately
- Derivative databases may inherit commercial-use restrictions from the original source databases; consult readme.txt and legal disclaimers before commercial use

## Evidence

- [readme] --delta_mz - tolerance for the MS1 peak m/z values when searching in the databases, i.e. candidates should be withing +/- delta_mz from MS1 peak m/z value (delta_mz is in ppm): [evidence span withheld — anchor status: not_found]
- [readme] --max_results - limit the number of candidate compounds per MS1 peak to this value. Default: 10, but you may wish to increase it for large compound databases.: [evidence span withheld — anchor status: not_found]
- [methods] chemdistiller.chemdb.manager module ----------------------------------- .. automodule:: chemdistiller.chemdb.manager: "chemdistiller.chemdb.manager module ----------------------------------- .. automodule:: chemdistiller.chemdb.manager"
- [abstract] combines automated large-scale annotation of metabolites using tandem MS data with a compiled database containing tens of millions of compounds: "combines automated large-scale annotation of metabolites using tandem MS data with a compiled database containing tens of millions of compounds"
- [results] For the small database (HMDB, ChEBI, MassBank and NIST14; totaling ~130 thousand compounds), 24 candidates were on average retrieved per single MS1 peak.: [evidence span withheld — anchor status: not_found]
