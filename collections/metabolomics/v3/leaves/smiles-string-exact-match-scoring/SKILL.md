---
name: smiles-string-exact-match-scoring
description: Use when evaluating SMILES-generating models (NMR2Struct multitask or
  substructure-to-structure transformer) where correctness is judged by whether the
  target canonical SMILES appears among the top-k generated predictions.
when_to_use_negative:
- When the analysis goal is similarity assessment rather than exact identification
  — use Tanimoto similarity instead (89.6% of best incorrect predictions had Tanimoto
  ≥0.50, average 0.68, which exact match scores as zero)
- When model outputs are substructure vectors or spectra rather than SMILES strings
  — exact string matching does not apply
- When stereochemical correctness must be assessed — the model does not yet predict
  stereochemistry, so exact SMILES match cannot judge stereo accuracy
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_2258
tools:
- name: RDKit
  role: Canonicalize target and prediction SMILES strings before exact-match comparison
  repo: https://github.com/MarklandGroup/NMR2Struct
- name: NMR2Struct
  role: Framework that generates the SMILES predictions and writes processed_predictions.h5
    scored by this metric
  repo: https://github.com/MarklandGroup/NMR2Struct
- name: transformer
  role: Encoder-decoder model that generates 15 SMILES predictions per input via top-k
    sampling (k=5)
  repo: https://github.com/MarklandGroup/NMR2Struct
provenance:
  source_task_ids:
  - task_003
  - task_008
  source_papers:
  - doi: 10.1021/acscentsci.4c01132
    title: Accurate and Efficient Structure Elucidation from Routine One-Dimensional
      NMR Spectra Using Multitask Machine Learning
  source_content_hash: sha256:100865d280d5afac7ba0b2eb93c2bc6821bf7028567cc55380bdfd0724d673bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarklandGroup/NMR2Struct
  tools:
  - Anaconda
  - NMR2Struct
  - RDKit
  - Zenodo record 13892026
  - convolutional neural network (CNN)
  - transformer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# SMILES string exact-match scoring

## Summary

Scores generated molecular structures by exact canonical-SMILES match: a prediction counts as correct only if the target's canonical SMILES string appears verbatim among the model's generated SMILES predictions (e.g., top-15 from top-k sampling). This is the metric behind the paper's headline 69.6% spectrum-to-structure and 93.2% substructure-to-structure accuracies.

## When to use

Use when evaluating SMILES-generating models (NMR2Struct multitask or substructure-to-structure transformer) where correctness is judged by whether the target canonical SMILES appears among the top-k generated predictions.

## When NOT to use

- When the analysis goal is similarity assessment rather than exact identification — use Tanimoto similarity instead (89.6% of best incorrect predictions had Tanimoto ≥0.50, average 0.68, which exact match scores as zero)
- When model outputs are substructure vectors or spectra rather than SMILES strings — exact string matching does not apply
- When stereochemical correctness must be assessed — the model does not yet predict stereochemistry, so exact SMILES match cannot judge stereo accuracy

## Inputs

- processed_predictions.h5 with valid_predictions group and per-molecule prediction_strings
- Target canonical SMILES strings (e.g., stored as binary-encoded arrays in .npy smiles files)
- RDKit canonicalization of targets and predictions

## Outputs

- Per-molecule exact-match hit indicators (binary correct/incorrect)
- Overall top-15 exact-match accuracy (e.g., 69.6% simulated, ~33.0% experimental, 93.2% substructure-to-structure)
- Optionally, size-stratified accuracy table (e.g., results.tsv by heavy-atom bin)

## How to apply

Generate 15 SMILES predictions per input via top-k sampling (k=5), then canonicalize both the target and prediction SMILES strings with RDKit before comparison, as the paper does when computing substructure vectors from canonicalized SMILES. Load predictions from processed_predictions.h5 (valid_predictions group containing prediction_strings) and count a molecule as correct when its canonical target SMILES string appears among its valid predictions (exact string match, top-15). When reading SMILES from the NMR2Struct .npy label files, decode each entry with x.decode('utf-8'). Aggregate the per-molecule binary hits into an overall accuracy; for size-stratified analysis, group hits by the num_heavy_atoms field (e.g., bins at 10, 17, 18-19 heavy atoms) before averaging.

## Related tools

- **RDKit** (Canonicalize target and prediction SMILES strings before exact-match comparison) — https://github.com/MarklandGroup/NMR2Struct
- **NMR2Struct** (Framework that generates the SMILES predictions and writes processed_predictions.h5 scored by this metric) — https://github.com/MarklandGroup/NMR2Struct
- **transformer** (Encoder-decoder model that generates 15 SMILES predictions per input via top-k sampling (k=5)) — https://github.com/MarklandGroup/NMR2Struct

## Examples

```
from rdkit import Chem; hit = Chem.MolToSmiles(Chem.MolFromSmiles(target)) in [Chem.MolToSmiles(Chem.MolFromSmiles(p.decode('utf-8'))) for p in preds[:15]]
```

## Evaluation signals

- Substructure-to-structure test-set accuracy under this metric should reproduce ~93.2% (96.0% at 10 heavy atoms, 92.8% at 17 heavy atoms)
- Multitask spectrum-to-structure top-15 accuracy on simulated spectra should reproduce 69.6%; on the 106 experimental spectra, ~33.0%
- Every target and prediction must be canonicalized with RDKit before comparison — raw SMILES string equality can undercount hits due to non-canonical string variants
- Size-stratified accuracy must decline with heavy-atom count (77.5% at 10 heavy atoms to 52.0% at 19 heavy atoms for the multitask model)
- All SMILES read from .npy files must be decoded with x.decode('utf-8') before matching

## Limitations

- Exact match gives zero credit to near-misses; the paper reports 89.6% of best incorrect predictions had Tanimoto similarity ≥0.50 with average 0.68, information lost under this metric
- Accuracy drops below 80% for molecules with 18 or 19 heavy atoms, and only ~1% of training molecules have 18-19 heavy atoms, so exact-match scores are unreliable at the largest sizes
- Results depend on the decoding choices of the generator — top-k sampling with k=5 and 15 predictions per input defines the pool against which the exact match is computed

## Evidence

- [results] top-k random sampling36 was used to generate SMILES from a given substructure array with k = 5, and 15 predictions were generated for each input.: "top-k random sampling36 was used to generate SMILES from a given substructure array with k = 5, and 15 predictions were generated for each input."
- [results] After canonicalizing all the SMILES strings using RDKit,31 the substructure vector for each molecule was computed using RDKit's substructure match functionality: "After canonicalizing all the SMILES strings using RDKit,31 the substructure vector for each molecule was computed using RDKit’s substructure match functionality"
- [readme] Be sure to do x.decode('utf-8') when reading strings from this format.: "Be sure to do ```x.decode('utf-8')``` when reading strings from this format."
- [readme] This inference workflow is intended for running inference on a model on a dataset that was used for training, i.e. getting predictions over a test set to then compute metrics on.: "This inference workflow is intended for running inference on a model on a dataset that was used for training, i.e. getting predictions over a test set to then compute metrics on."
