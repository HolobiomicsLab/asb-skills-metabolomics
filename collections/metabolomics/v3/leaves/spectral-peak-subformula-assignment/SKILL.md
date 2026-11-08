---
name: spectral-peak-subformula-assignment
description: Use when you have tandem MS/MS spectra with candidate precursor formulae
  (e.g. from SIRIUS decomp on NPLIB1/CASMI22) and need peak-wise subformula labels
  to score formula candidates with a neural model like MIST-CF without fragmentation
  trees.
when_to_use_negative:
- The input is negative-mode MS/MS data — the released models were trained only for
  positive mode, so subformula-assignment-based scoring is out of scope
- You already have fragmentation trees from SIRIUS and want the canonical tree-based
  pipeline rather than the tree-free subformula protocol
- You need to integrate MIST-CF candidate scores with MS1 isotopic scores — the paper
  explicitly does not address this integration
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MIST-CF
  role: Energy-based neural model whose architecture includes the internal subformula
    assignment protocol for scoring candidate formula–adduct assignments against MS2
    spectra
  repo: https://github.com/samgoldman97/mist-cf
- name: SIRIUS
  role: Provides the `SIRIUS decomp` dynamic-programming module used to enumerate
    candidate formulae for an observed MS1 mass before subformula assignment and scoring
  repo: https://github.com/boecker-lab/sirius/releases/download/
- name: FastFilter
  role: Lightweight neural formula filtering model that uses the same subformula featurization
    to prefilter candidates before MIST-CF re-ranking
  repo: https://github.com/samgoldman97/mist-cf
- name: Chemical Formula Transformer
  role: Prior Transformer architecture whose tree-based featurization MIST-CF replaces
    with the peak subformula assignment routine
  repo: https://github.com/samgoldman97/mist-cf
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jcim.3c01082
    title: 'MIST-CF: Chemical Formula Inference from Tandem Mass Spectra'
  source_content_hash: sha256:58345e72ff74535b35dad351526d28c75d89e6b4162e8bc12ea520932290aa56
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/samgoldman97/mist-cf
  tools:
  - Chemical Formula Transformer
  - FastFilter
  - MIST-CF
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# spectral-peak-subformula-assignment

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

A peak-level subformula assignment routine from MIST-CF that assigns chemical subformulae to individual MS2 fragment peaks and neutral losses from a candidate precursor formula, replacing SIRIUS fragmentation-tree construction as the spectral featurization for energy-based re-ranking of formula–adduct candidates.

## When to use

Use when you have tandem MS/MS spectra with candidate precursor formulae (e.g. from SIRIUS decomp on NPLIB1/CASMI22) and need peak-wise subformula labels to score formula candidates with a neural model like MIST-CF without fragmentation trees.

## When NOT to use

- The input is negative-mode MS/MS data — the released models were trained only for positive mode, so subformula-assignment-based scoring is out of scope
- You already have fragmentation trees from SIRIUS and want the canonical tree-based pipeline rather than the tree-free subformula protocol
- You need to integrate MIST-CF candidate scores with MS1 isotopic scores — the paper explicitly does not address this integration

## Inputs

- Positive-mode MS/MS spectra with precursor MS1 mass (e.g. .mgf, e.g. data/demo_specs.mgf from CASMI22)
- Candidate chemical formula–adduct assignments enumerated by SIRIUS decomp (up to 256 per spectrum)
- Instrument type covariate used to measure the MS/MS
- Trained MIST-CF (or FastFilter/FFN/xformer) model checkpoints
- Preprocessed NPLIB1 ('canopus_train') and/or CASMI22 datasets from Zenodo record 8151490

## Outputs

- Per-peak fragment subformula and neutral-loss fragment formula assignments for each candidate precursor formula
- Candidate scores (agreement between candidate formula and spectrum) used for re-ranking
- Top-1 / top-k ranked formula–adduct assignments per spectrum (e.g. ~74% top-1 accuracy on NPLIB1)
- benchmarking_results.csv-style accuracy table when run via run_scripts/benchmarking/eval_models.py

## How to apply

For each unknown spectrum, enumerate candidate precursor formulae from the MS1 mass using SIRIUS's `SIRIUS decomp` dynamic-programming module (SIRIUS v5.5.7 headless, set via the SIRIUS_PATH environment variable). For each candidate formula–adduct pair (multiple positive-mode adducts beyond [M+H]+ are supported), assign subformulae to each MS2 fragment peak and to the neutral loss fragment formula rather than building fragmentation trees — this subformula assignment protocol was introduced in MIST-CF to replace tree-based featurization. The assigned fragment and neutral-loss subformulae, together with the instrument type, are then embedded (via sinusoidal formula embeddings as in SCARF) and fed to the Transformer that scores agreement between the candidate and the spectrum. Scores across all candidates are used to re-rank up to ~256 formula–adduct assignments per test spectrum; on NPLIB1 the MIST-CF subformula-Transformer achieved ~74% top-1 accuracy (0.741 NPLIB1-only, 0.769 with NIST20 added), beating the next best architecture by >10% absolute top-1 accuracy.

## Related tools

- **MIST-CF** (Energy-based neural model whose architecture includes the internal subformula assignment protocol for scoring candidate formula–adduct assignments against MS2 spectra) — https://github.com/samgoldman97/mist-cf
- **SIRIUS** (Provides the `SIRIUS decomp` dynamic-programming module used to enumerate candidate formulae for an observed MS1 mass before subformula assignment and scoring) — https://github.com/boecker-lab/sirius/releases/download/
- **FastFilter** (Lightweight neural formula filtering model that uses the same subformula featurization to prefilter candidates before MIST-CF re-ranking) — https://github.com/samgoldman97/mist-cf
- **Chemical Formula Transformer** (Prior Transformer architecture whose tree-based featurization MIST-CF replaces with the peak subformula assignment routine) — https://github.com/samgoldman97/mist-cf

## Examples

```
. quickstart/download_model.sh && . quickstart/run_model.sh  # scores data/demo_specs.mgf via MIST-CF subformula assignment
```

## Evaluation signals

- Re-ranking up to 256 formula and adduct candidates per held-out NPLIB1 test spectrum should yield ~0.741 top-1 accuracy for MIST-CF trained on NPLIB1 alone and ~0.769 with NIST20 added
- The MIST-CF subformula-Transformer should beat the next best architecture by a >10% absolute top-1 accuracy margin on NPLIB1
- Per-spectrum subformula assignments must cover both fragment peaks and neutral losses for each candidate formula–adduct pair
- Benchmark output benchmarking_results.csv should report per-model top-1 accuracy comparable to the paper's Table 1 (MIST-CF vs FFN, MS1-Only, Transformer)
- On positive-mode CASMI2022 spectra, MIST-CF should approach the winning SIRIUS solution's formula annotation accuracy without manual curation or postprocessing

## Limitations

- Models were trained only for positive-mode data; negative-mode MS/MS is unsupported
- MS1 isotopic scores are not integrated with MIST-CF candidate scores
- Adduct switching in MS2 subformula assignment is not considered
- The approach still relies on SIRIUS's fast algorithmic decomposition of exact masses into formula candidates
- Use of forward structure-to-spectrum models for data augmentation has not been explored

## Evidence
