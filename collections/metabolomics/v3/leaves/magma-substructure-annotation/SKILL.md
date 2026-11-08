---
name: magma-substructure-annotation
description: 'Use when bootstrapping training data for ICEBERG''s Generate module
  from labeled NIST spectra: enumerate fragment substructures for each molecule and
  assign peaks within 20 ppm mass tolerance using the minimal MAGMa heuristic score.'
when_to_use_negative:
- When you only need formula-level spectra predictions rather than fragment-level
  DAGs — SCARF predicts spectra at the level of chemical formula and does not require
  MAGMa substructure enumeration
- When spectra are unlabeled or lack candidate formulas, since peak assignment relies
  on fragment masses and the MAGMa heuristic score computed from the molecule and
  its subformulae
- When you intend to use the newer GLACIER single-stage simulator instead of ICEBERG's
  two-part Generate/Score pipeline, since GLACIER does not consume MAGMa-constructed
  fragmentation trees in the same way
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MAGMa
  role: Provides the combinatorial substructure-annotation algorithm and heuristic
    score used to enumerate fragments and assign spectral peaks during DAG construction
- name: ICEBERG
  role: 'Downstream consumer: the constructed fragmentation DAGs bootstrap training
    data for the Generate module of ICEBERG'
  repo: github.com/samgoldman97/ms-pred
- name: Python
  role: Runs the repo's data pipeline scripts (assign_subformulae.py, run_magma.sh,
    create_msg_simulation_dataset.py) that implement enumeration, deduplication, and
    DAG export
  repo: github.com/coleygroup/ms-pred
- name: uv
  role: Environment manager used to install the ms-pred dependency stack before running
    the DAG construction scripts
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c04654
    title: Generating Molecular Fragmentation Graphs with Autoregressive Neural Networks
  source_content_hash: sha256:6cca2da793c60f2942a487d806e80894ce29e15287253e372ca36ef902c80b7d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ICEBERG
  - MAGMa
  - Python
  - uv
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# magma-substructure-annotation

## Summary

Bootstraps ICEBERG training data by enumerating MAGMa-style combinatorial substructures for each molecule-spectrum pair and assigning observed spectral peaks to fragment nodes in an explanatory fragmentation DAG. Produces hydrogen-shift-annotated DAGs (JSON/GraphML in dag_annotations.json) used to train the Generate module.

## When to use

Use when bootstrapping training data for ICEBERG's Generate module from labeled NIST spectra: enumerate fragment substructures for each molecule and assign peaks within 20 ppm mass tolerance using the minimal MAGMa heuristic score.

## When NOT to use

- When you only need formula-level spectra predictions rather than fragment-level DAGs — SCARF predicts spectra at the level of chemical formula and does not require MAGMa substructure enumeration
- When spectra are unlabeled or lack candidate formulas, since peak assignment relies on fragment masses and the MAGMa heuristic score computed from the molecule and its subformulae
- When you intend to use the newer GLACIER single-stage simulator instead of ICEBERG's two-part Generate/Score pipeline, since GLACIER does not consume MAGMa-constructed fragmentation trees in the same way

## Inputs

- Processed NIST20 spectra: labels.tsv and spec_files.hdf5 from the ms-pred data pipeline
- Molecular structures exported as .SDF from the NIST raw data
- Subformula-assigned data produced by data_scripts/forms/assign_subformulae.py

## Outputs

- Per-molecule fragmentation DAGs as JSON/GraphML files (fragment nodes, edges, hydrogen shifts, assigned peaks)
- dag_annotations.json aggregating all annotated DAGs for downstream ICEBERG Generate training

## How to apply

Load molecules and labeled spectra from the processed NIST20 dataset (labels.tsv and spec_files.hdf5), then enumerate combinatorial atom-removal fragmentations following the MAGMa substructure-annotation algorithm up to a maximum breakage depth of 3. Deduplicate enumerated fragments via Weisfeiler-Lehman graph hashes while recording hydrogen rearrangement shifts for each fragment. Assign observed peaks to fragment nodes using a 20 ppm mass tolerance as the primary filter and the minimal MAGMa heuristic score as the secondary filter, so each peak is explained by a fragment whose mass matches within tolerance and that carries the lowest heuristic score. Apply greedy parent-cover pruning so every retained fragment is justified by a parent breakage event, then add neutral-loss nodes connecting fragment nodes to their observed neutral losses. The rationale, per the paper, is to 'lift combinatorial bond-breaking approaches (specifically MAGMa) into a neural network space by using them to build a training dataset, from which our model learns to make fast estimates'; export each molecule's fragmentation DAG (nodes, edges, hydrogen shifts, assigned peaks) as JSON/GraphML collected in dag_annotations.json. Run the MAGMa annotation step via the repo's data_scripts/dag/run_magma.sh after subformula assignment with data_scripts/forms/assign_subformulae.py.

## Related tools

- **MAGMa** (Provides the combinatorial substructure-annotation algorithm and heuristic score used to enumerate fragments and assign spectral peaks during DAG construction)
- **ICEBERG** (Downstream consumer: the constructed fragmentation DAGs bootstrap training data for the Generate module of ICEBERG) — github.com/samgoldman97/ms-pred
- **Python** (Runs the repo's data pipeline scripts (assign_subformulae.py, run_magma.sh, create_msg_simulation_dataset.py) that implement enumeration, deduplication, and DAG export) — github.com/coleygroup/ms-pred
- **uv** (Environment manager used to install the ms-pred dependency stack before running the DAG construction scripts)

## Examples

```
. data_scripts/dag/run_magma.sh
```

## Evaluation signals

- dag_annotations.json exists and parses as valid JSON containing one annotated DAG per molecule
- Every per-molecule DAG contains fragment nodes annotated with hydrogen rearrangement shifts
- All assigned peaks match their fragment masses within the 20 ppm tolerance
- Every retained fragment is justified by a parent breakage event (greedy parent-cover pruning invariant)
- Neutral-loss nodes are present connecting fragment nodes to their observed neutral losses

## Limitations

- Fragments are selected to explain peaks by mass and heuristic score, but there are no guarantees that they are the correct physical explanations due to equivalent-mass atom and bond rearrangements and the cosine-similarity training objective
- MAGMa bootstrapping is computationally expensive — roughly 1 CPU second per molecule, nearly an order of magnitude more than neural approaches like SCARF
- Support for the NPLIB1 dataset was retired in the main branch; legacy code lives on the iceberg_1.0 branch
- NIST20/23 is commercial and licensed; processing requires purchased raw data exported as .SDF

## Evidence

- [methods] Peak assignment uses 20 ppm tolerance plus minimal MAGMa score: [evidence span withheld — anchor status: not_found]
- [methods] DAG construction workflow with WL hashing and pruning: [evidence span withheld — anchor status: not_found]
- [intro] Rationale for bootstrapping MAGMa into neural space: [evidence span withheld — anchor status: not_found]
