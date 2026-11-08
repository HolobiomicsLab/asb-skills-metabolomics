---
name: set-transformer-fragment-intensity-scoring
description: Use when you have a candidate set of molecular fragments from a fragmentation-tree
  model (e.g. the top 100 from ICEBERG Generate) and need intensity predictions to
  assemble a full predicted MS/MS spectrum with fragment-level attribution.
when_to_use_negative:
- You need a formula-level spectrum without fragment attribution — SCARF is a faster
  alternative at ~1/10 the compute cost.
- You only need breakage events / fragmentation-tree topology without intensities
  — the Generate module alone suffices.
- You cannot provide a molecular graph (e.g. you have only a formula or subformula)
  — the Score module operates over fragment embeddings derived from a molecule structure.
edam_operation: http://edamontology.org/operation_0394
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ICEBERG (ms-pred)
  role: The two-stage Generate-then-Score pipeline; the Score module is the Set Transformer
    described here, run after the gated-GNN Generate module.
  repo: https://github.com/coleygroup/ms-pred
- name: uv / Python / PyTorch
  role: Environment and framework for building, training, and running the Score module;
    create the environment with `uv sync`.
- name: MAGMa
  role: Bootstrapped to construct fragmentation-tree ground-truth labels that supervise
    both Generate training and the fragment-hydrogen-shift intensity targets.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c04654
    title: Generating Molecular Fragmentation Graphs with Autoregressive Neural Networks
  source_content_hash: sha256:b2c6b707fd4e0d86efaf9626214a0b98e07b4bb07df5c25e136c16882fcd9eac
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coleygroup/ms-pred
  tools:
  - ICEBERG (ms-pred)
  - MAGMa
  - PyTorch
  - Python
  - uv / Python / PyTorch
  edam_operation: http://edamontology.org/operation_0394
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# set-transformer-fragment-intensity-scoring

## Summary

The ICEBERG Score module is a Set Transformer that predicts an intensity for each of the top ~100 fragments (and their hydrogen shifts) generated for a molecule, trained to maximize a vectorized cosine similarity against the observed MS/MS spectrum.

## When to use

Use when you have a candidate set of molecular fragments from a fragmentation-tree model (e.g. the top 100 from ICEBERG Generate) and need intensity predictions to assemble a full predicted MS/MS spectrum with fragment-level attribution.

## When NOT to use

- You need a formula-level spectrum without fragment attribution — SCARF is a faster alternative at ~1/10 the compute cost.
- You only need breakage events / fragmentation-tree topology without intensities — the Generate module alone suffices.
- You cannot provide a molecular graph (e.g. you have only a formula or subformula) — the Score module operates over fragment embeddings derived from a molecule structure.

## Inputs

- Input molecule graph (from ms-pred processed dataset, e.g. nist20/nist23 .SDF-derived spectra or MassSpecGym msg_* datasets)
- Top-N candidate fragments and hydrogen shifts output by the ICEBERG Generate module checkpoint
- Trained Generate and Score module checkpoints (.ckpt) from the ms-pred repository

## Outputs

- Predicted MS/MS spectrum as JSON with peaks annotated by fragment and hydrogen-shift attribution
- Intensity value per candidate fragment-hydrogen-shift pair

## How to apply

Run the trained Generate module (a gated-GNN with atom-wise bond-breakage probabilities) on the input molecule graph to enumerate the top 100 most likely fragments, then feed these fragment embeddings into the Score module. Build the Score module as a Set Transformer with binned attention that predicts one intensity per fragment-hydrogen-shift pair, capturing hydrogen rearrangements and isotopic peaks. Train it against the MAGMa-annotated ground-truth fragmentation labels using a vectorized cosine-similarity loss so the full predicted spectrum best matches the observed spectrum. Execute Generate then Score on held-out test molecules and serialize the predicted spectra as JSON, annotating each peak by its fragment and hydrogen-shift attribution so fragment-attributed spectra can be used for retrieval or elucidation.

## Related tools

- **ICEBERG (ms-pred)** (The two-stage Generate-then-Score pipeline; the Score module is the Set Transformer described here, run after the gated-GNN Generate module.) — https://github.com/coleygroup/ms-pred
- **uv / Python / PyTorch** (Environment and framework for building, training, and running the Score module; create the environment with `uv sync`.)
- **MAGMa** (Bootstrapped to construct fragmentation-tree ground-truth labels that supervise both Generate training and the fragment-hydrogen-shift intensity targets.)

## Examples

```
uv sync --extra cu124 && source .venv/bin/activate
```

## Evaluation signals

- Predicted spectrum should match the experimental spectrum when scored with cosine similarity; ICEBERG establishes state-of-the-art performance on NIST20 and NPLIB1 spectrum prediction and retrieval.
- Each predicted peak in the output JSON should carry a fragment + hydrogen-shift attribution; unattributed peaks indicate a pipeline bug.
- Fragment enumeration should produce on the order of the top 100 most likely fragments per molecule (as configured by the Generate threshold sweep).
- Peak count and m/z values should be physically consistent with the input molecule formula (no peaks outside possible subformula masses).
- Runtime should be roughly 1 CPU second per molecule — significantly slower than SCARF is expected, but fast enough for batch inference over a test set.

## Limitations

- There are no guarantees that generated fragment explanations are the correct physical explanations, due to equivalent-mass atom and bond rearrangements and because the model is trained with a cosine-similarity objective.
- Higher computational cost (~1 CPU second per molecule, ~10x SCARF) limits throughput.
- NPLIB1 dataset support has been retired in the main ms-pred branch (still available on the iceberg_1.0 branch).

## Evidence

- [results] two-stage Generate-then-Score pipeline: [evidence span withheld — anchor status: not_found]
- [discussion] Set Transformer with binned attention: [evidence span withheld — anchor status: not_found]
- [results] JSON serialization of predicted spectra: [evidence span withheld — anchor status: not_found]
