---
name: mass-spectrometry-dataset-preprocessing
description: 'Use when preparing MS/MS spectra (e.g., NPLIB1, NIST20, biomols, CASMI2022)
  for formula inference: candidate sets from mass decomposition are too large for
  exhaustive scoring and must be ranked/filtered before model training or evaluation.'
when_to_use_negative:
- Input spectra are negative ion mode — MIST-CF/FastFilter models were trained only
  for positive-mode data, so candidate filtering would be unsupported.
- Candidates already come from a curated database search (e.g., NIST, GNPS, HMDB,
  PubChem) rather than de novo mass decomposition, so RDBE-permissive enumeration
  and FastFilter downselection are unnecessary.
- A NIST20-trained model is required for Orbitrap/high-resolution data but unavailable
  — the released quickstart model (public NPLIB1/CANOPUS data) may underperform on
  such spectra.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: SIRIUS
  role: Provides the 'SIRIUS decomp' dynamic-programming module used to enumerate
    all potential formulae for an observed MS1 mass, restricted to RDBE-plausible
    formulae; its fast implementation is why MIST-CF still relies on it for candidate
    generation.
  repo: https://github.com/boecker-lab/sirius
- name: FastFilter
  role: Light-weight neural network energy model that scores and ranks enumerated
    formula candidates against each spectrum, downselecting the RDBE-permissive set
    to a top-256 list retaining the true formula ~99% of the time.
  repo: https://github.com/samgoldman97/mist-cf
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jcim.3c01082
    title: 'MIST-CF: Chemical Formula Inference from Tandem Mass Spectra'
  source_content_hash: sha256:f77fe798be476e90a5c0b4f6d578a78ee50dffc878a4797c039d297a95be7fe2
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/boecker-lab/sirius
  tools:
  - FastFilter
  - SIRIUS
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# mass-spectrometry-dataset-preprocessing

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Preprocess tandem mass spectrometry datasets for de novo chemical formula assignment by enumerating RDBE-plausible formula candidates from MS1 exact masses with SIRIUS's 'decomp' dynamic-programming module, then downselecting the (often >5,000) candidates per spectrum with the FastFilter energy model to a tractable top-256 set.

## When to use

Use when preparing MS/MS spectra (e.g., NPLIB1, NIST20, biomols, CASMI2022) for formula inference: candidate sets from mass decomposition are too large for exhaustive scoring and must be ranked/filtered before model training or evaluation.

## When NOT to use

- Input spectra are negative ion mode — MIST-CF/FastFilter models were trained only for positive-mode data, so candidate filtering would be unsupported.
- Candidates already come from a curated database search (e.g., NIST, GNPS, HMDB, PubChem) rather than de novo mass decomposition, so RDBE-permissive enumeration and FastFilter downselection are unnecessary.
- A NIST20-trained model is required for Orbitrap/high-resolution data but unavailable — the released quickstart model (public NPLIB1/CANOPUS data) may underperform on such spectra.

## Inputs

- MS/MS training spectra with precursor (MS1) exact masses (e.g., preprocessed NPLIB1, NPLIB1+NIST20, biomols, or CASMI 2022 MGF datasets)
- Enumerated RDBE-permissive formula candidate sets per spectrum, produced by SIRIUS's dynamic-programming 'decomp' module
- Pretrained FastFilter checkpoint (trained per src/mist_cf/fast_form_score/train.py with configs/fast_filter.yaml)
- SIRIUS binary with SIRIUS_PATH environment variable set

## Outputs

- Per-spectrum ranked formula candidate lists (FastFilter-scored, top-256 downselection)
- Pre-filter candidate-count distribution across spectra
- fast_filter_stats.csv summary table of top-256 true-formula recovery rates

## How to apply

First install the SIRIUS framework (e.g., version 5.5.7 headless from the boecker-lab/sirius releases) and export its location as the SIRIUS_PATH environment variable, since MIST-CF relies on SIRIUS's 'decomp' module to enumerate all potential formulae for an observed MS1 mass via dynamic programming, restricted to RDBE-plausible formulae. Next, set up the mist-cf repository by cloning it and creating the mamba environment from environment.yml, then either use the pretrained FastFilter checkpoint or retrain it with run_scripts/public_data_train/train_fast_filter.sh using settings from configs/fast_filter.yaml (the fast filter model is trained on the biomols dataset of biologically relevant molecules). Run src/mist_cf/fast_form_score/predict.py to score every candidate formula against its spectrum with the FastFilter energy model and rank candidates per spectrum; expect roughly 15% of NPLIB1/NIST20 training spectra to carry more than 5,000 RDBE-permissive candidates, making this downselection essential for tractability. Finally, evaluate rankings with run_scripts/fast_filter/launch_fast_pred.py and write fast_filter_stats.csv; correct execution is evidenced by ~99% top-256 recovery of the true formula on training spectra.

## Related tools

- **SIRIUS** (Provides the 'SIRIUS decomp' dynamic-programming module used to enumerate all potential formulae for an observed MS1 mass, restricted to RDBE-plausible formulae; its fast implementation is why MIST-CF still relies on it for candidate generation.) — https://github.com/boecker-lab/sirius
- **FastFilter** (Light-weight neural network energy model that scores and ranks enumerated formula candidates against each spectrum, downselecting the RDBE-permissive set to a top-256 list retaining the true formula ~99% of the time.) — https://github.com/samgoldman97/mist-cf

## Examples

```
device=linux64; version=5.5.7; wget https://github.com/boecker-lab/sirius/releases/download/v$version/sirius-$version-$device-headless.zip && echo 'export SIRIUS_PATH=${your_sirius_path}' >> ~/.bashrc
```

## Evaluation signals

- Pre-filter candidate-count distribution shows ~15% of NPLIB1/NIST20 training spectra with more than 5,000 RDBE-permissive formula candidates, confirming 'decomp' ran with correct element/RDBE settings.
- FastFilter top-256 recovery rate of the true formula reaches approximately 99% on training spectra, matching the reported threshold.
- fast_filter_stats.csv is produced by run_scripts/fast_filter/launch_fast_pred.py with per-spectrum rankings and recovery summary rows.
- Every spectrum receives a non-empty ranked candidate list; spectra whose true formula falls outside the top-256 should be flagged as pipeline failures rather than silently dropped.

## Limitations

- Relies on SIRIUS's algorithmic decomposition of exact masses into formula candidates; a SIRIUS binary (SIRIUS_PATH) is a hard dependency even when all scoring is learned.
- Models were trained only for positive-mode data, so negative-mode spectra are out of scope.
- The released quickstart model (trained on public NPLIB1/CANOPUS data) may be less performant than NIST20-trained models, particularly for Orbitrap or higher resolution data; NIST20-trained model downloads require a NIST license.
- Filtering is capped at the top 256 candidates; any spectrum whose true formula ranks below 256 is lost to downstream steps.

## Evidence
