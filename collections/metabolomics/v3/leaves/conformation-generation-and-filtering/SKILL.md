---
name: conformation-generation-and-filtering
description: Use when predicting CCS values from SMILES structures for protonated/deprotonated/neutral
  molecular states of conformationally flexible metabolites, where a structurally
  distinct conformer ensemble must be prepared before QM optimization and CCS calculation.
when_to_use_negative:
- Input molecules contain elements outside the CHNO coverage of the ANI-1x/ANI-1ccx
  potentials — the README states these potentials 'provide predictions for the CHNO
  elements' only.
- The task already has an experimentally measured CCS or a machine-learning CCS predictor
  and does not require a quantum-chemistry conformer ensemble; the article positions
  this workflow as a QM-based alternative for metabolites lacking experimental values.
- Only a single rigid molecular structure is needed and conformational flexibility
  is not relevant — the workflow's value is in handling 'conformationally flexible
  metabolites with complex molecular structures'.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3047
- http://edamontology.org/topic_0091
tools:
- name: RDKit
  role: 'Conformation generation toolkit: takes ionization-state-resolved SMILES and
    produces the initial conformer ensemble to be filtered'
  repo: https://www.rdkit.org
- name: ASE-ANI
  role: Machine-learning QM (ANI-1x/ANI-1ccx neural network potentials, ASE interface)
    used to filter and energetically screen the RDKit-generated conformations
  repo: https://github.com/isayev/ASE_ANI
- name: AutoGraph
  role: In-house unsupervised clustering code applied after ASE-ANI filtering to obtain
    structurally distinct conformations
- name: Dimorphite-DL
  role: Upstream ionization-state determination that defines which molecular states
    need conformer generation
  repo: https://durrantlab.pitt.edu/dimorphite-dl
- name: Snakemake
  role: Workflow manager orchestrating the conformer generation and filtering steps
    within the snakemake_ccs pipeline
  repo: https://github.com/DasSusanta/snakemake_ccs
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1021/jasms.1c00315
    title: <i>In Silico</i> Collision Cross Section Calculations to Aid Metabolite
      Annotation
  source_content_hash: sha256:93f8ef5e1f69652492403cc829a9e700f452e37b1ca4765800ea98e8b1dc0da1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ASE-ANI
  - AutoGraph
  - Dimorphite-DL
  - Gaussian 16
  - QUICK
  - RDKit
  - Snakemake
  - hpccs
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3047
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# conformation-generation-and-filtering

## Summary

Generates conformations for each ionization state of a metabolite with RDKit and then filters/energy-ranks them with the ASE-ANI neural network potential (ANI-1x/ANI-1ccx for CHNO molecules), before downstream QM optimization and Boltzmann-averaged CCS prediction in the snakemake_ccs workflow.

## When to use

Use when predicting CCS values from SMILES structures for protonated/deprotonated/neutral molecular states of conformationally flexible metabolites, where a structurally distinct conformer ensemble must be prepared before QM optimization and CCS calculation.

## When NOT to use

- Input molecules contain elements outside the CHNO coverage of the ANI-1x/ANI-1ccx potentials — the README states these potentials 'provide predictions for the CHNO elements' only.
- The task already has an experimentally measured CCS or a machine-learning CCS predictor and does not require a quantum-chemistry conformer ensemble; the article positions this workflow as a QM-based alternative for metabolites lacking experimental values.
- Only a single rigid molecular structure is needed and conformational flexibility is not relevant — the workflow's value is in handling 'conformationally flexible metabolites with complex molecular structures'.

## Inputs

- SMILES structures of metabolites (per ionization state from Dimorphite-DL)
- RDKit-generated conformer geometries for each molecular state

## Outputs

- ASE-ANI-filtered conformer ensemble per molecular state
- Structurally distinct conformers obtained via unsupervised clustering (AutoGraph)
- Clustered conformer set ready for B3LYP/6-31+G(d,p) optimization and HPCCS CCS computation

## How to apply

First run Dimorphite-DL ionization-state determination to enumerate all molecular states (e.g. for carnosine: one neutral, three monoprotonated ions, one doubly protonated ion), then feed each state's SMILES into RDKit to generate conformations. Next apply the ASE-ANI machine-learning QM model — the ANI-1x and ANI-1ccx neural network potentials covering CHNO elements — to filter and energetically screen the RDKit ensemble, discarding high-energy or physically implausible conformers. Follow the filtering with an unsupervised clustering step using AutoGraph (in-house clustering code) to retain structurally distinct conformations for each cluster. This pre-QM filtering is essential because the full workflow is an eight-step process that combines force fields, machine learning QM, and standard QM methods, and it makes CCS computation tractable for large numbers of conformationally flexible metabolites with complex molecular structures. The filtered, clustered conformer ensemble is then handed to QUICK for quantum calculations and HPCCS for trajectory-method CCS computation, with Boltzmann weighting used to obtain the final average CCS in ccs.txt.

## Related tools

- **RDKit** (Conformation generation toolkit: takes ionization-state-resolved SMILES and produces the initial conformer ensemble to be filtered) — https://www.rdkit.org
- **ASE-ANI** (Machine-learning QM (ANI-1x/ANI-1ccx neural network potentials, ASE interface) used to filter and energetically screen the RDKit-generated conformations) — https://github.com/isayev/ASE_ANI
- **AutoGraph** (In-house unsupervised clustering code applied after ASE-ANI filtering to obtain structurally distinct conformations)
- **Dimorphite-DL** (Upstream ionization-state determination that defines which molecular states need conformer generation) — https://durrantlab.pitt.edu/dimorphite-dl
- **Snakemake** (Workflow manager orchestrating the conformer generation and filtering steps within the snakemake_ccs pipeline) — https://github.com/DasSusanta/snakemake_ccs

## Examples

```
conda activate snakemake && bash scheduler.sh   # runs the snakemake_ccs workflow (RDKit → ASE-ANI → AutoGraph → QUICK → HPCCS) producing ccs.txt
```

## Evaluation signals

- Boltzmann-averaged CCS values from the filtered ensemble must agree within ±3% with experimental literature values, TWIMS values determined in the study, and consensus averages.
- For carnosine, model 4 (doubly protonated state) must yield a computed CCS of ~150.21 Å² (error 0.11% vs the consensus [M + H]+ value of 150.1 Å²), demonstrating the filtering pipeline preserved the correct dominant conformer (model 4 contributing 99% from conformer 3).
- After ASE-ANI filtering, the unsupervised AutoGraph clustering step must produce structurally distinct conformations — verify that multiple clusters are represented in the ensemble passed to QUICK/HPCCS.
- ASE-ANI must run under its supported environment: Python 3.6, NVIDIA GPU with compute capability ≥5.0, CUDA 9.2 — the README's quick test (examples/ani_quicktest.py) should reproduce Initial Energy ≈ -2078.502822821320 eV and Final Energy ≈ -2078.504266011399 eV.
- The workflow must complete end-to-end and produce ccs.txt in the ensemble (or ensemble_fast) folder containing Boltzmann average CCS values.

## Limitations

- ASE-ANI is deprecated and no longer supported; the README directs users to the TorchANI implementation instead.
- The ASE-ANI prototype works only under Ubuntu variants of Linux with an NVIDIA GPU, Python 3.6, and CUDA 9.2.
- Current ANI-1x and ANI-1ccx potentials only provide predictions for CHNO elements, limiting applicability to metabolites containing other elements.
- Detailed workflow step descriptions are deferred to the Supporting Information, and no software version numbers are stated for RDKit or ASE-ANI beyond the Gaussian 16 package name.
- No concrete repository URL is provided in the article text itself for the described workflow scripts (the Snakemake workflow repository URL is only available via the README).

## Evidence

- [intro] RDKit for conformation generation, ASE-ANI for conformation filtering: [evidence span withheld — anchor status: not_found]
- [methods] followed by an unsupervised clustering step using in-house unsupervised clustering code (viz. AutoGraph) to obtain structurally distinct conformations: [evidence span withheld — anchor status: not_found]
- [readme] ASE-ANI README: deprecated, use TorchANI; potentials cover CHNO elements only: [evidence span withheld — anchor status: not_found]
