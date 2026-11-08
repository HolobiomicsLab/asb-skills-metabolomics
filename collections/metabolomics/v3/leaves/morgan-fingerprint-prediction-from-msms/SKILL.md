---
name: morgan-fingerprint-prediction-from-msms
description: Use when raw tandem MS/MS spectra must be converted to 4,096-bit Morgan
  fingerprints for molecular-structure retrieval from PubChem, or when benchmarking
  fingerprint-based retrieval against MIST on the CANOPUS benchmark.
when_to_use_negative:
- Spectra are in negative ionization mode or from multiply charged precursor ions
  — the model was trained only on positive-mode, singly charged data, so fine-tuned
  fingerprint prediction is out of scope
- MS1-level information (isotopic patterns, adduct distributions) is essential to
  the annotation — DreaMS focuses solely on tandem MS/MS and disregards these features
- A production-grade fingerprint predictor is required — the paper states DreaMS fingerprint
  fine-tuning 'should be understood as a proof of concept that our approach can achieve
  competitive performance with MIST', not a deployable tool
tools:
- name: PyTorch
  role: Framework for attaching the DeepSets-like fingerprint head and linear property
    head to the pre-trained DreaMS transformer and fine-tuning
- name: RDKit
  role: Computing the 4,096-bit Morgan fingerprint labels used as fine-tuning targets
    and for property predictions (QED, Bertz, synthetic accessibility)
- name: MIST
  role: Baseline deep-learning fingerprint predictor whose accuracy@k on the CANOPUS
    benchmark defines the on-par success criterion
  repo: https://github.com/samgoldman97/mist
- name: HDF5
  role: Compact binary format in which the GeMS MS/MS spectra and fine-tuning datasets
    are stored for deep learning
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:e9dd9e477aa8336c918a1b35205f8b6780bca269dde6739f49adef41925cdcee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/samgoldman97/mist
  tools:
  - HDF5
  - MIST
  - PyTorch
  - RDKit
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# morgan-fingerprint-prediction-from-msms

## Summary

Predicts 4,096-bit Morgan fingerprints from raw MS/MS spectra by attaching a DeepSets-like prediction head to the self-supervised pre-trained DreaMS transformer (116M parameters) and fine-tuning it on spectra paired with molecular structures; predicted fingerprints are then used to retrieve molecular structures from PubChem, benchmarked against MIST.

## When to use

Use when raw tandem MS/MS spectra must be converted to 4,096-bit Morgan fingerprints for molecular-structure retrieval from PubChem, or when benchmarking fingerprint-based retrieval against MIST on the CANOPUS benchmark.

## When NOT to use

- Spectra are in negative ionization mode or from multiply charged precursor ions — the model was trained only on positive-mode, singly charged data, so fine-tuned fingerprint prediction is out of scope
- MS1-level information (isotopic patterns, adduct distributions) is essential to the annotation — DreaMS focuses solely on tandem MS/MS and disregards these features
- A production-grade fingerprint predictor is required — the paper states DreaMS fingerprint fine-tuning 'should be understood as a proof of concept that our approach can achieve competitive performance with MIST', not a deployable tool

## Inputs

- Raw tandem MS/MS spectra (positive-ionization mode, singly charged precursors) in the DreaMS HDF5-based binary format from Zenodo records 10997887 and 13843034
- Paired MS/MS spectra–molecular structure fine-tuning datasets from the DreaMS Zenodo deposit
- MIST CANOPUS benchmark (8,000 spectra from 7,000 unique molecules) from github.com/samgoldman97/mist
- Pre-trained DreaMS model weights (Zenodo record 10997887)

## Outputs

- 4,096-bit Morgan fingerprint probability/prediction vectors per input MS/MS spectrum
- Retrieved molecular structures ranked by accuracy@k against PubChem
- Relative-prediction-error curves for Bertz complexity vs XGBoost and MS2Prop baselines
- Aggregated benchmark scores in dreams_finetuning_results.csv

## How to apply

Load the pre-trained DreaMS transformer and the fine-tuning datasets of MS/MS spectra paired with molecular structures from the DreaMS Zenodo deposits (records 10997887 and 13843034), stored in the HDF5-based binary format designed for deep learning. Attach a DeepSets-like head to the frozen pre-trained backbone and fine-tune it to predict 4,096-bit Morgan fingerprints from each spectrum, computed with RDKit from paired structures. Optionally add a linear head on the same fine-tuned network to predict 11 molecular properties (QED, synthetic accessibility, Bertz complexity and others). Evaluate fingerprint retrieval accuracy@k on the MIST CANOPUS benchmark (8,000 spectra from 7,000 unique molecules, from the official MIST GitHub repository) and compare accuracy@k against MIST and FFN baselines; separately compute relative-prediction-error curves for Bertz complexity against XGBoost and MS2Prop. Success means accuracy@k falls within MIST's reported range on CANOPUS and the Bertz error curve matches the published comparison; if thresholds are not re-derivable, accept reproduced scores by expert review against the published figures.

## Related tools

- **PyTorch** (Framework for attaching the DeepSets-like fingerprint head and linear property head to the pre-trained DreaMS transformer and fine-tuning)
- **RDKit** (Computing the 4,096-bit Morgan fingerprint labels used as fine-tuning targets and for property predictions (QED, Bertz, synthetic accessibility))
- **MIST** (Baseline deep-learning fingerprint predictor whose accuracy@k on the CANOPUS benchmark defines the on-par success criterion) — https://github.com/samgoldman97/mist
- **HDF5** (Compact binary format in which the GeMS MS/MS spectra and fine-tuning datasets are stored for deep learning)

## Evaluation signals

- Fingerprint retrieval accuracy@k on the MIST CANOPUS benchmark (8,000 spectra, 7,000 unique molecules) falls within MIST's reported range
- Comparison against MIST and FFN baselines reproduces the published Fig. 4e / Extended Data Table 1 pattern of on-par performance
- Bertz relative-prediction-error curves match the paper's comparison against XGBoost and MS2Prop baselines
- Results aggregate cleanly into dreams_finetuning_results.csv with per-benchmark accuracy@k and error-curve entries
- If quantitative thresholds are not re-derivable from the deposit, reproduced scores accepted only via expert review against published figures

## Limitations

- The MIST CANOPUS benchmark is limited in size, covering 8,000 spectra from 7,000 unique molecules, so on-par performance is evaluated on a small test set
- Fingerprint prediction fine-tuning is a proof of concept, not production-ready
- Method uses only tandem mass spectra, disregarding MS1 isotopic patterns and adduct distributions, and was trained only on positive-mode, singly charged spectra

## Evidence

- [abstract] on-par performance with MIST: "the fine-tuned DreaMS neural network performs on par with the deep learning model MIST in the retrieval of molecular structures using predicted fingerprints"
- [discussion] CANOPUS benchmark provenance: "we use the CANOPUS benchmark from the official GitHub repository (https://github.com/samgoldman97/mist)"
- [discussion] benchmark size limitation: "the MIST CANOPUS benchmark is limited in size, covering 8,000 spectra from 7,000 unique molecules"
- [discussion] proof-of-concept status: "should be understood as a proof of concept that our approach can achieve competitive performance with MIST"
- [readme] MIST fingerprint-prediction capability: [evidence span withheld — anchor status: external_doc]
