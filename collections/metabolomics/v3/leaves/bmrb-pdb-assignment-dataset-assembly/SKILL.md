---
name: bmrb-pdb-assignment-dataset-assembly
description: Use when building a training corpus for a chemical shift prediction model
  from public assignments and coordinates, specifically pairing BMRB entries with
  their associated PDB entries and evaluating a new predictor (e.
when_to_use_negative:
- When the goal is to assign new peaks or process raw FIDs rather than assemble a
  training corpus from public BMRB/PDB data — use NMRFx Processor/analysis tools instead.
- When the molecule set is paramagnetic, contains non-standard residues, or lacks
  a matching PDB entry — the article restricts the protein training dataset to standard
  residues with matching BMRB and PDB entries.
- When evaluating a predictor whose training set was not assembled from BMRB assignments
  plus PDB-derived structural features (e.g., ShiftX2-style 61-protein test set) —
  the comparison is only apples-to-apples within the same assembled dataset.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3272
tools:
- name: NMRFx
  role: Application in which the chemical shift prediction models are built and evaluated;
    provides the integrated environment for assembling the BMRB/PDB training set and
    running the FM-vs-SVR comparison.
  repo: github.com/nanalysis/nmrfx
- name: Tribuo
  role: Library used to train the FM chemical shift prediction models via stochastic
    gradient descent over the assembled BMRB/PDB dataset.
- name: BMRB
  role: Source of experimentally assigned chemical shifts for both RNA and protein
    training entries.
- name: PDB
  role: Source of the associated structural coordinates used to compute the structural
    features paired with each BMRB shift.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42004-025-01812-8
    title: NMR data processing, visualization, analysis and structure calculation
      with NMRFx
  source_content_hash: sha256:d1aa9014bde47880821e2bf6931d649a009cbb05f5c21872288adb25c532e11b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/nanalysis/nmrfx
  tools:
  - BMRB
  - NMRFx
  - PDB
  - Tribuo
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# bmrb-pdb-assignment-dataset-assembly

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assembles a paired dataset of BMRB chemical shift assignments and matching PDB structures (371 RNA molecules; 1480 non-paramagnetic proteins) to train and evaluate NMRFx Factorization Machine chemical shift predictors against prior SVR baselines.

## When to use

Use when building a training corpus for a chemical shift prediction model from public assignments and coordinates, specifically pairing BMRB entries with their associated PDB entries and evaluating a new predictor (e.g., Factorization Machines) against a prior SVR model by cross-validated RMSD per atom type (1H, 13C, 15N).

## When NOT to use

- When the goal is to assign new peaks or process raw FIDs rather than assemble a training corpus from public BMRB/PDB data — use NMRFx Processor/analysis tools instead.
- When the molecule set is paramagnetic, contains non-standard residues, or lacks a matching PDB entry — the article restricts the protein training dataset to standard residues with matching BMRB and PDB entries.
- When evaluating a predictor whose training set was not assembled from BMRB assignments plus PDB-derived structural features (e.g., ShiftX2-style 61-protein test set) — the comparison is only apples-to-apples within the same assembled dataset.

## Inputs

- BMRB entries with chemical shift assignments for 371 RNA molecules
- Associated PDB coordinate files for those 371 RNA molecules
- BMRB/PDB matched entries for 1480 non-paramagnetic proteins with standard residues (protein predictor variant)
- Prior SVR model definitions/baselines (NMRViewJ-era) for comparison

## Outputs

- Per-atom-type cross-validated RMSD values for Factorization Machine models (1H, 13C, 15N)
- Per-atom-type cross-validated RMSD values for SVR baselines
- Comparison table (FM vs SVR) equivalent to the SI evaluation table
- Training records pairing each assigned chemical shift with structural features from the paired PDB structure

## How to apply

Retrieve the BMRB chemical shift assignments for the target set of molecules (371 RNAs for the RNA predictor, 1480 non-paramagnetic standard-residue proteins for the protein predictor) and the corresponding PDB coordinates. Pair each assigned chemical shift with structural features computed from the matching PDB structure to generate training records. Train the Factorization Machine models with stochastic gradient descent as implemented in the Tribuo library, one model per atom class (1H, 13C, 15N), and evaluate them by cross-validated RMSD between predicted and experimentally assigned shifts for each atom type. Train the prior SVR models on the same assembled data to obtain a baseline for the FM-vs-SVR comparison. Confirm that the FM model handles the increased number of training points while maintaining high performance relative to the SVR baseline.

## Related tools

- **NMRFx** (Application in which the chemical shift prediction models are built and evaluated; provides the integrated environment for assembling the BMRB/PDB training set and running the FM-vs-SVR comparison.) — github.com/nanalysis/nmrfx
- **Tribuo** (Library used to train the FM chemical shift prediction models via stochastic gradient descent over the assembled BMRB/PDB dataset.)
- **BMRB** (Source of experimentally assigned chemical shifts for both RNA and protein training entries.)
- **PDB** (Source of the associated structural coordinates used to compute the structural features paired with each BMRB shift.)

## Examples

```
# Assemble the 371-RNA BMRB/PDB paired training set and evaluate FM vs SVR by cross-validated RMSD per atom type
python build_shift_dataset.py --molecule-set rna --n 371 --bmrb --pdb --atom-types 1H,13C,15N --model fm --baseline svr --cv rmsd
```

## Evaluation signals

- The number of retrieved RNA molecules with BMRB assignments and an associated PDB entry should equal 371.
- The number of protein entries with matching BMRB and PDB coordinates should equal 1480, all non-paramagnetic and standard residues.
- Cross-validated RMSD per atom type (1H, 13C, 15N) for the FM model should match the values in the SI evaluation table.
- SVR baseline cross-validated RMSD should be computed on the identical assembled dataset so the FM-vs-SVR comparison is valid.
- The FM model should demonstrate the reported property of handling a significant increase in training data points while maintaining high performance.

## Limitations

- The RNA model cross-validated RMSD evaluation table is only provided in the SI, not in the main text.
- Chemical shift prediction attribute values are only listed in Table S6 of the SI.
- Protein training set is restricted to non-paramagnetic proteins with standard residues and matching BMRB/PDB entries, so paramagnetic or non-standard-residue entries are excluded.
- The article states that NMRFx is under active development, so exact tooling details may evolve.

## Evidence

- [discussion] The current dataset consists of 371 RNA molecules that have assignments in the BMRB and associated PDB structures.: "The current dataset consists of 371 RNA molecules that have assignments in the BMRB and associated PDB structures."
- [discussion] To train the protein shift prediction model, a dataset of 1480 non-paramagnetic proteins comprising standard residues with matching BMRB and PDB entries was used.: "To train the protein shift prediction model, a dataset of 1480 non-paramagnetic proteins comprising standard residues with matching BMRB and PDB entries was used."
- [discussion] FM models were trained with stochastic gradient descent as implemented in the Tribuo library41.: "FM models were trained with stochastic gradient descent as implemented in the Tribuo library41."
- [discussion] The updated FM model robustly handles a significant increase in training data points while maintaining high performance.: "The updated FM model robustly handles a significant increase in training data points while maintaining high performance."
- [discussion] A table in the SI provides an evaluation of the model by cross-validated RMSD for each atom type (1H, 13C, 15N): "A table in the SI provides an evaluation of the model by cross-validated RMSD for each atom type (1H, 13C, 15N)"
