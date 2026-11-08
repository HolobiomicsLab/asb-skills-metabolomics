---
name: nmr-structure-calculation
description: Use when you have NEF files containing NMR constraints for proteins and
  need to generate an ensemble of ranked structural conformers via torsion-angle dynamics.
when_to_use_negative:
- Input is already a set of PDB structures without NMR restraints (e.g. pure homology
  models)
- You need full Cartesian-space molecular dynamics rather than torsion-angle dynamics
- Processing raw FIDs for spectra rather than performing a structure calculation
edam_operation: http://edamontology.org/operation_0325
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: NMRFx
  role: Performs the torsion-angle structure calculation from NEF constraint files
    and outputs ranked PDBx/mmCIF ensembles
  repo: https://github.com/nanalysis/nmrfx
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42004-025-01812-8
    title: NMR data processing, visualization, analysis and structure calculation
      with NMRFx
  source_content_hash: sha256:74a422584364d95384eb2560f06ff5d440bc04a0a9b9fc4a657d905650077f0c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/nanalysis/nmrfx
  repo_url: https://github.com/nanalysis/nmrfx
  tools:
  - NMRFx
  edam_operation: http://edamontology.org/operation_0325
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# nmr-structure-calculation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes protein NMR solution structures from deposited NEF constraint files using NMRFx's torsion-angle structure generator, producing ensembles of ranked conformers that can be superimposed onto reference PDB structures.

## When to use

Use when you have NEF files containing NMR constraints for proteins and need to generate an ensemble of ranked structural conformers via torsion-angle dynamics.

## When NOT to use

- Input is already a set of PDB structures without NMR restraints (e.g. pure homology models)
- You need full Cartesian-space molecular dynamics rather than torsion-angle dynamics
- Processing raw FIDs for spectra rather than performing a structure calculation

## Inputs

- NEF files containing NMR constraints (chemical shifts, distance restraints, dihedral angle restraints, etc.)

## Outputs

- Ensemble of 10 lowest-cost PDBx/mmCIF structures per protein
- Cost-function / quality statistics for retained conformers

## How to apply

Run the batch structure-calculation command 'nmrfxs batch -n 200 -k 10 -a <nef-file>', which performs a torsion-angle structure calculation, generates an ensemble of 200 candidate structures, and retains the 10 best structures ranked by the cost function. Superimpose (align) the 10 retained structures onto a representative conformer to produce an aligned PDBx/mmCIF ensemble file. Validate by confirming each protein yields a 10-structure PDBx/mmCIF ensemble and that aggregated cost-function/quality statistics match reference values.

## Related tools

- **NMRFx** (Performs the torsion-angle structure calculation from NEF constraint files and outputs ranked PDBx/mmCIF ensembles) — https://github.com/nanalysis/nmrfx

## Examples

```
nmrfxs batch -n 200 -k 10 -a protein_nef.nef
```

## Evaluation signals

- Each input protein yields a 10-structure PDBx/mmCIF ensemble
- Retained conformers show strong agreement with the corresponding experimentally determined PDB structure after superposition
- Aggregated cost-function values across the dataset match reference tables/figures
- Number of candidate structures generated is 200 per protein as specified by -n 200

## Limitations

- Structure calculation is restricted to torsion-angle dynamics; may not capture full Cartesian-space flexibility
- Result quality depends on completeness and accuracy of the input NEF constraint file
- Structure agreement to the reference is judged by superposition, not by an independent global metric (e.g. no reported RMSD threshold)

## Evidence

- [discussion] 10 best structures retained by cost function: [evidence span withheld — anchor status: not_found]
- [discussion] Superposition onto reference: [evidence span withheld — anchor status: not_found]
- [discussion] dataset scale: [evidence span withheld — anchor status: not_found]
