---
name: molecular-conformer-ensemble-processing
description: Use when predicting CCS values or preferred conformations for a metabolite
  from its SMILES structure, especially when experimental TWIMS CCS is available and
  the molecule is conformationally flexible.
when_to_use_negative:
- The input is not a molecular structure (e.g., already a feature table or spectra
  list) rather than a SMILES structure needing conformer generation
- The molecule is small and rigid such that the global-minimum conformer alone suffices
  and ensemble Boltzmann averaging adds no benefit
- Elements outside the CHNO scope of the ANI-1x/ANI-1ccx potentials are present during
  the ASE-ANI filtering step (original ANI potentials cover CHNO only)
edam_operation: http://edamontology.org/operation_0249
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3332
tools:
- name: RDKit
  role: Conformation generation from SMILES-derived protonated/deprotonated structures
  repo: https://www.rdkit.org
- name: ASE-ANI
  role: Conformation filtering using ANI-1x/ANI-1ccx neural network potentials
  repo: https://github.com/isayev/ASE_ANI
- name: QUICK
  role: Quantum calculations producing conformer geometries and atomic charges
  repo: https://github.com/merzlab/QUICK
- name: HPCCS
  role: Room-temperature N2 trajectory CCS calculation for each conformer
  repo: https://github.com/cepid-cces/hpccs
- name: Snakemake
  role: Workflow manager orchestrating the ensemble pipeline via scheduler.sh
  repo: https://github.com/DasSusanta/snakemake_ccs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/jasms.1c00315
    title: <i>In Silico</i> Collision Cross Section Calculations to Aid Metabolite
      Annotation
  source_content_hash: sha256:0c341278fdb60b7486fb784d00e6fd2ade0d905dd159fb1987e9da57cf6adaca
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ASE-ANI
  - HPCCS
  - QUICK
  - RDKit
  - Snakemake
  edam_operation: http://edamontology.org/operation_0249
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# molecular-conformer-ensemble-processing

## Summary

Generates and filters a molecular conformer ensemble, then computes per-conformer CCS values and a Boltzmann-weighted average to predict the preferred gas-phase conformation for ion mobility-mass spectrometry comparison.

## When to use

Use when predicting CCS values or preferred conformations for a metabolite from its SMILES structure, especially when experimental TWIMS CCS is available and the molecule is conformationally flexible.

## When NOT to use

- The input is not a molecular structure (e.g., already a feature table or spectra list) rather than a SMILES structure needing conformer generation
- The molecule is small and rigid such that the global-minimum conformer alone suffices and ensemble Boltzmann averaging adds no benefit
- Elements outside the CHNO scope of the ANI-1x/ANI-1ccx potentials are present during the ASE-ANI filtering step (original ANI potentials cover CHNO only)

## Inputs

- Molecular structure as SMILES string
- Conformer ensemble geometries with DFT atomic charges (e.g., B3LYP/6-311++G(d,p) output)
- paths.json with software paths, cluster.yaml with walltime/memory/node settings, arguments.json with job options

## Outputs

- Per-conformer CCS values from HPCCS trajectory calculations
- Boltzmann-average CCS written to ccs.txt in the ensemble (or ensemble_fast) folder
- Ranking of conformers identifying which conformer's CCS best matches the TWIMS experimental value

## How to apply

Determine the gas-phase protonation state (typically [M+H]+ or [M-H]-), then generate conformations with RDKit and filter them with ASE-ANI using ANI-1x/ANI-1ccx neural network potentials, followed by unsupervised clustering (AutoGraph) to retain structurally distinct conformations. Run quantum calculations with QUICK to obtain geometries and B3LYP/6-311++G(d,p) atomic charges for each conformer. Build HPCCS inputs from each conformer's geometry and charges and run the room-temperature N2 trajectory method to obtain per-conformer CCS values. Compute the Boltzmann-weighted average of the per-conformer CCS values over the ensemble, since averaging can outperform the global-minimum conformer alone (e.g., L-tryptophan model 3: Boltzmann-average 159.69 Å² vs experimental 154.22 Å²). Automate the whole pipeline with the Snakemake workflow, which writes per-conformer and Boltzmann-average results to ccs.txt in the ensemble folder.

## Related tools

- **RDKit** (Conformation generation from SMILES-derived protonated/deprotonated structures) — https://www.rdkit.org
- **ASE-ANI** (Conformation filtering using ANI-1x/ANI-1ccx neural network potentials) — https://github.com/isayev/ASE_ANI
- **QUICK** (Quantum calculations producing conformer geometries and atomic charges) — https://github.com/merzlab/QUICK
- **HPCCS** (Room-temperature N2 trajectory CCS calculation for each conformer) — https://github.com/cepid-cces/hpccs
- **Snakemake** (Workflow manager orchestrating the ensemble pipeline via scheduler.sh) — https://github.com/DasSusanta/snakemake_ccs

## Examples

```
conda activate snakemake; bash scheduler.sh
```

## Evaluation signals

- Computed CCS values should agree with experimental TWIMS values within ±3%
- The L-tryptophan model 3 benchmark should reproduce a Boltzmann-average CCS of 159.69 Å² (from conformers 4 and 9: 166.44 and 156.03 Å²) against the experimental 154.22 Å²
- Conformer ranking should identify conformer 3 as the one whose CCS best matches the TWIMS measurement for L-tryptophan
- ccs.txt is produced in the ensemble (or ensemble_fast) folder only after successful completion of all Snakemake jobs
- HPCCS config.in must contain the correct number of conformations matching the ensemble size, with tab-separated parameters on a single line

## Limitations

- ASE-ANI is deprecated and no longer supported; the ANI-1x/ANI-1ccx potentials cover only CHNO elements, and binaries target Python 3.6 with CUDA 9.2 on Ubuntu/NVIDIA GPU systems
- The workflow's stated agreement with experiment is ±3%, so CCS differences smaller than this threshold cannot resolve conformer assignments
- Detailed per-step descriptions and the workflow diagram (Figure 1) are deferred to the Supporting Information, and no software version numbers are stated for RDKit, ASE-ANI, Gaussian 16, or HPCCS

## Evidence

- [other] For L-tryptophan model 3, Boltzmann averaging the conformer ensemble gives a CCS value of 159.69 Å², computed from conformations 4 and 9 (166.44 and 156.03 Å²): [evidence span withheld — anchor status: not_found]
- [methods] clustering filter: [evidence span withheld — anchor status: not_found]
- [readme] hpccs config: [evidence span withheld — anchor status: external_doc]
