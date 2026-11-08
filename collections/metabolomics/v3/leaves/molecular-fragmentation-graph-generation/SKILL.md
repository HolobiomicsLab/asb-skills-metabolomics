---
name: molecular-fragmentation-graph-generation
description: Use when simulating MS/MS spectra directly from a molecular structure,
  when you need fragment-level (not formula-level) peak attribution with hydrogen-shift
  annotations, or when building fragmentation-tree training labels with MAGMa.
when_to_use_negative:
- A formula-level (not fragment-level) prediction is sufficient — SCARF predicts spectra
  at the level of chemical formula and is ~10x faster (~1 CPU second per molecule
  for ICEBERG).
- Single-stage, maximum-speed simulation is the priority — GLACIER in the same ms-pred
  repo is described as an even faster and more accurate single-stage MS/MS simulator.
- 'You need guaranteed correct physical explanations: there are no guarantees that
  generated fragment attributions are the correct physical explanations due to equivalent-mass
  atom and bond rearrangements and the cosine-similarity training objective.'
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ICEBERG (ms-pred)
  role: Implementation of the two-module Generate (gated-GNN breakage predictor) and
    Score (Set Transformer intensity scorer) pipeline with pretrained weights and
    evaluation scripts
  repo: https://github.com/coleygroup/ms-pred
- name: MAGMa
  role: Bootstrapped to construct fragmentation trees that serve as ground-truth training
    labels for the Generate module
- name: PyTorch
  role: Framework used to build and train both the GNN Generate module and the Set
    Transformer Score module
- name: uv
  role: Manages the Python environment used to run the ICEBERG training and inference
    scripts
  repo: https://docs.astral.sh/uv/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c04654
    title: Generating Molecular Fragmentation Graphs with Autoregressive Neural Networks
  source_content_hash: sha256:cfb6e472205ddb9cc30604517677d08b916725225b440f5fec55974106b24426
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coleygroup/ms-pred
  tools:
  - ICEBERG (ms-pred)
  - MAGMa
  - PyTorch
  - Python
  - uv
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# molecular-fragmentation-graph-generation

## Summary

Implements ICEBERG, a two-stage Generate-then-Score MS/MS spectrum predictor: a gated-GNN predicts atom-wise bond breakage probabilities to reconstruct a molecular fragmentation graph, then a Set Transformer assigns intensities to each generated fragment-hydrogen-shift pair. It lifts the combinatorial MAGMa fragmentation approach into a fast neural model trained on experimental MS/MS data.

## When to use

Use when simulating MS/MS spectra directly from a molecular structure, when you need fragment-level (not formula-level) peak attribution with hydrogen-shift annotations, or when building fragmentation-tree training labels with MAGMa.

## When NOT to use

- A formula-level (not fragment-level) prediction is sufficient — SCARF predicts spectra at the level of chemical formula and is ~10x faster (~1 CPU second per molecule for ICEBERG).
- Single-stage, maximum-speed simulation is the priority — GLACIER in the same ms-pred repo is described as an even faster and more accurate single-stage MS/MS simulator.
- You need guaranteed correct physical explanations: there are no guarantees that generated fragment attributions are the correct physical explanations due to equivalent-mass atom and bond rearrangements and the cosine-similarity training objective.

## Inputs

- Molecule graphs (SMILES/structures) from MS/MS libraries such as NIST20/NIST23 (.SDF), NPLIB1, CASMI22, or MassSpecGym 1.5 TSV
- MAGMa-annotated fragmentation trees serving as ground-truth breakage labels for the Generate module
- Observed MS/MS spectra (peak m/z and intensity vectors) paired with molecular structures for Score-module training
- Subformulae-assigned data files produced by data_scripts/forms/assign_subformulae.py
- Pretrained Generate (gen_ckpt) and Score (inten_ckpt) checkpoints for inference or structural elucidation

## Outputs

- Trained Generate-module checkpoint predicting per-bond breakage probabilities
- Trained Score-module (Set Transformer) checkpoint predicting fragment intensities
- Predicted MS/MS spectra serialized as JSON, with each peak annotated by its fragment and hydrogen-shift attribution
- Fragmentation graphs (top ~100 fragments per molecule) for interpretability and retrieval evaluation

## How to apply

Train the Generate module as a gated GNN that assigns a breakage probability to each bond of the molecule graph, using binary cross-entropy against MAGMa-annotated ground-truth fragmentation labels from the processed ms-pred dataset (e.g. nist20/nist23 SDF data assigned to subformulae via data_scripts/forms/assign_subformulae.py). Tree generation is posed as a sequential prediction over atoms, bootstrapped by MAGMa-constructed fragmentation trees. At inference the Generate module emits the top ~100 most likely fragments for the input molecule. The Score module then embeds these fragments in a Set Transformer (with binned attention) that predicts one intensity per fragment-hydrogen-shift pair, trained to maximize a vectorized cosine similarity against the observed spectrum. Finally, serialize the predicted spectra as JSON, with each peak annotated by its fragment and hydrogen-shift attribution, and sweep the fragment-generation threshold (run_scripts/iceberg/nist23/02_sweep_gen_thresh.py) to tune the number of generated fragments.

## Related tools

- **ICEBERG (ms-pred)** (Implementation of the two-module Generate (gated-GNN breakage predictor) and Score (Set Transformer intensity scorer) pipeline with pretrained weights and evaluation scripts) — https://github.com/coleygroup/ms-pred
- **MAGMa** (Bootstrapped to construct fragmentation trees that serve as ground-truth training labels for the Generate module)
- **PyTorch** (Framework used to build and train both the GNN Generate module and the Set Transformer Score module)
- **uv** (Manages the Python environment used to run the ICEBERG training and inference scripts) — https://docs.astral.sh/uv/

## Examples

```
python data_scripts/create_msg_simulation_dataset.py --strategy msg_all --audit-out data/spec_datasets/msg_all/audit.json
```

## Evaluation signals

- Score module training loss should reflect the vectorized cosine similarity objective against the observed spectrum, decreasing over epochs.
- Generated fragmentation trees should agree with MAGMa ground-truth labels (binary cross-entropy loss on atom-wise breakage probabilities should converge).
- Predicted spectra exported as JSON should have every peak annotated with a fragment and hydrogen-shift attribution.
- Sweep the number of generated fragments (02_sweep_gen_thresh.py) and verify spectrum-prediction cosine similarity and retrieval ranking improve toward ICEBERG's reported state-of-the-art performance on NIST20 and NPLIB1.
- End-to-end demo run (notebooks/iceberg_2025_biorxiv/iceberg_demo_pubchem_elucidation.ipynb) should complete in <2 minutes with a regular desktop GPU using gen_ckpt and inten_ckpt from the YAML config.

## Limitations

- There are no guarantees that the generated fragment explanations are the correct physical explanations, because of equivalent-mass atom and bond rearrangements and the cosine-similarity training objective.
- Higher computational cost: roughly 1 CPU second per molecule, nearly an order of magnitude more than other neural approaches like SCARF.
- Support for the NPLIB1 (CANOPUS) dataset has been retired from the main branch; it requires the legacy iceberg_1.0 branch.
- NIST20/NIST23 training data is commercial and cannot be redistributed; pretrained NIST weights require proof of license, while open MassSpecGym-trained weights are also available.

## Evidence
