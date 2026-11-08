---
name: weisfeiler-lehman-fragment-deduplication
description: Use when enumerating MAGMa substructure-annotation fragments (up to breakage
  depth 3) from molecule–spectrum pairs and you must collapse isomorphic fragment
  graphs into canonical nodes before assigning spectral peaks and building the fragmentation
  DAG.
when_to_use_negative:
- When working at the chemical-formula level rather than fragment-graph level (e.g.,
  SCARF-style subformula classification, where formula deduplication suffices)
- When only running inference with pretrained ICEBERG/GLACIER weights — deduplication
  happens once at training-data construction, not at prediction time
- When spectra come from instruments/protocols where the 20 ppm mass tolerance is
  inappropriate (e.g., low-resolution MS/MS)
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MAGMa
  role: Enumerates combinatorial atom-removal substructure annotations (up to depth
    3) whose fragment graphs are subsequently hash-deduplicated; its heuristic score
    also filters peak assignments
  repo: https://github.com/samgoldman97/ms-pred
- name: Python
  role: Implements Weisfeiler-Lehman hashing, hydrogen-shift bookkeeping, greedy parent-cover
    pruning, and DAG export in the ms-pred data scripts
  repo: https://github.com/samgoldman97/ms-pred
- name: ICEBERG
  role: 'Downstream consumer: the canonical deduplicated fragment DAGs are the training
    targets for ICEBERG''s Generate module'
  repo: https://github.com/samgoldman97/ms-pred
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c04654
    title: Generating Molecular Fragmentation Graphs with Autoregressive Neural Networks
  source_content_hash: sha256:036287a214068756e25149c288078d12fbbc46e6988d1c9d0a894238f203815c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/samgoldman97/ms-pred
  tools:
  - ICEBERG
  - MAGMa
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# weisfeiler-lehman-fragment-deduplication

## Summary

Canonicalizes the combinatorially enumerated MAGMa-style fragment substructures used to bootstrap ICEBERG training data by deduplicating isomorphic fragment graphs with Weisfeiler-Lehman hashes, while retaining per-fragment hydrogen rearrangement shifts so peak annotation stays chemically faithful.

## When to use

Use when enumerating MAGMa substructure-annotation fragments (up to breakage depth 3) from molecule–spectrum pairs and you must collapse isomorphic fragment graphs into canonical nodes before assigning spectral peaks and building the fragmentation DAG.

## When NOT to use

- When working at the chemical-formula level rather than fragment-graph level (e.g., SCARF-style subformula classification, where formula deduplication suffices)
- When only running inference with pretrained ICEBERG/GLACIER weights — deduplication happens once at training-data construction, not at prediction time
- When spectra come from instruments/protocols where the 20 ppm mass tolerance is inappropriate (e.g., low-resolution MS/MS)

## Inputs

- Processed NIST20/NIST23 labeled spectra (labels.tsv and spec_files.hdf5)
- Molecular structures exported from NIST raw data as .SDF
- Subformulae assignments from data_scripts/forms/assign_subformulae.py
- MAGMa substructure-annotation enumeration output (up to breakage depth 3)

## Outputs

- Per-molecule fragmentation DAG with deduplicated fragment nodes, hydrogen-shift annotations, and assigned peaks (JSON/GraphML files)
- dag_annotations.json collecting all per-molecule DAGs for ICEBERG Generate training

## How to apply

For each molecule from the processed NIST dataset (labels.tsv and spec_files.hdf5), enumerate combinatorial atom-removal fragmentations following the MAGMa substructure-annotation algorithm to a maximum breakage depth of 3. As fragments are enumerated, compute a Weisfeiler-Lehman graph hash for each fragment graph and use it as the deduplication key so structurally identical substructures produced via different atom orders collapse to a single canonical node, while recording the hydrogen rearrangement shifts observed for each unique fragment. After deduplication, assign observed peaks to fragment nodes using a 20 ppm mass tolerance as the primary filter and the minimal MAGMa heuristic score as a secondary filter, then apply greedy parent-cover pruning so every retained fragment is justified by a parent breakage event. Add neutral-loss nodes connecting fragments to their observed neutral losses and export the DAG (nodes, edges, hydrogen shifts, assigned peaks) as JSON/GraphML collected in dag_annotations.json; this canonical fragment set is what ICEBERG's Generate module is trained on.

## Related tools

- **MAGMa** (Enumerates combinatorial atom-removal substructure annotations (up to depth 3) whose fragment graphs are subsequently hash-deduplicated; its heuristic score also filters peak assignments) — https://github.com/samgoldman97/ms-pred
- **Python** (Implements Weisfeiler-Lehman hashing, hydrogen-shift bookkeeping, greedy parent-cover pruning, and DAG export in the ms-pred data scripts) — https://github.com/samgoldman97/ms-pred
- **ICEBERG** (Downstream consumer: the canonical deduplicated fragment DAGs are the training targets for ICEBERG's Generate module) — https://github.com/samgoldman97/ms-pred

## Examples

```
. data_scripts/dag/run_magma.sh
```

## Evaluation signals

- dag_annotations.json exists and parses without errors
- Every per-molecule DAG contains fragment nodes annotated with hydrogen-rearrangement shifts
- All assigned peaks match fragment masses within the 20 ppm tolerance
- Every retained fragment is justified by a parent breakage event (greedy parent-cover pruning invariant)
- No two distinct nodes in a DAG share the same Weisfeiler-Lehman graph hash (deduplication invariant)

## Limitations

- Generated fragment explanations are not guaranteed to be the correct physical explanations due to equivalent-mass atom and bond rearrangements
- Bootstrapped DAG construction costs roughly 1 CPU second per molecule, nearly an order of magnitude more than neural alternatives like SCARF
- NIST20/NIST23 training data is commercial; NPLIB1 support has been retired from the ms-pred main branch (moved to the iceberg_1.0 branch)

## Evidence

- [methods] Deduplicate the enumerated fragments using Weisfeiler-Lehman graph hashes while recording the hydrogen rearrangement shifts for each fragment.: [evidence span withheld — anchor status: not_found]
- [methods] using mass differences under 20 parts-per-million as the primary filter and the minimal MAGMa heuristic score as a secondary filter: [evidence span withheld — anchor status: not_found]
