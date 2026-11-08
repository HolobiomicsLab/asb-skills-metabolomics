---
name: msms-spectrum-formula-ranking
description: Use when you have MS/MS spectra (positive mode, NPLIB1/NIST20-style)
  with mass-enumerated formula candidates and need to downselect to a tractable top-k
  set that retains the true formula.
when_to_use_negative:
- The spectra are negative-mode — models were trained only for positive-mode data
- The task requires integrating formula scores with MS1 isotopic scores or handling
  adduct switching in MS2 subformula assignment, neither of which this method addresses
- You need full structural annotation (fingerprint/structure IDs) rather than formula
  ranking — use CSI:FingerID-style downstream tools instead
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: FastFilter
  role: Light-weight neural network energy model that scores and ranks each enumerated
    candidate formula against the MS/MS spectrum, downselecting to a top-256 set
  repo: github.com/samgoldman97/mist-cf
- name: SIRIUS
  role: Provides the 'SIRIUS decomp' dynamic-programming module that enumerates RDBE-permissive
    formula candidates for each observed MS1 mass before ranking
  repo: https://github.com/boecker-lab/sirius
- name: MIST-CF
  role: Parent pipeline (formula transformer energy-based model) into which the filtered
    formula ranking feeds for de novo formula/adduct annotation
  repo: https://github.com/samgoldman97/mist-cf
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jcim.3c01082
    title: 'MIST-CF: Chemical Formula Inference from Tandem Mass Spectra'
  source_content_hash: sha256:db60de54250df4703c5c6e5047566bfeea43c5da6e16af4cbbb5af388be9d924
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - FastFilter
  - MIST-CF
  - SIRIUS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# msms-spectrum-formula-ranking

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Rank enumerated chemical formula candidates for an MS/MS spectrum by scoring candidate-spectrum agreement with the FastFilter energy model (part of MIST-CF), downselecting large SIRIUS 'decomp' candidate sets while retaining the true formula.

## When to use

Use when you have MS/MS spectra (positive mode, NPLIB1/NIST20-style) with mass-enumerated formula candidates and need to downselect to a tractable top-k set that retains the true formula.

## When NOT to use

- The spectra are negative-mode — models were trained only for positive-mode data
- The task requires integrating formula scores with MS1 isotopic scores or handling adduct switching in MS2 subformula assignment, neither of which this method addresses
- You need full structural annotation (fingerprint/structure IDs) rather than formula ranking — use CSI:FingerID-style downstream tools instead

## Inputs

- MS/MS training or evaluation spectra (MGF-style, e.g. data/demo_specs.mgf from NPLIB1, NIST20, or CASMI2022)
- Enumerated formula candidate sets per spectrum from SIRIUS 'SIRIUS decomp' restricted to RDBE-plausible formulae
- Pretrained FastFilter checkpoint (trained via run_scripts/public_data_train/train_fast_filter.sh)
- Fast filter config file (configs/fast_filter.yaml)

## Outputs

- Per-spectrum ranked formula candidate lists (FastFilter energy scores)
- Pre-filter candidate-count distribution across spectra
- Top-256 true-formula recovery rate (expected ~99%)
- Summary table fast_filter_stats.csv from run_scripts/fast_filter/launch_fast_pred.py

## How to apply

First enumerate RDBE-plausible formula candidates for each exact MS1 mass using SIRIUS's dynamic-programming 'decomp' module (SIRIUS_PATH must point to a headless SIRIUS build, e.g. v5.5.7). Load the pretrained FastFilter checkpoint trained per src/mist_cf/fast_form_score/train.py with settings from configs/fast_filter.yaml, then run src/mist_cf/fast_form_score/predict.py to score every candidate formula against its spectrum and rank candidates per spectrum. Compute the pre-filter candidate-count distribution across spectra and the top-256 recovery rate of the true formula; the key threshold is that FastFilter keeps the true formula within its top 256 candidates ~99% of the time even for spectra with >5,000 candidates. Evaluate rankings with run_scripts/fast_filter/launch_fast_pred.py and write the summary to fast_filter_stats.csv. Rationale: ranking by a learned energy model replaces fragmentation-tree scoring so the pipeline stays tractable on large candidate sets.

## Related tools

- **FastFilter** (Light-weight neural network energy model that scores and ranks each enumerated candidate formula against the MS/MS spectrum, downselecting to a top-256 set) — github.com/samgoldman97/mist-cf
- **SIRIUS** (Provides the 'SIRIUS decomp' dynamic-programming module that enumerates RDBE-permissive formula candidates for each observed MS1 mass before ranking) — https://github.com/boecker-lab/sirius
- **MIST-CF** (Parent pipeline (formula transformer energy-based model) into which the filtered formula ranking feeds for de novo formula/adduct annotation) — https://github.com/samgoldman97/mist-cf

## Examples

```
python run_scripts/fast_filter/launch_fast_pred.py  # after running src/mist_cf/fast_form_score/predict.py to score candidates and rank them per spectrum
```

## Evaluation signals

- Top-256 recovery rate of the true formula is ~99% across NPLIB1 and NIST20 training spectra
- Pre-filter candidate-count distribution is reported; >15% of spectra should have >5,000 RDBE-permissive candidates before filtering
- fast_filter_stats.csv is produced by run_scripts/fast_filter/launch_fast_pred.py with per-dataset ranking statistics
- Every candidate in the input enumeration is scored (no candidates dropped before ranking), confirming the filter rather than a pre-truncation is doing the work
- Environment check: SIRIUS_PATH is set and the SIRIUS decomp module runs, since enumeration depends on it

## Limitations

- Models are trained only for positive-mode data; negative-mode support is absent
- MIST-CF/FastFilter scores are not integrated with MS1 isotopic scores, and adduct switching in MS2 subformula assignment is not considered
- The approach still relies on SIRIUS's algorithmic decomposition of exact masses into formula candidates
- The publicly released quickstart model (trained on public NPLIB1/CANOPUS data) may underperform the NIST20-trained model, particularly on Orbitrap or higher-resolution data; NIST20-trained model links are only available on request to NIST licensees

## Evidence
