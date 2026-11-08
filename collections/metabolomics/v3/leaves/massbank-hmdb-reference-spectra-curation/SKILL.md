---
name: massbank-hmdb-reference-spectra-curation
description: Use when building a reference tandem MS test set from MassBank/HMDB spectra
  with known chemical structures, or when a held-out known-compound split is needed
  to benchmark metabolite annotation retrieval statistics.
when_to_use_negative:
- Input spectra are not in ChemDistiller internal format — the README states conversions
  from other formats were not yet implemented, so spectra must first be converted
  to the hierarchical internal format
- The goal is de novo annotation of unknown compounds with no known structures — retrieval
  statistics are only printed when compounds have known chemical structures
- The task is scoring-algorithm design (e.g. element isotope-signature or formula
  scoring internals) rather than compiling and benchmarking against reference spectra
edam_operation: http://edamontology.org/operation_2421
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: ChemDistiller v0.1
  role: Annotation engine run via annotate.py over the curated MassBank/HMDB test
    spectra to produce ranked candidate annotations and retrieval statistics
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: h5py
  role: Library dependency for reading the HDF5 (*.h5) compound databases loaded into
    .../ChemDistiller/DBs/
- name: RDKit
  role: Optional dependency used to generate images of 2D chemical structures of compounds
    for the HTML annotation report
  repo: http://www.rdkit.org/
- name: Anaconda
  role: Recommended Python distribution supplying pre-installed dependencies (SciPy,
    NumPy, h5py) for running the annotation pipeline
  repo: https://www.anaconda.com/download/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:fd95f58d3cbc7c50fa1e0c173615f6391371bb7045536e120d4cee947031b459
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Anaconda
  - ChemDistiller
  - ChemDistiller v0.1
  - NumPy
  - Python 2 and 3
  - RDKit
  - SciPy
  - annotate.py
  - h5py
  edam_operation: http://edamontology.org/operation_2421
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# massbank-hmdb-reference-spectra-curation

## Summary

Compiles reference tandem MS spectra from MassBank and HMDB into a known-compound evaluation set for ChemDistiller, applying an 80:20 train/test split over a 6297-compound reference set and measuring top-k retrieval rates against small and tens-of-millions compound databases.

## When to use

Use when building a reference tandem MS test set from MassBank/HMDB spectra with known chemical structures, or when a held-out known-compound split is needed to benchmark metabolite annotation retrieval statistics.

## When NOT to use

- Input spectra are not in ChemDistiller internal format — the README states conversions from other formats were not yet implemented, so spectra must first be converted to the hierarchical internal format
- The goal is de novo annotation of unknown compounds with no known structures — retrieval statistics are only printed when compounds have known chemical structures
- The task is scoring-algorithm design (e.g. element isotope-signature or formula scoring internals) rather than compiling and benchmarking against reference spectra

## Inputs

- MassBank reference tandem MS spectra
- HMDB reference tandem MS spectra
- 6297-compound reference set with known chemical structures (known compounds)
- Pre-built compound databases in HDF5 format (*.h5) with pre-calculated fingerprints and fragmentation patterns
- SVM models for FingerScorer (Linear_weighted default or Radial_weighted, subfolders '1' and '-1')
- Spectra in ChemDistiller internal hierarchical format (see int_format.md)

## Outputs

- test_split_manifest.csv recording the held-out ~1259 test spectra from the 80:20 split
- annotated_spectra.json (JSON formatted annotations per test spectrum)
- Ranked candidate compound lists per spectrum from chemdistiller.chemdb.results
- Retrieval statistics: top-1/top-5 (small database) and top-20 (large database) rates under baseline, +Element, and +Known formula conditions
- table1_retrieval_rates.csv comparing computed rates against the published Table 1 values

## How to apply

Download the public MassBank and HMDB reference tandem MS spectra, compile the 6297-compound reference set, and apply an 80:20 train/test split, recording the held-out ~20% (~1259) test spectra in test_split_manifest.csv. Load the chemical reference databases (a small one and the tens-of-millions compound one with pre-calculated 'fingerprints' and fragmentation patterns) as HDF5 files into .../ChemDistiller/DBs/, and use the Radial_weighted SVMs for FingerScorer rather than the default Linear_weighted set. First validate the engine by running annotate.py in test mode over the 863 provided MassBank spectra, confirming that retrieval statistics are printed since these compounds have known structures. Then run annotate.py over each held-out test spectrum with a --delta_mz MS1 tolerance in ppm and --max_results raised above the default 10 for the large database, combining FingerScorer and FragScorer evidence and repeating under +Element and +Known formula conditions. Finally compute top-1/top-5 retrieval rates (small database) and top-20 retrieval rates (large database) per condition in positive and negative mode, e.g. the combined scorer reaching 86%/78% top-5 (small DB).

## Related tools

- **ChemDistiller v0.1** (Annotation engine run via annotate.py over the curated MassBank/HMDB test spectra to produce ranked candidate annotations and retrieval statistics) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **h5py** (Library dependency for reading the HDF5 (*.h5) compound databases loaded into .../ChemDistiller/DBs/)
- **RDKit** (Optional dependency used to generate images of 2D chemical structures of compounds for the HTML annotation report) — http://www.rdkit.org/
- **Anaconda** (Recommended Python distribution supplying pre-installed dependencies (SciPy, NumPy, h5py) for running the annotation pipeline) — https://www.anaconda.com/download/

## Examples

```
python annotate.py --test
```

## Evaluation signals

- Test-mode run over the 863 MassBank spectra completes in a few minutes and prints retrieval statistics because the compounds have known chemical structures
- Output folder contains annotated_spectra.json, log.txt, an annotated_spectra folder, and a Report folder with index.html
- Held-out test split contains ~20% of the 6297-compound reference set (~1259 spectra), recorded in test_split_manifest.csv
- Combined FingerScorer+FragScorer top-5 retrieval on the small database reaches ~86% (positive mode) and ~78% (negative mode), with FingerScorer alone at 80%/71.5% and FragScorer alone at 82.5%/76.5%
- Retrieval rates are comparable to or exceeding current state-of-the-art solutions while retaining higher throughput, scalability and processing speed

## Limitations

- Spectra must be supplied in ChemDistiller internal hierarchical format; conversions from other formats were not yet implemented at the time of the README
- The distribution ships only a small set of compound databases and simple SVM models; larger HDF5 databases and Radial_weighted SVMs must be downloaded separately (mediafire-hosted) and unpacked into .../ChemDistiller/DBs/ and .../ChemDistiller/SVMs/
- Derivative databases may carry licensing restrictions from their original source compound databases, so the readme.txt and original database disclaimers must be consulted before commercial use
- When replacing the default SVMs, both '1' and '-1' subfolders in .../ChemDistiller/SVMs/ must be deleted first to avoid confusion

## Evidence

- [readme] Test-mode verification over MassBank spectra: [evidence span withheld — anchor status: external_doc]
- [readme] Retrieval statistics require known structures: [evidence span withheld — anchor status: external_doc]
- [abstract] Reference database scale with pre-calculated fingerprints: "a compiled database containing tens of millions of compounds with pre-calculated 'fingerprints' and fragmentation patterns"
- [intro] ChemDistiller is a tandem MS annotation engine: "ChemDistiller is the high-throughput annotation engine for tandem MS spectra annotation."
- [readme] Spectra must be in internal format: [evidence span withheld — anchor status: not_found]
