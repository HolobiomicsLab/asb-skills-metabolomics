---
name: tandem-ms-spectral-library-annotation
description: Use when tandem MS spectra must be annotated against a small or tens-of-millions-compound
  reference database, and candidates must be ranked and scored with fingerprint and
  fragprint evidence, optionally with isotope/formula filters.
when_to_use_negative:
- 'Your spectra are in vendor or standard exchange formats (mzML, etc.): at the time
  of v0.1 they must be converted to ChemDistiller''s internal hierarchical format
  first, since conversions were not yet implemented'
- You need peak detection or feature detection on raw profile data — ChemDistiller
  is an annotation engine for already-processed tandem MS spectra
- The spectra lack MS1 precursor m/z information, since candidate search requires
  candidates within +/- delta_mz ppm of the MS1 peak m/z
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ChemDistiller
  role: Core annotation engine combining FingerScorer and FragScorer scores to rank
    compound candidates from tandem MS spectra
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: annotate.py
  role: Command-line entry point for running annotation; accepts --delta_mz, --ncpu,
    --max_results and --svm_folder arguments
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: RDKit
  role: Optional dependency used to generate images of 2D chemical structures of compounds
    for the HTML report
  repo: http://www.rdkit.org/
- name: h5py
  role: Required library for loading the HDF5-format chemical compound databases
  repo: http://www.h5py.org/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:2534d045459f82b6cdf27025594cadc9ebdee7cc057d66448a88245530cd9f41
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDistiller
  - NumPy
  - Python 2 and 3
  - RDKit
  - SciPy
  - annotate.py
  - h5py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# tandem-ms-spectral-library-annotation

## Summary

Annotate tandem MS spectra by scoring candidate metabolites against large chemical compound databases using ChemDistiller's FingerScorer (SVM-based fingerprint matching) and FragScorer (fragmentation-pattern matching) evidence combined. Enables retrieval-rate benchmarking (top-1/top-5/top-20) against reference spectra with known structures.

## When to use

Use when tandem MS spectra must be annotated against a small or tens-of-millions-compound reference database, and candidates must be ranked and scored with fingerprint and fragprint evidence, optionally with isotope/formula filters.

## When NOT to use

- Your spectra are in vendor or standard exchange formats (mzML, etc.): at the time of v0.1 they must be converted to ChemDistiller's internal hierarchical format first, since conversions were not yet implemented
- You need peak detection or feature detection on raw profile data — ChemDistiller is an annotation engine for already-processed tandem MS spectra
- The spectra lack MS1 precursor m/z information, since candidate search requires candidates within +/- delta_mz ppm of the MS1 peak m/z

## Inputs

- Tandem MS spectra in ChemDistiller internal hierarchical text format (see int_format.md)
- Compound database in HDF5 format (*.h5) with pre-calculated fingerprints and fragmentation patterns
- SVM model folders ('1' and '-1' subfolders) for FingerScorer, e.g. Linear_weighted or Radial_weighted
- Reference spectra with known structures (e.g. MassBank) for computing retrieval statistics
- Known precursor molecular formulas (optional, for the +Known formula condition via chemdistiller.filters.formula)

## Outputs

- annotated_spectra.json — JSON formatted annotations per spectrum
- annotated_spectra folder — individual annotated spectra in internal hierarchical text format
- log.txt — log of the current run duplicating console output
- Report folder with index.html — HTML annotation report with 2D chemical structure images (requires RDKit)
- Retrieval statistics (top-1/top-5 for small database, top-20 for large database) when spectra have known structures

## How to apply

Install Anaconda Python (2 or 3, 64-bit) with SciPy, NumPy and h5py (conda install -c anaconda h5py); RDKit is optional and only used for rendering 2D structure images in reports. Download compound databases in HDF5 format (*.h5) into the .../ChemDistiller/DBs/ subfolder and the preferred SVMs (Linear_weighted default, or Radial_weighted, which is larger/slower but more predictive) into .../ChemDistiller/SVMs/, then place the input spectra (in ChemDistiller's internal hierarchical format, see int_format.md) in a folder. Run annotate.py with the input folder, output folder, --ncpu (e.g. 5), --delta_mz tolerance for MS1 m/z in ppm (e.g. 20), and --max_results raised above the default 10 when using large compound databases so candidate lists are long enough for top-20 retrieval evaluation. ChemDistiller combines FingerScorer SVM predictions with FragScorer fragprint evidence and applies filters such as element/isotope signatures or known-formula restriction (chemdistiller.filters.element, chemdistiller.filters.formula). First validate the setup with python annotate.py --test, which annotates 863 MassBank spectra and prints retrieval statistics since these compounds have known structures.

## Related tools

- **ChemDistiller** (Core annotation engine combining FingerScorer and FragScorer scores to rank compound candidates from tandem MS spectra) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **annotate.py** (Command-line entry point for running annotation; accepts --delta_mz, --ncpu, --max_results and --svm_folder arguments) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **RDKit** (Optional dependency used to generate images of 2D chemical structures of compounds for the HTML report) — http://www.rdkit.org/
- **h5py** (Required library for loading the HDF5-format chemical compound databases) — http://www.h5py.org/

## Examples

```
python annotate.py Input_Spectra_folder [output_folder] --ncpu 5 --delta_mz 20 --max_results 100
```

## Evaluation signals

- Running python annotate.py --test completes and reports retrieval statistics for the 863 provided MassBank test spectra within a few minutes, writing results to ../ChemDistiller/testlogs/ in a date/time-named subfolder
- The output directory contains all four expected artefacts: annotated_spectra.json, log.txt, an annotated_spectra folder, and a Report folder with index.html
- For a small reference database of known compounds, expected top-5 retrieval with the combined FingerScorer+FragScorer is approximately 86% (positive mode) and 78% (negative mode), versus 80%/71.5% for FingerScorer alone and 82.5%/76.5% for FragScorer alone
- Every returned candidate compound lies within +/- delta_mz ppm of the input MS1 peak m/z value
- The number of ranked candidates per MS1 peak never exceeds --max_results

## Limitations

- Version 0.1 supports only ChemDistiller's internal hierarchical spectral format; conversions from other formats were not yet available at the time of release
- The default distribution ships only a small set of compound databases and simple SVMs due to size limitations; full-scale analysis requires downloading large HDF5 databases and Radial_weighted SVMs, which are larger and slower but more predictive than the default Linear_weighted SVMs
- Derived compound databases generated from open chemical databases may carry individual commercial-use restrictions; consult the readme.txt in the database download and the legal disclaimers of source databases before commercial use
- Performance figures are tied to spectra of compounds with known structures; retrieval statistics cannot be computed for compounds of unknown identity

## Evidence

- [abstract] Here, we introduce ChemDistiller, a customizable engine that combines automated large-scale annotation of metabolites using tandem MS data: "Here, we introduce ChemDistiller, a customizable engine that combines automated large-scale annotation of metabolites using tandem MS data"
- [intro] python annotate.py --help: "Try running __ChemDistiller__ from the command line: python annotate.py --help"
- [methods] chemdistiller.chemdb.database module: "chemdistiller.chemdb.database module"
- [readme] This will initiate a test annotation run using default settings and the test data provided in .../ChemDistiller/data/MassBankTestSpectra/*.: [evidence span withheld — anchor status: external_doc]
- [readme] candidates should be withing +/- delta_mz from MS1 peak m/z value (delta_mz is in ppm): [evidence span withheld — anchor status: external_doc]
- [readme] __Compound Databases__ in HDF5 format (*.h5): [evidence span withheld — anchor status: external_doc]
