---
name: bertz-relative-prediction-error-analysis
description: Use when fine-tuning a pre-trained DreaMS transformer on MS/MS spectra
  for molecular-property prediction and you need to benchmark Bertz complexity regression
  against XGBoost and MS2Prop baselines.
when_to_use_negative:
- The task is molecular-structure retrieval from fingerprints rather than property
  regression — use the fingerprint accuracy@k evaluation on the MIST CANOPUS benchmark
  instead
- The goal is zero-shot spectral similarity or library matching — fine-tuned property-prediction
  heads are not the appropriate DreaMS mode for those tasks
- Input spectra are not tandem MS/MS spectra; the method disregards MS1 isotopic patterns,
  adduct distributions, negative ionization mode, and multiply charged precursors
edam_operation: http://edamontology.org/operation_2451
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_0091
tools:
- name: DreaMS
  role: Pre-trained 116M-parameter transformer fine-tuned with a linear head to predict
    11 molecular properties including Bertz complexity from MS/MS spectra
  repo: https://github.com/pluskal-lab/DreaMS
- name: PyTorch
  role: Framework for loading the DreaMS checkpoint, attaching the linear property-prediction
    head, and fine-tuning/evaluating on held-out spectra
- name: RDKit
  role: Computation of Bertz complexity (and other molecular properties such as QED
    and synthetic accessibility) from molecular structures as regression targets and
    for error-curve evaluation
- name: XGBoost
  role: Baseline regressor whose Bertz relative-prediction-error curve is compared
    against fine-tuned DreaMS
- name: MS2Prop
  role: State-of-the-art MS/MS property-prediction baseline against which the Bertz
    relative-prediction-error curve is benchmarked
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:71fc25452a7e3110471aec0595625cf15f24cc87b0e93ef7ab970e57bc177aa5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pluskal-lab/DreaMS
  tools:
  - DreaMS
  - MIST
  - MS2Prop
  - PyTorch
  - RDKit
  - XGBoost
  edam_operation: http://edamontology.org/operation_2451
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# Bertz relative-prediction-error analysis

## Summary

Evaluation protocol for the fine-tuned DreaMS property-prediction head, benchmarking molecular-property regression (QED, synthetic accessibility, Bertz complexity) via relative-prediction-error curves for Bertz against XGBoost and MS2Prop baselines.

## When to use

Use when fine-tuning a pre-trained DreaMS transformer on MS/MS spectra for molecular-property prediction and you need to benchmark Bertz complexity regression against XGBoost and MS2Prop baselines.

## When NOT to use

- The task is molecular-structure retrieval from fingerprints rather than property regression — use the fingerprint accuracy@k evaluation on the MIST CANOPUS benchmark instead
- The goal is zero-shot spectral similarity or library matching — fine-tuned property-prediction heads are not the appropriate DreaMS mode for those tasks
- Input spectra are not tandem MS/MS spectra; the method disregards MS1 isotopic patterns, adduct distributions, negative ionization mode, and multiply charged precursors

## Inputs

- Pre-trained DreaMS model weights (Zenodo record 10997887)
- Fine-tuning datasets of MS/MS spectra paired with molecular structures (Zenodo record 13843034, HDF5-based binary format)
- Bertz complexity reference values computed from molecular structures (SMILES) via RDKit
- XGBoost and MS2Prop baseline relative-prediction-error results from the published paper

## Outputs

- Relative-prediction-error curves for the Bertz complexity property
- Comparison of DreaMS property predictions against XGBoost and MS2Prop baselines
- dreams_finetuning_results.csv aggregating accuracy@k scores and Bertz error-curve results

## How to apply

Load the pre-trained 116-million-parameter DreaMS transformer and the MS/MS–structure fine-tuning datasets from the DreaMS Zenodo deposits (records 10997887 and 13843034). Attach a linear head on top of the fine-tuned network and train it to predict 11 molecular properties (QED, synthetic accessibility, Bertz complexity and others) from raw tandem mass spectra. After training, compute relative-prediction-error curves specifically for the Bertz property on the held-out evaluation spectra and compare the curve against the XGBoost and MS2Prop baselines as reported in the paper. Aggregate the accuracy@k scores from the companion fingerprint-retrieval task and the Bertz error-curve results into dreams_finetuning_results.csv. Success is judged by the Bertz relative-prediction-error curve matching the paper's reported comparison against XGBoost and MS2Prop; if quantitative thresholds are not re-derivable from the deposit, accept the reproduced benchmark scores by expert review against the published figures.

## Related tools

- **DreaMS** (Pre-trained 116M-parameter transformer fine-tuned with a linear head to predict 11 molecular properties including Bertz complexity from MS/MS spectra) — https://github.com/pluskal-lab/DreaMS
- **PyTorch** (Framework for loading the DreaMS checkpoint, attaching the linear property-prediction head, and fine-tuning/evaluating on held-out spectra)
- **RDKit** (Computation of Bertz complexity (and other molecular properties such as QED and synthetic accessibility) from molecular structures as regression targets and for error-curve evaluation)
- **XGBoost** (Baseline regressor whose Bertz relative-prediction-error curve is compared against fine-tuned DreaMS)
- **MS2Prop** (State-of-the-art MS/MS property-prediction baseline against which the Bertz relative-prediction-error curve is benchmarked)

## Examples

```
pip install -r requirements.txt
```

## Evaluation signals

- The Bertz relative-prediction-error curve qualitatively matches the paper's reported comparison against XGBoost and MS2Prop baselines
- The fine-tuned network correctly predicts all 11 molecular properties (QED, synthetic accessibility, Bertz complexity and others) from the same fine-tuned backbone
- Companion fingerprint accuracy@k on the MIST CANOPUS benchmark (8,000 spectra from 7,000 unique molecules) falls within MIST's reported range
- All scores are aggregated into dreams_finetuning_results.csv with both error-curve and accuracy@k results
- If quantitative thresholds are not re-derivable from the Zenodo deposit, reproduced scores are accepted only via expert review against the published figures

## Limitations

- Fine-tuning of DreaMS for fingerprint/property prediction is a proof of concept, not a production-ready tool
- The MIST CANOPUS benchmark is limited in size, covering 8,000 spectra from 7,000 unique molecules
- Only positive-ionization, singly charged MS/MS spectra are covered; MS1 isotopic patterns and adduct distributions are disregarded

## Evidence

- [abstract] property-prediction head fine-tuned for 11 properties including Bertz: "we adapted the network to the prediction of spectral similarity, molecular fingerprints, chemical properties and the identification of fluorine-containing molecules"
- [other] Bertz relative-prediction-error protocol and baselines: [evidence span withheld — anchor status: not_found]
- [other] property head training detail: [evidence span withheld — anchor status: not_found]
- [discussion] proof-of-concept caveat: "should be understood as a proof of concept that our approach can achieve competitive performance with MIST"
- [discussion] benchmark size limitation: "the MIST CANOPUS benchmark is limited in size, covering 8,000 spectra from 7,000 unique molecules"
