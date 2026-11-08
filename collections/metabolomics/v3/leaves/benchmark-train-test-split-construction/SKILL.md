---
name: benchmark-train-test-split-construction
description: Use when evaluating a tandem MS annotation/scoring engine against published
  retrieval statistics and you need a reproducible held-out test split of reference
  spectra with known chemical structures.
when_to_use_negative:
- Input spectra have unknown chemical structures with no ground-truth annotations
  — retrieval statistics cannot be computed without known compounds
- You need de novo annotation of your own samples rather than a benchmark against
  published Table 1 statistics
- Spectra are not convertible into ChemDistiller's internal hierarchical format (other-format
  conversions were not yet implemented in v0.1)
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_0092
tools:
- name: ChemDistiller v0.1
  role: Annotation engine whose scoring (FingerScorer, FragScorer, totalscore combination)
    is benchmarked on the held-out split
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: annotate.py
  role: Command-line entry point run over the held-out test spectra to produce ranked
    annotations and retrieval statistics
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: chemdistiller.chemdb.manager / chemdb.database / chemdb.results
  role: Modules that load the fingerprint/fragprint-precalculated compound databases
    and collect ranked candidate lists for retrieval-rate computation
- name: RDKit
  role: Optional dependency used to generate 2D chemical structure images of annotated
    benchmark compounds in the HTML report
  repo: http://www.rdkit.org/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:da17b2adc133f1e6aea13f33d7c7766f0d1590e29f5d4fc4c962366681fe2574
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDistiller
  - ChemDistiller v0.1
  - NumPy
  - Python 2 and 3
  - RDKit
  - SciPy
  - annotate.py
  - chemdistiller.chemdb.manager / chemdb.database / chemdb.results
  - h5py
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# benchmark-train-test-split-construction

## Summary

Constructs a held-out train/test benchmark for tandem MS metabolite annotation: a ~6297-compound MassBank/HMDB reference set split 80:20, with retrieval rates (top-1/top-5 on a small database, top-20 on a tens-of-millions compound large database) computed per scorer condition (FingerScorer, FragScorer, combined, ±Element/±Known-formula filters).

## When to use

Use when evaluating a tandem MS annotation/scoring engine against published retrieval statistics and you need a reproducible held-out test split of reference spectra with known chemical structures.

## When NOT to use

- Input spectra have unknown chemical structures with no ground-truth annotations — retrieval statistics cannot be computed without known compounds
- You need de novo annotation of your own samples rather than a benchmark against published Table 1 statistics
- Spectra are not convertible into ChemDistiller's internal hierarchical format (other-format conversions were not yet implemented in v0.1)

## Inputs

- Public MassBank and HMDB reference tandem MS spectra (ChemDistiller internal hierarchical format, e.g. data/MassBankTestSpectra/*)
- 6297-compound reference set with known chemical structures
- Small and large chemical compound databases in HDF5 (*.h5) format with pre-calculated fingerprints and fragmentation patterns
- SVM models for FingerScorer (Linear_weighted default; Radial_weighted recommended), subfolders '1' and '-1' in .../ChemDistiller/SVMs/

## Outputs

- test_split_manifest.csv listing the ~1259 held-out (20%) test spectra
- Ranked candidate lists per test spectrum (annotated_spectra.json, annotated_spectra folder, Report/index.html in a timestamped output subfolder)
- table1_retrieval_rates.csv with top-1/top-5 (small DB) and top-20 (large DB) retrieval rates per scorer condition and mode
- log.txt and printed retrieval statistics in .../ChemDistiller/testlogs/

## How to apply

Download the public MassBank and HMDB reference tandem MS spectra and compile the 6297-compound reference set, then apply an 80:20 train/test split, recording the held-out ~20% (~1259) test spectra in a test_split_manifest.csv so the evaluation is reproducible. Load the small and large chemical reference databases (tens of millions of compounds) in HDF5 (*.h5) format with pre-calculated fingerprints and fragmentation patterns via chemdistiller.chemdb.database and chemdistiller.chemdb.manager. Run annotate.py over each held-out test spectrum against the small database under baseline, +Element (isotope-signature scoring) and +Known-formula conditions, combining fingerprint and fragprint evidence via chemdistiller.scorers.totalscore, then repeat against the large database collecting ranked candidate lists via chemdistiller.chemdb.results. Compute top-1/top-5 retrieval rates for the small database and top-20 for the large database per condition and per ionization mode, writing the comparison against published values to table1_retrieval_rates.csv. Candidate matching uses an MS1 m/z tolerance (--delta_mz, in ppm), and --max_results should be increased (e.g. 100) when benchmarking against large compound databases.

## Related tools

- **ChemDistiller v0.1** (Annotation engine whose scoring (FingerScorer, FragScorer, totalscore combination) is benchmarked on the held-out split) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **annotate.py** (Command-line entry point run over the held-out test spectra to produce ranked annotations and retrieval statistics) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **chemdistiller.chemdb.manager / chemdb.database / chemdb.results** (Modules that load the fingerprint/fragprint-precalculated compound databases and collect ranked candidate lists for retrieval-rate computation)
- **RDKit** (Optional dependency used to generate 2D chemical structure images of annotated benchmark compounds in the HTML report) — http://www.rdkit.org/

## Examples

```
python annotate.py Input_Spectra_folder output_folder --ncpu 5 --delta_mz 20 --max_results 100
```

## Evaluation signals

- Combined FingerScorer+FragScorer top-5 rates on the small database should match ~86% (positive mode) and ~78% (negative mode); FingerScorer alone 80%/71.5%; FragScorer alone 82.5%/76.5%
- test_split_manifest.csv must contain ~1259 held-out spectra, i.e. ~20% of the 6297-compound reference set
- Running 'python annotate.py --test' on the 863 MassBank test spectra must complete in a few minutes, produce annotated_spectra.json, log.txt, an annotated_spectra folder and an HTML Report, and print retrieval statistics for the known compounds
- Candidates accepted into ranking must lie within +/- delta_mz (ppm) of the MS1 peak m/z value; verify the tolerance was applied consistently across conditions
- Large-database runs must retain enough candidates per peak (--max_results raised, e.g. to 100) for top-20 statistics to be meaningful

## Limitations

- Spectra must be in ChemDistiller's internal hierarchical format; conversions from other formats were 'should be available in the next few weeks' at v0.1
- Default SVMs supplied with the repo are small/simple due to size limitations — use the recommended Radial_weighted SVMs from the mediafire download for full benchmark power, deleting both '1' and '-1' subfolders before replacing
- When replacing default SVMs via --svm_folder, note Linear SVMs are smaller/faster but have worse prediction power than Radial
- Derivative compound databases may carry commercial-use restrictions from their original sources; consult readme.txt and original database disclaimers before commercial use

## Evidence

- [abstract] Retrieval rates comparable to state-of-the-art: "show retrievals rates comparable to or exceeding the ones obtainable by the current state-of-the-art solutions in the field while offering higher throughput, scalability and processing speed"
- [abstract] Database with precalculated fingerprints: "combines automated large-scale annotation of metabolites using tandem MS data with a compiled database containing tens of millions of compounds with pre-calculated 'fingerprints' and fragmentation"
- [readme] Test-mode run on MassBank test spectra: [evidence span withheld — anchor status: external_doc]
- [readme] Retrieval statistics printed for known compounds: [evidence span withheld — anchor status: external_doc]
- [readme] delta_mz MS1 tolerance: [evidence span withheld — anchor status: external_doc]
