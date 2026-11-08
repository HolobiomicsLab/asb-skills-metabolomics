---
name: tandem-ms-model-inference
description: Use when labeled MS2 spectra (e.g., HDF5 2xN m/z/intensity arrays with
  lipid_species/adduct/precursor metadata) must be passed through a trained deep learning
  model to generate lipid species predictions and per-spectrum accuracy metrics.
when_to_use_negative:
- Input is raw mzML profile data needing preprocessing (smoothing, baseline correction,
  peak picking, base peak normalization, m/z 50-1600 trimming) rather than already-peak-picked
  HDF5 test spectra
- The goal is training or hyperparameter tuning of the transformer (e.g., with Ray
  Tune) rather than running inference and evaluation on a held-out test set
- Spectra are blank measurements with no target lipid species, where the expected
  model output is the end-of-sequence token rather than a scored lipid species prediction
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: LipiDetective
  role: Transformer model that decodes lipid species identifications from MS2 spectra;
    run in evaluation mode with greedy or beam search decoding
  repo: https://github.com/LipiTUM/lipidetective
- name: h5py
  role: Loads the HDF5 test dataset of spectra and their lipid_species/adduct/precursor
    metadata attributes via the LipiDetective H5Dataset loader
- name: PyTorch
  role: Provides the transformer module underlying LipiDetective used to generate
    predictions during inference
- name: PyTorch Lightning
  role: Drives the evaluation loop with the custom Evaluator that aggregates per-spectrum
    metrics into test_metrics.csv
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.10.07.617094
    title: LipiDetective a deep learning model for the identification of molecular
      lipid species in tandem mass spectra
  source_content_hash: sha256:4d586b8845bcfc5ef5a255b2e4938b55d7957ebeb997befbd0af03b9aca8e1e6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LipiTUM/lipidetective
  tools:
  - LipiDetective
  - PyTorch
  - PyTorch Lightning
  - h5py
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# tandem-ms-model-inference

## Summary

Runs a trained LipiDetective transformer to decode lipid species identifications from tandem MS2 spectra, scoring predictions via lipid component accuracy rather than only whole-species match.

## When to use

Use when labeled MS2 spectra (e.g., HDF5 2xN m/z/intensity arrays with lipid_species/adduct/precursor metadata) must be passed through a trained deep learning model to generate lipid species predictions and per-spectrum accuracy metrics.

## When NOT to use

- Input is raw mzML profile data needing preprocessing (smoothing, baseline correction, peak picking, base peak normalization, m/z 50-1600 trimming) rather than already-peak-picked HDF5 test spectra
- The goal is training or hyperparameter tuning of the transformer (e.g., with Ray Tune) rather than running inference and evaluation on a held-out test set
- Spectra are blank measurements with no target lipid species, where the expected model output is the end-of-sequence token rather than a scored lipid species prediction

## Inputs

- HDF5 dataset of labeled test MS2 spectra (2xN m/z/intensity arrays per spectrum with lipid_species, adduct, and precursor metadata attributes)
- Trained LipiDetective transformer checkpoint
- YAML configuration file specifying paths to HDF5 test files and model parameters

## Outputs

- Per-spectrum lipid species predictions in shorthand nomenclature
- test_metrics.csv with per-spectrum and overall loss, total prediction accuracy, and lipid component accuracy columns

## How to apply

Load the labeled test spectra from the HDF5 dataset (spectra stored as 2xN m/z/intensity arrays with lipid_species, adduct, and precursor metadata attributes) using h5py and the LipiDetective H5Dataset loader. Generate lipid species predictions for each test spectrum with the trained LipiDetective transformer, using greedy or beam search decoding. Tokenize predicted and true lipid species names into components (headgroup, fatty acid side chains, adduct) by regular-expression matching, sorting fatty acids by increasing carbon chain length then number of double bonds. Compute per-spectrum lipid component accuracy as correctly predicted tokens divided by total tokens, plus total prediction accuracy requiring all tokens to match. Aggregate and write per-spectrum and overall metrics, including loss and accuracy columns, to test_metrics.csv via PyTorch Lightning's evaluation loop with the custom Evaluator. The rationale is that component accuracy (60-80% on average) captures partial correctness even when the full species prediction is inaccurate, since the model composes predictions from relevant lipid components.

## Related tools

- **LipiDetective** (Transformer model that decodes lipid species identifications from MS2 spectra; run in evaluation mode with greedy or beam search decoding) — https://github.com/LipiTUM/lipidetective
- **h5py** (Loads the HDF5 test dataset of spectra and their lipid_species/adduct/precursor metadata attributes via the LipiDetective H5Dataset loader)
- **PyTorch** (Provides the transformer module underlying LipiDetective used to generate predictions during inference)
- **PyTorch Lightning** (Drives the evaluation loop with the custom Evaluator that aggregates per-spectrum metrics into test_metrics.csv)

## Examples

```
uv run lipidetective --config path/to/config.yaml
```

## Evaluation signals

- test_metrics.csv exists and contains a lipid component accuracy value in the range 0-1 for each test spectrum
- Overall component accuracy summary is consistent with the fraction of correctly predicted tokens across spectra
- Predicted and true lipid species names tokenize into the same component types (headgroup, fatty acid side chains, adduct) with fatty acids sorted by chain length then double bonds
- Total prediction accuracy (all tokens matching) is lower than average component accuracy, consistent with the reported 60-80% component accuracy range
- Loss and accuracy columns are present in test_metrics.csv as produced by the PyTorch Lightning Evaluator

## Limitations

- Complete molecular lipid species prediction is often not fully correct even when most component tokens are right, so whole-species accuracy understates model performance
- The model composes predictions from lipid components and can fail on peaks relevant for prediction, such as 224.0 m/z, that lack annotation in the Alex123 database
- Requires spectra preprocessed and stored in HDF5 format; prediction on raw mzML files is handled by a separate workflow path in the tool

## Evidence

- [results] LipiDetective's lipid component accuracy is less variable and much higher than its total prediction accuracy, averaging between 60% and 80%: [evidence span withheld — anchor status: not_found]
- [intro] it does not have to rely on matching the overall spectra but can identify relevant lipid components from which it composes the final prediction: "it does not have to rely on matching the overall spectra but can identify relevant lipid components from which it composes the final prediction"
- [methods] LipiDetective was implemented by modifying the transformer module provided by PyTorch [26]: "LipiDetective was implemented by modifying the transformer module provided by PyTorch [26]"
- [methods] all spectra were saved in a Hierarchical Data Format 5 (HDF5) file: "all spectra were saved in a Hierarchical Data Format 5 (HDF5) file"
- [readme] Accepts YAML configuration file for streamlined and reproducible setup. Supports training with HDF5 files and prediction using mzML files: "Accepts YAML configuration file for streamlined and reproducible setup.
- **Data Handling:** Supports training with HDF5 files and prediction using mzML files"
