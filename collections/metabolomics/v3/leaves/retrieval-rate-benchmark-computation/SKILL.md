---
name: retrieval-rate-benchmark-computation
description: 'Use when benchmarking a metabolite annotation engine against MetFrag,
  CFM-ID or CSI:FingerID: you have tandem MS/MS test spectra with known structures
  and ranked candidate lists per tool, and need top-1/top-k retrieval rates for side-by-side
  comparison.'
when_to_use_negative:
- Test spectra compounds lack known chemical structures — correct hits cannot be scored
  and retrieval statistics cannot be computed (ChemDistiller's built-in test set is
  chosen precisely because structures are known)
- Tools are run on different datasets, splits, or precursor mass tolerances — the
  comparison is then not 'under the same conditions' and top-k rates are not comparable
- The task is a single metabolite annotation run rather than an engine-vs-engine performance
  benchmark
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ChemDistiller
  role: Annotation engine under test; retrieval statistics are computed from its ranked
    annotations on the shared test spectra and compound databases
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: MetFrag
  role: 'Baseline: ranks database candidates by in silico fragmentation against measured
    MS/MS peaks under the benchmark precursor mass tolerance'
- name: CFM-ID
  role: 'Baseline: run in single-energy mode to generate predicted spectra and score
    database candidates per test spectrum'
- name: SIRIUS
  role: 'Baseline: computes fragmentation trees and queries CSI:FingerID to rank candidate
    compounds by fingerprint match'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:b388b2c53dec4e2509527c4ac2fc5b66c088b85345dd0f5c4b6b281e8cee0652
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CFM-ID
  - CSI:FingerID
  - ChemDistiller
  - MetFrag
  - SIRIUS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# retrieval-rate-benchmark-computation

## Summary

Computes top-k (top-1) retrieval rates for tandem MS metabolite annotation engines and baseline tools (MetFrag, CFM-ID, SIRIUS/CSI:FingerID) on identical train/test spectra splits, to support claims like ChemDistiller's rates being comparable to or exceeding state-of-the-art solutions.

## When to use

Use when benchmarking a metabolite annotation engine against MetFrag, CFM-ID or CSI:FingerID: you have tandem MS/MS test spectra with known structures and ranked candidate lists per tool, and need top-1/top-k retrieval rates for side-by-side comparison.

## When NOT to use

- Test spectra compounds lack known chemical structures — correct hits cannot be scored and retrieval statistics cannot be computed (ChemDistiller's built-in test set is chosen precisely because structures are known)
- Tools are run on different datasets, splits, or precursor mass tolerances — the comparison is then not 'under the same conditions' and top-k rates are not comparable
- The task is a single metabolite annotation run rather than an engine-vs-engine performance benchmark

## Inputs

- Train/test tandem MS/MS spectra splits (e.g. MassBank test spectra, 863 spectra in the ChemDistiller test set, in ChemDistiller internal hierarchical format)
- Compiled compound database with pre-calculated fingerprints and fragmentation patterns (HDF5 *.h5 files)
- Ranked candidate compound result lists from each annotation tool (MetFrag, CFM-ID, SIRIUS/CSI:FingerID, ChemDistiller)

## Outputs

- Top-k (top-1) retrieval rates per tool on the test split
- Side-by-side comparison of reproduced baseline rates against published Table 1 values
- Retrieval statistics printout / annotated_spectra.json results from a ChemDistiller test-mode run

## How to apply

Evaluate all tools under identical conditions: load the same train/test tandem MS spectra splits and the compiled compound database (with pre-calculated fingerprints and fragmentation patterns). Run each tool on every test spectrum — MetFrag ranks database candidates by in silico fragmentation under the benchmark's precursor mass tolerance, CFM-ID in single-energy mode scores predicted spectra, SIRIUS/CSI:FingerID ranks by fingerprint match, and ChemDistiller via annotate.py with a --delta_mz ppm MS1 tolerance. Parse each tool's ranked result lists, score correct molecular-formula/structure hits, and compute top-k retrieval rates on the test split. Validation: reproduced baseline rates must match the published Table 1 values within a small tolerance (e.g. MetFrag 51.7%/45% vs ChemDistiller 50%/44% top-1 on the small database), supporting the claim that retrieval rates are comparable to or exceeding state-of-the-art solutions.

## Related tools

- **ChemDistiller** (Annotation engine under test; retrieval statistics are computed from its ranked annotations on the shared test spectra and compound databases) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **MetFrag** (Baseline: ranks database candidates by in silico fragmentation against measured MS/MS peaks under the benchmark precursor mass tolerance)
- **CFM-ID** (Baseline: run in single-energy mode to generate predicted spectra and score database candidates per test spectrum)
- **SIRIUS** (Baseline: computes fragmentation trees and queries CSI:FingerID to rank candidate compounds by fingerprint match)

## Examples

```
python annotate.py --test
```

## Evaluation signals

- Reproduced baseline top-1 rates match published Table 1 values within a small tolerance (e.g. MetFrag 51.7%/45% vs ChemDistiller best scorer 50%/44% on the small database)
- ChemDistiller test-mode run completes on the 863 provided MassBank spectra (a few minutes) and prints retrieval statistics because compounds have known structures
- All tools were evaluated on the same train/test splits and compound database with the same precursor mass tolerance (--delta_mz in ppm) for candidates
- Retrieval-rate comparison supports the claim of rates 'comparable to or exceeding' state-of-the-art tools, not worse under identical conditions

## Limitations

- Spectra must be in ChemDistiller internal hierarchical format; conversions from other formats were not yet implemented at the time of the README
- The shipped version includes only a small set of compound databases and simple SVM models; larger-scale benchmarking requires downloading larger HDF5 databases and Radial_weighted SVMs (recommended over Linear for better prediction power)
- Derivative databases may have commercial-use restrictions inherited from source compound databases; consult readme.txt and original database disclaimers before commercial use

## Evidence

- [readme] Test-mode retrieval scoring on known structures: "Since these compounds also have the known chemical structures (known compounds)
    the retrieval statistics should also be printed."
- [readme] Test data and expected runtime/output: "This will initiate a test annotation run using default settings and the test
    data provided in .../ChemDistiller/data/MassBankTestSpectra/*.
    The test should take a few minutes for 863 spectra"
- [abstract] Claim the benchmark supports: "show retrievals rates comparable to or exceeding the ones obtainable by the current state-of-the-art solutions in the field while offering higher throughput"
- [abstract] Database contents used for identical-condition evaluation: "a compiled database containing tens of millions of compounds with pre-calculated 'fingerprints' and fragmentation"
