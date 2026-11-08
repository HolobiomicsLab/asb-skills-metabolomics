---
name: neutral-loss-node-generation
description: Use when constructing fragmentation DAGs from molecule–spectrum pairs
  where fragment nodes already carry 20 ppm peak assignments and observed neutral
  losses must be represented as explicit graph nodes linking fragment pairs.
when_to_use_negative:
- When working with formula-level predictors such as SCARF, which predict spectra
  at the subformula level rather than on molecular fragment graphs
- Before peaks have been assigned to fragment nodes within the 20 ppm tolerance —
  neutral-loss nodes depend on those assignments
- When only predicted spectra are available and no observed neutral losses can be
  grounded in experimental peak differences
tools:
- name: MAGMa
  role: Upstream substructure-annotation enumeration that produces the fragment nodes
    to which neutral-loss nodes are attached
- name: Python
  role: Implements the DAG construction pipeline (deduplication, pruning, peak assignment,
    neutral-loss node insertion, export)
  repo: https://github.com/coleygroup/ms-pred
- name: ICEBERG
  role: Downstream model trained on the neutral-loss-augmented fragmentation DAGs
    bootstrapped from MAGMa
  repo: https://github.com/coleygroup/ms-pred
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c04654
    title: Generating Molecular Fragmentation Graphs with Autoregressive Neural Networks
  source_content_hash: sha256:41be5140823eb220e6546e2ab15ac3293f595b0ccd126be51aed5958fe9ffb05
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
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# neutral-loss-node-generation

## Summary

Augments per-molecule fragmentation DAGs—built by MAGMa-style substructure enumeration for ICEBERG training data—with neutral-loss nodes that connect fragment nodes to their observed neutral losses, completing the explanatory graph structure exported for model training.

## When to use

Use when constructing fragmentation DAGs from molecule–spectrum pairs where fragment nodes already carry 20 ppm peak assignments and observed neutral losses must be represented as explicit graph nodes linking fragment pairs.

## When NOT to use

- When working with formula-level predictors such as SCARF, which predict spectra at the subformula level rather than on molecular fragment graphs
- Before peaks have been assigned to fragment nodes within the 20 ppm tolerance — neutral-loss nodes depend on those assignments
- When only predicted spectra are available and no observed neutral losses can be grounded in experimental peak differences

## Inputs

- Molecule–spectrum pairs from the processed NIST dataset (labels.tsv and spec_files.hdf5)
- Enumerated fragment nodes with hydrogen rearrangement shifts and peaks assigned within 20 ppm (post greedy parent-cover pruning)
- Observed neutral losses derived from assigned peak differences

## Outputs

- Per-molecule fragmentation DAGs containing fragment nodes plus neutral-loss nodes, edges, hydrogen shifts, and assigned peaks
- JSON/GraphML exports collected in dag_annotations.json

## How to apply

After enumerating MAGMa-style combinatorial atom-removal fragments up to depth 3, deduplicating them via Weisfeiler-Lehman graph hashes with hydrogen rearrangement shifts, applying greedy parent-cover pruning, and assigning observed peaks to fragment nodes using a 20 ppm mass tolerance plus the minimal MAGMa heuristic score, add neutral-loss nodes that connect each fragment node to its observed neutral loss. This makes the DAG capture both direct breakage events and secondary losses, mirroring the physically-grounded fragmentation representation ICEBERG's Generate module is trained on. Export the completed DAG (fragment nodes, neutral-loss nodes, edges, hydrogen shifts, assigned peaks) per molecule as JSON/GraphML entries collected in dag_annotations.json, running the DAG pipeline via data_scripts/dag/run_magma.sh in the ms-pred environment.

## Related tools

- **MAGMa** (Upstream substructure-annotation enumeration that produces the fragment nodes to which neutral-loss nodes are attached)
- **Python** (Implements the DAG construction pipeline (deduplication, pruning, peak assignment, neutral-loss node insertion, export)) — https://github.com/coleygroup/ms-pred
- **ICEBERG** (Downstream model trained on the neutral-loss-augmented fragmentation DAGs bootstrapped from MAGMa) — https://github.com/coleygroup/ms-pred

## Examples

```
source .venv/bin/activate && . data_scripts/dag/run_magma.sh
```

## Evaluation signals

- dag_annotations.json exists and parses, and every per-molecule DAG contains fragment nodes with hydrogen-shift annotations
- All assigned peaks match fragment masses within the 20 ppm tolerance
- Each neutral-loss node connects a fragment node to its observed neutral loss, with node masses consistent with the fragment-to-fragment mass difference
- Every retained fragment (and its loss edges) is justified by a parent breakage event after greedy parent-cover pruning
- Expert review confirms chemical plausibility of retained breakage events

## Limitations

- There are no guarantees that generated fragment explanations are the correct physical explanations due to equivalent-mass atom and bond rearrangements and the cosine-similarity training objective
- DAG construction via MAGMa bootstrapping costs roughly 1 CPU second per molecule, nearly an order of magnitude more than neural approaches like SCARF
- NPLIB1 dataset support is retired in the ms-pred main branch; legacy workflows require the iceberg_1.0 branch

## Evidence

- [other] Add neutral-loss nodes connecting fragment nodes to their observed neutral losses.: [evidence span withheld — anchor status: not_found]
