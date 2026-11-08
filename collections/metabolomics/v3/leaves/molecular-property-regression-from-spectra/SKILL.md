---
name: molecular-property-regression-from-spectra
description: Use when raw MS/MS peak lists must be converted to continuous molecular
  property values (e.g. QED, synthetic accessibility, Bertz complexity) and spectrum-to-structure
  reference libraries are unavailable.
when_to_use_negative:
- The goal is molecular-structure retrieval by fingerprint search in PubChem — that
  is a separate fingerprint-prediction and accuracy@k retrieval task, not property
  regression
- Spectra are outside the training distribution the article covers, e.g. negative
  ionization mode or multiply charged precursor ions, which the authors explicitly
  note were not incorporated
- A production-ready property predictor is required — the authors state this fine-tuning
  'should be understood as a proof of concept that our approach can achieve competitive
  performance with MIST'
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Framework for attaching the linear regression head to the pre-trained DreaMS
    transformer and running supervised fine-tuning
- name: RDKit
  role: Computes ground-truth molecular property labels (QED, synthetic accessibility,
    Bertz complexity) from the paired molecular structures
- name: HDF5
  role: Binary storage format for the fine-tuning MS/MS spectra paired with structures
- name: DreaMS (pluskal-lab/DreaMS)
  role: Supplies the pre-trained self-supervised encoder and fine-tuning datasets
    via Zenodo records 10997887 and 13843034
  repo: https://github.com/pluskal-lab/DreaMS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:3b2c5db7b8e7c49e4b52569fe94a9f770982aba8bb84886748022bf620c94216
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pluskal-lab/DreaMS
  tools:
  - DreaMS (pluskal-lab/DreaMS)
  - HDF5
  - MIST
  - PyTorch
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# molecular-property-regression-from-spectra

## Summary

Fine-tunes the pre-trained 116M-parameter DreaMS transformer on MS/MS spectra paired with molecular structures so a regression head predicts chemical properties (QED, synthetic accessibility, Bertz complexity, among 11 total) directly from raw tandem mass spectra.

## When to use

Use when raw MS/MS peak lists must be converted to continuous molecular property values (e.g. QED, synthetic accessibility, Bertz complexity) and spectrum-to-structure reference libraries are unavailable.

## When NOT to use

- The goal is molecular-structure retrieval by fingerprint search in PubChem — that is a separate fingerprint-prediction and accuracy@k retrieval task, not property regression
- Spectra are outside the training distribution the article covers, e.g. negative ionization mode or multiply charged precursor ions, which the authors explicitly note were not incorporated
- A production-ready property predictor is required — the authors state this fine-tuning 'should be understood as a proof of concept that our approach can achieve competitive performance with MIST'

## Inputs

- MS/MS spectra with paired molecular structures from the DreaMS Zenodo deposits (records 10997887 and 13843034), stored in the project's HDF5-based binary format
- Pre-trained DreaMS transformer model weights (116M parameters)
- RDKit-computed ground-truth property labels for 11 properties (QED, synthetic accessibility, Bertz complexity, etc.)
- Baseline Bertz relative-prediction-error curves for XGBoost and MS2Prop for comparison

## Outputs

- Fine-tuned DreaMS network with a regression head predicting 11 molecular properties from raw spectra
- Relative-prediction-error curves for the Bertz property versus XGBoost and MS2Prop baselines
- Aggregated benchmark results in dreams_finetuning_results.csv

## How to apply

Load the pre-trained DreaMS transformer weights from Zenodo (records 10997887 and 13843034) together with the fine-tuning datasets of MS/MS spectra paired with molecular structures, stored in the project's HDF5-based binary format. Freeze or lightly adapt the pre-trained encoder and attach a linear head, then train it to regress 11 molecular properties in parallel — QED, synthetic accessibility, Bertz complexity and others — from the 1,024-dimensional DreaMS embeddings. Ground-truth property labels are computed from structures with RDKit (QED, SA score, Bertz). Evaluate on the Bertz property by comparing relative-prediction-error curves against XGBoost and MS2Prop baselines, aggregating results into dreams_finetuning_results.csv. The article treats this fine-tuning as a proof of concept that self-supervised spectral representations transfer to supervised annotation tasks, not as a production property predictor.

## Related tools

- **PyTorch** (Framework for attaching the linear regression head to the pre-trained DreaMS transformer and running supervised fine-tuning)
- **RDKit** (Computes ground-truth molecular property labels (QED, synthetic accessibility, Bertz complexity) from the paired molecular structures)
- **HDF5** (Binary storage format for the fine-tuning MS/MS spectra paired with structures)
- **DreaMS (pluskal-lab/DreaMS)** (Supplies the pre-trained self-supervised encoder and fine-tuning datasets via Zenodo records 10997887 and 13843034) — https://github.com/pluskal-lab/DreaMS

## Evaluation signals

- The Bertz relative-prediction-error curve matches the paper's reported comparison against XGBoost and MS2Prop baselines
- All 11 properties (QED, synthetic accessibility, Bertz complexity and others) are regressed by a single linear head on top of the same fine-tuned network
- Quantitative thresholds are re-derivable from the Zenodo deposit (records 10997887 and 13843034); if not, reproduced scores are validated by expert review against the published figures
- Results are aggregated into dreams_finetuning_results.csv with the expected schema

## Limitations

- Fine-tuning for property prediction is explicitly a proof of concept, not a production-ready tool
- Training used only a subset of available mass spectra; full potential of training on raw unannotated spectra remains to be unlocked
- The method focuses solely on tandem mass spectra, disregarding MS1 isotopic patterns and adduct distributions, and excludes negative ionization mode and multiply charged precursors

## Evidence

- [abstract] supervised fine-tuning for properties: "we adapted the network to the prediction of spectral similarity, molecular fingerprints, chemical properties and the identification of fluorine-containing molecules"
- [discussion] proof-of-concept status: "should be understood as a proof of concept that our approach can achieve competitive performance with MIST"
- [discussion] benchmark scope limitation: "the MIST CANOPUS benchmark is limited in size, covering 8,000 spectra from 7,000 unique molecules"
- [abstract] spectra stored in HDF5: "we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning"
