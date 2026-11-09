---
name: molecular-conformation-generation
description: Use when starting from a SMILES string of a small metabolite (10–40 atoms)
  whose gas-phase ion states ([M + H]+ or [M − H]−) have already been assigned, and
  a 3D conformer ensemble is needed before geometry optimization and CCS calculation.
when_to_use_negative:
- Input is not a SMILES structure but already optimized 3D geometries — use the ASE-ANI
  filtering / AutoGraph clustering stage instead.
- The molecule contains elements outside ANI-1ccx's CHNO coverage (e.g. S, F, Cl)
  and no alternative potential is available, since downstream relaxation would fail.
- Goal is direct CCS computation from a single known structure rather than ensemble-based
  prediction.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3332
- http://edamontology.org/topic_0081
tools:
- name: RDKit
  role: 'Conformation generation toolkit: produces the per-protonation-state 3D conformer
    ensembles from SMILES-derived ion structures'
  repo: https://www.rdkit.org
- name: Dimorphite-DL
  role: Upstream ionization state determination that defines which protonated/deprotonated
    forms conformers are generated for
- name: ASE-ANI
  role: 'Downstream conformation filtering: relaxes the RDKit-generated ensemble with
    the ANI-1ccx ML potential before clustering'
  repo: https://github.com/isayev/ASE_ANI
- name: Snakemake (snakemake_ccs)
  role: Workflow manager that chains RDKit conformer generation into the automated,
    parallelized CCS prediction pipeline
  repo: https://github.com/DasSusanta/snakemake_ccs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/jasms.1c00315
    title: <i>In Silico</i> Collision Cross Section Calculations to Aid Metabolite
      Annotation
  source_content_hash: sha256:c1930328ac28cb21d647cc03da435c017b243ebfb2db8e23798f1b086520c6c2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ASE-ANI
  - AutoGraph
  - Dimorphite-DL
  - RDKit
  - Snakemake (snakemake_ccs)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3332
  - http://edamontology.org/topic_0081
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# molecular-conformation-generation

## Summary

Generates large 3D conformer ensembles from SMILES-derived protonated/deprotonated ion structures using the RDKit toolkit, as the first structural step of a high-throughput collision cross section (CCS) prediction workflow. Producing an exhaustive ensemble is necessary so downstream ML-QM minimization and unsupervised clustering can distill structurally distinct conformations.

## When to use

Use when starting from a SMILES string of a small metabolite (10–40 atoms) whose gas-phase ion states ([M + H]+ or [M − H]−) have already been assigned, and a 3D conformer ensemble is needed before geometry optimization and CCS calculation.

## When NOT to use

- Input is not a SMILES structure but already optimized 3D geometries — use the ASE-ANI filtering / AutoGraph clustering stage instead.
- The molecule contains elements outside ANI-1ccx's CHNO coverage (e.g. S, F, Cl) and no alternative potential is available, since downstream relaxation would fail.
- Goal is direct CCS computation from a single known structure rather than ensemble-based prediction.

## Inputs

- SMILES structure string for the target metabolite
- Assigned protonation/ionization state(s) ([M + H]+, [M − H]−, neutral) from Dimorphite-DL

## Outputs

- 3D conformer ensemble (large set of per-state conformer geometries) for each ionization state
- Conformer coordinates passed downstream to ASE-ANI filtering (and ultimately cluster_representatives.sdf)

## How to apply

First determine the molecular protonation state of the gas-phase ions (typically [M + H]+ or [M − H]−), e.g. with Dimorphite-DL, since conformers must be generated per ionization state. Then generate the conformations for these states using the RDKit toolkit; for conformationally flexible molecules like carnosine this produces very large ensembles (e.g. 990–996 conformers per carnosine state across its neutral, monoprotonated, and doubly protonated forms). Within the Snakemake workflow (DasSusanta/snakemake_ccs), conformer generation runs after Dimorphite-DL ionization-state determination and before ASE-ANI conformation filtering, which relaxes structures with the ANI-1ccx neural network potential. Rationale: an over-complete ensemble guarantees coverage of the low-energy conformational space; subsequent ASE-ANI minimization and in-house AutoGraph unsupervised clustering (reducing carnosine's ~990-member sets to 12–16 unique conformations) remove redundancy, so no manual cluster selection is required at this stage.

## Related tools

- **RDKit** (Conformation generation toolkit: produces the per-protonation-state 3D conformer ensembles from SMILES-derived ion structures) — https://www.rdkit.org
- **Dimorphite-DL** (Upstream ionization state determination that defines which protonated/deprotonated forms conformers are generated for)
- **ASE-ANI** (Downstream conformation filtering: relaxes the RDKit-generated ensemble with the ANI-1ccx ML potential before clustering) — https://github.com/isayev/ASE_ANI
- **Snakemake (snakemake_ccs)** (Workflow manager that chains RDKit conformer generation into the automated, parallelized CCS prediction pipeline) — https://github.com/DasSusanta/snakemake_ccs

## Examples

```
bash scheduler.sh  # after configuring paths.json, cluster.yaml, and arguments.json in the snakemake_ccs workflow
```

## Evaluation signals

- Each ionization state from Dimorphite-DL yields its own conformer ensemble (e.g. five carnosine states → five separate sets).
- Ensemble sizes for flexible molecules are large (carbosine-like: ~990–996 conformers per state), confirming exhaustive rather than sparse sampling.
- All conformers correspond to the correct protonation state ([M + H]+ or [M − H]−) before entering ASE-ANI filtering.
- Downstream AutoGraph clustering returns fewer clusters than input conformers (e.g. 16, 13, 15, 12, 14 from the ~990-member carnosine sets), confirming the ensemble covered redundant conformations.
- The workflow completes through the Snakemake DAG and ultimately produces Boltzmann-averaged CCS values in ccs.txt within the ensemble folder.

## Limitations

- Very large ensembles (hundreds to ~1000 conformers per state) are expected and are only tractable because downstream ASE-ANI and clustering steps condense them; generation alone does not yield distinct conformations.
- Conformer generation is per-protonation-state; skipping Dimorphite-DL risks missing states (e.g. carnosine's three monoprotonated and one doubly protonated forms).
- Details of each workflow step are deferred to the Supporting Information, and no software version numbers are stated for RDKit in the methods text.
- Computation is intended for molecules with atom counts roughly 10–40; scaling to much larger metabolites is not demonstrated here.

## Evidence

- [methods] Downstream clustering condenses the ensemble: [evidence span withheld — anchor status: not_found]
