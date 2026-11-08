---
name: dft-geometry-optimization-and-charge-calculation
description: Use when a conformational ensemble has been filtered and clustered (e.g.
  with ASE-ANI and AutoGraph) and a structurally distinct representative conformer
  must be optimized and charged before running CCS calculations.
when_to_use_negative:
- When only a quick, approximate CCS estimate is needed and ML-predicted charges/energies
  (e.g. from the ANI machine-learning model in ASE-ANI) are sufficient without DFT
  refinement
- When the molecule falls outside the CHNO element coverage of the upstream ANI potential
  and no DFT-level treatment is planned for the whole ensemble
- When the input is already a fully quantum-optimized structure with computed charges
  — re-optimizing is redundant
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3332
tools:
- name: Gaussian 16
  role: Performs the B3LYP/6-31+G(d,p) geometry optimization and the B3LYP/6-311++G(d,p)
    atomic charge calculation on cluster-representative conformers
- name: QUICK
  role: Alternative quantum calculation engine invoked by the Snakemake workflow for
    the DFT step
  repo: https://github.com/merzlab/QUICK
- name: ASE-ANI
  role: Upstream tool that filters and clusters conformations so that only one representative
    per cluster needs DFT optimization
  repo: https://github.com/isayev/ASE_ANI
- name: HPCCS
  role: Downstream consumer of the DFT-optimized, charged conformers for trajectory
    CCS calculation
  repo: https://github.com/cepid-cces/hpccs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/jasms.1c00315
    title: <i>In Silico</i> Collision Cross Section Calculations to Aid Metabolite
      Annotation
  source_content_hash: sha256:de188f0f6aefcc1adb514926a8f3fe97d472cb839f34508a6566c7865bda4792
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/merzlab/QUICK
  tools:
  - ASE-ANI
  - AutoGraph
  - Dimorphite-DL
  - Gaussian 16
  - HPCCS
  - QUICK
  - RDKit
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# dft-geometry-optimization-and-charge-calculation

## Summary

Refines a representative conformer per cluster via DFT geometry optimization and computes atomic charges, providing the quantum-refined input structures that the subsequent CCS trajectory calculation requires.

## When to use

Use when a conformational ensemble has been filtered and clustered (e.g. with ASE-ANI and AutoGraph) and a structurally distinct representative conformer must be optimized and charged before running CCS calculations.

## When NOT to use

- When only a quick, approximate CCS estimate is needed and ML-predicted charges/energies (e.g. from the ANI machine-learning model in ASE-ANI) are sufficient without DFT refinement
- When the molecule falls outside the CHNO element coverage of the upstream ANI potential and no DFT-level treatment is planned for the whole ensemble
- When the input is already a fully quantum-optimized structure with computed charges — re-optimizing is redundant

## Inputs

- Representative conformer per cluster from the AutoGraph clustering step (3D coordinates)
- Molecular ionization state determined by Dimorphite-DL (e.g. [M + H]+ or [M − H]− protonation state)

## Outputs

- DFT-optimized 3D geometry per cluster representative (B3LYP/6-31+G(d,p))
- Atomic partial charges per optimized structure (B3LYP/6-311++G(d,p), Gaussian 16)
- Charged, optimized conformer ensemble ready for Boltzmann weighting and HPCCS trajectory calculation

## How to apply

After clustering the conformer ensemble into structurally distinct groups, select one representative conformer per cluster and optimize its geometry at the B3LYP/6-31+G(d,p) level. Then compute atomic charges for the optimized structure at the higher-level B3LYP/6-311++G(d,p) basis using the Gaussian 16 software package. These quantum-refined geometries and charges are the Boltzmann-weighted inputs that HPCCS consumes for the room-temperature N2 trajectory CCS calculation. The workflow can alternatively be executed via QUICK as the quantum calculation engine within the Snakemake-managed pipeline. Optimization level (B3LYP/6-31+G(d,p) for geometry) and charge level (B3LYP/6-311++G(d,p)) are deliberately different: the larger basis resolves electrostatics more accurately, which the trajectory CCS calculation is sensitive to.

## Related tools

- **Gaussian 16** (Performs the B3LYP/6-31+G(d,p) geometry optimization and the B3LYP/6-311++G(d,p) atomic charge calculation on cluster-representative conformers)
- **QUICK** (Alternative quantum calculation engine invoked by the Snakemake workflow for the DFT step) — https://github.com/merzlab/QUICK
- **ASE-ANI** (Upstream tool that filters and clusters conformations so that only one representative per cluster needs DFT optimization) — https://github.com/isayev/ASE_ANI
- **HPCCS** (Downstream consumer of the DFT-optimized, charged conformers for trajectory CCS calculation) — https://github.com/cepid-cces/hpccs

## Examples

```
bash scheduler.sh  # runs the full Snakemake CCS workflow; on completion produces ccs.txt with Boltzmann average CCS values in the ensemble (or ensemble_fast) folder
```

## Evaluation signals

- Geometry optimization converges without error at B3LYP/6-31+G(d,p) for each cluster representative
- Atomic charges are computed at the distinct higher basis B3LYP/6-311++G(d,p), not the optimization basis
- Optimized structures retain the ionization state determined by Dimorphite-DL (correct protonation of [M + H]+ / [M − H]−)
- Downstream HPCCS Boltzmann-average CCS output (ccs.txt) falls within ±3% of the consensus experimental reference, consistent with the workflow's stated agreement
- One optimized structure per cluster — no cluster is left unrepresented in the charged ensemble

## Limitations

- Computed CCS agreement is bounded at ±3% relative to experimental values; deviations beyond this indicate upstream errors (conformer selection, protonation state) rather than the DFT step itself
- DFT optimization/charging is performed only on one representative per cluster, so alternative low-energy conformers within a cluster are not independently refined
- The ASE-ANI upstream potential covers only CHNO elements; molecules containing other elements require different QM handling
- QUICK installation URL was truncated in the article; users should consult the merzlab/QUICK GitHub repository directly

## Evidence

- [other] Optimize a representative conformer per cluster at B3LYP/6-31+G(d,p) and compute atomic charges at B3LYP/6-311++G(d,p) in Gaussian 16: [evidence span withheld — anchor status: not_found]
- [readme] QUICK: For quantum calculations. Available at: https://github.com/merzlab/QUICK: [evidence span withheld — anchor status: external_doc]
