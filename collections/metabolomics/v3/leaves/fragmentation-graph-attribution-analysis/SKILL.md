---
name: fragmentation-graph-attribution-analysis
description: Use when you have ICEBERG's per-fragment DAG outputs for held-out test
  molecules and need to verify that predicted bond breakages (e.g., C-O/C-N vs C-C)
  and hydrogen-shift peak attributions are chemically plausible.
when_to_use_negative:
- Input spectra come from a formula-level model such as SCARF, which lacks per-fragment
  DAGs and bond-breakage annotations needed for attribution analysis.
- The goal is purely black-box spectrum intensity prediction or retrieval benchmarking
  with no interpretability/attribution requirement.
- Working with the NPLIB1 (CANOPUS) dataset on the ms-pred main branch — its support
  is retired there and requires the legacy iceberg_1.0 branch.
tools:
- name: ICEBERG
  role: Generates fragmentation DAGs with per-breakage bond types and per-peak hydrogen-rearrangement
    annotations used for the attribution audit
  repo: https://github.com/coleygroup/ms-pred
- name: MAGMa
  role: Provides the exhaustive combinatorial fragmentation trees used to bootstrap
    ICEBERG's training data, grounding the bond-breakage semantics being audited
- name: Python
  role: Runs ICEBERG inference (predict_smis.py) and the downstream aggregation of
    bond-type frequencies and per-peak annotation extraction
  repo: https://github.com/coleygroup/ms-pred
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c04654
    title: Generating Molecular Fragmentation Graphs with Autoregressive Neural Networks
  source_content_hash: sha256:ed8a1aa2e5a750bd2ea163e5e9bf1ec7ac0a06f50b16890625dd12f41b432b62
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coleygroup/ms-pred
  tools:
  - ICEBERG
  - MAGMa
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# fragmentation-graph-attribution-analysis

## Summary

Audits the mechanistic interpretability of ICEBERG's predicted fragmentation DAGs on held-out NIST20/NIST23 test molecules by aggregating broken-bond-type frequencies and inspecting per-peak hydrogen-rearrangement annotations, testing whether the model's peak explanations align with chemical intuition.

## When to use

Use when you have ICEBERG's per-fragment DAG outputs for held-out test molecules and need to verify that predicted bond breakages (e.g., C-O/C-N vs C-C) and hydrogen-shift peak attributions are chemically plausible.

## When NOT to use

- Input spectra come from a formula-level model such as SCARF, which lacks per-fragment DAGs and bond-breakage annotations needed for attribution analysis.
- The goal is purely black-box spectrum intensity prediction or retrieval benchmarking with no interpretability/attribution requirement.
- Working with the NPLIB1 (CANOPUS) dataset on the ms-pred main branch — its support is retired there and requires the legacy iceberg_1.0 branch.

## Inputs

- Pretrained ICEBERG generator checkpoint (gen_ckpt) and intensity checkpoint (inten_ckpt)
- Held-out test molecule labels (SMILES structures) from the processed NIST20/NIST23 dataset split
- ICEBERG inference output: fragmentation DAGs with per-breakage bond types and per-peak hydrogen-rearrangement annotations

## Outputs

- bond_breakage_stats.csv — summary table of broken-bond-type frequencies across all held-out molecules
- Annotated records for the most intense peaks with fragment identity and hydrogen-shift fields
- Flagged list of cases where a -1H shift corresponds to a neutral H2O loss

## How to apply

Load the pretrained ICEBERG generator (gen_ckpt) and intensity (inten_ckpt) checkpoints together with the held-out test-split molecule labels from the processed dataset (e.g., NIST20/NIST23 test split). Run ICEBERG inference via src/ms_pred/iceberg/predict_smis.py to produce fragmentation DAGs annotated with per-breakage bond types and per-peak hydrogen-rearrangement fields (~1 CPU second per molecule). Aggregate the frequencies of broken bond types across all held-out molecules into a summary table (bond_breakage_stats.csv) and check whether weaker heteroatom bonds (C-O, C-N) break more reliably than C-C bonds and complex ring breakages. Extract annotated records for the most intense peaks, retaining per-peak fragment identity and hydrogen-shift fields, and flag cases where a -1H shift corresponds to a neutral H2O loss. Finally, combine machine-checkable field-presence validation with expert review of the -1H/H2O-loss correspondence, since ICEBERG's cosine-similarity training objective gives no guarantee that peak explanations are the correct physical explanations.

## Related tools

- **ICEBERG** (Generates fragmentation DAGs with per-breakage bond types and per-peak hydrogen-rearrangement annotations used for the attribution audit) — https://github.com/coleygroup/ms-pred
- **MAGMa** (Provides the exhaustive combinatorial fragmentation trees used to bootstrap ICEBERG's training data, grounding the bond-breakage semantics being audited)
- **Python** (Runs ICEBERG inference (predict_smis.py) and the downstream aggregation of bond-type frequencies and per-peak annotation extraction) — https://github.com/coleygroup/ms-pred

## Examples

```
git clone https://github.com/coleygroup/ms-pred.git && cd ms-pred && uv sync --extra cu124 && source .venv/bin/activate && python src/ms_pred/iceberg/predict_smis.py
```

## Evaluation signals

- bond_breakage_stats.csv exists and contains bond-type frequency columns
- Every annotated intense-peak record carries fragment identity and hydrogen-shift fields (machine-checkable via field_present checks)
- Aggregated frequencies show weaker heteroatom bonds (C-O, C-N) broken more reliably than C-C bonds and complex ring breakages
- Runtime sanity: inference proceeds at roughly 1 CPU second per molecule
- Expert review judges the correspondence between -1H shifts and neutral H2O-loss peaks as chemically plausible

## Limitations

- There are no guarantees that the generated fragment explanations are the correct physical explanations, due to equivalent-mass atom and bond rearrangements and the cosine-similarity training objective.
- Inference costs roughly 1 CPU second per molecule, nearly an order of magnitude more than neural alternatives like SCARF, limiting throughput for very large molecule sets.
- Pretrained weights on NIST'20/23/26 require proof of a NIST license; otherwise only open-source MassSpecGym-trained weights are available.
- Module renames between releases (e.g., ms_pred.dag_pred becoming ms_pred.iceberg) were not recorded in the changelog, which can break older analysis scripts.

## Evidence
