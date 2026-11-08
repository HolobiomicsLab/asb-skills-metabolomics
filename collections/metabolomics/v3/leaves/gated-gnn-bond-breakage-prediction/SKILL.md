---
name: gated-gnn-bond-breakage-prediction
description: Use when you need to reconstruct or retrain the Generate stage of ICEBERG
  — i.e., predict which bonds in a molecular graph break under CID and enumerate candidate
  fragment subformulae from MAGMa ground-truth fragmentation trees.
when_to_use_negative:
- You need spectra predicted at the chemical-formula level only — SCARF (subformula
  classification) is the faster alternative, and ICEBERG costs ~1 CPU second per molecule,
  nearly an order of magnitude more than SCARF.
- You want a single-stage simulator without explicit fragment attributions — GLACIER
  in the same repo is a faster, more accurate single-stage alternative.
- You lack MAGMa-annotated fragmentation trees or the license to generate them, since
  the Generate module is trained by bootstrapping MAGMa labels.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: ICEBERG
  role: Hybrid two-part model whose Generate module is the gated-GNN bond-breakage
    predictor trained here
  repo: https://github.com/coleygroup/ms-pred
- name: MAGMa
  role: Bootstrapped to construct fragmentation trees serving as ground-truth breakage
    labels for the BCE training loss
- name: PyTorch
  role: Framework for implementing and training the gated-GNN Generate module
- name: uv
  role: Environment manager to install the ms-pred package (uv sync --extra cu124)
  repo: https://docs.astral.sh/uv/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c04654
    title: Generating Molecular Fragmentation Graphs with Autoregressive Neural Networks
  source_content_hash: sha256:23bdcfea9e67284e17951e57510b2162a5f80b93bb68a667057d664b4cbadf55
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coleygroup/ms-pred
  tools:
  - ICEBERG
  - MAGMa
  - PyTorch
  - Python
  - uv
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# gated-gnn-bond-breakage-prediction

## Summary

Trains ICEBERG's Generate module, a gated graph neural network that assigns an atom-wise breakage probability to each molecular bond, using binary cross-entropy against MAGMa-annotated fragmentation-tree labels to enumerate the top ~100 most likely fragments of a molecule for downstream intensity scoring.

## When to use

Use when you need to reconstruct or retrain the Generate stage of ICEBERG — i.e., predict which bonds in a molecular graph break under CID and enumerate candidate fragment subformulae from MAGMa ground-truth fragmentation trees.

## When NOT to use

- You need spectra predicted at the chemical-formula level only — SCARF (subformula classification) is the faster alternative, and ICEBERG costs ~1 CPU second per molecule, nearly an order of magnitude more than SCARF.
- You want a single-stage simulator without explicit fragment attributions — GLACIER in the same repo is a faster, more accurate single-stage alternative.
- You lack MAGMa-annotated fragmentation trees or the license to generate them, since the Generate module is trained by bootstrapping MAGMa labels.

## Inputs

- MAGMa-annotated fragmentation trees (ground-truth breakage labels) for training molecules
- Molecular graphs / SMILES from the processed ms-pred spec dataset (e.g., NIST20/NIST23 or MassSpecGym 1.5 TSV-derived splits)
- Subformulae assignment files produced by data_scripts/forms/assign_subformulae.py

## Outputs

- Trained Generate module checkpoint (gen_ckpt) assigning per-bond breakage probabilities
- Enumerated top-100 candidate fragments (with hydrogen shifts) per molecule, consumed by the ICEBERG Score module
- Predicted fragment-attributed spectra serialized as JSON

## How to apply

Load the MAGMa-annotated fragmentation-tree training data and molecule graphs from the processed ms-pred dataset (e.g., NIST20/NIST23 subformulae produced via assign_subformulae.py and run_magma.sh). Build the Generate module as a gated-GNN that outputs a breakage probability per bond, posed as sequential prediction over atoms, and train it with a binary cross-entropy loss against the MAGMa ground-truth fragmentation labels. Save the trained Generate checkpoint, then run it over training molecules to enumerate the top 100 most likely fragments, which are passed to the Score module (a Set Transformer trained on vectorized cosine similarity). At test time the Generate module is executed before Score to produce fragment attributions for the predicted spectra; configuration uses the ms-pred repo's iceberg run scripts with a gen_ckpt checkpoint path.

## Related tools

- **ICEBERG** (Hybrid two-part model whose Generate module is the gated-GNN bond-breakage predictor trained here) — https://github.com/coleygroup/ms-pred
- **MAGMa** (Bootstrapped to construct fragmentation trees serving as ground-truth breakage labels for the BCE training loss)
- **PyTorch** (Framework for implementing and training the gated-GNN Generate module)
- **uv** (Environment manager to install the ms-pred package (uv sync --extra cu124)) — https://docs.astral.sh/uv/

## Examples

```
git clone https://github.com/coleygroup/ms-pred.git && cd ms-pred && uv sync --extra cu124 && source .venv/bin/activate
```

## Evaluation signals

- Breakage probabilities trained with binary cross-entropy should correlate with MAGMa ground-truth fragmentation labels on a held-out validation split.
- At inference, the Generate module should yield the top 100 most likely fragments whose masses match the input molecule's subformulae (valid hydrogen-shift accounting).
- Downstream, Generate+Score spectra should improve cosine similarity against observed NIST20/NIST23 spectra relative to baselines (NEIMS, MassFormer, 3DMolMS, GrAFF-MS, SCARF).
- Retrieval evaluation (bootstrap_metrics.py) should show the fragment-attributed spectra improving metabolite identification from candidate databases.
- Every predicted peak in the serialized JSON should carry a valid fragment and hydrogen-shift attribution.

## Limitations

- Generated fragment explanations are not guaranteed to be the correct physical explanations because of equivalent-mass atom and bond rearrangements and the cosine-similarity training objective of the Score stage.
- Higher computational cost of roughly 1 CPU second per molecule, nearly an order of magnitude more than neural approaches like SCARF, limits throughput.
- The Generate module is only as good as its MAGMa bootstrapped labels; NPLIB1 support has been retired from the main branch of ms-pred.

## Evidence

- [other] test time, the Generate module produces the top 100 most likely fragments: [evidence span withheld — anchor status: not_found]
