---
name: fingerprint-retrieval-accuracy-evaluation
description: Use when a fine-tuned tandem-MS model outputs molecular fingerprints
  and you must quantify molecular-structure retrieval quality against MIST/FFN baselines
  on the CANOPUS benchmark (8,000 spectra, 7,000 unique molecules).
when_to_use_negative:
- The goal is library spectral matching by embedding similarity or one of the 44 standard
  spectral similarity measures — use the retrieval/similarity fine-tuning evaluation
  instead, not fingerprint accuracy@k
- The benchmark requires de novo formula or precursor chemical-formula prediction
  — that is MIST-CF territory, not fingerprint-based retrieval
- Production-grade fingerprint prediction is required — the article explicitly frames
  fingerprint fine-tuning as proof of concept, not a production-ready method
edam_operation: http://edamontology.org/operation_2945
tools:
- name: DreaMS
  role: Pre-trained 116M-parameter transformer fine-tuned with a DeepSets-like head
    to predict 4,096-bit Morgan fingerprints from MS/MS spectra
  repo: https://github.com/pluskal-lab/DreaMS
- name: MIST
  role: Deep-learning baseline whose reported accuracy@k defines the 'on par' range;
    also source of the CANOPUS benchmark
  repo: https://github.com/samgoldman97/mist
- name: RDKit
  role: Morgan fingerprint computation and molecular-structure retrieval against PubChem
- name: PyTorch
  role: Training and inference for the fine-tuned fingerprint-prediction and property-prediction
    heads
- name: HDF5
  role: Compact binary storage format for the GeMS MS/MS spectra used in fine-tuning
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:087b4caea061a9c078f986f31bb432762102f345d8b5eaa870b44c96c8f31c51
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pluskal-lab/DreaMS
  tools:
  - DreaMS
  - HDF5
  - MIST
  - PyTorch
  - RDKit
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Fingerprint-retrieval accuracy evaluation

## Summary

Benchmark protocol for judging a fine-tuned MS/MS neural network (e.g. DreaMS) by how accurately its predicted 4,096-bit Morgan fingerprints retrieve molecular structures from PubChem, using accuracy@k on the MIST CANOPUS benchmark against the MIST and FFN baselines.

## When to use

Use when a fine-tuned tandem-MS model outputs molecular fingerprints and you must quantify molecular-structure retrieval quality against MIST/FFN baselines on the CANOPUS benchmark (8,000 spectra, 7,000 unique molecules).

## When NOT to use

- The goal is library spectral matching by embedding similarity or one of the 44 standard spectral similarity measures — use the retrieval/similarity fine-tuning evaluation instead, not fingerprint accuracy@k
- The benchmark requires de novo formula or precursor chemical-formula prediction — that is MIST-CF territory, not fingerprint-based retrieval
- Production-grade fingerprint prediction is required — the article explicitly frames fingerprint fine-tuning as proof of concept, not a production-ready method

## Inputs

- MS/MS spectra paired with molecular structures (fine-tuning datasets from DreaMS Zenodo records 10997887 and 13843034, HDF5-based binary format)
- MIST CANOPUS benchmark: 8,000 spectra from 7,000 unique molecules (from the official MIST GitHub repository)
- Pre-trained DreaMS model weights (Zenodo record 10997887)
- PubChem compound database as the retrieval library for structure lookup

## Outputs

- Predicted 4,096-bit Morgan fingerprint vectors per spectrum
- accuracy@k retrieval scores for DreaMS vs MIST and FFN baselines
- Relative-prediction-error curves for the Bertz property vs XGBoost and MS2Prop
- dreams_finetuning_results.csv aggregating accuracy@k and Bertz error-curve results

## How to apply

Attach a DeepSets-like head to the pre-trained 116M-parameter DreaMS transformer and fine-tune it on MS/MS spectra paired with molecular structures (from Zenodo records 10997887 and 13843034) to predict 4,096-bit Morgan fingerprints. Optionally train a linear head on the same fine-tuned network to predict 11 molecular properties (QED, synthetic accessibility, Bertz complexity, etc.). Convert predicted fingerprints into molecular-structure retrieval against PubChem using RDKit, and compute accuracy@k on the MIST CANOPUS benchmark obtained from the official MIST GitHub repository (github.com/samgoldman97/mist), comparing against MIST and FFN baselines. Separately compute relative-prediction-error curves for the Bertz property against XGBoost and MS2Prop baselines. Aggregate accuracy@k scores and Bertz error curves into a dreams_finetuning_results.csv file. The success criterion is that fine-tuned DreaMS performs on par with MIST — accuracy@k within MIST's reported range — and the Bertz error curve matches the published comparison; if quantitative thresholds cannot be re-derived from the deposit, accept reproduced scores via expert review against the published figures.

## Related tools

- **DreaMS** (Pre-trained 116M-parameter transformer fine-tuned with a DeepSets-like head to predict 4,096-bit Morgan fingerprints from MS/MS spectra) — https://github.com/pluskal-lab/DreaMS
- **MIST** (Deep-learning baseline whose reported accuracy@k defines the 'on par' range; also source of the CANOPUS benchmark) — https://github.com/samgoldman97/mist
- **RDKit** (Morgan fingerprint computation and molecular-structure retrieval against PubChem)
- **PyTorch** (Training and inference for the fine-tuned fingerprint-prediction and property-prediction heads)
- **HDF5** (Compact binary storage format for the GeMS MS/MS spectra used in fine-tuning)

## Examples

```
. quickstart/00_download_models.sh && . quickstart/01_run_models.sh
```

## Evaluation signals

- accuracy@k on the MIST CANOPUS benchmark falls within MIST's reported range (the 'on par' success criterion)
- Predicted fingerprints are 4,096-bit Morgan fingerprints and retrieve matching structures from PubChem
- Bertz relative-prediction-error curve reproduces the paper's comparison against XGBoost and MS2Prop baselines
- Benchmark size matches the published 8,000 spectra / 7,000 unique molecules CANOPUS setup
- All accuracy@k scores and Bertz curves are aggregated into dreams_finetuning_results.csv

## Limitations

- The MIST CANOPUS benchmark is limited in size, covering 8,000 spectra from 7,000 unique molecules
- Fingerprint-prediction fine-tuning should be understood as proof of concept that DreaMS can achieve competitive performance with MIST, not a production-ready annotator
- If quantitative thresholds are not re-derivable from the Zenodo deposit, the reproduced benchmark scores must be accepted by expert review against the published figures

## Evidence

- [abstract] on-par performance vs MIST: "the fine-tuned DreaMS neural network performs on par with the deep learning model MIST in the retrieval of molecular structures using predicted fingerprints"
- [discussion] benchmark source and size: "we use the CANOPUS benchmark from the official GitHub repository (https://github.com/samgoldman97/mist)"
- [discussion] benchmark size limitation: "the MIST CANOPUS benchmark is limited in size, covering 8,000 spectra from 7,000 unique molecules"
- [discussion] fingerprint prediction scope: "should be understood as a proof of concept that our approach can achieve competitive performance with MIST"
- [readme] MIST model capabilities: [evidence span withheld — anchor status: external_doc]
