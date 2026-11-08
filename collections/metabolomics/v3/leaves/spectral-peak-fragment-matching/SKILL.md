---
name: spectral-peak-fragment-matching
description: Use when constructing explanatory fragmentation DAGs from molecule-spectrum
  pairs (e.g., processed NIST labels.tsv/spec_files.hdf5) and you must map each observed
  spectral peak to a candidate fragment formula before model training or attribution
  analysis.
when_to_use_negative:
- The goal is fast spectrum prediction from formula alone without fragment-level attribution
  (use SCARF-style subformula classification instead).
- Spectra lack the mass accuracy needed for 20 ppm matching, making peak-to-fragment
  assignment unreliable.
- Support for the retired NPLIB1 (CANOPUS) dataset is required — that legacy workflow
  lives on the iceberg_1.0 branch, not the main pipeline.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MAGMa
  role: Substructure-annotation algorithm whose enumeration (up to breakage depth
    3) and heuristic scoring supply candidate fragments and the tie-breaking score
    for peak assignment
- name: ICEBERG
  role: 'Downstream consumer: the peak-to-fragment assignments bootstrap the training
    DAGs on which the ICEBERG Generate and Score modules are trained'
  repo: https://github.com/samgoldman97/ms-pred
- name: Python
  role: Implements DAG construction, Weisfeiler-Lehman hashing, 20 ppm matching, greedy
    pruning, and export to dag_annotations.json in the ms-pred repository
  repo: https://github.com/coleygroup/ms-pred
provenance:
  source_task_ids:
  - task_008
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c04654
    title: Generating Molecular Fragmentation Graphs with Autoregressive Neural Networks
  source_content_hash: sha256:ce7783fc0e68bc4bd461708bddc914b24efb766f7d47b2a0114684fc90afeb40
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/samgoldman97/ms-pred
  tools:
  - ICEBERG
  - MAGMa
  - Python
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# spectral-peak-fragment-matching

## Summary

Assigns observed MS/MS peaks to fragment nodes in a MAGMa-style fragmentation DAG using a 20 ppm mass tolerance as the primary filter and the minimal MAGMa heuristic score as a secondary filter, producing per-peak fragment attributions with hydrogen rearrangement annotations used to bootstrap ICEBERG training data.

## When to use

Use when constructing explanatory fragmentation DAGs from molecule-spectrum pairs (e.g., processed NIST labels.tsv/spec_files.hdf5) and you must map each observed spectral peak to a candidate fragment formula before model training or attribution analysis.

## When NOT to use

- The goal is fast spectrum prediction from formula alone without fragment-level attribution (use SCARF-style subformula classification instead).
- Spectra lack the mass accuracy needed for 20 ppm matching, making peak-to-fragment assignment unreliable.
- Support for the retired NPLIB1 (CANOPUS) dataset is required — that legacy workflow lives on the iceberg_1.0 branch, not the main pipeline.

## Inputs

- Processed NIST dataset labels.tsv and spec_files.hdf5 (molecule-spectrum pairs, e.g., NIST20/NIST23)
- Molecular structures (SMILES/SDF) for MAGMa substructure enumeration
- MAGMa substructure annotation outputs (run via data_scripts/dag/run_magma.sh)

## Outputs

- Per-molecule fragmentation DAG (nodes, edges, hydrogen shifts, assigned peaks) as JSON/GraphML, collected in dag_annotations.json
- Per-peak fragment attributions with hydrogen-rearrangement shift fields

## How to apply

Enumerate combinatorial atom-removal fragments for each molecule following the MAGMa substructure-annotation algorithm up to a maximum breakage depth of 3, deduplicating fragments with Weisfeiler-Lehman graph hashes while recording hydrogen rearrangement shifts. For each observed peak, apply a 20 parts-per-million mass difference tolerance as the primary filter, then break ties among mass-matching candidates using the minimal MAGMa heuristic score as the secondary filter. Retain only fragments justified by a parent breakage event via greedy parent-cover pruning, then add neutral-loss nodes connecting fragment nodes to their observed neutral losses. Export each molecule's DAG (nodes, edges, hydrogen shifts, assigned peaks) so that downstream ICEBERG Generate/Score training and per-peak attribution analyses (e.g., -1H shifts corresponding to neutral H2O loss) can consume the annotations.

## Related tools

- **MAGMa** (Substructure-annotation algorithm whose enumeration (up to breakage depth 3) and heuristic scoring supply candidate fragments and the tie-breaking score for peak assignment)
- **ICEBERG** (Downstream consumer: the peak-to-fragment assignments bootstrap the training DAGs on which the ICEBERG Generate and Score modules are trained) — https://github.com/samgoldman97/ms-pred
- **Python** (Implements DAG construction, Weisfeiler-Lehman hashing, 20 ppm matching, greedy pruning, and export to dag_annotations.json in the ms-pred repository) — https://github.com/coleygroup/ms-pred

## Examples

```
. data_scripts/dag/run_magma.sh
```

## Evaluation signals

- dag_annotations.json exists, parses, and every per-molecule DAG contains fragment nodes with hydrogen-shift annotations.
- All assigned peaks match their fragment masses within the 20 ppm tolerance.
- Every retained fragment is justified by a parent breakage event (greedy parent-cover pruning invariant).
- Breakage depth of enumerated fragments does not exceed 3, and duplicate fragments are merged via Weisfeiler-Lehman graph hash.
- Expert review confirms chemical plausibility of retained breakage events and of -1H shifts corresponding to neutral H2O loss.

## Limitations

- There are no guarantees that generated fragment explanations are the correct physical explanations due to equivalent-mass atom and bond rearrangements and the cosine-similarity training objective.
- Peak assignment is capped at breakage depth 3 in MAGMa enumeration, so deeper ring breakages may go unattributed.
- Heteroatom bonds (C-O, C-N) break more reliably than C-C bonds in the resulting attributions; C-C and complex ring breakage assignments warrant extra scrutiny.

## Evidence

- [other] Fragments in the constructed fragmentation graph are selected to explain each spectral peak using mass differences under 20 parts-per-million as the primary filter and the minimal MAGMa heuristic score as a secondary filter.: [evidence span withheld — anchor status: not_found]
