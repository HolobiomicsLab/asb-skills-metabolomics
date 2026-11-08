---
name: ccs-percent-error-benchmarking
description: Use when a metabolite has SMILES input plus an experimental/consensus
  CCS reference and you need per-state computed CCS values and percent errors (e.g.,
  carnosine's five states vs 150.1 Å² [M + H]+).
when_to_use_negative:
- The metabolite contains elements beyond CHNO — the ANI-1x/ANI-1ccx potentials used
  for conformation filtering only predict for CHNO elements
- No experimental or consensus CCS reference exists for the ion state of interest,
  so a percent-error benchmark is undefined
- The goal is acquiring or calibrating raw TWIMS IM-MS data rather than comparing
  computed CCS values to published/measured references
edam_operation: http://edamontology.org/operation_0242
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: Dimorphite-DL
  role: Determines ionization states, generating the per-state models (one neutral,
    three monoprotonated, one doubly protonated for carnosine) that each get a computed
    CCS
  repo: https://durrantlab.pitt.edu/dimorphite-dl
- name: RDKit
  role: Generates conformations for each ionization state prior to filtering
  repo: https://www.rdkit.org
- name: ASE-ANI
  role: Machine-learning QM conformation filtering before clustering and DFT refinement
  repo: https://github.com/isayev/ASE_ANI
- name: QUICK
  role: Quantum calculations in the workflow's QM refinement step
  repo: https://github.com/merzlab/QUICK
- name: HPCCS
  role: Performs the room-temperature N2 trajectory CCS calculation over the Boltzmann-weighted
    conformer ensemble for each state
  repo: https://github.com/cepid-cces/hpccs
- name: Gaussian 16
  role: B3LYP/6-31+G(d,p) conformer optimization and B3LYP/6-311++G(d,p) charge computation
- name: Snakemake
  role: Workflow manager automating the end-to-end per-state CCS prediction and emitting
    ccs.txt
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/jasms.1c00315
    title: <i>In Silico</i> Collision Cross Section Calculations to Aid Metabolite
      Annotation
  source_content_hash: sha256:d31182fc7af1656b598305b482ca82654fe4b652eebf546bcb74eef4257b3215
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ASE-ANI
  - AutoGraph
  - Dimorphite-DL
  - Gaussian 16
  - HPCCS
  - QUICK
  - RDKit
  - Snakemake
  edam_operation: http://edamontology.org/operation_0242
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# ccs-percent-error-benchmarking

## Summary

Computes per-molecular-state collision cross sections (CCS) for an ionizable metabolite with the Snakemake CCS workflow and benchmarks each computed state against a consensus experimental reference (e.g., carnosine [M + H]+ at 150.1 Å²) to report percent errors and identify the best-matching protonation state.

## When to use

Use when a metabolite has SMILES input plus an experimental/consensus CCS reference and you need per-state computed CCS values and percent errors (e.g., carnosine's five states vs 150.1 Å² [M + H]+).

## When NOT to use

- The metabolite contains elements beyond CHNO — the ANI-1x/ANI-1ccx potentials used for conformation filtering only predict for CHNO elements
- No experimental or consensus CCS reference exists for the ion state of interest, so a percent-error benchmark is undefined
- The goal is acquiring or calibrating raw TWIMS IM-MS data rather than comparing computed CCS values to published/measured references

## Inputs

- SMILES structure of the metabolite (e.g., carnosine)
- Consensus/consensus-average experimental CCS reference value with ion type, e.g., [M + H]+ = 150.1 Å² from TWIMS experiments
- TWIMS instrumental parameters (Supporting Information Tables S1 and S2) when validating experimental reference values

## Outputs

- Computed CCS value (Å²) per molecular state / model (Boltzmann average from `ccs.txt`)
- Percent error per model relative to the experimental reference (e.g., model 4 = 150.21 Å², 0.11% error)
- Ranking of molecular states by agreement with the experimental [M + H]+ CCS

## How to apply

Generate the possible ionization states with Dimorphite-DL — for carnosine this yields one neutral, three monoprotonated, and one doubly protonated state. Generate conformers with RDKit, filter them with the ASE-ANI machine-learning QM model, and cluster with the in-house AutoGraph unsupervised clustering code to retain structurally distinct conformers. Optimize a representative conformer per cluster at B3LYP/6-31+G(d,p) and compute charges at B3LYP/6-311++G(d,p) (Gaussian 16; QUICK for quantum calculations). Run the room-temperature N2 trajectory method in HPCCS over the Boltzmann-weighted conformer ensemble to obtain the average CCS per state, which the Snakemake workflow emits as `ccs.txt` Boltzmann average values. Finally, compare each state's computed CCS against the consensus experimental value (carnosine: 150.1 Å²) and report percent error; model 4 should reproduce 150.21 Å² (0.11% error) while neutral and doubly-protonated states deviate more. Validation criterion: all computed values must agree with experimental references within ±3%.

## Related tools

- **Dimorphite-DL** (Determines ionization states, generating the per-state models (one neutral, three monoprotonated, one doubly protonated for carnosine) that each get a computed CCS) — https://durrantlab.pitt.edu/dimorphite-dl
- **RDKit** (Generates conformations for each ionization state prior to filtering) — https://www.rdkit.org
- **ASE-ANI** (Machine-learning QM conformation filtering before clustering and DFT refinement) — https://github.com/isayev/ASE_ANI
- **QUICK** (Quantum calculations in the workflow's QM refinement step) — https://github.com/merzlab/QUICK
- **HPCCS** (Performs the room-temperature N2 trajectory CCS calculation over the Boltzmann-weighted conformer ensemble for each state) — https://github.com/cepid-cces/hpccs
- **Gaussian 16** (B3LYP/6-31+G(d,p) conformer optimization and B3LYP/6-311++G(d,p) charge computation)
- **Snakemake** (Workflow manager automating the end-to-end per-state CCS prediction and emitting ccs.txt)

## Examples

```
bash scheduler.sh
```

## Evaluation signals

- Every computed CCS must fall within ±3% of the experimental reference value, consistent with the workflow's reported agreement
- For carnosine, the best state (model 4) must return ~150.21 Å², i.e., ~0.11% error against the consensus [M + H]+ value of 150.1 Å²
- Exactly five molecular states must be generated for carnosine: one neutral, three monoprotonated ions, and one doubly protonated ion
- The workflow must terminate successfully and produce a `ccs.txt` file containing Boltzmann average CCS values in the `ensemble` (or `ensemble_fast`) folder
- Neutral and doubly-protonated carnosine states must show larger deviations from the TWIMS reference than the best-matching monoprotonated model

## Limitations

- ASE-ANI repository is deprecated and no longer supported; users are pointed to the TorchANI implementation, and the binaries require Python 3.6 with CUDA 9.2 on Ubuntu with an NVIDIA GPU (compute capability ≥5.0)
- ±3% is the accuracy envelope — exact reproduction of experimental values is not expected, and multiple monoprotonated states can be near-degenerate
- Agreement depends on validated TWIMS experimental references; the authors note there are no standardized procedures for IM data collection, collation, standardization, and sharing

## Evidence

- [results] Five molecular states generated for carnosine and per-model comparison against experimental references: "for carnosine we have generated five states including one neutral, three monoprotonated ions, and one doubly protonated ion"
- [intro] ±3% agreement threshold used to judge computed-vs-experimental CCS accuracy: "The CCS values calculated using our workflow were in good agreement with available experimental values in the literature (±3%), with the values determined herein (±3%)"
- [readme] Workflow output artifact: Boltzmann average CCS values written to ccs.txt: "After successful job completion, the workflow will generate a `ccs.txt` file in the `ensemble` (or `ensemble_fast`) folder. The `ccs.txt` file contains the Boltzmann average CCS values computed using"
- [readme] Per-state model generation begins with Dimorphite-DL ionization-state determination: "Dimorphite-DL: For ionization state determination. Available at: https://durrantlab.pitt.edu/dimorphite-dl"
- [readme] ANI potential element coverage limits which metabolites can be benchmarked: [evidence span withheld — anchor status: external_doc]
