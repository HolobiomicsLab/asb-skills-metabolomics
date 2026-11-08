---
name: frequency-based-scaffold-candidate-aggregation
description: Use when pre-computed ranked scaffold candidate lists (e.g., ranks_msgym_pred.pkl
  from Zenodo record 17115068) are available for MS/MS test spectra and one predicted
  scaffold per chemical formula is needed to compute Scaffold Prediction Accuracy
  (SPA).
when_to_use_negative:
- Evaluating MADGEN with the oracle retriever — the oracle lookup table always yields
  the correct scaffold given the spectrum and chemical formula, so no aggregation
  over ranked candidates applies.
- Running stage 2 of MADGEN — this skill only selects the scaffold; scaffold-conditioned
  de novo molecule generation with spectra guidance is a separate step.
- No trained contrastive retrieval model or pre-computed candidate ranking exists
  — frequency aggregation cannot be performed without the top-k ranked candidate scaffolds.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MADGEN
  role: Predictive retriever whose contrastive ranking produces the top-k candidate
    scaffolds that the frequency aggregation consumes
  repo: https://github.com/HassounLab/MADGEN
- name: RDKit
  role: Extracts ground-truth core scaffolds from the molecular graph representations
    for comparison against the aggregated prediction
- name: Python 3.9
  role: Runs the aggregation script over ranking pickles and writes spa_evaluation.csv
    (madgen conda environment)
- name: wget
  role: Downloads the processed spectra pickles (Zenodo 15036069) and pre-computed
    predictive ranking pickles (Zenodo 17115068)
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:a2987b5c8c321b6fb535c51a513d9ef118f4c012d037ddd8173bfebba9be239b
  source_papers:
  - doi: 10.48550/arxiv.2501.01950
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HassounLab/MADGEN
  tools:
  - MADGEN
  - Python
  - Python 3.9
  - RDKit
  - conda
  - wget
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# frequency-based-scaffold-candidate-aggregation

## Summary

MADGEN's post-retrieval step that converts a predictive retriever's ranked scaffold candidates into a single predicted scaffold per test MS/MS spectrum by counting scaffold frequencies among the top-k ranked candidates per chemical formula. It yields the Scaffold Prediction Accuracy (SPA) that upper-bounds MADGEN's end-to-end generation performance under the predictive-retrieval setting.

## When to use

Use when pre-computed ranked scaffold candidate lists (e.g., ranks_msgym_pred.pkl from Zenodo record 17115068) are available for MS/MS test spectra and one predicted scaffold per chemical formula is needed to compute Scaffold Prediction Accuracy (SPA).

## When NOT to use

- Evaluating MADGEN with the oracle retriever — the oracle lookup table always yields the correct scaffold given the spectrum and chemical formula, so no aggregation over ranked candidates applies.
- Running stage 2 of MADGEN — this skill only selects the scaffold; scaffold-conditioned de novo molecule generation with spectra guidance is a separate step.
- No trained contrastive retrieval model or pre-computed candidate ranking exists — frequency aggregation cannot be performed without the top-k ranked candidate scaffolds.

## Inputs

- Pre-processed spectra pickles msgym.pkl and canopus.pkl from Zenodo record 15036069 (normalized intensities, low-intensity peaks below threshold removed)
- Pre-computed predictive retrieval rankings ranks_msgym_pred.pkl and ranks_canopus_pred.pkl from Zenodo record 17115068
- Ground-truth molecular graph representations (SMILES) of test-set molecules for RDKit scaffold extraction

## Outputs

- One predicted scaffold (SMILES) per test spectrum/chemical formula
- Per-dataset Scaffold Prediction Accuracy (SPA) table written to spa_evaluation.csv

## How to apply

Load the pre-processed spectra pickles (msgym.pkl, canopus.pkl) whose intensities were normalized with low-intensity peaks removed below a predefined noise threshold, then read the ranked scaffold candidates for each test spectrum from the predictive-retrieval pickle (ranks_msgym_pred.pkl or ranks_canopus_pred.pkl). For each data point, take the top-k ranked candidate scaffolds and count the frequency of each scaffold, selecting the most frequently occurring scaffold per chemical formula as the prediction. Extract ground-truth core scaffolds from the molecular graph representations with RDKit, noting that scaffolds are unique to each split so no leakage occurs. Compare predicted versus ground-truth scaffolds and compute SPA per dataset (NIST23, CANOPUS, MassSpecGym), writing the results to spa_evaluation.csv. Validate by reproducing the reported pattern: the best SPA is on the dataset with the least scaffold diversity (~40.3%), with lower SPA on CANOPUS and MassSpecGym where scaffold diversity is higher.

## Related tools

- **MADGEN** (Predictive retriever whose contrastive ranking produces the top-k candidate scaffolds that the frequency aggregation consumes) — https://github.com/HassounLab/MADGEN
- **RDKit** (Extracts ground-truth core scaffolds from the molecular graph representations for comparison against the aggregated prediction)
- **Python 3.9** (Runs the aggregation script over ranking pickles and writes spa_evaluation.csv (madgen conda environment))
- **wget** (Downloads the processed spectra pickles (Zenodo 15036069) and pre-computed predictive ranking pickles (Zenodo 17115068))

## Examples

```
wget https://zenodo.org/records/17115068/files/ranks_msgym_pred.pkl?download=1 -O ./data/msgym/raw/ranks_msgym_pred.pkl
```

## Evaluation signals

- Reproduced SPA spans the reported range of 13.2% to 40.3%, with the highest SPA (~40.3%) on NIST23 (the least scaffold-diverse dataset) and lower SPA on CANOPUS and MassSpecGym.
- Exactly one predicted scaffold is emitted per chemical formula — the most frequently occurring scaffold among the top-k ranked candidates.
- The predicted scaffold must be drawn from the ranked candidate list of the same split; scaffolds are unique to each split, so a predicted scaffold matching a different split's scaffold set indicates leakage.
- The predicted scaffold exactly matches the RDKit-extracted ground-truth core scaffold (exact-match criterion underlying SPA).
- Results table spa_evaluation.csv is produced with one SPA value per dataset.

## Limitations

- The NIST23 dataset is commercially available and requires separate licensing; it is not downloadable from the Zenodo records, so the aggregation cannot be re-run for NIST23 without licensed data.
- SPA is substantially lower on CANOPUS and MassSpecGym because their scaffolds are more complex/diverse, so aggregated predictions frequently miss the true scaffold and limit downstream generation quality.
- Predictive retrieval results are only distributed for MSGym and CANOPUS (ranks_msgym_pred.pkl, ranks_canopus_pred.pkl); NIST ranks must be generated by retraining the scaffold_retrieval module.

## Evidence

- [methods] Frequency-based aggregation selects the modal scaffold among top-k ranked candidates per chemical formula.: [evidence span withheld — anchor status: not_found]
