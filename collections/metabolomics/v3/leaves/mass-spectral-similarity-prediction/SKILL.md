---
name: mass-spectral-similarity-prediction
description: Use when MS/MS spectra in MGF/h5py format must be scored against a spectral
  library (e.g., GNPS/MassBank) to retrieve structural analogues, or when similarity
  scores must reflect subtle structural modifications.
when_to_use_negative:
- When only simple heuristic comparison suffices and no analogue search or explanation
  is needed — the paper targets structural-analogue retrieval, where Cosine/Modified
  Cosine underperform
- When the input is not tandem MS/MS spectra (e.g., MS1 features or an already-computed
  feature table)
- When no trained model or training data (paired MS/MS spectra with structural similarity
  labels) is available, since TransExION is a supervised method
edam_operation: http://edamontology.org/operation_3647
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3473
tools:
- name: TransExION
  role: Transformer-based supervised model that predicts MS/MS spectral similarity
    correlated with structural similarity and detects related fragments via mass difference
  repo: github.com/banhdzui/TransExION
- name: PyTorch
  role: Framework in which TransExION is implemented and trained/evaluated
  repo: github.com/banhdzui/TransExION
- name: h5py
  role: File format for spectra, pair-id, and structural-similarity inputs at large
    scale
  repo: github.com/banhdzui/TransExION
- name: pyteomics
  role: Listed dependency for handling MS/MS spectral data
  repo: github.com/banhdzui/TransExION
- name: RDKit
  role: Used to compute structural similarity matrices for supervised training labels
  repo: github.com/banhdzui/TransExION
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-024-00858-5
    title: 'TransExION: a transformer based explainable similarity metric for comparing
      IONS in tandem mass spectrometry'
  source_content_hash: sha256:e7b9f644dced3b65a72176d4fdb315beaf36f5117b87e5dae55c7eb842bf2463
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Matplotlib
  - PyTorch
  - RDKit
  - TransExION
  - h5py
  - pyteomics
  edam_operation: http://edamontology.org/operation_3647
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# mass-spectral-similarity-prediction

## Summary

Predict MS/MS spectral similarity between query and reference spectra with the Transformer-based TransExION model, whose scores correlate with structural similarity (Tanimoto), enabling spectral library search for structural analogues beyond Cosine, Modified Cosine, Spec2Vec, and MS2DeepScore.

## When to use

Use when MS/MS spectra in MGF/h5py format must be scored against a spectral library (e.g., GNPS/MassBank) to retrieve structural analogues, or when similarity scores must reflect subtle structural modifications.

## When NOT to use

- When only simple heuristic comparison suffices and no analogue search or explanation is needed — the paper targets structural-analogue retrieval, where Cosine/Modified Cosine underperform
- When the input is not tandem MS/MS spectra (e.g., MS1 features or an already-computed feature table)
- When no trained model or training data (paired MS/MS spectra with structural similarity labels) is available, since TransExION is a supervised method

## Inputs

- MS/MS query spectra file (MGF, converted to h5py .db)
- MS/MS reference/library spectra file (h5py .db, e.g., GNPS_MassBank_train.db)
- Spectrum pair id file (h5py, from CSV via ms_covert_csv2h5py.py)
- Pre-trained TransExION model file (e.g., GNPS_MassBank.ms.model)
- Structural similarity matrices (.ref files) if training a new model

## Outputs

- CSV of predicted spectral similarity scores per spectrum pair (e.g., GNPS_MassBank_result.csv)
- Trained TransExION model file and training log (when training)
- Intermediate spectra representations reusable by other models

## How to apply

Convert MGF spectra to h5py using ms_convert_mgf2h5py.py, then either load a pre-trained model (e.g., GNPS_MassBank.ms.model) or train one with ms_train_spectal_similarity.py on the Zenodo-recorded GNPS_MassBank_train/val sets (batchsize 64, lr 0.0001, decay-weight 0.0) against precomputed structural similarity matrices from ms_compute_structural_similarity.py. Compute spectral similarity with ms_test_spectral_similarity.py, passing --db (reference library, h5py), --query (query spectra, h5py), --pairs (optional h5py spectrum-pair id file built with ms_covert_csv2h5py.py), and --model; if --pairs is empty, every query is paired with every reference. The model's key design is that, in addition to aligned product ions and neutral losses, the mass difference between query and reference fragments is considered, which lets it detect related fragments across modifications such as +15.99 Da (one oxygen). The intermediate spectra representation can also be reused by other downstream models for unknown structure elucidation.

## Related tools

- **TransExION** (Transformer-based supervised model that predicts MS/MS spectral similarity correlated with structural similarity and detects related fragments via mass difference) — github.com/banhdzui/TransExION
- **PyTorch** (Framework in which TransExION is implemented and trained/evaluated) — github.com/banhdzui/TransExION
- **h5py** (File format for spectra, pair-id, and structural-similarity inputs at large scale) — github.com/banhdzui/TransExION
- **pyteomics** (Listed dependency for handling MS/MS spectral data) — github.com/banhdzui/TransExION
- **RDKit** (Used to compute structural similarity matrices for supervised training labels) — github.com/banhdzui/TransExION

## Examples

```
python ms_test_spectral_similarity.py --db GNPS_MassBank_train.db --query GNPS_MassBank_test.db --model GNPS_MassBank.ms.model --output GNPS_MassBank_result.csv
```

## Evaluation signals

- TransExION should outperform Cosine, Modified Cosine, Spec2Vec, and MS2DeepScore on independent test sets, especially at high Tanimoto distance cut-offs (nearly-identical structure matching)
- Prediction error should be low and stable across the entire structure similarity range
- Similarity scores computed with --pairs must match the number of specified pairs; with --pairs empty, every query-reference combination must be scored
- Scores must correlate with structural (Tanimoto) similarity as encoded in the .ref files used for training/validation
- Reproducing paper case studies should recover expected mass-difference associations (e.g., +15.99 Da for DL-beta-Homophenylalanine, -14.02 Da for Scopularide F)

## Limitations

- Requires a supervised trained model and structural-similarity labels; performance claims in the discussion lack reported quantitative metrics or confidence intervals
- The Tanimoto distance cut-off for 'nearly-identical structure matching' and details of the explanation module are not specified in the discussion
- Data volume is large-scale: all inputs must first be converted to h5py format for processing

## Evidence

- [readme] TransExION is a supervised learning method estimating spectral similarity between MS/MS spectra that are strongly correlated to their structural similarity: "TransExION is a supervised learning method estimating ***spectral similarity*** between MS/MS spectra that are strongly correlated to their structural similarity."
- [intro] Mass difference considered in similarity: "The major novelty of TransExION is that, in addition to aligned product ions and neutral losses, the mass difference between query and reference fragments are also considered for spectral similarity"
- [discussion] Outperforms existing similarity measures: "TransExION outperforms simple, heuristic similarity measures, such as Cosine and Modified Cosine, and the unsupervised Spec2Vec model."
- [readme] Testing script usage: "To compute spectral similarity between spectra using a trained TransExION model, the script ```ms_test_spectral_similarity.py``` can be used with parameters"
- [readme] Pairs file semantics: "If the parameter ```--pairs``` is empty, then every query spectrum is paired with every reference spectra to compute spectral similarity."
