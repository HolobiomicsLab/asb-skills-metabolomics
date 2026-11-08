---
name: quantum-chemistry-ccs-computation
description: Use when you need to compute CCS values from SMILES structures for metabolites
  (or other small molecules) whose experimental CCS is unavailable, or when benchmarking
  in silico CCS against TWIMS-measured values for structural annotation.
when_to_use_negative:
- Input is already an experimentally measured CCS value or an existing annotated library
  — no need to recompute in silico.
- Molecules containing elements outside the CHNO coverage of the ANI-1x/ANI-1ccx potentials
  used for filtering.
- You need a rapid single-point CCS estimate without HPC resources — the workflow
  requires Snakemake, QUICK, and parallelized HPC computation.
edam_operation: http://edamontology.org/operation_0394
edam_topics:
- http://edamontology.org/topic_3332
- http://edamontology.org/topic_0154
tools:
- name: Dimorphite-DL
  role: Determines ionization states / protonation state of the input molecule
  repo: https://durrantlab.pitt.edu/dimorphite-dl
- name: RDKit
  role: Generates conformations from SMILES
  repo: https://www.rdkit.org
- name: ASE-ANI
  role: Filters/evaluates conformations with ANI-1x/ANI-1ccx neural network potentials
    as a fast machine-learning pre-screen
  repo: https://github.com/isayev/ASE_ANI
- name: QUICK
  role: Performs higher-level quantum (DFT) calculations on filtered conformers
  repo: https://github.com/merzlab/QUICK
- name: HPCCS
  role: Computes collision cross sections from the optimized conformer geometries
  repo: https://github.com/cepid-cces/hpccs
- name: Snakemake
  role: Orchestrates the eight-step workflow, parallelized on HPC, invoked via scheduler.sh
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/jasms.1c00315
    title: <i>In Silico</i> Collision Cross Section Calculations to Aid Metabolite
      Annotation
  source_content_hash: sha256:4abdca0022a4337ab8c0ee39a6e93c3b05b5f32d487aab50831b46fc442bf7d9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ASE-ANI
  - Dimorphite-DL
  - HPCCS
  - QUICK
  - RDKit
  - Snakemake
  - hpccs
  edam_operation: http://edamontology.org/operation_0394
  edam_topics:
  - http://edamontology.org/topic_3332
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# quantum-chemistry-ccs-computation

## Summary

Quantum-chemistry-based collision cross section (CCS) computation from SMILES input, combining machine-learning potentials (ASE-ANI), DFT-level calculations (QUICK), and trajectory CCS estimation (HPCCS) to support metabolite annotation in metabolomics.

## When to use

Use when you need to compute CCS values from SMILES structures for metabolites (or other small molecules) whose experimental CCS is unavailable, or when benchmarking in silico CCS against TWIMS-measured values for structural annotation.

## When NOT to use

- Input is already an experimentally measured CCS value or an existing annotated library — no need to recompute in silico.
- Molecules containing elements outside the CHNO coverage of the ANI-1x/ANI-1ccx potentials used for filtering.
- You need a rapid single-point CCS estimate without HPC resources — the workflow requires Snakemake, QUICK, and parallelized HPC computation.

## Inputs

- SMILES structures of metabolites
- Adduct state specification (e.g. protonated/deprotonated/neutral molecular states)
- Configuration files: paths.json, cluster.yaml, arguments.json

## Outputs

- Boltzmann-averaged CCS values in ccs.txt (in the ensemble/ensemble_fast folder)
- Per-conformer CCS values and Boltzmann weights
- Structurally distinct, energetically filtered conformer geometries

## How to apply

Start from a SMILES structure and first determine the gas-phase protonation state (typically [M + H]+ or [M − H]−) with Dimorphite-DL. Generate conformations with RDKit, then filter and score them with ASE-ANI neural network potentials (ANI-1x/ANI-1ccx; CHNO predictions) as a machine-learning pre-screen. Pass surviving structurally distinct conformations (e.g. after unsupervised clustering) to QUICK for higher-level quantum calculations, and feed those geometries to HPCCS for trajectory-based CCS computation. For multiple adducts or molecular states, run each in parallel on HPC resources, then combine results into a Boltzmann-weighted average so the final CCS reflects the thermodynamically preferred conformer set. Judge success by agreement of computed CCS with literature and TWIMS-measured values (targeted within ±3%) and by ability to compute values for large, conformationally flexible metabolites with 10–40 atoms.

## Related tools

- **Dimorphite-DL** (Determines ionization states / protonation state of the input molecule) — https://durrantlab.pitt.edu/dimorphite-dl
- **RDKit** (Generates conformations from SMILES) — https://www.rdkit.org
- **ASE-ANI** (Filters/evaluates conformations with ANI-1x/ANI-1ccx neural network potentials as a fast machine-learning pre-screen) — https://github.com/isayev/ASE_ANI
- **QUICK** (Performs higher-level quantum (DFT) calculations on filtered conformers) — https://github.com/merzlab/QUICK
- **HPCCS** (Computes collision cross sections from the optimized conformer geometries) — https://github.com/cepid-cces/hpccs
- **Snakemake** (Orchestrates the eight-step workflow, parallelized on HPC, invoked via scheduler.sh)

## Examples

```
bash scheduler.sh  # after configuring paths.json, cluster.yaml, arguments.json
```

## Evaluation signals

- Computed CCS for the benchmark metabolites agrees with literature experimental values within ±3%
- Computed CCS agrees with TWIMS values measured in the same study within ±3%
- Boltzmann-average in ccs.txt is dominated by the lowest-energy conformer set (e.g. carnosine model 4: conformer 3 contributes ~99% at 150.21 Å²)
- All prerequisites (Dimorphite-DL, RDKit, ASE-ANI, QUICK, hpccs) resolvable via paths.json before scheduler.sh launch
- Workflow completes without errors and produces a non-empty ccs.txt in the ensemble/ensemble_fast output folder

## Limitations

- ASE-ANI (the original prototype interface) is deprecated in favor of TorchANI and only supports CHNO elements, Python 3.6 and CUDA 9.2 on Ubuntu Linux with an NVIDIA GPU
- Requires HPC resources and parallelized computation; not suitable for low-throughput or CPU-only environments
- hpccs is distributed under an academic license from UNICAMP, restricting commercial use
- Workflow details for each step are deferred to the paper's Supporting Information; READMEs alone do not document all eight steps
- No standardized procedures exist for IM data collection/collation, limiting direct comparison of CCS across labs

## Evidence

- [intro] Workflow tooling and orchestration: [evidence span withheld — anchor status: not_found]
- [methods] Ionization state determination: [evidence span withheld — anchor status: not_found]
- [methods] Conformer clustering: [evidence span withheld — anchor status: not_found]
- [readme] ASE-ANI deprecation / element coverage: [evidence span withheld — anchor status: external_doc]
