---
name: snakemake-workflow-execution
description: Use when SMILES structures of metabolites are available and in silico
  CCS values are needed across protonated/deprotonated/neutral states, requiring automated
  multi-tool orchestration on HPC.
when_to_use_negative:
- When only a single metabolite with a single fixed protonation state is needed and
  a simpler manual HPCCS/Gaussian run suffices — the Snakemake machinery targets high-throughput,
  multi-adduct, multi-model batches
- When experimentally measured TWIMS CCS values already exist for the metabolite and
  no in silico prediction is required
- When elements outside CHNO are involved — the ANI-1x/ANI-1ccx potentials used for
  conformation filtering only predict CHNO elements
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_0091
tools:
- name: Snakemake
  role: Workflow manager orchestrating the eight-step CCS prediction pipeline and
    parallelized HPC execution
- name: Dimorphite-DL
  role: Determines ionization states ([M + H]+ / [M − H]−) from SMILES input
  repo: https://durrantlab.pitt.edu/dimorphite-dl
- name: RDKit
  role: Generates the conformer ensembles for each protonation state
  repo: https://www.rdkit.org
- name: ASE-ANI
  role: Filters/minimizes conformers using ANI-1ccx neural network potentials
  repo: https://github.com/isayev/ASE_ANI
- name: QUICK
  role: Quantum calculations (DFT geometry and charges, e.g. B3LYP/6-311++G(d,p))
    on clustered conformers
  repo: https://github.com/merzlab/QUICK
- name: hpccs
  role: Computes per-conformer CCS via trajectory method; Boltzmann averages written
    to ccs.txt
  repo: https://github.com/cepid-cces/hpccs
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_006
  source_papers:
  - doi: 10.1021/jasms.1c00315
    title: <i>In Silico</i> Collision Cross Section Calculations to Aid Metabolite
      Annotation
  source_content_hash: sha256:0599bddc3896de05127d22126ba829730fc3d910b76d412acc702fe208ab7601
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ASE-ANI
  - AutoGraph
  - Dimorphite-DL
  - QUICK
  - RDKit
  - Snakemake
  - hpccs
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# snakemake-workflow-execution

## Summary

Configure and launch the eight-step snakemake_ccs Snakemake workflow that predicts Boltzmann-averaged collision cross sections (CCS) from SMILES input, chaining Dimorphite-DL, RDKit, ASE-ANI, QUICK, and HPCCS into a single automated run.

## When to use

Use when SMILES structures of metabolites are available and in silico CCS values are needed across protonated/deprotonated/neutral states, requiring automated multi-tool orchestration on HPC.

## When NOT to use

- When only a single metabolite with a single fixed protonation state is needed and a simpler manual HPCCS/Gaussian run suffices — the Snakemake machinery targets high-throughput, multi-adduct, multi-model batches
- When experimentally measured TWIMS CCS values already exist for the metabolite and no in silico prediction is required
- When elements outside CHNO are involved — the ANI-1x/ANI-1ccx potentials used for conformation filtering only predict CHNO elements

## Inputs

- SMILES structures of metabolites (workflow input)
- paths.json with software installation paths
- cluster.yaml with walltime, memory, and node specifications
- arguments.json with job/run options
- Conformer ensemble geometries and DFT atomic charges in the ensemble folder (for downstream CCS steps)

## Outputs

- ccs.txt in the ensemble (or ensemble_fast) folder containing Boltzmann average CCS values
- cluster_representatives.sdf with representative conformer coordinates and cluster assignments

## How to apply

Specify the paths to the prerequisite software (Dimorphite-DL for ionization state determination, RDKit for conformation generation, ASE-ANI for conformation filtering, QUICK for quantum calculations, hpccs for CCS calculation) in paths.json. Configure walltime, memory, and node specifications in cluster.yaml and supply run options in arguments.json, then launch with 'bash scheduler.sh' in the snakemake conda environment (installed via mamba with conda-forge and bioconda channels). The workflow determines protonation states (typically [M + H]+ or [M − H]−), generates conformers with RDKit, filters them with the ANI-1ccx ML potential via ASE-ANI, clusters to structurally distinct conformations (AutoGraph), runs QUICK quantum calculations (e.g. B3LYP/6-311++G(d,p) charges), and computes per-conformer CCS with HPCCS' room-temperature N2 trajectory method before Boltzmann-averaging. On successful completion, collect the ccs.txt file in the ensemble (or ensemble_fast) folder containing the Boltzmann average CCS values.

## Related tools

- **Snakemake** (Workflow manager orchestrating the eight-step CCS prediction pipeline and parallelized HPC execution)
- **Dimorphite-DL** (Determines ionization states ([M + H]+ / [M − H]−) from SMILES input) — https://durrantlab.pitt.edu/dimorphite-dl
- **RDKit** (Generates the conformer ensembles for each protonation state) — https://www.rdkit.org
- **ASE-ANI** (Filters/minimizes conformers using ANI-1ccx neural network potentials) — https://github.com/isayev/ASE_ANI
- **QUICK** (Quantum calculations (DFT geometry and charges, e.g. B3LYP/6-311++G(d,p)) on clustered conformers) — https://github.com/merzlab/QUICK
- **hpccs** (Computes per-conformer CCS via trajectory method; Boltzmann averages written to ccs.txt) — https://github.com/cepid-cces/hpccs

## Examples

```
mamba create -c conda-forge -c bioconda -n snakemake snakemake; conda activate snakemake; bash scheduler.sh
```

## Evaluation signals

- Computed CCS values agree within ±3% with TWIMS experimental values and literature references (e.g. L-tryptophan model 3 Boltzmann average of 159.69 Å² vs. experimental 154.22 Å²)
- ccs.txt file is generated in the ensemble (or ensemble_fast) folder after successful job completion
- AutoGraph clustering reduces the RDKit-generated ensemble (e.g. carnosine's ~990–996 generated conformers down to 12–16 unique conformations)
- Boltzmann averaging outperforms the global-minimum conformer prediction for flexible metabolites (ranked conformers identify the best-matching structure)
- paths.json, cluster.yaml, and arguments.json are configured and the workflow runs end-to-end via bash scheduler.sh without failed jobs

## Limitations

- ASE-ANI is deprecated and no longer supported; the README directs users to the TorchANI implementation, and the binaries only work on Ubuntu Linux with NVIDIA GPUs (CUDA 9.2, Python 3.6)
- HPCCS must be compiled from source (GCC 7.2 recommended for memory alignment) and its config parameters are tab-separated in a single line
- Detailed workflow step descriptions are deferred to the Supporting Information; no software version numbers are stated for RDKit, ASE_ANI, or HPCCS
- The paper's CCS agreement of ±3% assumes metabolites with 10–40 atoms; conformationally flexible metabolites with complex structures were the benchmarked scope

## Evidence

- [results] Boltzmann averaging result: [evidence span withheld — anchor status: not_found]
