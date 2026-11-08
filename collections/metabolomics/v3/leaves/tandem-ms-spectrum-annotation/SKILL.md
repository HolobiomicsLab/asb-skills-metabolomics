---
name: tandem-ms-spectrum-annotation
description: Use when you need to assign metabolite identities to unknown tandem MS
  spectra by searching a compiled compound database (tens of millions of structures
  with pre-calculated fingerprints and fragmentation patterns), especially when throughput
  and scalability matter.
when_to_use_negative:
- Input spectra are not convertible to ChemDistiller's internal hierarchical format
  (format conversions from other formats were still being implemented at v0.1)
- The task is reproducing the comparative benchmark against MetFrag, CFM-ID and CSI:FingerID
  rather than performing annotation itself — that requires running the external tools
  under identical train/test splits
- The target is peak-picking or raw LC-MS imaging data processing rather than ranking
  candidate compound structures for MS/MS peaks
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: ChemDistiller
  role: Core annotation engine ranking database candidates for tandem MS spectra via
    fingerprints, fragmentation patterns and scorers
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: Anaconda
  role: Recommended Python distribution providing pre-installed dependencies for running
    ChemDistiller
- name: RDKit
  role: Optional dependency used to generate images of 2D chemical structures of compounds
    for the HTML annotation report
  repo: http://www.rdkit.org/
- name: SciPy
  role: Required Python library for the engine
- name: NumPy
  role: Required Python library for the engine
- name: h5py
  role: Required library for reading HDF5 compound database files (*.h5)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:01abb9522edde61d072c789ed0d8c873dc68b6d046eb939287db87fca5302647
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Anaconda
  - CFM-ID
  - CSI:FingerID
  - ChemDistiller
  - MetFrag
  - NumPy
  - RDKit
  - SIRIUS
  - SciPy
  - h5py
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6190476190476191
n_claims: 21
---

# tandem-ms-spectrum-annotation

## Summary

Automated high-throughput annotation of tandem MS (MS/MS) metabolite spectra by ranking candidate compounds from a large compound database using pre-calculated molecular fingerprints, fragmentation patterns, and scorers (including SVM-based FingerScorer).

## When to use

Use when you need to assign metabolite identities to unknown tandem MS spectra by searching a compiled compound database (tens of millions of structures with pre-calculated fingerprints and fragmentation patterns), especially when throughput and scalability matter.

## When NOT to use

- Input spectra are not convertible to ChemDistiller's internal hierarchical format (format conversions from other formats were still being implemented at v0.1)
- The task is reproducing the comparative benchmark against MetFrag, CFM-ID and CSI:FingerID rather than performing annotation itself — that requires running the external tools under identical train/test splits
- The target is peak-picking or raw LC-MS imaging data processing rather than ranking candidate compound structures for MS/MS peaks

## Inputs

- Tandem MS spectra in ChemDistiller internal hierarchical format (text format described in int_format.md), e.g. the MassBankTestSpectra set of 863 spectra
- Compound databases in HDF5 format (*.h5) with pre-calculated fingerprints and fragmentation patterns, copied to .../ChemDistiller/DBs/
- SVM model folders ('1' and '-1' subfolders) for the FingerScorer, placed in .../ChemDistiller/SVMs/ or passed via --svm_folder

## Outputs

- annotated_spectra.json — JSON formatted annotations for each input spectrum
- log.txt — run log duplicating console output, including retrieval statistics when structures are known
- annotated_specta folder — individual annotated spectra in the internal hierarchical text format
- Report folder with index.html — HTML annotation report for web-browser navigation

## How to apply

Prepare spectra in ChemDistiller's internal hierarchical format and place compound databases (HDF5 *.h5 files) in the .../ChemDistiller/DBs/ subfolder, selecting Linear_weighted or the recommended Radial_weighted SVM models for FingerScorer (Radial is larger and slower but has better prediction power). Run the annotation engine from the command line with python annotate.py, setting --delta_mz to the MS1 m/z tolerance in ppm (e.g. 20 ppm) so database candidates must fall within +/- delta_mz of the MS1 peak, and --max_results to cap candidates per MS1 peak (default 10, increase for large databases). The engine scores candidates via scorers including the SVM FingerScorer against pre-calculated fingerprints and fragmentation patterns, ranking them per spectrum. Use --ncpu for multiprocessor parallelism to exploit ChemDistiller's scalability, and optionally point to a custom SVM folder with --svm_folder. Inspect the outputs: annotated_spectra.json, log.txt, individual annotated spectra, and an HTML Report with index.html for browsing annotations.

## Related tools

- **ChemDistiller** (Core annotation engine ranking database candidates for tandem MS spectra via fingerprints, fragmentation patterns and scorers) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **Anaconda** (Recommended Python distribution providing pre-installed dependencies for running ChemDistiller)
- **RDKit** (Optional dependency used to generate images of 2D chemical structures of compounds for the HTML annotation report) — http://www.rdkit.org/
- **SciPy** (Required Python library for the engine)
- **NumPy** (Required Python library for the engine)
- **h5py** (Required library for reading HDF5 compound database files (*.h5))

## Examples

```
python annotate.py /home/me/my_spectra /home/me/my_results --ncpu 5 --delta_mz 20 --max_results 100
```

## Evaluation signals

- Running 'python annotate.py --test' on the bundled 863 MassBank spectra completes in a few minutes and prints retrieval statistics, since these compounds have known chemical structures
- Retrieval statistics (top-k rates) computed on known-compound test spectra fall in the range reported in the paper, e.g. around 50%/44% for ChemDistiller's best scorer at top 1 on the small database
- Output folder contains annotated_spectra.json, log.txt, the annotated_specta subfolder, and Report/index.html
- Candidate compounds for each MS1 peak are all within +/- delta_mz ppm of the MS1 peak m/z value, and per-peak candidates are capped at --max_results (default 10)
- For every spectrum in annotated_spectra.json, the ranked candidate list is non-empty and the correct molecular formula/structure appears at the expected top-k position for known test compounds

## Limitations

- Spectra must be supplied in ChemDistiller's internal hierarchical format at v0.1; conversions from other formats were not yet available
- The shipped distribution includes only a small set of compound databases and simple SVM models due to size limitations; larger databases and better (Radial_weighted) SVMs must be downloaded separately from external mediafire links
- Radial_weighted SVMs give better prediction power but are larger and slower than Linear_weighted SVMs
- RDKit support is optional and only affects 2D structure image generation in the report, not annotation itself
- When replacing default SVMs, both '1' and '-1' subfolders in .../ChemDistiller/SVMs/ must be deleted first to avoid confusion
- Derived compound databases may inherit licensing restrictions from source databases; consult readme.txt in the DBs download before commercial use

## Evidence

- [intro] High-throughput annotation of tandem MS spectra: "__ChemDistiller__ is the high-throughput annotation engine for tandem MS spectra annotation."
- [abstract] Database of compounds with pre-calculated fingerprints and fragmentation: "combines automated large-scale annotation of metabolites using tandem MS data with a compiled database containing tens of millions of compounds with pre-calculated 'fingerprints' and fragmentation"
- [abstract] Retrieval rates comparable to or exceeding state-of-the-art: "show retrievals rates comparable to or exceeding the ones obtainable by the current state-of-the-art solutions in the field while offering higher throughput, scalability and processing speed"
- [readme] Test-mode run over MassBank spectra: [evidence span withheld — anchor status: external_doc]
- [readme] delta_mz tolerance for MS1 candidate search: [evidence span withheld — anchor status: not_found]
- [readme] SVM choice for FingerScorer: [evidence span withheld — anchor status: not_found]
- [readme] Support for Python 2 and 3 with SciPy, NumPy, h5py: "__ChemDistiller__ supports Python 2 and 3 (64-bit version recommended) and requires _SciPy_, _NumPy_, _h5py_ libraries."
