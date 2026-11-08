---
name: tandem-ms-spectral-prediction
description: Use when experimental MS/MS spectra of unknown metabolites (e.g., recurrent
  unidentified spectra like ARUS) need in silico annotation by comparing predicted
  spectra of candidate SMILES against the experimental spectrum at 2-decimal m/z resolution.
when_to_use_negative:
- When peak intensities must be predicted — SingleFrag deliberately predicts only
  the presence or absence of each peak, not intensity
- When molecules lack SMILES representations, since the ANN input is exclusively a
  Mol2vec embedding of the SMILES
- When spectral comparison needs better than 2-decimal m/z resolution — the toy input
  format with integer masses is insufficient and the models operate at 2-decimal resolution
edam_operation: http://edamontology.org/operation_3216
tools:
- name: SingleFrag
  role: Deep learning tool providing per-fragment ANN models (ModelVal*.pickle), spectraPrediction.ipynb,
    and model/threshold files to predict and reconstruct MS/MS spectra
  repo: https://github.com/MaribelPR/SingleFrag
- name: mol2vec
  role: Embeds each candidate SMILES into the 300-dimensional vector that serves as
    input to the SingleFrag ANN models
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.11.04.621329
    title: 'SingleFrag: A deep learning tool for MS/MS fragment and spectral prediction
      and metabolite annotation'
  source_content_hash: sha256:052c6c56da22435a8116689ef14ed554bff2b4ac017a94935ba0df48a877dfc9
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/MaribelPR/SingleFrag
  tools:
  - SingleFrag
  - mol2vec
  edam_operation: http://edamontology.org/operation_3216
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# tandem-ms-spectral-prediction

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Predicts MS/MS spectra for candidate molecules in silico using SingleFrag, which trains a separate ANN per fragment-ion m/z position (mol2vec-embedded SMILES input) to predict peak presence/absence, then reconstructs full spectra to rank candidates against experimental spectra for metabolite annotation.

## When to use

Use when experimental MS/MS spectra of unknown metabolites (e.g., recurrent unidentified spectra like ARUS) need in silico annotation by comparing predicted spectra of candidate SMILES against the experimental spectrum at 2-decimal m/z resolution.

## When NOT to use

- When peak intensities must be predicted — SingleFrag deliberately predicts only the presence or absence of each peak, not intensity
- When molecules lack SMILES representations, since the ANN input is exclusively a Mol2vec embedding of the SMILES
- When spectral comparison needs better than 2-decimal m/z resolution — the toy input format with integer masses is insufficient and the models operate at 2-decimal resolution

## Inputs

- Candidate molecule SMILES (inputMols_git.xlsx, SMILES column)
- Experimental MS/MS spectra of unknown metabolites (e.g., ARUS database spectra)
- Trained ANN models (ANN/ModelVal*.pickle, m/z encoded as mz*100)
- Mol2vec model (Mol2vecModelGeneral_rep_dades1.model)
- Peak-count file (AllMostFreqMolGeneral_rep_dades1.pickle) and per-position thresholds (thresholdsANN.pickle)

## Outputs

- Per-MZ-position prediction files (results/dfPredictionPeak_*.pickle)
- Reconstructed predicted spectra for all candidates (results/predSpectra.json)
- Ranked candidate list (top-10 per experimental spectrum) by cosine similarity
- Graphical comparison of predicted vs. experimental spectra

## How to apply

Set up a Python 3.8.19 environment from requirements.txt and download the SingleFrag dataset from Figshare, placing trained ANN models (ANN/ModelVal*.pickle, filenames encoding m/z as mz*100), the Mol2vec model, and thresholdsANN.pickle under data/. Embed each candidate SMILES into a 300-dimensional vector with Mol2vecModelGeneral_rep_dades1.model, then run the per-MZ-position ANN for the 1,000 most frequented m/z positions in the training set, calling a peak present when the prediction is >= the position-specific threshold in thresholdsANN.pickle. Save per-position predictions to results/dfPredictionPeak_*.pickle and reconstruct each spectrum into results/predSpectra.json using spectraPrediction.ipynb, which uses only the ANN models because they outperformed GNN and COM. Compare each predicted spectrum against the experimental spectrum at 2-decimal m/z resolution, rank candidates by cosine similarity, and retain the top-10 per spectrum for expert manual elucidation.

## Related tools

- **SingleFrag** (Deep learning tool providing per-fragment ANN models (ModelVal*.pickle), spectraPrediction.ipynb, and model/threshold files to predict and reconstruct MS/MS spectra) — https://github.com/MaribelPR/SingleFrag
- **mol2vec** (Embeds each candidate SMILES into the 300-dimensional vector that serves as input to the SingleFrag ANN models)

## Examples

```
jupyter notebook spectraPrediction.ipynb  # after arranging data/ANN/ModelVal*.pickle, thresholdsANN.pickle, and inputMols_git.xlsx per README layout
```

## Evaluation signals

- A peak is predicted at an m/z position only when the ANN prediction is >= the position-specific value in thresholdsANN.pickle
- Only ANN model predictions should be used for reconstruction, as ANN outperformed GNN and COM
- Spectra must be reconstructed from all per-position prediction files into results/predSpectra.json covering every candidate molecule
- Validated benchmark: glucuronyl-2-hydroxyhippurate, 8-hydroxyquinoline glucuronide, and a truxilline isomer should rank among the top-10 candidates for their ARUS spectra with a significant cosine-similarity (Δcos) gap separating them from remaining candidates
- Comparison against experimental spectra must be performed at 2-decimal m/z resolution

## Limitations

- Prediction is binary (peak presence/absence per m/z bin); intensities are not modeled
- Only the 1,000 most frequented m/z positions in the training set have trained ANN models, so peaks outside these bins cannot be predicted
- Authors deliberately kept neural network architectures simple, so architectures are not optimized — e.g., the GNN's slightly lower performance 'should not be seen as a fundamental limitation of graph neural networks in the domain of spectral prediction'
- Environment is pinned to Python 3.8.19; all pickle files were created with that version

## Evidence
