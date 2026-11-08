---
name: hdf5-prediction-results-analysis
description: Use when NMR2Struct inference (nmr_infer or nmr_infer_single_spectrum)
  has produced an hdf5 predictions file and you need to compute exact-match accuracy,
  e.g. benchmarking the 106 experimental spectra against the 69.6% simulated baseline.
when_to_use_negative:
- You are evaluating the substructure-to-structure transformer instead of the spectrum-to-structure
  multitask model — that task's accuracy (93.2%, k=5 sampling) is scored differently
  against 957-substructure inputs
- The hdf5 file contains raw 1H/13C spectra rather than model predictions — spectral
  preprocessing (interpolation to the 28000-point 1H grid, 80-bin 13C binning) must
  happen before inference, not after
- You need stereochemistry-resolved evaluation — the model does not predict stereochemistry,
  so exact-match scoring is undefined for stereoisomers
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3474
tools:
- name: NMR2Struct
  role: Generates the processed_predictions.h5 top-15 SMILES predictions that this
    skill scores, via its analysis/inference config sections
  repo: https://github.com/MarklandGroup/NMR2Struct
- name: RDKit
  role: Canonicalizes predicted and target SMILES strings for exact-match comparison
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acscentsci.4c01132
    title: Accurate and Efficient Structure Elucidation from Routine One-Dimensional
      NMR Spectra Using Multitask Machine Learning
  source_content_hash: sha256:30ac1670f857f007a72c82affeafac7f5a3665df44cf097893410a05b3aaae53
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarklandGroup/NMR2Struct
  tools:
  - Anaconda
  - NMR2Struct
  - RDKit
  - convolutional neural network (CNN)
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# hdf5-prediction-results-analysis

## Summary

Scores the top-15 SMILES predictions written by NMR2Struct multitask-model inference into processed_predictions.h5 against known target molecules, computing exact-match (top-k) accuracy — e.g. the 33.0% experimental accuracy vs the 69.6% simulated baseline reported for 1H/13C NMR structure elucidation.

## When to use

Use when NMR2Struct inference (nmr_infer or nmr_infer_single_spectrum) has produced an hdf5 predictions file and you need to compute exact-match accuracy, e.g. benchmarking the 106 experimental spectra against the 69.6% simulated baseline.

## When NOT to use

- You are evaluating the substructure-to-structure transformer instead of the spectrum-to-structure multitask model — that task's accuracy (93.2%, k=5 sampling) is scored differently against 957-substructure inputs
- The hdf5 file contains raw 1H/13C spectra rather than model predictions — spectral preprocessing (interpolation to the 28000-point 1H grid, 80-bin 13C binning) must happen before inference, not after
- You need stereochemistry-resolved evaluation — the model does not predict stereochemistry, so exact-match scoring is undefined for stereoisomers

## Inputs

- processed_predictions.h5 — hdf5 file of raw top-15 SMILES predictions per spectrum written by nmr_infer
- Ground-truth target SMILES (numpy .npy binary-encoded strings; decode with x.decode('utf-8'))
- NMR2Struct YAML config with global_args (save directory) and analysis sections

## Outputs

- Overall exact-match (top-15) accuracy over the 106 experimental spectra (~33.0% expected)
- experimental_eval_results.json recording the experimental accuracy and the gap vs the 69.6% simulated baseline
- Per-spectrum hit/no-hit flags for the 15 candidate SMILES

## How to apply

Load processed_predictions.h5 from the save directory defined in the config's global_args section; each spectrum should have 15 candidate SMILES generated with top-k sampling (k=5). Canonicalize every prediction and the target SMILES with RDKit, then mark a spectrum as a hit when the target canonical SMILES appears among the valid predictions. Average the per-spectrum hit flags over the 106 experimental 1H/13C spectra (expected ~33.0%) and record the simulated-to-experimental gap relative to the 69.6% simulated accuracy in experimental_eval_results.json. Use the config's analysis section settings, since the README states inference metrics are computed on hdf5 raw predictions written at write_freq intervals.

## Related tools

- **NMR2Struct** (Generates the processed_predictions.h5 top-15 SMILES predictions that this skill scores, via its analysis/inference config sections) — https://github.com/MarklandGroup/NMR2Struct
- **RDKit** (Canonicalizes predicted and target SMILES strings for exact-match comparison)

## Examples

```
import h5py; preds = h5py.File('processed_predictions.h5','r')['spectra']; hits = sum(target in [p.decode('utf-8') for p in row[:15]] for row in preds)/106
```

## Evaluation signals

- Recomputed experimental accuracy over the 106 spectra is reproducible at approximately 33.0%
- Recorded simulated-to-experimental gap equals ~36.6 percentage points (69.6% simulated minus ~33.0% experimental) and is written to experimental_eval_results.json
- Every spectrum's prediction set contains exactly 15 SMILES candidates (top-k sampling with k=5)
- All predictions canonicalize successfully with RDKit before comparison; invalid SMILES are excluded rather than counted as misses
- A hit is scored only when the target canonical SMILES string matches a prediction exactly

## Limitations

- The model was trained on simulated spectra (MestreNova-predicted), which 'are not fully captured by the simulated spectra used to train the current model' — hence the large simulated-to-experimental gap
- Framework is limited to molecules with up to 19 heavy (non-hydrogen) atoms; accuracy on larger or elementally diverse compounds is out of scope
- Predictions are non-deterministic SMILES generated by top-k sampling, so accuracy can vary run-to-run without a fixed seed

## Evidence

- [abstract] Top-15 accuracy metric: "we show that our approach predicts the exact molecule 69.6% of the time within the first 15 predictions, reducing the search space by up to 11 orders of magnitude."
- [results] Canonicalization-based scoring: [evidence span withheld — anchor status: not_found]
- [readme] hdf5 prediction output location: "Running inference writes the raw predictions as an hdf5 file to the save directory that you specified under the ```global_args``` section"
