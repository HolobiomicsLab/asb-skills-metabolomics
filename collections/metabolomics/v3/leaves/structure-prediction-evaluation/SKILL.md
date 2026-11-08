---
name: structure-prediction-evaluation
description: Use when scoring de novo structure predictions from tandem MS2 spectra
  against known ground-truth SMILES, requiring exact-match, Tanimoto>0.9, and Tanimoto>0.8
  recovery rates on a held-out spectral test set.
when_to_use_negative:
- 'Input spectra are high-resolution and high-accuracy: the article notes TeFT may
  not outperform other deep models on high-quality resolution spectra, so comparison-focused
  evaluation should note this regime mismatch.'
- 'Spectra were recorded in negative ion mode: TeFT was only trained on positive ion
  mode (H+) spectra, so evaluation of such data would be out of scope.'
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: TeFT
  role: De novo structure generation model whose MS2-derived structure predictions
    are being evaluated against ground truth
  repo: thumingo/TeFT
- name: SIRIUS4
  role: Baseline structure prediction method used for comparative evaluation (27.6%
    exact, 39% >0.9 similarity)
- name: Transformer
  role: Deep learning module generating the complete target molecular structure from
    MS2 spectra within TeFT
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42004-024-01189-0
    title: Structural annotation of unknown molecules in a miniaturized mass spectrometer
      based on a transformer enabled fragment tree method
  source_content_hash: sha256:6bb84ec3181605261016aec93f3b32df8526225f37a3f9a65f4bd79748247d76
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thumingo/TeFT
  tools:
  - SIRIUS
  - SIRIUS4
  - TeFT
  - Transformer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# structure-prediction-evaluation

## Summary

Evaluates de novo MS2-to-structure prediction models (e.g., TeFT vs. SIRIUS4) by comparing predicted vs. ground-truth molecular structures as SMILES using exact-match rates and MACCS-key/Morgan fingerprint Tanimoto similarity thresholds on a held-out test set.

## When to use

Use when scoring de novo structure predictions from tandem MS2 spectra against known ground-truth SMILES, requiring exact-match, Tanimoto>0.9, and Tanimoto>0.8 recovery rates on a held-out spectral test set.

## When NOT to use

- Input spectra are high-resolution and high-accuracy: the article notes TeFT may not outperform other deep models on high-quality resolution spectra, so comparison-focused evaluation should note this regime mismatch.
- Spectra were recorded in negative ion mode: TeFT was only trained on positive ion mode (H+) spectra, so evaluation of such data would be out of scope.

## Inputs

- Non-repetitive MS2 test spectra compiled from open-source mass spectrometry databases (GNPS, HMDB 5.0, MoNA)
- Predicted candidate molecular structures from the model under evaluation (e.g., TeFT output)
- Ground-truth real molecular structures for each test spectrum
- Baseline predictions from a comparison model (e.g., SIRIUS4)

## Outputs

- Counts of exact structure matches and matches above 0.9 and 0.8 Tanimoto similarity thresholds
- Percentage recovery rates (e.g., 195/660, 311/660, 439/660)
- Aggregate metrics written to results_table.csv

## How to apply

Compile a non-repetitive MS2 test set (e.g., 660 spectra drawn from open-source databases GNPS, HMDB 5.0, and MoNA) and run the prediction model, such as TeFT from thumingo/TeFT, over each spectrum to obtain a best candidate molecular structure. Convert both predicted and real structures to SMILES and compute MACCS-key and Morgan fingerprint Tanimoto similarity scores between them. Count spectra where the best candidate exactly matches the real structure (Tanimoto = 1) and where similarity exceeds the 0.9 and 0.8 thresholds, then convert counts to percentages out of the total test set. Report the aggregate metrics in a results table and benchmark against a baseline such as SIRIUS4 (reported 27.6% exact matches and 39% >0.9 similarity on the same 660-spectrum set).

## Related tools

- **TeFT** (De novo structure generation model whose MS2-derived structure predictions are being evaluated against ground truth) — thumingo/TeFT
- **SIRIUS4** (Baseline structure prediction method used for comparative evaluation (27.6% exact, 39% >0.9 similarity))
- **Transformer** (Deep learning module generating the complete target molecular structure from MS2 spectra within TeFT)

## Examples

```
from rdkit import DataStructs; sim = DataStructs.TanimotoSimilarity(fp_pred, fp_true)
```

## Evaluation signals

- Reproduce the reported counts on the 660-spectrum test set: 195 exact matches (Tanimoto = 1), 311 candidates above 0.9 similarity, and 439 above 0.8 similarity.
- Exact-match spectra must show Tanimoto similarity of 1 for both MACCS-key and Morgan fingerprint comparisons.
- Computed percentages must equal counts/660 (approximately 30%, 47%, and 67% respectively).
- Baseline SIRIUS4 comparison on the same set should fall at or below TeFT figures (27.6% exact, 39% >0.9).
- Aggregate metrics are written to results_table.csv with the expected row/column structure.

## Limitations

- TeFT was trained only on spectra recorded in positive ion mode (H+), restricting applicability of evaluations to that ionization mode.
- The model may not show exceptional performance advantages over other deep learning models on high-quality/high-resolution spectra compared to low-resolution miniaturized mass spectrometer data.
- Evaluation relies on converting both predicted and real structures to standardized SMILES; failures or ambiguities in SMILES standardization will distort fingerprint similarity scores.

## Evidence

- [discussion] TeFT de novo generation from low-res MSn: "We propose a novel prediction model, TeFT, designed for de novo structure generation from low-resolution tandem mass spectrometry (MSn) spectra"
- [discussion] Training data sources GNPS/HMDB/MoNA: "The training set utilized for the Transformer model was created by compiling open-source spectrometry databases GNPS"
- [abstract] Evaluation on open-source databases successful: "The evaluation results on different open-source databases indicated that the proposed model achieved remarkable results"
- [readme] README description of TeFT: "TeFT, a predict model combines various types of fragmentation tree models and the deep learning Transformer module."
