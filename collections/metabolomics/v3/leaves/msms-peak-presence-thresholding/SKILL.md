---
name: msms-peak-presence-thresholding
description: 'Use when reconstructing in silico MS/MS spectra from candidate SMILES
  with SingleFrag: for each of the most frequented MZ positions, an ANN prediction
  must be converted into a binary peak call via a position-specific threshold.'
when_to_use_negative:
- You need quantitative peak intensities for spectral matching — SingleFrag deliberately
  predicts presence/absence rather than intensity
- You already have empirical reference spectra for the compounds, in which case library
  matching (e.g. NIST MS/MS) is more appropriate than in silico prediction
- Your molecules fall outside the trained MZ-frequency space (rare MZ positions lack
  a dedicated model/threshold pair)
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: SingleFrag
  role: Provides the trained per-MZ-position ANN models and thresholdsANN.pickle used
    to make and binarize peak-presence predictions
  repo: https://github.com/MaribelPR/SingleFrag
- name: mol2vec
  role: Embeds each candidate SMILES into the 300-dimensional vector that serves as
    ANN input
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2024.11.04.621329
    title: 'SingleFrag: A deep learning tool for MS/MS fragment and spectral prediction
      and metabolite annotation'
  source_content_hash: sha256:20a1e91211798f2a2e8ae81eb22d5eb449cb0571162133cfe775c4d48eb352dd
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
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# msms-peak-presence-thresholding

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

A peak-calling step from the SingleFrag workflow in which trained per-MZ-position ANN models predict peak presence/absence, and each prediction is binarized using position-specific thresholds rather than predicting peak intensities.

## When to use

Use when reconstructing in silico MS/MS spectra from candidate SMILES with SingleFrag: for each of the most frequented MZ positions, an ANN prediction must be converted into a binary peak call via a position-specific threshold.

## When NOT to use

- You need quantitative peak intensities for spectral matching — SingleFrag deliberately predicts presence/absence rather than intensity
- You already have empirical reference spectra for the compounds, in which case library matching (e.g. NIST MS/MS) is more appropriate than in silico prediction
- Your molecules fall outside the trained MZ-frequency space (rare MZ positions lack a dedicated model/threshold pair)

## Inputs

- 300-dimensional mol2vec embedding of each candidate molecule SMILES (from Mol2vecModelGeneral_rep_dades1.model)
- Trained per-MZ-position ANN models (data/ANN/ModelVal*.pickle, m/z encoded as mz*100)
- Position-specific peak thresholds (data/modelData/thresholdsANN.pickle)
- Training-set MZ-frequency table (data/modelData/AllMostFreqMolGeneral_rep_dades1.pickle)

## Outputs

- Per-MZ-position prediction files (results/dfPredictionPeak_*.pickle)
- Reconstructed predicted MS/MS spectra per candidate molecule (results/predSpectra.json), at 2-decimal m/z resolution

## How to apply

For each of the 1,000 most frequented MZ positions in the SingleFrag training set, load the corresponding trained ANN model (ModelVal*.pickle, where the file name encodes m/z as mz*100) and run it on the 300-dimensional mol2vec embedding of the candidate SMILES. If the model's prediction for a position is greater than or equal to that position's threshold in thresholdsANN.pickle, call a peak present in that m/z bin. Write the per-position predictions to results/dfPredictionPeak_*.pickle, then reconstruct the full predicted spectrum for each candidate into results/predSpectra.json using spectraPrediction.ipynb. The presence/absence formulation is deliberate: the authors found binarized peak presence more robust than predicting intensities across experimental conditions, and ANN models outperformed the GNN and combined (COM) variants.

## Related tools

- **SingleFrag** (Provides the trained per-MZ-position ANN models and thresholdsANN.pickle used to make and binarize peak-presence predictions) — https://github.com/MaribelPR/SingleFrag
- **mol2vec** (Embeds each candidate SMILES into the 300-dimensional vector that serves as ANN input)

## Examples

```
jupyter execute spectraPrediction.ipynb  # loads ModelVal*.pickle ANN models, applies thresholdsANN.pickle per MZ position, writes results/dfPredictionPeak_*.pickle and results/predSpectra.json
```

## Evaluation signals

- Every called peak corresponds to a prediction ≥ its position-specific threshold in thresholdsANN.pickle (≥, not >)
- dfPredictionPeak_*.pickle files exist for each of the 1,000 most frequented MZ positions, with file names encoding m/z as mz*100
- Reconstructed predSpectra.json contains one predicted spectrum per input candidate at 2-decimal m/z resolution
- For validated ARUS cases, the true candidate (e.g. glucuronyl-2-hydroxyhippurate, 8-hydroxyquinoline glucuronide, a truxilline isomer) shows a significant cosine-similarity (Δcos) gap over remaining candidates
- Predicted-vs-experimental graphical comparison is available for molecules with known experimental spectra

## Limitations

- Thresholds and models were only defined for the 1,000 most frequented MZ positions of the training set; rarer fragments are not predicted
- GNN models showed slightly lower performance than ANN models, attributed to deliberately simple, unoptimized architectures rather than a fundamental GNN limitation
- The toy example spectrum format contains only integer m/z, while the model predicts with 2-decimal resolution — integer-level comparisons underuse the model's resolution

## Evidence
