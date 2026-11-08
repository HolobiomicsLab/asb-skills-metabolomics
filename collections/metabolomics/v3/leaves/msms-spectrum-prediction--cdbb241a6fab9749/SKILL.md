---
name: msms-spectrum-prediction
description: Use when you have molecule SMILES but no experimental MS/MS reference
  spectrum, and you need predicted spectra for library generation, metabolite annotation,
  or benchmarking against tools like CFM-ID and 3DMolMS.
when_to_use_negative:
- A curated experimental MS/MS spectrum already exists for the compound at the relevant
  collision energy — an empirical library match is preferable, since NIST MS/MS provides
  systematically curated multi-collision-energy spectra
- The target is peak intensity prediction rather than peak presence/absence — SingleFrag
  deliberately focuses on presence/absence and does not model intensities
- The molecule falls outside 3DMolMS's supported atom-type set (C, O, N, H, P, S,
  F, Cl, B, Br, I) or exceeds 300 atoms if using the 3D-conformation-based alternative
edam_operation: http://edamontology.org/operation_3860
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: SingleFrag
  role: Primary deep learning tool that predicts individual fragments separately via
    ANN, GNN, and combined (COM) models; includes spectraPrediction.ipynb and ANNmodel.py/GNNmodel.py/COMmodel.py
  repo: https://github.com/MaribelPR/SingleFrag
- name: mol2vec
  role: Embeds each input SMILES into the 300-dimensional vector used as ANN model
    input (Mol2vecModelGeneral_rep_dades1.model)
- name: 3DMolMS
  role: Alternative deep learning baseline that predicts MS/MS spectra from 3D molecular
    conformations, used for comparison and installable via molnetpack
  repo: https://github.com/JosieHong/3DMolMS
- name: CFM-ID
  role: Rule-based in silico fragmentation baseline against which SingleFrag predictions
    are benchmarked
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2024.11.04.621329
    title: 'SingleFrag: A deep learning tool for MS/MS fragment and spectral prediction
      and metabolite annotation'
  source_content_hash: sha256:f9a653f93cee1f4cf48113eb09a3f8da0699f7b61a23730cc3d5b2e107cad04f
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/MaribelPR/SingleFrag
  tools:
  - 3DMolMS
  - CFM-ID
  - SingleFrag
  - mol2vec
  edam_operation: http://edamontology.org/operation_3860
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

# msms-spectrum-prediction

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Predict MS/MS spectra of small molecules in silico by training/running per-MZ-position deep learning models (SingleFrag ANN/GNN/COM) that call peak presence or absence, enabling spectral prediction for compounds lacking empirical reference spectra.

## When to use

Use when you have molecule SMILES but no experimental MS/MS reference spectrum, and you need predicted spectra for library generation, metabolite annotation, or benchmarking against tools like CFM-ID and 3DMolMS.

## When NOT to use

- A curated experimental MS/MS spectrum already exists for the compound at the relevant collision energy — an empirical library match is preferable, since NIST MS/MS provides systematically curated multi-collision-energy spectra
- The target is peak intensity prediction rather than peak presence/absence — SingleFrag deliberately focuses on presence/absence and does not model intensities
- The molecule falls outside 3DMolMS's supported atom-type set (C, O, N, H, P, S, F, Cl, B, Br, I) or exceeds 300 atoms if using the 3D-conformation-based alternative

## Inputs

- Molecule SMILES strings (e.g., the inputMols_git.xlsx file with smiles, name, spectrum, link columns)
- SingleFrag Figshare dataset: trained ANN/GNN/COM ModelVal*.pickle files per MZ position, Mol2vecModelGeneral_rep_dades1.model, AllMostFreqMolGeneral_rep_dades1.pickle, thresholdsANN/GNN/COM.pickle
- Optional experimental MS/MS spectra for comparison (e.g., HMDB MS/MS spectra, 2-decimal m/z resolution)

## Outputs

- Per-MZ-position prediction files (results/dfPredictionPeak_*.pickle)
- Reconstructed predicted MS/MS spectra (results/predSpectra.json)
- Graphical comparison of predicted vs. experimental spectra when an experimental spectrum is supplied
- For benchmarking: per-tool per-molecule tables of precision, recall, accuracy, F1, and cosine similarity on a common 0.2 m/z grid

## How to apply

Set up a Python 3.8.19 environment with the packages in SingleFrag's requirements.txt, then download the SingleFrag dataset from Figshare (trained ANN/GNN/COM models for the 1,000 most frequented MZ positions, the Mol2vec model, and thresholdsANN/GNN/COM.pickle). Encode each input SMILES as a 300-dimensional mol2vec vector and run spectraPrediction.ipynb over the most-frequented MZ positions; for each position, a peak is called when the model's prediction meets or exceeds the per-position threshold from the corresponding thresholds pickle. To scale to many molecules, first write per-MZ-position prediction files (results/dfPredictionPeak_*.pickle), then reconstruct each full spectrum into results/predSpectra.json. The rationale for the per-fragment design is that training a separate model for each fragment ion — and predicting presence/absence rather than intensity — captures position-specific features and is more robust across experimental conditions than whole-spectrum or intensity-regression models. If retraining, prepare training/validation/test sets and use ANNmodel.py, GNNmodel.py, and COMmodel.py, each trained to predict peak presence at a specified MZ position.

## Related tools

- **SingleFrag** (Primary deep learning tool that predicts individual fragments separately via ANN, GNN, and combined (COM) models; includes spectraPrediction.ipynb and ANNmodel.py/GNNmodel.py/COMmodel.py) — https://github.com/MaribelPR/SingleFrag
- **mol2vec** (Embeds each input SMILES into the 300-dimensional vector used as ANN model input (Mol2vecModelGeneral_rep_dades1.model))
- **3DMolMS** (Alternative deep learning baseline that predicts MS/MS spectra from 3D molecular conformations, used for comparison and installable via molnetpack) — https://github.com/JosieHong/3DMolMS
- **CFM-ID** (Rule-based in silico fragmentation baseline against which SingleFrag predictions are benchmarked)

## Examples

```
python ANNmodel.py  --mz_position 7300  # train a model for m/z 73.0; then run spectraPrediction.ipynb to produce results/dfPredictionPeak_*.pickle and results/predSpectra.json
```

## Evaluation signals

- On the 2,929-compound test set at 0.2 m/z binning, predicted spectra should achieve higher precision, accuracy, and cosine similarity than CFM-ID and 3DMolMS baselines, with recall comparable to 3DMolMS
- Predicted spectra must contain peaks only at the 1,000 most frequented MZ positions, and each peak call must satisfy its per-position threshold from thresholdsANN/GNN/COM.pickle
- reconstructions in results/predSpectra.json should match the union of peaks called in the per-position dfPredictionPeak_*.pickle files
- Input embeddings must be 300-dimensional mol2vec vectors derived from the SMILES; dimensionality mismatch indicates a wrong or corrupted Mol2vec model
- GNN performance is expected to be slightly lower than ANN — a combined-model run whose COM predictions fall below GNN-level quality suggests a threshold misapplication

## Limitations

- Models are limited to the 1,000 most frequented MZ positions in the training set, so rare fragments outside these bins cannot be predicted
- The GNN model shows slightly lower performance than the ANN, attributed to deliberately simple, unoptimized graph features and layer architectures rather than a fundamental limitation
- Predictions cover peak presence/absence only; peak intensities and multiple collision-energy behavior are not modeled
- ANN models alone are used in the notebook prediction workflow because they achieved better results than GNN and COM models

## Evidence

- [results] Compared on the test set at 0.2 m/z binning, all three SingleFrag variants achieve higher precision, accuracy, and cosine similarity than CFM-ID and 3DMolMS: [evidence span withheld — anchor status: not_found]
