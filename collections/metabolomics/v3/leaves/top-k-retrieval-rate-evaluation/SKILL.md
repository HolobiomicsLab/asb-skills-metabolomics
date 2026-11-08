---
name: top-k-retrieval-rate-evaluation
description: Use when an annotation engine has scored known-compound tandem MS spectra
  against reference chemical databases and you need to quantify how often the correct
  metabolite appears in the top-k ranked candidates.
when_to_use_negative:
- Spectra are from unknown/novel compounds with no ground-truth structure, so no correct-answer
  rank can be determined for retrieval statistics
- The task is de novo formula prediction or peak annotation rather than ranking candidates
  against a reference chemical database
- Only MS1 data is available without tandem MS fragmentation evidence needed by FingerScorer/FragScorer
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ChemDistiller v0.1
  role: Annotation engine whose FingerScorer/FragScorer ranking output is evaluated
    for top-k retrieval
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: annotate.py
  role: Command-line driver that runs annotation over test spectra and, for known
    compounds, prints retrieval statistics
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: chemdistiller.chemdb.results
  role: Module used to collect ranked candidate lists per test spectrum from the large
    database runs
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: chemdistiller.scorers.totalscore
  role: Combines fingerprint and fragprint evidence into the combined FingerScorer+FragScorer
    score being benchmarked
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: chemdistiller.filters.formula
  role: Restricts candidates to those matching the known precursor formula for the
    +Known formula condition
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: h5py
  role: Required library for reading the HDF5 (*​.h5) compound databases used in retrieval
    evaluation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:80f5ad61c3db44d90460153aef09f4c5881aa5cf5c3d57116aeb8e862567b10f
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
  - chemdistiller.chemdb.results
  - chemdistiller.filters.formula
  - chemdistiller.scorers.totalscore
  - h5py
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# top-k-retrieval-rate-evaluation

## Summary

Evaluates a tandem MS metabolite annotation engine by computing top-k retrieval rates (top-1/top-5 on a small database, top-20 on a large database) over a held-out test split of known-compound spectra, comparing FingerScorer, FragScorer, and their combination under baseline, +Element, and +Known formula conditions.

## When to use

Use when an annotation engine has scored known-compound tandem MS spectra against reference chemical databases and you need to quantify how often the correct metabolite appears in the top-k ranked candidates.

## When NOT to use

- Spectra are from unknown/novel compounds with no ground-truth structure, so no correct-answer rank can be determined for retrieval statistics
- The task is de novo formula prediction or peak annotation rather than ranking candidates against a reference chemical database
- Only MS1 data is available without tandem MS fragmentation evidence needed by FingerScorer/FragScorer

## Inputs

- Held-out test split of known-compound tandem MS spectra (~1259 spectra from the 6297-compound MassBank/HMDB reference set, 80:20 split), in ChemDistiller internal hierarchical format
- Small reference chemical database (HDF5 *.h5) with pre-calculated fingerprints and fragmentation patterns
- Large reference chemical database (tens of millions of compounds, HDF5 *.h5) with pre-calculated fingerprints and fragmentation patterns
- Known precursor formula per test compound for the +Known formula condition
- Trained FingerScorer SVM models (Linear_weighted or Radial_weighted)

## Outputs

- Ranked candidate compound lists per test spectrum for each database and scorer condition
- table1_retrieval_rates.csv with top-1/top-5 (small DB) and top-20 (large DB) retrieval rates per condition and ionization mode
- Comparison of computed retrieval rates against published Table 1 values

## How to apply

Split the 6297-compound MassBank/HMDB reference set 80:20 and hold out the ~20% (~1259) test spectra in a test_split_manifest.csv. Run annotate.py over each held-out spectrum against the small database, combining fingerprint (FingerScorer) and fragprint (FragScorer) evidence via chemdistiller.scorers.totalscore, then repeat under the +Element condition (chemdistiller.scorers.element isotope-signature scoring) and the +Known formula condition (chemdistiller.filters.formula restriction to candidates matching the known precursor formula). Re-run against the large (tens-of-millions compounds) database with pre-calculated fingerprints and fragmentation patterns, collecting ranked candidate lists via chemdistiller.chemdb.results. Compute the fraction of test spectra whose true compound appears in the top-1 and top-5 of the ranked list for the small database and in the top-20 for the large database, per scorer condition and ionization mode (positive/negative). Benchmark correctness by reproducing published Table 1 values, e.g. combined FingerScorer+FragScorer top-5 of 86%/78% (positive/negative) versus 80%/71.5% for FingerScorer alone and 82.5%/76.5% for FragScorer alone on the small database.

## Related tools

- **ChemDistiller v0.1** (Annotation engine whose FingerScorer/FragScorer ranking output is evaluated for top-k retrieval) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **annotate.py** (Command-line driver that runs annotation over test spectra and, for known compounds, prints retrieval statistics) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **chemdistiller.chemdb.results** (Module used to collect ranked candidate lists per test spectrum from the large database runs) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **chemdistiller.scorers.totalscore** (Combines fingerprint and fragprint evidence into the combined FingerScorer+FragScorer score being benchmarked) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **chemdistiller.filters.formula** (Restricts candidates to those matching the known precursor formula for the +Known formula condition) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **h5py** (Required library for reading the HDF5 (*​.h5) compound databases used in retrieval evaluation)

## Examples

```
python annotate.py Input_Spectra_folder /home/me/my_results --ncpu 5 --delta_mz 20 --max_results 100
```

## Evaluation signals

- Computed top-5 combined-scorer rates on the small database match published Table 1 values: 86% (positive mode) and 78% (negative mode)
- Single-scorer baselines fall below the combined score: FingerScorer alone 80%/71.5% and FragScorer alone 82.5%/76.5% (top-5, positive/negative)
- test_split_manifest.csv contains ~1259 held-out spectra, i.e. ~20% of the 6297-compound reference set
- Retrieval statistics are reported separately for positive and negative ionization modes and for baseline / +Element / +Known formula conditions
- Running the packaged test mode over the 863 MassBank test spectra produces retrieval statistics in ../ChemDistiller/testlogs/ without errors

## Limitations

- The distribution ships only a small set of chemical compound databases and simple SVM models; larger databases and more complex (larger, slower) Radial_weighted SVMs must be downloaded separately
- Input spectra must be converted to ChemDistiller's internal hierarchical format; conversions from other formats were not yet implemented at v0.1
- Generated databases inherit licensing restrictions of the original compound databases, so commercial use requires checking each source database's legal disclaimer
- Retrieval statistics are only computable when the test compounds have known chemical structures

## Evidence

- [results] For top-5 annotation with the small database, the combined FingerScorer+FragScorer achieved 86%/78% (positive/negative mode): [evidence span withheld — anchor status: not_found]
- [results] versus 80%/71.5% for FingerScorer alone and 82.5%/76.5% for FragScorer alone: [evidence span withheld — anchor status: not_found]
- [abstract] a compiled database containing tens of millions of compounds with pre-calculated 'fingerprints' and fragmentation: "combines automated large-scale annotation of metabolites using tandem MS data with a compiled database containing tens of millions of compounds with pre-calculated 'fingerprints' and fragmentation"
- [readme] This will initiate a test annotation run using default settings and the test data provided in .../ChemDistiller/data/MassBankTestSpectra/*.: [evidence span withheld — anchor status: external_doc]
- [readme] Since these compounds also have the known chemical structures (known compounds) the retrieval statistics should also be printed.: [evidence span withheld — anchor status: external_doc]
