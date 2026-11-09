---
name: peak-presence-threshold-classification
description: Use when you need in silico MS/MS spectra for small molecules from SMILES
  as binary peak-present/absent calls on a discretized m/z grid, e.g. to build in
  silico libraries or benchmark against CFM-ID/3DMolMS.
when_to_use_negative:
- The analysis requires accurate peak intensities or relative abundance ratios — SingleFrag
  deliberately predicts only peak presence/absence, not intensities.
- Input is already an experimental spectrum to be annotated against a library — this
  skill generates predicted spectra from SMILES, not annotations (though the predicted
  spectra can subsequently feed annotation).
- The molecules' fragment peaks fall largely outside the 1,000 most frequented MZ
  positions of the training set — no per-position models or thresholds exist for those
  bins.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SingleFrag
  role: Source of the per-MZ-position ANN/GNN/COM models, thresholds, and spectraPrediction.ipynb
    used to make peak-presence calls and reconstruct spectra
  repo: https://github.com/MaribelPR/SingleFrag
- name: mol2vec
  role: Embeds each input SMILES into the 300-dimensional vector that serves as ANN
    model input
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.11.04.621329
    title: 'SingleFrag: A deep learning tool for MS/MS fragment and spectral prediction
      and metabolite annotation'
  source_content_hash: sha256:91374dac1db4cab9d80674189724aab8c4d6fe25c3ad10d81107f53471b68a3e
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/MaribelPR/SingleFrag
  tools:
  - 3DMolMS
  - CFM-ID
  - SingleFrag
  - mol2vec
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# peak-presence-threshold-classification

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

SingleFrag-style MS/MS spectral prediction that trains/uses a separate neural network per fragment ion (1,000 most frequented MZ positions) and converts each per-position model score into a binary peak call using position-specific thresholds (thresholdsANN/GNN/COM.pickle), instead of regressing whole-spectrum intensities.

## When to use

Use when you need in silico MS/MS spectra for small molecules from SMILES as binary peak-present/absent calls on a discretized m/z grid, e.g. to build in silico libraries or benchmark against CFM-ID/3DMolMS.

## When NOT to use

- The analysis requires accurate peak intensities or relative abundance ratios — SingleFrag deliberately predicts only peak presence/absence, not intensities.
- Input is already an experimental spectrum to be annotated against a library — this skill generates predicted spectra from SMILES, not annotations (though the predicted spectra can subsequently feed annotation).
- The molecules' fragment peaks fall largely outside the 1,000 most frequented MZ positions of the training set — no per-position models or thresholds exist for those bins.

## Inputs

- inputMols file (inputMols_git.xlsx) with a 'smiles' column per compound
- Trained per-position models: data/ANN/ModelVal*.pickle, data/GNN/ModelVal*.pickle, data/COM/ModelVal*.pickle (singleFrag Figshare dataset)
- Mol2vec model file Mol2vecModelGeneral_rep_dades1.model (300-dimensional SMILES embedding)
- Threshold files thresholdsANN.pickle, thresholdsGNN.pickle, thresholdsCOM.pickle
- modelData/AllMostFreqMolGeneral_rep_dades1.pickle (peak counts per MZ bin in the training set)

## Outputs

- Per-MZ-position prediction files results/dfPredictionPeak_*.pickle
- Reconstructed predicted spectra results/predSpectra.json (one predicted MS/MS spectrum per input molecule)

## How to apply

Load input molecules from the inputMols file (e.g. inputMols_git.xlsx) and embed each SMILES into a 300-dimensional vector with the Mol2vec model (Mol2vecModelGeneral_rep_dades1.model). For each of the 1,000 most frequented MZ positions in the training set, run the corresponding trained model (ModelVal*.pickle for ANN, GNN, or COM) to obtain a probability of a peak at that position, then apply the per-position classification threshold from thresholdsANN/thresholdsGNN/thresholdsCOM.pickle — a prediction higher than or equal to the position's threshold calls the peak present. For efficiency on molecule batches, first write all predictions per MZ position to results/dfPredictionPeak_*.pickle, then reconstruct each spectrum into results/predSpectra.json. Use the ANN models preferentially, since they outperformed GNN and COM in the paper's benchmarks.

## Related tools

- **SingleFrag** (Source of the per-MZ-position ANN/GNN/COM models, thresholds, and spectraPrediction.ipynb used to make peak-presence calls and reconstruct spectra) — https://github.com/MaribelPR/SingleFrag
- **mol2vec** (Embeds each input SMILES into the 300-dimensional vector that serves as ANN model input)

## Examples

```
jupyter nbconvert --to notebook --execute spectraPrediction.ipynb
```

## Evaluation signals

- For each MZ position, the model probability must be compared against that position's own threshold from the matching thresholds*.pickle file — applying one global cutoff is incorrect.
- All predicted peaks must land on the 1,000 most-frequented MZ positions from AllMostFreqMolGeneral_rep_dades1.pickle; peaks elsewhere indicate a pipeline error.
- On the 2,929-compound test set with 0.2 m/z binning, precision, accuracy, F1, and cosine similarity versus experimental spectra should meet or exceed CFM-ID 4.0 and 3DMolMS (recall comparable to 3DMolMS).
- results/dfPredictionPeak_*.pickle must exist for every predicted MZ position before predSpectra.json reconstruction succeeds.
- Predicted spectra should carry two-decimal m/z resolution, matching the model training resolution (README: 'our model is able to predict the fragments with more resolution (considering 2 decimals)').

## Limitations

- Only the 1,000 most frequented MZ positions in the training set have trained models and thresholds; rarer fragments cannot be predicted.
- The ANN model is preferred because it 'got better results than the GNN and the COM'; GNN performance is slightly lower due to deliberately simple, unoptimized architectures, not a fundamental limitation.
- Peak presence/absence only — intensity information is not modeled, which limits comparison to intensity-based spectral similarity metrics.
- All .pickle artifacts were created with Python 3.8.19 and the environment is pinned to that version (requirements.txt).

## Evidence
