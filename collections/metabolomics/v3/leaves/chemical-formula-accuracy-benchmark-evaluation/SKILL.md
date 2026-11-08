---
name: chemical-formula-accuracy-benchmark-evaluation
description: Use when benchmarking formula-prediction methods on CASMI 2016/2017 or
  EMBL-MCF 2.0 spectra and you need comparable top-1/top-5 accuracy per tool and ion
  mode after excluding training-set overlaps.
when_to_use_negative:
- Evaluating on the internal held-out test set split from NIST20/NIST23/MoNA/GNPS
  training data by canonical SMILES — this skill targets external CASMI/EMBL-MCF benchmarks
  only
- Spectra with precursor m/z above 1500, which the workflow explicitly excludes as
  rare
- Compound-level structural elucidation or fingerprint prediction (CSI:FingerID/CANOPUS)
  rather than formula-level accuracy benchmarking
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: FIDDLE
  role: Primary deep-learning formula predictor evaluated via run_fiddle.py with Orbitrap
    TCN config and rescore checkpoint; also combines SIRIUS/BUDDY candidates via --buddy_path
    and --sirius_path
  repo: https://github.com/JosieHong/FIDDLE
- name: SIRIUS
  role: Top-down baseline run on the same benchmark spectra; outputs formatted as
    output_sirius.csv for combined evaluation
  repo: https://github.com/sirius-ms/sirius
- name: BUDDY
  role: Bottom-up baseline (msbuddy) run on the same benchmark spectra; outputs formatted
    as output_buddy.csv for combined evaluation
  repo: https://github.com/Philipbear/msbuddy
- name: MIST-CF
  role: Additional baseline whose scores are merged into the accuracy comparison per
    tool and ion mode
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-66060-9
    title: 'FIDDLE: a deep learning method for chemical formulas prediction from tandem
      mass spectra'
  source_content_hash: sha256:446e503678d38dacba6707c91c64a6290617322b8eca84bad3d34064fe2c9a8b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/JosieHong/FIDDLE
  repo_url: https://github.com/JosieHong/FIDDLE
  tools:
  - BUDDY
  - FIDDLE
  - MIST-CF
  - Q-TOF
  - SIRIUS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# chemical-formula-accuracy-benchmark-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce and compare molecular formula identification accuracy (top-1/top-5) of FIDDLE against SIRIUS, BUDDY, and MIST-CF on external metabolomics benchmarks (CASMI 2016, CASMI 2017, EMBL-MCF 2.0), with training-overlap compounds removed.

## When to use

Use when benchmarking formula-prediction methods on CASMI 2016/2017 or EMBL-MCF 2.0 spectra and you need comparable top-1/top-5 accuracy per tool and ion mode after excluding training-set overlaps.

## When NOT to use

- Evaluating on the internal held-out test set split from NIST20/NIST23/MoNA/GNPS training data by canonical SMILES — this skill targets external CASMI/EMBL-MCF benchmarks only
- Spectra with precursor m/z above 1500, which the workflow explicitly excludes as rare
- Compound-level structural elucidation or fingerprint prediction (CSI:FingerID/CANOPUS) rather than formula-level accuracy benchmarking

## Inputs

- CASMI 2016 benchmark MS/MS spectra
- CASMI 2017 benchmark MS/MS spectra
- EMBL-MCF 2.0 benchmark MS/MS spectra
- Spectra converted to mgf format with title, precursor_mz, precursor_type, collision_energy fields
- Pre-trained FIDDLE checkpoints (Orbitrap and Q-TOF TCN and rescore models)
- SIRIUS and BUDDY prediction outputs formatted as output_sirius.csv and output_buddy.csv
- FIDDLE training-set canonical SMILES list for overlap filtering

## Outputs

- output_fiddle_all.csv with combined FIDDLE/SIRIUS/BUDDY results
- Per-tool, per-ion-mode top-1 and top-5 accuracy tables on the external benchmarks
- Reproduced average top-5 accuracies (FIDDLE alone 75.1% positive / 66.2% negative; combined 80.0% / 73.8%) comparable to Fig. 5

## How to apply

Download FIDDLE pre-trained weights (fiddle_tcn_orbitrap.pt, fiddle_rescore_orbitrap.pt and Q-TOF equivalents) into ./check_point/ via ./running_scripts/download_models.sh. Convert benchmark spectra to mgf with required fields (title, precursor_mz, precursor_type, collision_energy) and run running_scripts/experiments_test_benchmark.sh, which invokes run_fiddle.py with the Orbitrap TCN config and rescore checkpoint, using a 5 ppm precursor tolerance for Orbitrap and 10 ppm for Q-TOF with precursor m/z confined to (0, 1500]. Run SIRIUS and BUDDY on the same spectra, format outputs as output_sirius.csv and output_buddy.csv, then rerun run_fiddle.py with --buddy_path and --sirius_path to get combined results in output_fiddle_all.csv. Filter out benchmark compounds overlapping training by canonical SMILES, merge MIST-CF baseline scores, and compute top-1/top-5 accuracy per tool and ion mode.

## Related tools

- **FIDDLE** (Primary deep-learning formula predictor evaluated via run_fiddle.py with Orbitrap TCN config and rescore checkpoint; also combines SIRIUS/BUDDY candidates via --buddy_path and --sirius_path) — https://github.com/JosieHong/FIDDLE
- **SIRIUS** (Top-down baseline run on the same benchmark spectra; outputs formatted as output_sirius.csv for combined evaluation) — https://github.com/sirius-ms/sirius
- **BUDDY** (Bottom-up baseline (msbuddy) run on the same benchmark spectra; outputs formatted as output_buddy.csv for combined evaluation) — https://github.com/Philipbear/msbuddy
- **MIST-CF** (Additional baseline whose scores are merged into the accuracy comparison per tool and ion mode)

## Examples

```
bash running_scripts/download_models.sh && bash running_scripts/experiments_test_benchmark.sh; python run_fiddle.py --buddy_path output_buddy.csv --sirius_path output_sirius.csv
```

## Evaluation signals

- Computed average top-5 accuracies fall within a few percentage points of 80.0% (positive) and 73.8% (negative) for combined FIDDLE+SIRIUS+BUDDY, and 75.1% / 66.2% for FIDDLE alone
- Training-overlap filtering by canonical SMILES removes no test-set compounds from training, matching the split guarantee
- Precursor mass tolerance applied is 5 ppm for Orbitrap and 10 ppm for Q-TOF, with precursor m/z within (0, 1500]
- output_sirius.csv and output_buddy.csv cover the same benchmark compounds so per-tool top-1/top-5 accuracy is computable in both ion modes

## Limitations

- Experiment condition effect is not completely eliminated; accuracy varies across conditions, especially those with less training data
- FIDDLE's performance degrades on chimeric (co-eluting) spectra as spectral mixing increases
- Data augmentation (Gaussian noise, mean 0, sd 0.1) was applied only for Q-TOF; no augmentation was used for Orbitrap training

## Evidence
