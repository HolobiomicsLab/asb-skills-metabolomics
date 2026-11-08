---
name: msms-spectra-reconstruction-from-per-position-predictions
description: Use when you have per-MZ-position peak predictions (one model per MZ
  bin, each outputting a presence score) and need to assemble whole predicted MS/MS
  spectra for a molecule or library of molecules.
when_to_use_negative:
- The task is training the per-position models rather than predicting spectra with
  them (use ANNmodel.py / GNNmodel.py / COMmodel.py instead)
- A single whole-spectrum predictor is used rather than one model per MZ position
  — this skill's thresholding and aggregation steps assume per-position outputs
- Predicted spectra for a single molecule, no per-position prediction files exist
  yet — run the per-MZ-position models first
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SingleFrag
  role: Source of the per-MZ-position ANN models and reconstruction workflow (spectraPrediction.ipynb)
  repo: https://github.com/MaribelPR/SingleFrag
- name: mol2vec
  role: Generates the 300-dimensional molecular embedding used as input to each per-position
    ANN model
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.11.04.621329
    title: 'SingleFrag: A deep learning tool for MS/MS fragment and spectral prediction
      and metabolite annotation'
  source_content_hash: sha256:f94ee3762e976abb72d15d636d077bce460f435e35781577867aa3fd669497fc
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/MaribelPR/SingleFrag
  tools:
  - SingleFrag
  - mol2vec
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# MS/MS spectra reconstruction from per-position predictions

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Reconstruct full predicted MS/MS spectra by aggregating per-MZ-position binary peak predictions (e.g. from SingleFrag's per-position ANN models) into a complete spectrum for each input molecule.

## When to use

Use when you have per-MZ-position peak predictions (one model per MZ bin, each outputting a presence score) and need to assemble whole predicted MS/MS spectra for a molecule or library of molecules.

## When NOT to use

- The task is training the per-position models rather than predicting spectra with them (use ANNmodel.py / GNNmodel.py / COMmodel.py instead)
- A single whole-spectrum predictor is used rather than one model per MZ position — this skill's thresholding and aggregation steps assume per-position outputs
- Predicted spectra for a single molecule, no per-position prediction files exist yet — run the per-MZ-position models first

## Inputs

- Input molecule SMILES (e.g. inputMols_git.xlsx)
- Trained per-MZ-position ANN models (ModelVal*.pickle)
- Mol2vec 300-d embedding model (Mol2vecModelGeneral_rep_dades1.model)
- Per-position peak thresholds (thresholdsANN.pickle)

## Outputs

- Per-MZ-position prediction files (results/dfPredictionPeak_***.pickle)
- Reconstructed predicted spectra (results/predSpectra.json)

## How to apply

For each molecule, run a separate trained model for each of the most frequented MZ positions (SingleFrag's per-position ANN models, ModelVal*.pickle) using the 300-dimensional mol2vec embedding of the input SMILES. Compare each model's prediction to its MZ-position-specific threshold from thresholdsANN.pickle; a peak is called present if the prediction meets or exceeds that position's threshold. Batch predictions are saved per MZ position as results/dfPredictionPeak_***.pickle, then all per-position predictions are merged to reconstruct each complete spectrum, written to results/predSpectra.json.

## Related tools

- **SingleFrag** (Source of the per-MZ-position ANN models and reconstruction workflow (spectraPrediction.ipynb)) — https://github.com/MaribelPR/SingleFrag
- **mol2vec** (Generates the 300-dimensional molecular embedding used as input to each per-position ANN model)

## Examples

```
jupyter nbconvert --to notebook --execute spectraPrediction.ipynb
```

## Evaluation signals

- Reconstructed spectrum contains peaks only at MZ positions for which a per-position model exists and whose prediction met that position's threshold from thresholdsANN.pickle
- results/predSpectra.json is produced containing one spectrum per input molecule
- Per-position prediction files results/dfPredictionPeak_***.pickle exist for each MZ position evaluated
- For molecules with known experimental spectra, the reconstructed predicted spectrum can be graphically compared against the empirical MS/MS spectrum
- Predictions use the ANN models only, since they got better results than GNN and COM models

## Limitations

- The workflow uses only integer-resolution example spectra in the toy input file (inputMols_git.xlsx), whereas the models predict fragments at 2-decimal resolution
- Only ANN models are used for reconstruction in the provided notebook because they outperformed GNN and COM models; results may differ with other model types
- Per-position models cover only the most frequented MZ positions in the training set, so low-frequency fragments may be missed

## Evidence
