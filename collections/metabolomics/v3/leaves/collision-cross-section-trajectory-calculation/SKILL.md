---
name: collision-cross-section-trajectory-calculation
description: Use when gas-phase ion geometries and their DFT atomic charges are available
  and a CCS value is needed for comparison to TWIMS measurements or metabolite annotation.
when_to_use_negative:
- Input is raw SMILES without generated conformers and DFT charges — use the full
  upstream Snakemake workflow (Dimorphite-DL, RDKit, ASE-ANI, QUICK) first instead
  of running HPCCS directly
- A rapid predicted-CCS estimate is needed for thousands of metabolites without HPC
  resources — a machine-learning CCS predictor or the POMICS web tool is more appropriate
- The molecule contains elements outside the CHNO scope validated by the ANI potentials
  used upstream in the workflow
edam_operation: http://edamontology.org/operation_2476
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3332
tools:
- name: HPCCS
  role: Performs the trajectory-method CCS calculation on conformer geometries with
    atomic charges, using the room-temperature N2 collision gas
  repo: https://github.com/cepid-cces/hpccs
- name: Snakemake
  role: Workflow manager that automates per-conformer CCS runs (bash scheduler.sh)
    with HPC cluster settings from cluster.yaml and arguments.json
- name: Gaussian 16
  role: Provides B3LYP/6-311++G(d,p) atomic charges and optimized geometries that
    serve as HPCCS inputs
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1021/jasms.1c00315
    title: <i>In Silico</i> Collision Cross Section Calculations to Aid Metabolite
      Annotation
  source_content_hash: sha256:c2282004e8bca8941103ac83f453d023186770cb962f9f5f634621369661c446
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/cepid-cces/hpccs
  tools:
  - ASE-ANI
  - AutoGraph
  - Dimorphite-DL
  - Gaussian 16
  - HPCCS
  - QUICK
  - RDKit
  - Snakemake
  edam_operation: http://edamontology.org/operation_2476
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# collision-cross-section-trajectory-calculation

## Summary

Computes collision cross section (CCS) values for molecular conformer ensembles using the trajectory method in HPCCS with atomic charges from DFT (B3LYP/6-311++G(d,p)), then Boltzmann-averages per-conformer CCS values to compare against experimental TWIMS references.

## When to use

Use when gas-phase ion geometries and their DFT atomic charges are available and a CCS value is needed for comparison to TWIMS measurements or metabolite annotation.

## When NOT to use

- Input is raw SMILES without generated conformers and DFT charges — use the full upstream Snakemake workflow (Dimorphite-DL, RDKit, ASE-ANI, QUICK) first instead of running HPCCS directly
- A rapid predicted-CCS estimate is needed for thousands of metabolites without HPC resources — a machine-learning CCS predictor or the POMICS web tool is more appropriate
- The molecule contains elements outside the CHNO scope validated by the ANI potentials used upstream in the workflow

## Inputs

- Conformer ensemble 3D geometries (e.g., from RDKit generation, ASE-ANI filtering/clustering, and DFT optimization)
- DFT atomic charges per conformer (B3LYP/6-311++G(d,p), e.g., from Gaussian 16 output)
- HPCCS configuration file (config/config.in: number of conformations, averaging cycles, velocity-integration points)
- Workflow configuration files: paths.json, cluster.yaml, arguments.json
- Experimental TWIMS CCS reference values (Tables S1/S2)

## Outputs

- Per-conformer CCS values (Å²) from the N2 trajectory method
- Boltzmann-averaged CCS value (Å²) per molecular state/model
- ccs.txt file in the ensemble (or ensemble_fast) folder with average and per-conformer results
- Percent errors versus experimental [M + H]+ / [M − H]− CCS references

## How to apply

Extract each conformer's geometry and atomic charges (e.g., B3LYP/6-311++G(d,p) Mulliken/ESP charges from a Gaussian 16 output) into HPCCS input files, verifying paths in paths.json. Run the room-temperature N2 trajectory method in HPCCS for each conformer in the ensemble (conformations, averaging cycles, and velocity-integration points are set in config/config.in, tab-separated on one line). Execute the pipeline via Snakemake (bash scheduler.sh) with walltime/memory/node settings from cluster.yaml and job options from arguments.json, targeting HPC parallel execution. Compute the Boltzmann-weighted average of per-conformer CCS values over the ensemble rather than using only the global-minimum conformer; for L-tryptophan model 3 this averaging (conformations 4 and 9: 166.44 and 156.03 Å²) yielded 159.69 Å² and a better match to the 154.22 Å² experimental TWIMS value. Compare the resulting CCS against the experimental reference (Tables S1/S2) and write results to ccs.txt in the ensemble folder; treat agreement within ±3% of experimental values as the accuracy criterion.

## Related tools

- **HPCCS** (Performs the trajectory-method CCS calculation on conformer geometries with atomic charges, using the room-temperature N2 collision gas) — https://github.com/cepid-cces/hpccs
- **Snakemake** (Workflow manager that automates per-conformer CCS runs (bash scheduler.sh) with HPC cluster settings from cluster.yaml and arguments.json)
- **Gaussian 16** (Provides B3LYP/6-311++G(d,p) atomic charges and optimized geometries that serve as HPCCS inputs)

## Examples

```
bash scheduler.sh  # runs the Snakemake pipeline (paths.json, cluster.yaml, arguments.json) through HPCCS; results in ensemble/ccs.txt
```

## Evaluation signals

- Computed CCS values agree within ±3% of experimental reference values (literature, in-house TWIMS, or consensus averages)
- Boltzmann-averaged CCS is closer to experiment than the global-minimum conformer alone (as shown for L-tryptophan model 3: 159.69 Å² vs 154.22 Å² experimental)
- ccs.txt is generated in the ensemble (or ensemble_fast) folder after job completion, containing the Boltzmann average CCS
- Carnosine model 4 benchmark reproduces 150.21 Å² with 0.11% error against the 150.1 Å² consensus experimental [M + H]+ CCS
- config/config.in parses correctly with parameters tab-separated on a single line (conformation count, ≥1 averaging cycle, ≥1 velocity-integration point)

## Limitations

- HPCCS requires compilation and works best with GCC 7.2 for memory-alignment performance; configuration parameters must be tab-split on one line in config/config.in
- Accuracy claims (±3%) are established for metabolites with 10–40 atoms; reliability outside this range is not demonstrated
- Upstream ASE-ANI is deprecated in favor of TorchANI, and ANI-1x/ANI-1ccx potentials cover only CHNO elements
- Detailed workflow steps are deferred to the Supporting Information, and some tool versions (RDKit, ASE_ANI, HPCCS) are not stated in the article

## Evidence

- [results] For L-tryptophan model 3, Boltzmann averaging the conformer ensemble gives a CCS value of 159.69 Å²: [evidence span withheld — anchor status: not_found]
- [methods] Trajectory method description: [evidence span withheld — anchor status: not_found]
- [readme] HPCCS configuration: [evidence span withheld — anchor status: external_doc]
