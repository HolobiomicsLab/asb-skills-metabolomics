---
name: gas-phase-ion-protonation-state-handling
description: Use when input is a SMILES structure of a metabolite for in silico CCS
  prediction or IM-MS annotation, and the relevant gas-phase ion species ([M + H]+
  or [M − H]−, plus multiply protonated forms) must be enumerated before conformation
  generation.
when_to_use_negative:
- Input is already a specific ion species or a single-protonation-state structure
  where no state enumeration is needed
- The comparison target is neutral-phase data (e.g., literature CCS values reported
  only for neutral compounds) rather than gas-phase ion measurements from TWIMS/IM-MS
edam_topics:
- http://edamontology.org/topic_3335
- http://edamontology.org/topic_0091
tools:
- name: Dimorphite-DL
  role: Ionization state determination — enumerates the protonation states of the
    input SMILES before any conformers are generated
  repo: https://durrantlab.pitt.edu/dimorphite-dl
- name: RDKit
  role: Generates conformations for each protonation state determined in this step,
    e.g., the per-state conformer sets for carnosine
  repo: https://www.rdkit.org
- name: Snakemake workflow (snakemake_ccs)
  role: Orchestrates the state-determination step and predicts CCS for multiple protonated/deprotonated
    adducts with parallelized HPC computation
  repo: https://github.com/DasSusanta/snakemake_ccs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/jasms.1c00315
    title: <i>In Silico</i> Collision Cross Section Calculations to Aid Metabolite
      Annotation
  source_content_hash: sha256:639731b607e9b3cba208c6a586ad5c41195952090716cc23a8ab2dbbc10b2199
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ASE-ANI
  - AutoGraph
  - Dimorphite-DL
  - RDKit
  - Snakemake workflow (snakemake_ccs)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3335
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# gas-phase-ion-protonation-state-handling

## Summary

Determines the molecular protonation state(s) of gas-phase ions (typically [M + H]+ or [M − H]−) so that downstream conformer generation, QM optimization, and CCS prediction operate on the correct ion species; in the ISiCLE-style Snakemake CCS workflow this is handled by Dimorphite-DL, with enumerations such as carnosine's five states (one neutral, three monoprotonated ions, one doubly protonated ion).

## When to use

Use when input is a SMILES structure of a metabolite for in silico CCS prediction or IM-MS annotation, and the relevant gas-phase ion species ([M + H]+ or [M − H]−, plus multiply protonated forms) must be enumerated before conformation generation.

## When NOT to use

- Input is already a specific ion species or a single-protonation-state structure where no state enumeration is needed
- The comparison target is neutral-phase data (e.g., literature CCS values reported only for neutral compounds) rather than gas-phase ion measurements from TWIMS/IM-MS

## Inputs

- SMILES structure string of a metabolite
- Target ionization conditions / adduct types (e.g., [M + H]+, [M − H]−, multiply protonated species)
- Dimorphite-DL ionization-state parameters (pH range) supplied via the workflow's arguments.json

## Outputs

- Set of protonation-state-specific molecular species (e.g., five states for carnosine: one neutral, three monoprotonated ions, one doubly protonated ion)
- State-specific SMILES/structures feeding per-state RDKit conformation generation and downstream CCS prediction
- Per-adduct CCS values aggregated across protonated/deprotonated adducts in the final ccs.txt output

## How to apply

Start from the SMILES input to the Snakemake workflow and run ionization-state determination with Dimorphite-DL (durrantlab.pitt.edu/dimorphite-dl), enumerating the possible protonation states of the molecule. Initially determine the molecular protonation state of the corresponding gas-phase ions, typically [M + H]+ or [M − H]−; where multiple states exist, enumerate all of them (e.g., for carnosine the workflow generated five states: one neutral, three monoprotonated ions, and one doubly protonated ion). Each enumerated state then flows independently through the rest of the pipeline — RDKit conformation generation, ASE-ANI filtering, QUICK/Gaussian quantum calculations, and hpccs CCS calculation — so that a CCS value is predicted for every protonated/deprotonated adduct and model. The rationale is that TWIMS experimental CCS values are measured for specific ionic species, so predicting CCS for the wrong ion state produces values that cannot be compared against experimental or literature CCS data.

## Related tools

- **Dimorphite-DL** (Ionization state determination — enumerates the protonation states of the input SMILES before any conformers are generated) — https://durrantlab.pitt.edu/dimorphite-dl
- **RDKit** (Generates conformations for each protonation state determined in this step, e.g., the per-state conformer sets for carnosine) — https://www.rdkit.org
- **Snakemake workflow (snakemake_ccs)** (Orchestrates the state-determination step and predicts CCS for multiple protonated/deprotonated adducts with parallelized HPC computation) — https://github.com/DasSusanta/snakemake_ccs

## Evaluation signals

- The number of enumerated ion states matches chemical expectation (e.g., carnosine yields five states: one neutral, three monoprotonated ions, and one doubly protonated ion)
- Each enumerated state corresponds to a recognized gas-phase ion adduct form, typically [M + H]+ or [M − H]− for singly charged metabolites
- Downstream steps produce per-state conformer ensembles and CCS predictions (no state silently dropped before QUICK/DFT and hpccs)
- Predicted CCS values per state fall within ±3% of TWIMS experimental/literature values, confirming the correct ion species was modeled

## Limitations

- Ionization-state enumeration multiplies the computational workload — multiple adducts and models each require separate conformer generation, optimization, and CCS runs
- The workflow is aimed at metabolites relevant to IM-MS annotation (benchmark atoms counts 10–40); behavior for large or highly flexible molecules is not demonstrated here
- No software version numbers are stated for Dimorphite-DL, RDKit, or the ASE-ANI step, so exact parameter reproducibility of the state-determination step is not documented

## Evidence
