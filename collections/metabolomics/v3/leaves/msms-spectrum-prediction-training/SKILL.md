---
name: msms-spectrum-prediction-training
description: Use when a task requires simulating MS/MS spectra from a molecular structure
  — e.
when_to_use_negative:
- The goal is formula-level rather than fragment-level prediction — SCARF (also in
  ms-pred) predicts spectra at the level of chemical formula and may be the more appropriate
  comparator
- Only a small NPLIB1/CANOPUS-style dataset is needed, since main-branch support for
  NPLIB1 is retired and requires the legacy iceberg_1.0 branch
- 'Throughput matters more than interpretability: ICEBERG costs roughly 1 CPU second
  per molecule, nearly an order of magnitude slower than SCARF'
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: ICEBERG
  role: 'The two-stage Generate-then-Score neural model itself: gated-GNN breakage
    predictor plus Set Transformer intensity scorer executed as a pipeline'
  repo: https://github.com/coleygroup/ms-pred
- name: MAGMa
  role: Bootstraps construction of fragmentation-tree ground-truth labels used to
    train the Generate module's breakage predictions
- name: SCARF
  role: Reimplemented baseline comparator whose average cosine similarity (~0.726
    on NIST20) is reported alongside ICEBERG in the benchmark table
  repo: https://github.com/samgoldman97/ms-pred
- name: MassFormer
  role: Baseline graph-transformer comparator evaluated with the same covariates and
    hyperparameter sweep settings via run_scripts/massformer_model/02_predict_massformer.py
  repo: https://github.com/Roestlab/massformer
- name: CFM-ID
  role: Baseline comparator run via run_scripts/cfm_id/ (not retrained) to sit alongside
    ICEBERG's cosine-similarity scores
- name: uv
  role: Environment manager used to create the ms-pred venv with a mutually exclusive
    accelerator extra (cpu/cu118/cu124) before training or prediction
- name: PyTorch
  role: Framework in which the Generate gated-GNN and Score Set Transformer modules
    are built and trained
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c04654
    title: Generating Molecular Fragmentation Graphs with Autoregressive Neural Networks
  source_content_hash: sha256:74a625c63078b8d63e78416c44cb7b79cfe78db8c751956cd7ebeede00e98068
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coleygroup/ms-pred
  tools:
  - CFM-ID
  - ICEBERG
  - MAGMa
  - MassFormer
  - PyTorch
  - Python
  - SCARF
  - uv
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# msms-spectrum-prediction-training

## Summary

Trains and applies ICEBERG, a two-stage Generate-then-Score MS/MS spectrum predictor that simulates bond breakage events with a gated GNN and scores fragments/hydrogen shifts with a Set Transformer, yielding interpretable, physically grounded predicted spectra.

## When to use

Use when a task requires simulating MS/MS spectra from a molecular structure — e.g. reproducing ICEBERG's Generate (fragment generation) and Score (intensity prediction) pipeline, training from MAGMa-annotated fragmentation trees, or benchmarking cosine similarity against baselines like SCARF, NEIMS, MassFormer, or CFM-ID on NIST20/NPLIB1.

## When NOT to use

- The goal is formula-level rather than fragment-level prediction — SCARF (also in ms-pred) predicts spectra at the level of chemical formula and may be the more appropriate comparator
- Only a small NPLIB1/CANOPUS-style dataset is needed, since main-branch support for NPLIB1 is retired and requires the legacy iceberg_1.0 branch
- Throughput matters more than interpretability: ICEBERG costs roughly 1 CPU second per molecule, nearly an order of magnitude slower than SCARF

## Inputs

- Processed spectra dataset from ms-pred (e.g. nist20 or msg_all MassSpecGym TSV), including MAGMa-annotated fragmentation-tree ground truth and molecule graphs
- NIST raw data exported as .SDF (nist20/nist23 commercial library) processed with reformat_nist_lcmsms_sdf.py
- Subformulae-assigned forms from data_scripts/forms/assign_subformulae.py
- Pretrained ICEBERG Generate (gen_ckpt) and Score (inten_ckpt) checkpoints, configured in configs/iceberg/iceberg_elucidation.yaml

## Outputs

- Predicted binned MS/MS spectra as JSON, with each peak annotated by its fragment and hydrogen-shift attribution
- Trained Generate-module checkpoint (atom-wise breakage probabilities) and Score-module checkpoint (Set Transformer intensities)
- Benchmark table (CSV) of per-model, per-dataset average cosine similarity values

## How to apply

First build the Generate module as a gated-GNN that assigns an atom-wise breakage probability to each bond, trained with binary cross-entropy against MAGMa ground-truth fragmentation labels from the processed ms-pred dataset (nist20). Save that checkpoint, then run Generate over training molecules to enumerate the top-100 predicted fragments; build the Score module as a Set Transformer over fragment embeddings that predicts one intensity per fragment-hydrogen-shift pair, trained to maximize a vectorized cosine similarity against the observed spectrum. Execute Generate followed by Score on held-out test molecules and serialize predictions as binned spectra JSON with fragment and hydrogen-shift attribution per peak. Benchmark accuracy by computing average cosine similarity at 0.1-Da binning against ground truth using analysis/spec_pred_eval.py, comparing to baselines such as SCARF (~0.726 on NIST20) and MassFormer/FixedVocab (~0.568 on NPLIB1); ICEBERG should reach roughly 0.627 on NPLIB1 and 0.727 on NIST20. For reproduction, create the environment with `uv sync --extra cu124` (or cu118/cpu), export NIST raw data to .SDF, and pretrain weights are available for NIST'23 or MassSpecGym-trained variants via the maintainer or DropBox links.

## Related tools

- **ICEBERG** (The two-stage Generate-then-Score neural model itself: gated-GNN breakage predictor plus Set Transformer intensity scorer executed as a pipeline) — https://github.com/coleygroup/ms-pred
- **MAGMa** (Bootstraps construction of fragmentation-tree ground-truth labels used to train the Generate module's breakage predictions)
- **SCARF** (Reimplemented baseline comparator whose average cosine similarity (~0.726 on NIST20) is reported alongside ICEBERG in the benchmark table) — https://github.com/samgoldman97/ms-pred
- **MassFormer** (Baseline graph-transformer comparator evaluated with the same covariates and hyperparameter sweep settings via run_scripts/massformer_model/02_predict_massformer.py) — https://github.com/Roestlab/massformer
- **CFM-ID** (Baseline comparator run via run_scripts/cfm_id/ (not retrained) to sit alongside ICEBERG's cosine-similarity scores)
- **uv** (Environment manager used to create the ms-pred venv with a mutually exclusive accelerator extra (cpu/cu118/cu124) before training or prediction)
- **PyTorch** (Framework in which the Generate gated-GNN and Score Set Transformer modules are built and trained)

## Examples

```
python run_scripts/iceberg/nist23/01_train_generate.py && python run_scripts/iceberg/nist23/02_train_score.py
```

## Evaluation signals

- ICEBERG's average cosine similarity on the NIST20 test split should be approximately 0.727, matching the reported value and validating the trained checkpoints
- On NPLIB1 (legacy iceberg_1.0 branch), average cosine similarity should be approximately 0.627, a ~10% improvement over MassFormer and FixedVocab (both ~0.568)
- The Generate module's binary cross-entropy loss against MAGMa ground-truth breakage labels should converge before the Score module is trained
- Serialized predicted spectra JSON must contain, for each peak, a fragment and hydrogen-shift attribution, enabling interpretability checks
- The generated fragment set should contain at most the top-100 most likely fragments per input molecule at inference time

## Limitations

- There are no guarantees that the generated fragment explanations are the correct physical explanations, because of equivalent-mass atom and bond rearrangements and the cosine-similarity training objective
- Higher computational cost of roughly 1 CPU second per molecule, nearly an order of magnitude more than other neural approaches like SCARF, limits throughput
- NPLIB1 dataset support is retired in the main branch and requires checking out the legacy iceberg_1.0 branch
- nist20/nist23 is a commercial dataset requiring a purchased license; pretrained weights on NIST'20/23 require emailing the maintainer with proof of license
- Changes between v1.0.0 and release 1.1.0 (e.g. ms_pred.dag_pred becoming ms_pred.iceberg) were not recorded in the changelog, and 1.1.0 is unreleased

## Evidence

- [results] Reported cosine similarity benchmarks: [evidence span withheld — anchor status: not_found]
