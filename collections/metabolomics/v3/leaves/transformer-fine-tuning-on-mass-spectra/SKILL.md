---
name: transformer-fine-tuning-on-mass-spectra
description: Use when raw tandem MS/MS spectra must be mapped to predicted molecular
  fingerprints or chemical-property values, and retrieval accuracy should be compared
  against transformer baselines like MIST on the CANOPUS benchmark.
when_to_use_negative:
- Spectra are in negative ionization mode or from multiply charged precursor ions
  — the model was trained only on positive-mode, singly charged data
- The task requires MS1 isotopic patterns or adduct distributions — the method 'is
  focused solely on tandem mass spectra, disregarding other important features'
- A production-ready structure-annotation pipeline is required — the fingerprint fine-tuning
  'should be understood as a proof of concept that our approach can achieve competitive
  performance with MIST'
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3382
tools:
- name: DreaMS
  role: Pre-trained 116M-parameter transformer backbone that is attached to DeepSets-like
    (fingerprints) and linear (properties) heads and fine-tuned on spectra from the
    Zenodo deposits
  repo: github.com__pluskal-lab__DreaMS
- name: PyTorch
  role: Framework for attaching heads to the transformer and running the fine-tuning
    and evaluation loops
- name: RDKit
  role: Computation of 4,096-bit Morgan fingerprints and the 11 chemical properties
    (QED, synthetic accessibility, Bertz complexity) used as fine-tuning labels
- name: MIST
  role: Deep-learning baseline whose accuracy@k retrieval range on the CANOPUS benchmark
    defines the success criterion for the fine-tuned DreaMS model
  repo: https://github.com/samgoldman97/mist
- name: HDF5
  role: Compact binary format in which GeMS spectra and selected LC–MS/MS metadata
    are stored for deep-learning fine-tuning
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:2521ab945ba0ee86c2f7050b533d39f24c834b7f67cccfee6d578340c61737b5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DreaMS
  - HDF5
  - MIST
  - PyTorch
  - RDKit
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Transformer fine-tuning on mass spectra

## Summary

Adapts the pre-trained 116M-parameter DreaMS transformer by attaching task-specific heads (DeepSets-like head for 4,096-bit Morgan fingerprints; linear head for 11 molecular properties such as QED, synthetic accessibility and Bertz complexity) and fine-tuning on MS/MS spectra paired with molecular structures, benchmarked against MIST on the CANOPUS benchmark.

## When to use

Use when raw tandem MS/MS spectra must be mapped to predicted molecular fingerprints or chemical-property values, and retrieval accuracy should be compared against transformer baselines like MIST on the CANOPUS benchmark.

## When NOT to use

- Spectra are in negative ionization mode or from multiply charged precursor ions — the model was trained only on positive-mode, singly charged data
- The task requires MS1 isotopic patterns or adduct distributions — the method 'is focused solely on tandem mass spectra, disregarding other important features'
- A production-ready structure-annotation pipeline is required — the fingerprint fine-tuning 'should be understood as a proof of concept that our approach can achieve competitive performance with MIST'

## Inputs

- MS/MS spectra paired with molecular structures from DreaMS Zenodo records 10997887 and 13843034 (stored in the DreaMS HDF5-based binary format)
- Pre-trained DreaMS model weights (Zenodo record 10997887)
- MIST CANOPUS benchmark: 8,000 spectra from 7,000 unique molecules (from github.com/samgoldman97/mist)
- RDKit-computable 4,096-bit Morgan fingerprints of the paired molecular structures

## Outputs

- Fine-tuned DreaMS model with DeepSets-like head predicting 4,096-bit Morgan fingerprints from spectra
- Fine-tuned DreaMS model with linear head predicting 11 molecular properties (QED, synthetic accessibility, Bertz complexity, others)
- accuracy@k retrieval scores on the CANOPUS benchmark vs MIST and FFN baselines
- Relative-prediction-error curves for the Bertz property vs XGBoost and MS2Prop baselines
- Aggregated results file dreams_finetuning_results.csv

## How to apply

Load the pre-trained DreaMS transformer weights and the MS/MS spectra paired with molecular structures from the DreaMS Zenodo deposits (records 10997887 and 13843034). Attach a DeepSets-like head to the frozen-or-tuned backbone and fine-tune it to predict 4,096-bit Morgan fingerprints from the raw spectra (fingerprints computable with RDKit from paired structures). Then add a linear head on top of the same fine-tuned network to regress 11 molecular properties (QED, synthetic accessibility, Bertz complexity, etc.). Evaluate fingerprint retrieval accuracy@k on the MIST CANOPUS benchmark (8,000 spectra from 7,000 unique molecules, obtained from the official MIST GitHub repository) and compare with MIST and FFN baselines; compute relative-prediction-error curves for the Bertz property against XGBoost and MS2Prop. Aggregate accuracy@k scores and Bertz error curves into dreams_finetuning_results.csv; the success criterion is DreaMS accuracy@k within MIST's reported range on CANOPUS, with expert review against published figures if thresholds cannot be re-derived from the deposit.

## Related tools

- **DreaMS** (Pre-trained 116M-parameter transformer backbone that is attached to DeepSets-like (fingerprints) and linear (properties) heads and fine-tuned on spectra from the Zenodo deposits) — github.com__pluskal-lab__DreaMS
- **PyTorch** (Framework for attaching heads to the transformer and running the fine-tuning and evaluation loops)
- **RDKit** (Computation of 4,096-bit Morgan fingerprints and the 11 chemical properties (QED, synthetic accessibility, Bertz complexity) used as fine-tuning labels)
- **MIST** (Deep-learning baseline whose accuracy@k retrieval range on the CANOPUS benchmark defines the success criterion for the fine-tuned DreaMS model) — https://github.com/samgoldman97/mist
- **HDF5** (Compact binary format in which GeMS spectra and selected LC–MS/MS metadata are stored for deep-learning fine-tuning)

## Examples

```
git clone https://github.com/pluskal-lab/DreaMS && cd DreaMS && pip install -r requirements.txt && ln -s ../tutorials tutorials
```

## Evaluation signals

- Fine-tuned DreaMS accuracy@k on the CANOPUS benchmark (8,000 spectra / 7,000 molecules) falls within MIST's reported range for molecular-structure retrieval via predicted fingerprints
- Relative-prediction-error curve for the Bertz property matches the paper's reported comparison against XGBoost and MS2Prop baselines
- Predicted fingerprint outputs have exactly 4,096 bits (Morgan fingerprint dimensionality) and the property head outputs exactly 11 values per spectrum
- Reproduced scores are aggregated into dreams_finetuning_results.csv with accuracy@k and Bertz error-curve entries
- If quantitative thresholds are not re-derivable from the Zenodo deposit, reproduced benchmark scores pass expert review against the published figures

## Limitations

- The MIST CANOPUS benchmark is limited in size, covering 8,000 spectra from 7,000 unique molecules
- Fingerprint fine-tuning is a proof of concept, not a production-ready annotation method
- Only a subset of available mass spectra was used for training; the full potential of training on raw unannotated spectra remains to be unlocked
- More diverse data (negative ionization mode, multiply charged precursors) are not incorporated, and MS1 isotopic patterns and adduct distributions are disregarded
- More advanced fine-tuning protocols are deferred to future work within the MassSpecGym benchmarking environment

## Evidence

- [abstract] Fine-tuned DreaMS on par with MIST on fingerprint retrieval: "the fine-tuned DreaMS neural network performs on par with the deep learning model MIST in the retrieval of molecular structures using predicted fingerprints"
- [abstract] Supervised fine-tuning tasks covered by the network: "we adapted the network to the prediction of spectral similarity, molecular fingerprints, chemical properties and the identification of fluorine-containing molecules"
- [discussion] CANOPUS benchmark source: "we use the CANOPUS benchmark from the official GitHub repository (https://github.com/samgoldman97/mist)"
- [discussion] Benchmark size limitation: "the MIST CANOPUS benchmark is limited in size, covering 8,000 spectra from 7,000 unique molecules"
- [discussion] Proof-of-concept caveat: "should be understood as a proof of concept that our approach can achieve competitive performance with MIST"
- [methods] Environment setup for running the code: "# Install requirements
pip install -r requirements.txt"
