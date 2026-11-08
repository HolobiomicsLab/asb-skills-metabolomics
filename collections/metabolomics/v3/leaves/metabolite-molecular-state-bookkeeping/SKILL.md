---
name: metabolite-molecular-state-bookkeeping
description: Use when starting CCS prediction from SMILES inputs of metabolites and
  you must enumerate all relevant ionization states (protonated, deprotonated, neutral)
  before conformation generation, ensuring agreement tables and consensus errors are
  attributed to the correct molecular state.
when_to_use_negative:
- Input is already a processed CCS table where the molecular state has been assigned
  — re-running state enumeration would duplicate or overwrite prior assignments.
- You need TWIMS drift-time calibration or arrival-time conversion rather than state
  enumeration; that belongs to the TWIMS validation steps (Tables S1 and S2), not
  this bookkeeping step.
- Your target molecules are outside the CHNO scope handled by the ANI potentials used
  for conformation filtering (ASE-ANI covers CHNO elements), since state enumeration
  would produce species the downstream filtering cannot process.
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3335
tools:
- name: Dimorphite-DL
  role: Performs ionization state determination, enumerating protonated/deprotonated/neutral
    states of each SMILES input before conformation generation.
  repo: https://durrantlab.pitt.edu/dimorphite-dl
- name: RDKit
  role: Consumes the enumerated molecular states and generates conformations for each
    state in the workflow.
  repo: https://www.rdkit.org
- name: Snakemake workflow (snakemake_ccs)
  role: Orchestrates state-aware CCS prediction, writing Boltzmann-average CCS values
    to ccs.txt.
  repo: https://github.com/DasSusanta/snakemake_ccs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/jasms.1c00315
    title: <i>In Silico</i> Collision Cross Section Calculations to Aid Metabolite
      Annotation
  source_content_hash: sha256:499979854552cd340ee54133e1c793c65b55329ab4bbec8ee54efa31a3ec9d81
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CCS calculations
  - Dimorphite-DL
  - RDKit
  - Snakemake workflow (snakemake_ccs)
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3335
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# metabolite-molecular-state-bookkeeping

## Summary

Enumerate the possible protonated/deprotonated/neutral molecular states of each metabolite (e.g. [M + H]+ or [M − H]−) so that downstream CCS calculations are performed per state and can be matched to the correct ion species during benchmarking against TWIMS and literature values.

## When to use

Use when starting CCS prediction from SMILES inputs of metabolites and you must enumerate all relevant ionization states (protonated, deprotonated, neutral) before conformation generation, ensuring agreement tables and consensus errors are attributed to the correct molecular state.

## When NOT to use

- Input is already a processed CCS table where the molecular state has been assigned — re-running state enumeration would duplicate or overwrite prior assignments.
- You need TWIMS drift-time calibration or arrival-time conversion rather than state enumeration; that belongs to the TWIMS validation steps (Tables S1 and S2), not this bookkeeping step.
- Your target molecules are outside the CHNO scope handled by the ANI potentials used for conformation filtering (ASE-ANI covers CHNO elements), since state enumeration would produce species the downstream filtering cannot process.

## Inputs

- SMILES structures of metabolites (e.g. the 20 benchmark metabolites with atom counts 10–40)
- Dimorphite-DL ionization-state configuration/arguments (arguments.json in the snakemake_ccs workflow)
- paths.json specifying locations of Dimorphite-DL and downstream tools

## Outputs

- Per-metabolite list of molecular states (neutral, monoprotonated, deprotonated, doubly protonated etc.)
- State-tagged SMILES structures fed to conformation generation (RDKit)
- ccs.txt containing Boltzmann-average CCS values keyed by metabolite and molecular state

## How to apply

First determine the molecular protonation state of the corresponding gas-phase ions, typically [M + H]+ or [M − H]−, before generating conformations with RDKit. Use Dimorphite-DL for ionization state determination so that multiple states are enumerated automatically; for example, carnosine yields five states including one neutral, three monoprotonated ions, and one doubly protonated ion. Carry the state label through the Snakemake workflow so each computed Boltzmann-average CCS (the ccs.txt output) is keyed by metabolite plus molecular state. When benchmarking, compare each calculated value only against the TWIMS experimental CCS values and literature database values for the same state, reporting percent errors and a consensus average; agreement is judged against the ±3% criterion reported by the authors.

## Related tools

- **Dimorphite-DL** (Performs ionization state determination, enumerating protonated/deprotonated/neutral states of each SMILES input before conformation generation.) — https://durrantlab.pitt.edu/dimorphite-dl
- **RDKit** (Consumes the enumerated molecular states and generates conformations for each state in the workflow.) — https://www.rdkit.org
- **Snakemake workflow (snakemake_ccs)** (Orchestrates state-aware CCS prediction, writing Boltzmann-average CCS values to ccs.txt.) — https://github.com/DasSusanta/snakemake_ccs

## Examples

```
python -c "from dimorphite_dl import protonate_smiles; print(protonate_smiles('C(C(=O)O)NCCc1ccc(O)c(O)c1', min_ph=5.0, max_ph=9.0))"
```

## Evaluation signals

- Each metabolite in the benchmark produces the expected number of states (e.g. carnosine → 5 states: one neutral, three monoprotonated, one doubly protonated).
- Every row in the resulting ccs.txt / agreement table carries a molecular-state label consistent with the Dimorphite-DL output.
- Percent errors computed against TWIMS values, literature database values, and the consensus average fall within ±3% for correctly assigned states.
- The state assigned to each ion matches the expected gas-phase species ([M + H]+ or [M − H]−) used in the TWIMS experiments described in the Supporting Information (Tables S1 and S2).

## Limitations

- Ionization-state enumeration depends on Dimorphite-DL heuristics; unusual tautomer/protonation patterns may be missed or over-generated.
- The paper only demonstrates the workflow on 20 metabolites with atom counts from 10 to 40; larger or more flexible molecules may produce more states and higher computational cost.
- Agreement within ±3% was validated only for these benchmark metabolites and their states; extrapolation to other compound classes is not established.

## Evidence
