---
name: ionization-state-determination
description: Use when starting from a SMILES structure of a small metabolite and you
  need to enumerate its protonated/deprotonated/neutral adduct states as input models
  for downstream conformation generation and collision cross section (CCS) calculation.
when_to_use_negative:
- The input is already an ionized/adduct-specific structure with a known gas-phase
  protonation state, so re-enumerating states would duplicate models
- The target is not a small organic metabolite (ANI-family potentials used downstream
  cover only CHNO/CHNOSFCl elements, so exotic-element species cannot proceed through
  the full pipeline)
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3332
- http://edamontology.org/topic_0154
tools:
- name: Dimorphite-DL
  role: Performs ionization state determination on input SMILES to enumerate protonated/deprotonated/neutral
    molecular-state models
  repo: https://durrantlab.pitt.edu/dimorphite-dl
- name: Snakemake
  role: Workflow manager that orchestrates the ionization-state determination step
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
  source_content_hash: sha256:eef20794c47bc9f852c404636b2aa9d0b634b8745ea7855ff50749da324ab306
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
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3332
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# ionization-state-determination

## Summary

Determines the possible gas-phase ionization/protonation states of a metabolite from its SMILES structure using Dimorphite-DL, enumerating neutral, monoprotonated ([M + H]+), deprotonated ([M − H]−), and multiply protonated models as the first step of the snakemake_ccs workflow before conformation generation and CCS prediction.

## When to use

Use when starting from a SMILES structure of a small metabolite and you need to enumerate its protonated/deprotonated/neutral adduct states as input models for downstream conformation generation and collision cross section (CCS) calculation.

## When NOT to use

- The input is already an ionized/adduct-specific structure with a known gas-phase protonation state, so re-enumerating states would duplicate models
- The target is not a small organic metabolite (ANI-family potentials used downstream cover only CHNO/CHNOSFCl elements, so exotic-element species cannot proceed through the full pipeline)

## Inputs

- SMILES structure(s) of the target metabolite(s)
- paths.json specifying the Dimorphite-DL installation path
- arguments.json with workflow run arguments

## Outputs

- Enumerated ionization-state models per metabolite (neutral, monoprotonated [M + H]+, deprotonated [M − H]−, multiply protonated variants)
- Per-state SMILES/model structures passed onward to RDKit conformation generation

## How to apply

Run Dimorphite-DL on the input SMILES to enumerate ionization states; the paper's workflow first 'determines the molecular protonation state of the corresponding gas-phase ions (typically [M + H]+ or [M − H]−)' before generating conformations with RDKit. Treat each enumerated state as a separate model: for the benchmark metabolite carnosine, five states were generated — one neutral, three monoprotonated ions, and one doubly protonated ion — and each model was carried independently through RDKit conformation generation, ASE-ANI filtering, Gaussian 16 optimization (B3LYP/6-31+G(d,p)) and charge computation (B3LYP/6-311++G(d,p)), and HPCCS trajectory CCS calculation. Compare each model's Boltzmann-averaged CCS against the experimental TWIMS reference to select the best-matching state; for carnosine, model 4 gave 150.21 Å² with only 0.11% error versus the consensus experimental [M + H]+ value of 150.1 Å². In the Snakemake implementation, paths to Dimorphite-DL are specified in paths.json and run parameters in arguments.json, launched via scheduler.sh.

## Related tools

- **Dimorphite-DL** (Performs ionization state determination on input SMILES to enumerate protonated/deprotonated/neutral molecular-state models) — https://durrantlab.pitt.edu/dimorphite-dl
- **Snakemake** (Workflow manager that orchestrates the ionization-state determination step within the snakemake_ccs pipeline) — https://github.com/DasSusanta/snakemake_ccs

## Examples

```
bash scheduler.sh  # after configuring paths.json (Dimorphite-DL path), cluster.yaml, and arguments.json
```

## Evaluation signals

- For a molecule with multiple titratable sites, multiple models are produced (carnosine yielded five states: one neutral, three monoprotonated, one doubly protonated) — a single model output signals a misconfigured run
- The downstream computed CCS for at least one model must fall within ±3% of the experimental TWIMS/literature reference value
- The best-matching model should correspond to the expected gas-phase ion type ([M + H]+ for positive mode, [M − H]− for negative mode)
- paths.json correctly points at the Dimorphite-DL installation before scheduler.sh is executed

## Limitations

- The ASE-ANI potentials used for downstream conformation filtering predict only CHNO elements (ANI-1x/ANI-1ccx); ANI-2x extends to CHNOSFCl, so metabolites outside these element sets cannot follow the full pipeline
- Detailed parameterization of the ionization step is deferred to the paper's Supporting Information, which is not reproduced in the main text
- Enumerating all possible states multiplies the downstream conformational/DFT workload per metabolite (e.g., five states for carnosine)

## Evidence
