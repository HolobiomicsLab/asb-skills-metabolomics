---
name: training-set-size-bias-attribution
description: Use when a structure-prediction model's accuracy must be stratified by
  molecule size (num_heavy_atoms) and a systematic decline — e.g. 77.5% at 10 atoms
  vs 52.0% at 19 atoms — needs to be attributed to data imbalance versus problem complexity.
when_to_use_negative:
- Predictions come from the substructure-to-structure transformer rather than the
  spectrum-to-structure multitask model — use the substructure-task reference accuracies
  (96.0%/92.8%) directly instead of recomputing from processed_predictions.h5.
- The goal is overall top-15 accuracy (69.6%) rather than size-stratified attribution;
  a single aggregate metric suffices.
- Molecules exceed 19 heavy atoms, outside the framework's demonstrated range, so
  size-binned extrapolation is not supported.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: NMR2Struct
  role: Source of the multitask model and its processed test-set prediction files
    (processed_predictions.h5); inference writes raw predictions as hdf5 to the save
    directory
  repo: https://github.com/MarklandGroup/NMR2Struct
- name: h5py
  role: Python library used to load valid_predictions and bad_predictions groups from
    processed_predictions.h5
- name: RDKit
  role: Canonicalization of SMILES strings so exact string match between targets and
    predictions is a valid accuracy criterion
- name: Zenodo record 13892026
  role: Provides the training, validation, and test data needed to compute training-set
    composition by heavy-atom count
  repo: https://zenodo.org/records/13892026
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acscentsci.4c01132
    title: Accurate and Efficient Structure Elucidation from Routine One-Dimensional
      NMR Spectra Using Multitask Machine Learning
  source_content_hash: sha256:3fa457e8e80612c4573805d06f8ee50cab5eb9b4dc27375ec8053c64e441a366
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarklandGroup/NMR2Struct
  tools:
  - NMR2Struct
  - RDKit
  - Zenodo record 13892026
  - h5py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Training-set size bias attribution

## Summary

Quantifies how multitask spectrum-to-structure model accuracy degrades with heavy-atom count and attributes the decline to both combinatorial growth of the search space and training-set sparsity (only ~1% of training molecules have 18–19 heavy atoms).

## When to use

Use when a structure-prediction model's accuracy must be stratified by molecule size (num_heavy_atoms) and a systematic decline — e.g. 77.5% at 10 atoms vs 52.0% at 19 atoms — needs to be attributed to data imbalance versus problem complexity.

## When NOT to use

- Predictions come from the substructure-to-structure transformer rather than the spectrum-to-structure multitask model — use the substructure-task reference accuracies (96.0%/92.8%) directly instead of recomputing from processed_predictions.h5.
- The goal is overall top-15 accuracy (69.6%) rather than size-stratified attribution; a single aggregate metric suffices.
- Molecules exceed 19 heavy atoms, outside the framework's demonstrated range, so size-binned extrapolation is not supported.

## Inputs

- processed_predictions.h5 (h5py file with valid_predictions and bad_predictions groups, each containing prediction_strings and per-molecule num_heavy_atoms)
- canonical SMILES target strings for the test set
- reference accuracies (96.0% at 10 heavy atoms, 92.8% at 17 heavy atoms) from the substructure-to-structure task

## Outputs

- results.tsv — tab-separated table of heavy-atom bin, accuracy, molecule-count magnitude, and accuracy decrease, with a note on training-set composition
- per-bin accuracy summary (e.g. 77.5% at 10 atoms, 52.0% at 19 atoms, ~25.5-percentage-point drop at 17→18–19 atoms)

## How to apply

Load the multitask model's test-set predictions from processed_predictions.h5 using h5py, reading the valid_predictions and bad_predictions groups. Group molecules by the num_heavy_atoms field stored per molecule into bins at 10, 17, and 18–19 heavy atoms. For each bin, compute accuracy as the fraction of targets whose canonical SMILES appears (exact string match) among the generated prediction_strings, mirroring the paper's metric of 15 predictions per input via top-k sampling with k=5. Cross-check the 10-atom and 17-atom bins against the published substructure-to-structure references of 96.0% and 92.8% to validate binning. Then compute the absolute accuracy decrease between adjacent bins — expected ~25.5 percentage points between 17 and 18–19 heavy atoms, with the 18–19 bin below 80% — alongside the ~5-order-of-magnitude growth in combinatorial molecule count between those sizes. Tabulate heavy-atom bin, accuracy, molecule-count magnitude, and decrease into results.tsv, annotating the decline with the note that only ~1% of training molecules contain 18–19 heavy atoms.

## Related tools

- **NMR2Struct** (Source of the multitask model and its processed test-set prediction files (processed_predictions.h5); inference writes raw predictions as hdf5 to the save directory) — https://github.com/MarklandGroup/NMR2Struct
- **h5py** (Python library used to load valid_predictions and bad_predictions groups from processed_predictions.h5)
- **RDKit** (Canonicalization of SMILES strings so exact string match between targets and predictions is a valid accuracy criterion)
- **Zenodo record 13892026** (Provides the training, validation, and test data needed to compute training-set composition by heavy-atom count) — https://zenodo.org/records/13892026

## Examples

```
import h5py; f = h5py.File('processed_predictions.h5','r'); preds = f['valid_predictions']
```

## Evaluation signals

- The 10-atom and 17-atom substructure-to-structure reference bins reproduce 96.0% and 92.8% within tolerance, confirming correct binning and matching.
- The multitask model's 10-heavy-atom bin lands near 77.5% and the 18–19 bin near 52.0%, with an absolute decrease of ~25.5 percentage points between the 17 and 18–19 bins.
- The 18–19 heavy-atom bin accuracy is below 80%, consistent with the published drop.
- Combinatorial molecule-count magnitude grows ~5 orders of magnitude between the 17-atom and 18–19-atom bins.
- Every target's accuracy criterion is an exact canonical-SMILES string match against prediction_strings, verified after RDKit canonicalization of both sides.

## Limitations

- The framework is demonstrated only on molecules with up to 19 heavy atoms, so size-stratified analysis cannot be extended beyond that.
- The decline partially reflects training-set imbalance: only ~1% of training molecules have 18–19 heavy atoms, confounding attribution to problem size alone.
- The model is trained on simulated MestreNova-predicted SpectraBase spectra, so size-stratified accuracy may not transfer to experimental spectra.

## Evidence

- [results] Substructure-to-structure accuracy by size: "at 10 heavy atoms, where the problem size is ∼2 million, the accuracy is 96.0% and only drops to 92.8% by 17 heavy atoms"
- [results] Accuracy drop for largest molecules: "the accuracy drops below 80% for the molecules with 18 or 19 heavy atoms"
- [readme] Predictions file format: "Running inference writes the raw predictions as an hdf5 file to the save directory that you specified under the ```global_args``` section"
- [results] SMILES canonicalization basis: "After canonicalizing all the SMILES strings using RDKit,31 the substructure vector for each molecule was computed"
