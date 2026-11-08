---
name: unsupervised-conformer-clustering
description: Use when a RDKit conformer ensemble (e.g. ~990-996 conformers per protonation
  state) has been geometry-relaxed with ASE-ANI and must be condensed to structurally
  distinct representatives before QM-level optimization and CCS calculation.
when_to_use_negative:
- Input is an unrelaxed RDKit conformer ensemble that has not yet been minimized with
  the ASE-ANI model — cluster on optimized geometries only
- The goal is exhaustive CCS sampling of every conformer rather than reduction to
  structurally distinct representatives before QM optimization
- 'Elements beyond CHNO are present: ANI-1x/ANI-1ccx potentials only provide predictions
  for CHNO elements, so other systems need a different potential or clustering route'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3047
tools:
- name: AutoGraph
  role: In-house unsupervised clustering code that groups structurally similar, ASE-ANI-relaxed
    conformers and selects one representative per cluster with no human supervision
- name: ASE-ANI
  role: Geometry-relaxes (minimizes) the RDKit-generated conformers with ANI-1ccx
    machine-learning potentials before clustering, so clusters reflect relaxed structures
  repo: https://github.com/isayev/ASE_ANI
- name: RDKit
  role: Generates the initial conformer ensemble per ionization state that AutoGraph
    clustering then condenses
  repo: https://www.rdkit.org
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1021/jasms.1c00315
    title: <i>In Silico</i> Collision Cross Section Calculations to Aid Metabolite
      Annotation
  source_content_hash: sha256:137560068394bf1be0cd50d968535610cc6f7eb9591a6d8f46f6a722e806c98b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/isayev/ASE_ANI
  tools:
  - ASE-ANI
  - AutoGraph
  - Dimorphite-DL
  - Gaussian 16
  - QUICK
  - RDKit
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3047
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# unsupervised-conformer-clustering

## Summary

Reduces large RDKit-generated conformer ensembles to a small set of structurally distinct conformations by running in-house AutoGraph unsupervised clustering on conformers previously relaxed with the ASE-ANI (ANI-1ccx) machine-learning QM model, one representative per cluster feeding downstream DFT optimization and CCS calculation.

## When to use

Use when a RDKit conformer ensemble (e.g. ~990-996 conformers per protonation state) has been geometry-relaxed with ASE-ANI and must be condensed to structurally distinct representatives before QM-level optimization and CCS calculation.

## When NOT to use

- Input is an unrelaxed RDKit conformer ensemble that has not yet been minimized with the ASE-ANI model — cluster on optimized geometries only
- The goal is exhaustive CCS sampling of every conformer rather than reduction to structurally distinct representatives before QM optimization
- Elements beyond CHNO are present: ANI-1x/ANI-1ccx potentials only provide predictions for CHNO elements, so other systems need a different potential or clustering route

## Inputs

- RDKit-generated conformer ensemble for a metabolite ionization state (e.g. [M + H]+ or [M − H]−), per-state counts of ~990-996 conformers
- Geometry-relaxed (minimized) conformers from the ASE-ANI QM-based machine-learning model optimization stage

## Outputs

- cluster_representatives.sdf containing one representative conformer per cluster with coordinates and cluster assignments
- Structurally distinct conformation set (e.g. 12-16 conformers per state for carnosine) that feeds the Gaussian 16 DFT step

## How to apply

After RDKit generates conformations for a metabolite's [M + H]+ or [M − H]− state, minimize each conformer with the ASE-ANI QM-based machine-learning model (ANI-1ccx potentials covering CHNO elements), since clustering raw unrelaxed geometries would be noise-dominated. Run the in-house AutoGraph unsupervised clustering code on the optimized geometries with no human supervision or manual cluster selection; for carnosine's five states this condensed ensembles of 990, 994, 994, 994, and 996 conformers down to 16, 13, 15, 12, and 14 unique conformations. Extract one representative conformation per cluster and write coordinates plus cluster assignments to cluster_representatives.sdf for the downstream Gaussian 16 B3LYP/6-31+G(d,p) optimization and HPCCS trajectory CCS step. Validate that the cluster count is far smaller than the input ensemble and that each representative is structurally distinct, confirming conformational-complexity reduction without manual intervention.

## Related tools

- **AutoGraph** (In-house unsupervised clustering code that groups structurally similar, ASE-ANI-relaxed conformers and selects one representative per cluster with no human supervision)
- **ASE-ANI** (Geometry-relaxes (minimizes) the RDKit-generated conformers with ANI-1ccx machine-learning potentials before clustering, so clusters reflect relaxed structures) — https://github.com/isayev/ASE_ANI
- **RDKit** (Generates the initial conformer ensemble per ionization state that AutoGraph clustering then condenses) — https://www.rdkit.org

## Examples

```
snakemake --cores 1 cluster_representatives.sdf
```

## Evaluation signals

- Number of output clusters is far smaller than the input conformer ensemble (e.g. 990→16 for carnosine's neutral state)
- cluster_representatives.sdf is produced containing coordinates plus cluster assignments for every cluster
- Each representative is a structurally distinct conformation, not a near-duplicate
- No manual cluster selection was involved — clustering is fully unsupervised
- Downstream CCS values computed from these representatives agree within ±3% of experimental TWIMS reference values (e.g. carnosine model 4: 150.21 Å², 0.11% error vs 150.1 Å²)

## Limitations

- ASE-ANI (ANI-1x/ANI-1ccx) is deprecated in favor of TorchANI and its prototype binaries target Python 3.6, CUDA 9.2, and NVIDIA GPUs on Ubuntu Linux
- ANI-1x/ANI-1ccx potentials provide predictions only for CHNO elements, limiting the metabolite space this clustering-precedes-DFT chain can cover
- Clustering quality depends on prior ANI-1ccx minimization; the workflow details for each step are described in the Supporting Information rather than the main text

## Evidence

- [results] AutoGraph reduces 990-996 generated conformers per state to 12-16 unique conformations: [evidence span withheld — anchor status: not_found]
- [methods] Unsupervised clustering with in-house AutoGraph yields structurally distinct conformations: [evidence span withheld — anchor status: not_found]
- [readme] ASE-ANI potentials predict only CHNO elements: [evidence span withheld — anchor status: external_doc]
- [readme] ASE-ANI is deprecated in favor of TorchANI: [evidence span withheld — anchor status: not_found]
