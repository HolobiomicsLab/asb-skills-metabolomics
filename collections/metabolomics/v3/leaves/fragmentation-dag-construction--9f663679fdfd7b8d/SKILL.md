---
name: fragmentation-dag-construction
description: 'Use when bootstrapping training data for a fragment-level spectrum predictor
  (e.g., ICEBERG Generate): you have labeled molecule–spectrum pairs (NIST20 labels.tsv
  + spec_files.hdf5) and need fragments that explain each observed peak.'
when_to_use_negative:
- You only need formula-level spectral predictions rather than fragment-level graphs
  — SCARF predicts spectra at the level of chemical formula instead
- Your input spectra are unlabeled or not paired with known molecular structures (NIST20
  is a commercial reference library; unannotated peaks cannot be assigned to fragments)
- You want inference only — use pretrained ICEBERG 2.1 or GLACIER weights rather than
  re-constructing MAGMa DAGs, which costs roughly 1 CPU second per molecule (~10x
  SCARF)
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MAGMa
  role: Combinatorial substructure-annotation algorithm used to enumerate fragments
    up to breakage depth 3 and to provide the heuristic score used as the secondary
    peak-assignment filter
- name: Python
  role: 'Implements the ms-pred data pipeline: subformula assignment, WL-hash deduplication,
    20 ppm peak assignment, greedy parent-cover pruning, and DAG export scripts'
  repo: https://github.com/coleygroup/ms-pred
- name: ms-data-parser
  role: Processes raw NIST .SDF exports (reformat_nist_lcmsms_sdf.py) into the processed
    library used downstream
  repo: https://github.com/rogerwwww/ms-data-parser
- name: MassFormer
  role: Its repo documents exporting NIST'20 raw library data, a prerequisite for
    building the labeled dataset
  repo: https://github.com/Roestlab/massformer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c04654
    title: Generating Molecular Fragmentation Graphs with Autoregressive Neural Networks
  source_content_hash: sha256:6dee9d4e8a441c1661a152b002fb7a3adc0d5d5bb6e394115cd77f92d7c37e3d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coleygroup/ms-pred
  tools:
  - MAGMa
  - MassFormer
  - Python
  - ms-data-parser
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# fragmentation-dag-construction

## Summary

Constructs per-molecule fragmentation DAGs by bootstrapping MAGMa substructure enumeration (max breakage depth 3) on labeled NIST20 molecule–spectrum pairs, assigning observed spectral peaks to fragment nodes with a 20 ppm mass tolerance plus minimal MAGMa heuristic score, then pruning and exporting DAGs as training data for ICEBERG's Generate module.

## When to use

Use when bootstrapping training data for a fragment-level spectrum predictor (e.g., ICEBERG Generate): you have labeled molecule–spectrum pairs (NIST20 labels.tsv + spec_files.hdf5) and need fragments that explain each observed peak.

## When NOT to use

- You only need formula-level spectral predictions rather than fragment-level graphs — SCARF predicts spectra at the level of chemical formula instead
- Your input spectra are unlabeled or not paired with known molecular structures (NIST20 is a commercial reference library; unannotated peaks cannot be assigned to fragments)
- You want inference only — use pretrained ICEBERG 2.1 or GLACIER weights rather than re-constructing MAGMa DAGs, which costs roughly 1 CPU second per molecule (~10x SCARF)

## Inputs

- Processed NIST20 (or NIST'23) molecule–spectrum pairs: labels.tsv and spec_files.hdf5
- Raw NIST library data exported as .SDF (processed via reformat_nist_lcmsms_sdf.py)
- Subformulae assignments produced by assign_subformulae.py

## Outputs

- dag_annotations.json collecting per-molecule fragmentation DAGs (fragment nodes, edges, hydrogen-shift annotations, assigned peaks)
- Per-molecule DAG files in JSON/GraphML format
- Fragmentation trees suitable as training targets for ICEBERG's Generate module

## How to apply

First preprocess the data by assigning subformulae with `data_scripts/forms/assign_subformulae.py`, then run MAGMa substructure annotation via `. data_scripts/dag/run_magma.sh`. For each molecule, enumerate combinatorial atom-removal fragmentations following the MAGMa substructure-annotation algorithm up to a maximum breakage depth of 3, and deduplicate enumerated fragments with Weisfeiler-Lehman graph hashes while recording hydrogen rearrangement shifts per fragment. Assign observed spectral peaks to fragment nodes using mass differences under 20 ppm as the primary filter and the minimal MAGMa heuristic score as the secondary filter. Apply greedy parent-cover pruning so every retained fragment is justified by a parent breakage event, then add neutral-loss nodes connecting fragments to their observed neutral losses. Export each molecule's DAG (nodes, edges, hydrogen shifts, assigned peaks) as JSON/GraphML collected in dag_annotations.json — these trees are the supervision ICEBERG's Generate module trains on.

## Related tools

- **MAGMa** (Combinatorial substructure-annotation algorithm used to enumerate fragments up to breakage depth 3 and to provide the heuristic score used as the secondary peak-assignment filter)
- **Python** (Implements the ms-pred data pipeline: subformula assignment, WL-hash deduplication, 20 ppm peak assignment, greedy parent-cover pruning, and DAG export scripts) — https://github.com/coleygroup/ms-pred
- **ms-data-parser** (Processes raw NIST .SDF exports (reformat_nist_lcmsms_sdf.py) into the processed library used downstream) — https://github.com/rogerwwww/ms-data-parser
- **MassFormer** (Its repo documents exporting NIST'20 raw library data, a prerequisite for building the labeled dataset) — https://github.com/Roestlab/massformer

## Examples

```
. data_scripts/dag/run_magma.sh
```

## Evaluation signals

- dag_annotations.json exists and parses successfully
- Every per-molecule DAG contains fragment nodes annotated with hydrogen rearrangement shifts
- All assigned peaks match fragment masses within the 20 ppm tolerance
- Every retained fragment is justified by a parent breakage event (greedy parent-cover pruning invariant)
- Retained breakage events pass expert review for chemical plausibility

## Limitations

- There are no guarantees that generated fragment explanations are the correct physical explanations due to equivalent-mass atom and bond rearrangements and the cosine-similarity training objective
- Higher computational cost (~1 CPU second per molecule, nearly an order of magnitude more than neural approaches like SCARF) limits throughput
- NIST20/23 is a commercial dataset requiring purchase; NPLIB1 support is retired from the ms-pred main branch (use the iceberg_1.0 branch)

## Evidence

- [discussion] Bootstrapping MAGMa to construct fragmentation trees for training: "bootstrapping MAGMa to construct fragmentation trees on which our model is trained"
- [intro] MAGMa used to build the training dataset for fast neural estimates: [evidence span withheld — anchor status: not_found]
- [methods] Subformula assignment is a required preprocessing step before DAG annotation: "Data should then be assigned to subformulae files using `data_scripts/forms/assign_subformulae.py`, which will preprocess the data."
