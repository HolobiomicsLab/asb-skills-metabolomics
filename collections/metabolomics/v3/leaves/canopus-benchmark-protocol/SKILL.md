---
name: canopus-benchmark-protocol
description: Use when you must benchmark a fingerprint-prediction model on tandem
  MS/MS spectra against MIST on the CANOPUS benchmark, or when reproducing DreaMS's
  fingerprint retrieval and molecular-property prediction results.
when_to_use_negative:
- When the task is spectral similarity or spectral-library retrieval rather than fingerprint-based
  structure retrieval — the paper fine-tunes a separate DreaMS head for spectral similarity,
  where it 'substantially outperforms 44 standard spectral similarity measures'.
- When the input spectra are in negative ionization mode or have multiply charged
  precursor ions — the paper notes these were not incorporated in training.
- When a large-scale or fully generalizable fingerprint-prediction method is required
  — the paper states this fine-tuning 'should be understood as a proof of concept
  that our approach can achieve competitive performance with MIST' on a benchmark
  that 'is limited in size, covering 8,000 spectra from 7,000 unique molecules'.
edam_operation: http://edamontology.org/operation_4009
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: MIST
  role: Baseline deep-learning model for fingerprint prediction from spectra; its
    official GitHub repository supplies the CANOPUS benchmark used for accuracy@k
    evaluation
  repo: https://github.com/samgoldman97/mist
- name: DreaMS
  role: Pre-trained 116M-parameter transformer fine-tuned with a DeepSets-like head
    to predict 4,096-bit Morgan fingerprints and a linear head for 11 molecular properties
- name: PyTorch
  role: Framework for attaching and fine-tuning the DeepSets-like fingerprint head
    and linear property head on the pre-trained DreaMS network
- name: RDKit
  role: Computation of Morgan fingerprints and the 11 molecular properties (QED, synthetic
    accessibility, Bertz complexity) from molecular structures
- name: HDF5
  role: Compact binary format in which the GeMS spectra and metadata are stored for
    deep learning inputs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:cc38d0cc6f81e078e9b38bd4460248cbde67b275521baafebcc427f8b285a9a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/samgoldman97/mist
  tools:
  - DreaMS
  - HDF5
  - MIST
  - PyTorch
  - RDKit
  edam_operation: http://edamontology.org/operation_4009
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# canopus-benchmark-protocol

## Summary

A protocol for evaluating MS/MS-to-structure models by predicting 4,096-bit Morgan fingerprints from tandem mass spectra and retrieving candidate structures from PubChem, scored by accuracy@k on the MIST CANOPUS benchmark (8,000 spectra, 7,000 unique molecules), with fine-tuned DreaMS compared against the MIST and FFN baselines.

## When to use

Use when you must benchmark a fingerprint-prediction model on tandem MS/MS spectra against MIST on the CANOPUS benchmark, or when reproducing DreaMS's fingerprint retrieval and molecular-property prediction results.

## When NOT to use

- When the task is spectral similarity or spectral-library retrieval rather than fingerprint-based structure retrieval — the paper fine-tunes a separate DreaMS head for spectral similarity, where it 'substantially outperforms 44 standard spectral similarity measures'.
- When the input spectra are in negative ionization mode or have multiply charged precursor ions — the paper notes these were not incorporated in training.
- When a large-scale or fully generalizable fingerprint-prediction method is required — the paper states this fine-tuning 'should be understood as a proof of concept that our approach can achieve competitive performance with MIST' on a benchmark that 'is limited in size, covering 8,000 spectra from 7,000 unique molecules'.

## Inputs

- MIST CANOPUS benchmark: 8,000 MS/MS spectra from 7,000 unique molecules (from the official MIST GitHub repository)
- Pre-trained DreaMS model weights (Zenodo record 10997887)
- Fine-tuning dataset of MS/MS spectra paired with molecular structures (Zenodo record 13843034)
- DreaMS-format HDF5 spectra files
- PubChem compound library as the structure-retrieval candidate database
- Morgan fingerprint (4,096-bit) labels computed from molecular structures via RDKit

## Outputs

- Accuracy@k retrieval scores for fine-tuned DreaMS, MIST, and FFN on the CANOPUS benchmark
- Relative-prediction-error curves for the Bertz property versus XGBoost and MS2Prop baselines
- Predicted 4,096-bit Morgan fingerprints and 11 molecular-property values (QED, synthetic accessibility, Bertz complexity, etc.) per spectrum
- Aggregated benchmark table: dreams_finetuning_results.csv

## How to apply

Obtain the CANOPUS benchmark from the official MIST GitHub repository (github.com/samgoldman97/mist) and the pre-trained DreaMS weights plus MS/MS–structure fine-tuning pairs from the DreaMS Zenodo deposits (records 10997887 and 13843034). Fine-tune the 116-million-parameter pre-trained DreaMS transformer by attaching a DeepSets-like head trained to predict 4,096-bit Morgan fingerprints from raw spectra; on the same fine-tuned network, train a linear head to predict 11 molecular properties (QED, synthetic accessibility, Bertz complexity and others) using RDKit. For retrieval, rank PubChem candidates by similarity to the predicted fingerprint and compute accuracy@k on the 8,000-spectrum CANOPUS test set, alongside the MIST and FFN baselines. In parallel, compute relative-prediction-error curves for the Bertz property and compare against XGBoost and MS2Prop. Aggregate all accuracy@k scores and Bertz error-curve results into a single results table (dreams_finetuning_results.csv).

## Related tools

- **MIST** (Baseline deep-learning model for fingerprint prediction from spectra; its official GitHub repository supplies the CANOPUS benchmark used for accuracy@k evaluation) — https://github.com/samgoldman97/mist
- **DreaMS** (Pre-trained 116M-parameter transformer fine-tuned with a DeepSets-like head to predict 4,096-bit Morgan fingerprints and a linear head for 11 molecular properties)
- **PyTorch** (Framework for attaching and fine-tuning the DeepSets-like fingerprint head and linear property head on the pre-trained DreaMS network)
- **RDKit** (Computation of Morgan fingerprints and the 11 molecular properties (QED, synthetic accessibility, Bertz complexity) from molecular structures)
- **HDF5** (Compact binary format in which the GeMS spectra and metadata are stored for deep learning inputs)

## Examples

```
git clone https://github.com/samgoldman97/mist && cd mist && . data_processing/canopus_train/00_download_canopus_data.sh
```

## Evaluation signals

- Fine-tuned DreaMS accuracy@k on the CANOPUS benchmark falls within MIST's reported range (on-par performance in molecular-structure retrieval using predicted fingerprints).
- The Bertz relative-prediction-error curve matches the paper's reported comparison against XGBoost and MS2Prop baselines.
- Predicted fingerprints are exactly 4,096-bit Morgan vectors and property predictions cover all 11 targets (QED, synthetic accessibility, Bertz complexity and others).
- Retrieval is performed against PubChem, mirroring the paper's database-lookup protocol, not against a different compound library.
- All accuracy@k scores and Bertz error-curve results are aggregated into a single dreams_finetuning_results.csv; if quantitative thresholds are not re-derivable from the deposit, reproduced benchmark scores are accepted only by expert review against the published figures.

## Limitations

- The MIST CANOPUS benchmark is limited in size, covering 8,000 spectra from 7,000 unique molecules.
- DreaMS fine-tuning for fingerprint prediction is a proof of concept, not a production-ready method.
- If quantitative thresholds cannot be re-derived from the Zenodo deposit, validation must fall back to expert review against the published figures.

## Evidence

- [abstract] Parity with MIST on fingerprint retrieval: "the fine-tuned DreaMS neural network performs on par with the deep learning model MIST in the retrieval of molecular structures using predicted fingerprints"
- [discussion] Benchmark source and size: "we use the CANOPUS benchmark from the official GitHub repository (https://github.com/samgoldman97/mist)"
- [discussion] Benchmark scale caveat: "the MIST CANOPUS benchmark is limited in size, covering 8,000 spectra from 7,000 unique molecules"
- [discussion] Proof-of-concept caveat: "should be understood as a proof of concept that our approach can achieve competitive performance with MIST"
- [readme] MIST capabilities reproduced in this protocol: [evidence span withheld — anchor status: external_doc]
