---
name: neural-network-spectra-formula-re-ranking
description: Use when you have held-out tandem MS spectra (e.g. NPLIB1, CASMI 2022),
  precomputed SIRIUS formula candidates, and need to compare neural re-ranking architectures
  by top-1 accuracy.
when_to_use_negative:
- Data are negative-ion-mode spectra — MIST-CF was trained only on positive-mode data
- You need candidate enumeration without SIRIUS — MIST-CF still relies on SIRIUS's
  algorithmic decomposition of exact masses into formula candidates
- You require MS1 isotopic pattern scoring or adduct switching during MS2 subformula
  assignment — neither is integrated
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MIST-CF
  role: Primary energy-based architecture that scores formula-adduct candidates from
    MS/MS via subformula assignment and formula transformer embeddings
  repo: github.com/samgoldman97/mist-cf
- name: SIRIUS
  role: Enumerates candidate formulae for each MS1 mass via 'SIRIUS decomp' dynamic
    programming; sets SIRIUS_PATH env variable
  repo: github.com/boecker-lab/sirius
- name: FastFilter
  role: Lightweight neural formula filtering model used as an alternative baseline
    architecture in the benchmarking comparison
  repo: github.com/samgoldman97/mist-cf
- name: Chemical Formula Transformer
  role: Original transformer architecture that MIST-CF extends; included as baseline
    for re-ranking comparison
  repo: github.com/samgoldman97/mist-cf
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jcim.3c01082
    title: 'MIST-CF: Chemical Formula Inference from Tandem Mass Spectra'
  source_content_hash: sha256:113b11f53b94cc1755f95e3c39b860136c185015a300ee10f1297a8a5174c69a
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - Chemical Formula Transformer
  - FastFilter
  - MIST-CF
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# neural-network-spectra-formula-re-ranking

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Benchmark protocol that re-ranks enumerated chemical formula and adduct candidates for held-out MS/MS spectra using energy-based neural architectures (MIST-CF, FFN, xformer, MS1-only) to quantify top-1/top-k accuracy gains on NPLIB1 and NPLIB1+NIST20 splits.

## When to use

Use when you have held-out tandem MS spectra (e.g. NPLIB1, CASMI 2022), precomputed SIRIUS formula candidates, and need to compare neural re-ranking architectures by top-1 accuracy.

## When NOT to use

- Data are negative-ion-mode spectra — MIST-CF was trained only on positive-mode data
- You need candidate enumeration without SIRIUS — MIST-CF still relies on SIRIUS's algorithmic decomposition of exact masses into formula candidates
- You require MS1 isotopic pattern scoring or adduct switching during MS2 subformula assignment — neither is integrated

## Inputs

- Preprocessed NPLIB1 dataset (canopus_train) from Zenodo 8151490
- CASMI 2022 challenge spectra (optional secondary evaluation)
- SIRIUS-enumerated chemical formula candidate lists per MS1 mass (up to 256 per spectrum)
- Trained model checkpoints: MIST-CF, FFN, xformer, and MS1-only
- SIRIUS 5.5.7 headless executable

## Outputs

- benchmarking_results.csv with per-model top-1 and top-k accuracy on NPLIB1
- Re-ranked formula and adduct candidate lists per test spectrum for each architecture
- Top-1 accuracy for MIST-CF with and without NIST20 training spectra

## How to apply

Download the preprocessed NPLIB1 (canopus_train) dataset from Zenodo record 8151490 and unzip into ./data/canopus_train, plus optionally CASMI 22 for prospective evaluation. Install SIRIUS 5.5.7 headless for linux64 from github.com/boecker-lab/sirius releases and export SIRIUS_PATH so 'SIRIUS decomp' can enumerate candidate formulae per MS1 mass. Load trained MIST-CF, FFN, xformer, and MS1-only checkpoints, score each of up to 256 formula-adduct candidates per test spectrum with each architecture, and re-rank candidates by score. Compile per-model top-1 and top-k accuracy into benchmarking_results.csv; on NPLIB1-only the MIST-CF model should reach roughly 74% top-1 accuracy, and the margin over the next best architecture should exceed 10%. Optionally repeat the training/evaluation with NIST20 Orbitrap spectra added (nist_canopus joint dataset), which should lift MIST-CF top-1 accuracy to about 77%, reflecting the benefit of higher-resolution commercial training data.

## Related tools

- **MIST-CF** (Primary energy-based architecture that scores formula-adduct candidates from MS/MS via subformula assignment and formula transformer embeddings) — github.com/samgoldman97/mist-cf
- **SIRIUS** (Enumerates candidate formulae for each MS1 mass via 'SIRIUS decomp' dynamic programming; sets SIRIUS_PATH env variable) — github.com/boecker-lab/sirius
- **FastFilter** (Lightweight neural formula filtering model used as an alternative baseline architecture in the benchmarking comparison) — github.com/samgoldman97/mist-cf
- **Chemical Formula Transformer** (Original transformer architecture that MIST-CF extends; included as baseline for re-ranking comparison) — github.com/samgoldman97/mist-cf

## Examples

```
python run_scripts/benchmarking/eval_models.py --sirius_path $SIRIUS_PATH
```

## Evaluation signals

- Top-1 accuracy of MIST-CF on NPLIB1-only split should be approximately 74% (0.741 reported)
- Top-1 accuracy when NIST20 Orbitrap training spectra are added should be about 77% (0.769 reported)
- Margin over the next best architecture should exceed 10% absolute top-1 accuracy
- Each test spectrum should be re-ranked against up to 256 formula-adduct candidates
- benchmarking_results.csv should be produced with per-model accuracy entries

## Limitations

- Models trained only on positive-mode data; negative-mode support is absent
- MIST-CF scores are not integrated with MS1 isotopic scores
- Adduct switching in MS2 subformula assignment is not considered
- Reliance on SIRIUS for formula candidate decomposition remains a dependency
- Forward structure-to-spectrum models for data augmentation have not been explored

## Evidence
